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


## Project Structure

```mdx
└── 📂 confidential-avs-example/          # Root directory for the Confidential AVS Example project
├── 📂 Execution_Service/                 # Service responsible for executing confidential tasks
│   ├── 📂 configs/                       # Configuration files for the Execution Service
│   │   └── 📄 app.config.js              # Main configuration file for the Execution Service
│   ├── 📂 src/                           # Source code for the Execution Service
│   │   ├── 📂 utils/                     # Utility modules for the Execution Service
│   │   │   ├── 📄 mcl.js                 # MCL cryptography utilities
│   │   │   ├── 📄 validateError.js       # Error validation utilities
│   │   │   └── 📄 validateResponse.js    # Response validation utilities
│   │   ├── 📄 dal.service.js             # Data access layer for Execution Service
│   │   ├── 📄 kyc.service.js             # KYC (Know Your Customer) service logic
│   │   └── 📄 task.controller.js         # Controller for handling task-related endpoints
│   ├── 📄 Dockerfile                     # Dockerfile for containerizing the Execution Service
│   ├── 📄 index.js                       # Entry point for the Execution Service
│   ├── 📄 package-lock.json              # NPM lock file for dependency management
│   ├── 📄 package.json                   # NPM package manifest for Execution Service
│   └── 📄 yarn.lock                      # Yarn lock file for dependency management
├── 📂 Validation_Service/                # Service responsible for validating confidential tasks
│   ├── 📂 configs/                       # Configuration files for the Validation Service
│   │   └── 📄 app.config.js              # Main configuration file for the Validation Service
│   ├── 📂 src/                           # Source code for the Validation Service
│   │   ├── 📂 utils/                     # Utility modules for the Validation Service
│   │   │   ├── 📄 validateError.js       # Error validation utilities
│   │   │   └── 📄 validateResponse.js    # Response validation utilities
│   │   ├── 📄 dal.service.js             # Data access layer for Validation Service
│   │   ├── 📄 oracle.service.js          # Oracle integration logic
│   │   ├── 📄 task.controller.js         # Controller for handling task-related endpoints
│   │   ├── 📄 validator.service.js       # Core validation logic
│   │   └── 📄 verify.service.js          # Service for verification processes
│   ├── 📄 Dockerfile                     # Dockerfile for containerizing the Validation Service
│   ├── 📄 index.js                       # Entry point for the Validation Service
│   ├── 📄 package.json                   # NPM package manifest for Validation Service
├── 📂 grafana/                           # Grafana monitoring and visualization setup
   │   ├── 📂 dashboards/                 # Predefined Grafana dashboards
│   │   └── 📄 othentic-cli.json          # Grafana dashboard configuration for Othentic CLI
│   └── 📂 provisioning/                  # Grafana provisioning configuration
│   ├── 📂 dashboards/                    # Dashboard provisioning configs
│   │   └── 📄 dashboards.yaml            # Dashboard provisioning YAML
│   └── 📂 datasources/                   # Datasource provisioning configs
│   └── 📄 datasources.yaml               # Datasource provisioning YAML
├── 📄 Dockerfile                         # Root Dockerfile, possibly for building base images or multi-service setups
├── 📄 README.md                          # Project documentation and instructions
├── 📄 docker-compose.yml                 # Docker Compose file for orchestrating multi-container services
└── 📄 prometheus.yaml # Prometheus monitoring configuration file
```

## Architecture

The Performer Node runs inside a TEE.

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
   cd simple-price-oracle-avs-example
   docker compose build --no-cache
   docker compose up
   curl -X POST http://localhost:4003/task/execute
   ```

Happy Building! 🚀

