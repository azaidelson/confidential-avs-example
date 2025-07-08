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

The Simple Price Oracle AVS Example demonstrates how to deploy a minimal AVS using Othentic Stack.

## Architecture

The Performer Node runs inside a SecretVM TEE, offering two crucial properties:
1. The user data is protected from the Node Operator
2. The source code running on the Performer Node is verifiable through Attestation

The Performer node executes tasks using the Task Execution Service and sends the results to the p2p network.

Attester Nodes validate task execution through the Validation Service. Based on the Validation Service's response, attesters sign the tasks. In this AVS:

### Task Execution logic:
The Performer Node receievs a picture of the user's identification document. The Performer Node uses a Confidential AI model provide by SecretAI to extract key fields from the ID:
- Country of citizenship
- Age
The Performer Node constructs the task results with three fields:
  Country of Origin: String
  Is Over 18: Boolean
  Is Over 21: Boolean

The task results are _signed_ using [Verifiable Message Signature](https://docs.scrt.network/secret-network-documentation/secretvm-confidential-virtual-machines/verifiable-message-signing) scheme that ties the message to a specific TEE attestation and guarantees that the results were produced on a Performer Node running a known and approved version of the code.

Note: the actual image of the user ID is not stored neither in the Performed Node nor in the SecretAI LLM server (this can be validated by auditing the source code)

### Validation Service logic:
The Attester Nodes performs the following logic:
a. Extract the Public Key from the Attestation Quote's user data field
b. Decode the message and verify the message signature
c. Confirm that the Attestation Quote was produced by the expected source code by comparing the MRTD, RTMR0, RTMR1, RTMR2 and RTMR3 to known good values. This guarantees that the Performer Node code was not tampered with

After these steps are completed, the Attester is sure that the message is produced by an untampered Performer Node and thus can be fully trusted.

Note: the Attester nodes also don't get access to the actual data of the user, ensuring complete privacy of the solution

---


## Prerequisites

- Node.js (v 22.6.0 )
- Foundry
- [Yarn](https://yarnpkg.com/)
- [Docker](https://docs.docker.com/engine/install/)

## Usage

1. Clone the repository:

   ```bash
   git clone https://github.com/Othentic-Labs/confidential-avs-example.git
   cd confidential-avs-example
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
   cd simple-price-oracle-avs-example
   docker compose build --no-cache
   docker compose up
   curl -X POST http://localhost:4003/task/execute
   ```

Happy Building! 🚀

