# Confidential AVS Example

---

## Table of Contents

1. [Overview](#overview)
2. [Project Structure](#project-structure)
3. [Architecture](#architecture)
4. [Prerequisites](#prerequisites)
5. [Installation](#installation)
6. [Usage](#usage)

---

## Overview

The Confidential AVS Example demonstrates how to create and deploy a minimal Confidential AVS.
The concept of Confidential AVS is that the user data remains confidential and is protected from both the Performer nodes and the Attester nodes.
Confidential AVS uses TEE-based SecretVM technology developed by Secret.

In Confidential AVS, Performer nodes runs inside a TEE-powered Confidential Virtual Machine (CVM), offering two key properties:
- Data Confidentiality: CVM's memory is encrytpted and cannot be accessed by the operator of the node
- Verifiability of the source code: CVM attestation allows to cryptographically prove the authenticity of the software running inside the CVM

The Perfomrmer receives a Tesk from a user and executes it (user data remains confidential), and then submits the tasks along with an attestation-tied signature to the Attesters.

Attesters validate the following:
- That the attestation belongs to a known Performer software package and version, proving that the code is genuine and has not been tampered with. 
- That the task message is actually signed by the code running in this software package

In this example, the User sends images of their ID to the Performer Node. The Performer Node analyzes the ID using a Vision Model (gemma3:4b in this case)
to extract the user's nationality and age. Then, the performer sends the sign results: the user's nationality and whether the user is over 21 years of age.
All the processing is done inside TEEs, so the user data cannot be accessed by any third party, including the operators of the Performer Node.

  ### Useful Links

- [Intro to SecretVM](https://docs.scrt.network/secret-network-documentation/secretvm-confidential-virtual-machines/introduction)
- [SecretVM Attestation Docs](https://docs.scrt.network/secret-network-documentation/secretvm-confidential-virtual-machines/attestation)
- [Verifiable Message Signing Docs](https://docs.scrt.network/secret-network-documentation/secretvm-confidential-virtual-machines/verifiable-message-signing)
- [SecretVM Source Code](https://github.com/scrtlabs/secretvm)



## Project Structure

```mdx
📂 confidential-avs-example
├── 📂 Execution_Service               # Task Performer Node logic. Must be deployed in SecretVM Tee.
│   ├── 📂 configs
│   │   └── app.config.js              # Express app config (dotenv, routes)
│   ├── 📂 src
│   │   ├── dal.service.js             # Interacts with Pinata for IPFS uploads
│   │   ├── oracle.service.js          # Fetches crypto prices from Binance API
│   │   ├── task.controller.js         # Express router for /execute POST endpoint
│   │   ├── kyc.service.js             # KYC-related service logic
│   │   ├── id.png                     # Sample image asset
│   │   └── 📂 utils
│   │       ├── mcl.js                 # Cryptographic utilities (MCL)
│   │       ├── validateError.js       # Custom error class for API responses
│   │       └── validateResponse.js    # Custom response class for API responses
│   ├── Dockerfile                     # Node.js 22.6 Docker setup, exposes 8080
│   ├── index.js                       # Server entry point, loads config, starts app
│   ├── package.json                   # Node.js dependencies and scripts
│   ├── package-lock.json              # NPM lockfile
│   └── yarn.lock                      # Yarn lockfile
│
├── 📂 Validation_Service              # Task validation logic (Express.js backend)
│   ├── 📂 configs
│   │   └── app.config.js              # Express app config (routes)
│   ├── 📂 src
│   │   ├── dal.service.js             # Interacts with Pinata for IPFS uploads
│   │   ├── oracle.service.js          # Fetches crypto prices from Binance API
│   │   ├── task.controller.js         # Express router for /validate POST endpoint
│   │   ├── validator.service.js       # Validates task result 
│   │   ├── verify.service.js          # Verifies a signed identity using the SGX quote and Ed25519 public key.
│   │   └── 📂 utils
│   │       ├── validateError.js       # Custom error class for API responses
│   │       └── validateResponse.js    # Custom response class for API responses
│   ├── Dockerfile                     # Node.js 22.6 Docker setup, exposes 8080
│   ├── index.js                       # Server entry point, loads config, starts app
│   ├── package.json                   # Node.js dependencies and scripts
│   ├── package-lock.json              # NPM lockfile
│   └── yarn.lock                      # Yarn lockfile
│
├── 📂 grafana                         # Grafana monitoring configuration
│   ├── 📂 dashboards
│   │   └── othentic-cli.json          # Grafana dashboard definition
│   └── 📂 provisioning
│       ├── 📂 dashboards
│       │   └── dashboards.yaml        # Dashboard provisioning config
│       └── 📂 datasources
│           └── datasources.yaml       # Datasource provisioning config
├── docker-compose.yml                 # Docker Compose for all services and monitoring
├── Dockerfile                         # (Root) Dockerfile (if used for meta/CI)
├── prometheus.yaml                    # Prometheus configuration
├── .gitignore                         # Git ignore rules
├── .env.example                       # Example environment variables
└── README.md                          # Project documentation
```

## Architecture

![image](https://github.com/user-attachments/assets/dc60d859-2c1e-4dfb-bacd-125f09f10bc8)

The Performer node runs inside the TEE. It receives confidential data from the user and performs the necessary calculation.

Attester Nodes validate task execution through the Validation Service. Based on the Validation Service's response, attesters sign the tasks. In this AVS:

Task Execution logic:
Receive an image of the user's identification document. Send to a Confidential AI model to extract the necessary data.
Return the user's country and two boolean fields - Age Over 18 and Age Over 21.
Sign the resulting message using the Verifiable Message Signing scheme https://docs.scrt.network/secret-network-documentation/secretvm-confidential-virtual-machines/verifiable-message-signing
Post the signed message.

Validation Service logic:
Receive the signed message.
Check the signature and the Attestation fields (MRTD, RTMR0, RTMR1, RTMR2, RTMR3)
If the signature is correct, and the attestation fields match known good values, accept the message.


---

## Prerequisites

- Node.js (v 22.6.0 )
- Foundry
- [Yarn](https://yarnpkg.com/)
- [Docker](https://docs.docker.com/engine/install/)

## Usage

1. Clone the repository:

   ```bash
   git clone https://github.com/Othentic-Labs/confidential-avs-example-example.git
   cd confidential-avs-example-example
   git checkout kyc-avs
   ```

2. Install Othentic CLI:

   ```bash
   npm i -g @othentic/cli
   npm i -g @othentic/node
   ```

3. Set up the TEE server by following the instructions below. Build the Docker image and start the server. Make sure to populate the `.env` file with `TEE_KYC_SERVER_URL` and any other required environment variables.
   ```bash
   cd ..
   git clone https://github.com/scrtlabs/kyc-avs-demo
   cd kyc-avs-demo
   docker build -t kyc-avs .
   ```

4. Follow the steps in the official documentation's [Quickstart](https://docs.othentic.xyz/main/welcome/getting-started/install-othentic-cli) Guide for setup and deployment.

   ```
   cd confidential-avs-example
   docker compose build --no-cache
   docker compose up
   curl -X POST http://localhost:4003/task/execute
   ```

Happy Building! 🚀

