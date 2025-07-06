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

  ### Useful Links

- [Intro to SecretVM](https://docs.scrt.network/secret-network-documentation/secretvm-confidential-virtual-machines/introduction)
- [SecretVM Attestation Docs](https://docs.scrt.network/secret-network-documentation/secretvm-confidential-virtual-machines/attestation)
- [Verifiable Message Signing Docs](https://docs.scrt.network/secret-network-documentation/secretvm-confidential-virtual-machines/verifiable-message-signing)
- [SecretVM Source Code](https://github.com/scrtlabs/secretvm)



## Project Structure

```mdx
📂 simple-price-oracle-avs-example
├── 📂 Execution_Service         # Implements Task execution logic - Express JS Backend
│   ├── 📂 config/
│   │   └── app.config.js        # An Express.js app setup with dotenv, and a task controller route for handling `/task` endpoints.
│   ├── 📂 src/
│   │   └── dal.service.js       # A module that interacts with Pinata for IPFS uploads
│   │   ├── oracle.service.js    # A utility module to fetch the current price of a cryptocurrency pair from the Binance API
│   │   ├── task.controller.js   # An Express.js router handling a `/execute` POST endpoint
│   │   ├── 📂 utils             # Defines two custom classes, CustomResponse and CustomError, for standardizing API responses
│   ├── Dockerfile               # A Dockerfile that sets up a Node.js (22.6) environment, exposes port 8080, and runs the application via index.js
|   ├── index.js                 # A Node.js server entry point that initializes the DAL service, loads the app configuration, and starts the server on the specified port
│   └── package.json             # Node.js dependencies and scripts
│
├── 📂 Validation_Service         # Implements task validation logic - Express JS Backend
│   ├── 📂 config/
│   │   └── app.config.js         # An Express.js app setup with a task controller route for handling `/task` endpoints.
│   ├── 📂 src/
│   │   └── dal.service.js        # A module that interacts with Pinata for IPFS uploads
│   │   ├── oracle.service.js     # A utility module to fetch the current price of a cryptocurrency pair from the Binance API
│   │   ├── task.controller.js    # An Express.js router handling a `/validate` POST endpoint
│   │   ├── validator.service.js  # A validation module that checks if a task result from IPFS matches the ETH/USDT price within a 5% margin.
│   │   ├── 📂 utils              # Defines two custom classes, CustomResponse and CustomError, for standardizing API responses.
│   ├── Dockerfile                # A Dockerfile that sets up a Node.js (22.6) environment, exposes port 8080, and runs the application via index.js.
|   ├── index.js                  # A Node.js server entry point that initializes the DAL service, loads the app configuration, and starts the server on the specified port.
│   └── package.json              # Node.js dependencies and scripts
│
├── 📂 grafana                    # Grafana monitoring configuration
├── docker-compose.yml            # Docker setup for Operator Nodes (Performer, Attesters, Aggregator), Execution Service, Validation Service, and monitoring tools
├── .env.example                  # An example .env file containing configuration details and contract addresses
├── README.md                     # Project documentation
└── prometheus.yaml               # Prometheus configuration for logs
```

## Architecture

![Price oracle sample](https://github.com/user-attachments/assets/03d544eb-d9c3-44a7-9712-531220c94f7e)

The Performer node executes tasks using the Task Execution Service and sends the results to the p2p network.

Attester Nodes validate task execution through the Validation Service. Based on the Validation Service's response, attesters sign the tasks. In this AVS:

Task Execution logic:


Validation Service logic:

---

## Prerequisites

- Node.js (v 22.6.0 )
- Foundry
- [Yarn](https://yarnpkg.com/)
- [Docker](https://docs.docker.com/engine/install/)

## Usage

1. Clone the repository:

   ```bash
   git clone https://github.com/Othentic-Labs/simple-price-oracle-avs-example.git
   cd simple-price-oracle-avs-example
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

