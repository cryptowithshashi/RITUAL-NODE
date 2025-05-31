
# Ritual Infernet Node — Full Setup Guide

Set up a Ritual node on your VPS or local system using Docker. This guide is beginner-friendly and includes explanations for every step.


## System Requirements

| Component   | Minimum                | Recommended                  | GPU-Heavy Workloads          |
| :---------- | :-------------------- | :-------------------------- | :--------------------------- |
| **CPU**     | Single-core vCPU       | 4 modern vCPU cores          | 4 modern vCPU cores          |
| **RAM**     | 128MB                  | 16GB                         | 64GB                         |
| **DISK**    | 512MB HDD              | 500GB IOPS-optimized SSD     | 500GB NVME                   |
| **GPU**     | —                      | —                            | CUDA-enabled GPU             |


### Pre-Requirements 

- EVM-compatible wallet with at least $5 worth of BASE ETH  
- Base Mainnet RPC URL (from ALCHEMY)  

## Step 1: System Update & Essential Tools

Update the system packages to the latest versions.
```bash
sudo apt update && sudo apt upgrade -y
```

Install essential tools used during setup

```bash
sudo apt -qy install curl git nano jq lz4 build-essential screen
```

## Step 2: Install Docker & Docker Compose (for VPS)

Add necessary dependencies for downloading Docker securely.

```bash
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

Add Docker's official GPG key for package verification.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Add the Docker repository to your sources list.


```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker Engine and enables the Docker service to auto-start.

```bash
sudo apt update && sudo apt install -y docker-ce && sudo systemctl enable --now docker
```

 Add your current user to the Docker group so you can run docker without sudo.

```bash
sudo usermod -aG docker $USER && newgrp docker
```

Download and installs the latest version of Docker Compose.

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | jq -r .tag_name)/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose
```

Confirms Docker and Docker Compose were installed correctly.


```bash
docker --version && docker-compose --version
```

## Step 3: Open Required Firewall Ports (for VPS)

Install UFW (Uncomplicated Firewall)

```bash
sudo apt install ufw -y
```

Open required ports for Ritual node and enables the firewall.

```bash
sudo ufw allow 22
sudo ufw allow 3000
sudo ufw allow 4000
sudo ufw allow 6379
sudo ufw allow 8545
sudo ufw allow ssh
sudo ufw enable
```

## Step 4: Clone Ritual Node Repo

Move you to your home directory.

```bash
cd $HOME
```

Download the Ritual Infernet Node code from GitHub.

```bash
git clone https://github.com/ritual-net/infernet-container-starter
```

Enter the cloned project folder.

```bash
cd infernet-container-starter
```
## Running the hello-world Container

Before running this step, make sure Docker is running in the background (especially if you're on a local PC or using Docker Desktop).

- Pull the Hello-World Container

```bash
docker pull ritualnetwork/hello-world-infernet:latest
```

This pulls the latest hello-world container image from Ritual Network's Docker Hub.

- Deploy the Container

```bash
project=hello-world make deploy-container
```

## Step 5: Configure Environment Variables

You’ll now customize multiple configuration files. This ensures your Infernet node uses your wallet and RPC settings properly.

### 5.1 - Configure Main Node File (`deploy/config.json`)

Replace default config file and open for editing:

```bash
rm ~/infernet-container-starter/deploy/config.json && nano ~/infernet-container-starter/deploy/config.json
```

Paste the following:

```
{
    "log_path": "infernet_node.log",
    "server": {
        "port": 4000,
        "rate_limit": {
            "num_requests": 100,
            "period": 100
        }
    },
    "chain": {
        "enabled": true,
        "trail_head_blocks": 3,
        "rpc_url": "ENTER BASE MAINNET URL",
        "registry_address": "0x3B1554f346DFe5c482Bb4BA31b880c1C18412170",
        "wallet": {
          "max_gas_limit": 4000000,
          "private_key": "ENTER YOUR PRIVATE KEY",
          "allowed_sim_errors": []
        },
        "snapshot_sync": {
          "sleep": 3,
          "batch_size": 500,
          "starting_sub_id": 240000,
          "sync_period": 30
        }
    },
    "startup_wait": 1.0,
    
    "redis": {
        "host": "redis",
        "port": 6379
    },
    "forward_stats": true,
    "containers": [
        {
            "id": "hello-world",
            "image": "ritualnetwork/hello-world-infernet:latest",
            "external": true,
            "port": "3000",
            "allowed_delegate_addresses": [],
            "allowed_addresses": [],
            "allowed_ips": [],
            "command": "--bind=0.0.0.0:3000 --workers=2",
            "env": {},
            "volumes": [],
            "accepted_payments": {},
            "generates_proofs": false
        }
    ]
}
```

Replace:

- ENTER BASE MAINNET URL with your actual RPC URL

- ENTER YOUR PRIVATE KEY with your EVM wallet private key (must start with 0x)

Then press CTRL+X, press Y, then hit Enter to save.

### 5.2 - Configure Hello-World Container (projects/hello-world/container/config.json)

Replace config file and edit:

```bash
rm ~/infernet-container-starter/projects/hello-world/container/config.json && nano ~/infernet-container-starter/projects/hello-world/container/config.json
```

Paste the following:

```bash
{
    "log_path": "infernet_node.log",
    "server": {
        "port": 4000,
        "rate_limit": {
            "num_requests": 100,
            "period": 100
        }
    },
    "chain": {
        "enabled": true,
        "trail_head_blocks": 3,
        "rpc_url": "ENTER BASE MAINNET URL",
        "registry_address": "0x3B1554f346DFe5c482Bb4BA31b880c1C18412170",
        "wallet": {
          "max_gas_limit": 4000000,
          "private_key": "ENTER YOUR PRIVATE KEY",
          "allowed_sim_errors": []
        },
        "snapshot_sync": {
          "sleep": 3,
          "batch_size": 500,
          "starting_sub_id": 240000,
          "sync_period": 30
        }
    },
    "startup_wait": 1.0,
   
    "redis": {
        "host": "redis",
        "port": 6379
    },
    "forward_stats": true,
    "containers": [
        {
            "id": "hello-world",
            "image": "ritualnetwork/hello-world-infernet:latest",
            "external": true,
            "port": "3000",
            "allowed_delegate_addresses": [],
            "allowed_addresses": [],
            "allowed_ips": [],
            "command": "--bind=0.0.0.0:3000 --workers=2",
            "env": {},
            "volumes": [],
            "accepted_payments": {},
            "generates_proofs": false
        }
    ]
}
```
Replace:

- ENTER BASE MAINNET URL with your actual RPC URL

- ENTER YOUR PRIVATE KEY with your EVM wallet private key (must start with 0x)

Then press CTRL+X, press Y, then hit Enter to save.

### 5.3 - Update Contract Deployment Script

Replace Deploy.s.sol with updated version:

```bash
rm ~/infernet-container-starter/projects/hello-world/contracts/script/Deploy.s.sol && nano ~/infernet-container-starter/projects/hello-world/contracts/script/Deploy.s.sol
```

Paste the following:

```bash
// SPDX-License-Identifier: BSD-3-Clause-Clear
pragma solidity ^0.8.13;

import {Script, console2} from "forge-std/Script.sol";
import {SaysGM} from "../src/SaysGM.sol";

contract Deploy is Script {
    function run() public {
        // Setup wallet
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        // Log address
        address deployerAddress = vm.addr(deployerPrivateKey);
        console2.log("Loaded deployer: ", deployerAddress);

        address registry = 0x3B1554f346DFe5c482Bb4BA31b880c1C18412170;
        // Create consumer
        SaysGM saysGm = new SaysGM(registry);
        console2.log("Deployed SaysHello: ", address(saysGm));

        // Execute
        vm.stopBroadcast();
        vm.broadcast();
    }
}
```

Save using CTRL+X, Y, Enter.

### 5.4 - Update the Makefile (Contract Compilation Config)

```bash
rm ~/infernet-container-starter/projects/hello-world/contracts/Makefile && nano ~/infernet-container-starter/projects/hello-world/contracts/Makefile
```

Paste this following:

```bash
# phony targets are targets that don't actually create a file
.phony: deploy

# anvil's third default address
sender := ENTER YOUR PRIVATE KEY
RPC_URL := ENTER YOUR BASE MAINNET URL

# deploying the contract
deploy:
	@PRIVATE_KEY=$(sender) forge script script/Deploy.s.sol:Deploy --broadcast --rpc-url $(RPC_URL)

# calling sayGM()
call-contract:
	@PRIVATE_KEY=$(sender) forge script script/CallContract.s.sol:CallContract --broadcast --rpc-url $(RPC_URL)
```

Replace:

- ENTER BASE MAINNET URL with your actual RPC URL

- ENTER YOUR PRIVATE KEY with your EVM wallet private key (must start with 0x)

Then press CTRL+X, press Y, then hit Enter to save.


### 5.5 - Update Docker Compose File

```bash
rm ~/infernet-container-starter/deploy/docker-compose.yaml && nano ~/infernet-container-starter/deploy/docker-compose.yaml
```

Paste this following: 

```bash
services:
  node:
    image: ritualnetwork/infernet-node:1.4.0
    ports:
      - "0.0.0.0:4000:4000"
    volumes:
      - ./config.json:/app/config.json
      - node-logs:/logs
      - /var/run/docker.sock:/var/run/docker.sock
    tty: true
    networks:
      - network
    depends_on:
      - redis
      - infernet-anvil
    restart:
      on-failure
    extra_hosts:
      - "host.docker.internal:host-gateway"
    stop_grace_period: 1m
    container_name: infernet-node

  redis:
    image: redis:7.4.0
    ports:
    - "6379:6379"
    networks:
      - network
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      - redis-data:/data
    restart:
      on-failure
    container_name: infernet-redis

  fluentbit:
    image: fluent/fluent-bit:3.1.4
    expose:
      - "24224"
    environment:
      - FLUENTBIT_CONFIG_PATH=/fluent-bit/etc/fluent-bit.conf
    volumes:
      - ./fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf
      - /var/log:/var/log:ro
    networks:
      - network
    restart:
      on-failure
    container_name: infernet-fluentbit

  infernet-anvil:
    image: ritualnetwork/infernet-anvil:1.0.0
    command: --host 0.0.0.0 --port 3000 --load-state infernet_deployed.json -b 1
    ports:
      - "8545:3000"
    networks:
      - network
    container_name: infernet-anvil

networks:
  network:

volumes:
  node-logs:
  redis-data:
```  

Save using CTRL+X, Y, Enter.

## Step 6: Configure Docker Compose

```bash
cd $home
```

```bash
docker compose -f infernet-container-starter/deploy/docker-compose.yaml stop
```

## Step 7: Install Foundry
We'll use **Foundry** for deploying and interacting with smart contracts.

- Download and install Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
```

- Add Foundry to your shell PATH

```bash
echo 'export PATH="$HOME/.foundry/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

- Finalize setup

```bash
foundryup
```

## Step 8: Install Required Forge Libraries

Now that Foundry is installed, let’s set up the required libraries for your smart contracts.

- Navigate to Contract Directory

```bash
cd ~/infernet-container-starter/projects/hello-world/contracts
```
- Clean up Old Libs (if any)

```bash
rm -rf lib/forge-std lib/infernet-sdk
```

- Install the standard Foundry testing library

```bash
forge install foundry-rs/forge-std
```

- Install the Ritual infernet SDK

```bash
forge install ritual-net/infernet-sdk
```

Confirm Install

```bash
ls lib/forge-std
ls lib/infernet-sdk
```

You should see the installed files/directories under both lib/forge-std and lib/infernet-sdk.


## Step 9: Start Docker Compose & Verify Containers

Navigate to your home directory

```bash
cd $HOME
```

Start Docker Compose in detached mode

```bash
docker compose -f infernet-container-starter/deploy/docker-compose.yaml up -d
```

This will start all containers in the background.

Verify all 5 containers are running

```bash
docker ps
```

Look for these containers running:

- infernet-node

- infernet-redis

- infernet-fluentbit

- infernet-anvil

and any others listed in your compose file. If any container isn’t running, check logs with:

```bash
docker logs <container_name>
```

## Step 10: Deploy the SaysGM Consumer Contract

Navigate to the main project directory

```bash
cd ~/infernet-container-starter
```

Run the Makefile target to deploy contracts for the hello-world project

```bash
project=hello-world make deploy-contracts
```

This command will:

- Use your private key and RPC URL from the config

- Deploy the SaysGM contract on the blockchain

- Output the deployed contract address in the console

**Save the deployed contract address somewhere safe You’ll need it later for calling the contract.**


## Step 11: Call the sayGM() Function on the Deployed Contract

Open the CallContract script for editing

```bash
nano ~/infernet-container-starter/projects/hello-world/contracts/script/CallContract.s.sol
```

Paste or edit the script like this:

- Replace YOUR_DEPLOYED_CONTRACT_ADDRESS with the actual address you saved from the previous deployment step.

```bash
// SPDX-License-Identifier: BSD-3-Clause-Clear
pragma solidity ^0.8.13;

import {Script, console2} from "forge-std/Script.sol";
import {SaysGM} from "../src/SaysGM.sol";

contract CallContract is Script {
    function run() public {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        SaysGM saysGm = SaysGM(0xYOUR_DEPLOYED_CONTRACT_ADDRESS);
        saysGm.sayGM();

        vm.stopBroadcast();
        console2.log("✅ Called sayGM() successfully!");
    }
}
```
Save it with ctrl + x, then press Y, and Enter.

### Run the Call Contract command

```bash
project=hello-world make call-contract
```
This will broadcast the sayGM() call on-chain using your wallet.



## Your Infernet Node is Now Running!

Congratulations! Your Infernet node setup is complete and running on your VPS.

### Verify Everything Is Working

To check if your containers are running properly:

```bash
docker ps
```

To view real-time logs from the node:

```bash
docker logs infernet-node
```

## Check Your Transactions

You can now visit https://basescan.org/ and search your wallet address.

You should see at least 2 transactions if everything was deployed and initialized correctly.

## How to Delete the Whole Node Data 

If you’ve misconfigured something or want to start clean from scratch, follow the steps below:

Stop Docker Compose:

```bash
cd $HOME
docker compose -f infernet-container-starter/deploy/docker-compose.yaml stop
```

Delete the Node Directory:

```bash
sudo rm -rf infernet-container-starter
```

## Disclaimer

This guide is provided for educational purposes only.
You are solely responsible for any actions you take using this code or instructions.
We do not guarantee any rewards, outcomes, or uptime. Use at your own risk.

Running testnet/mainnet nodes may involve interactions with external networks, smart contracts, and wallet keys. Always verify all commands and double-check any wallet/private key usage before proceeding.

Big credit to @rahulboy2002
for creating the original Infernet node setup repo and sharing the initial configuration publicly. This repo is a customized version inspired by his work, reorganized and restructured for clarity and ease of use.

## About Me

- **Twitter**: [https://x.com/SHASHI522004](https://x.com/SHASHI522004)
- **GitHub**: [https://github.com/cryptowithshashi](https://github.com/cryptowithshashi)
- **Telegram**: [https://t.me/crypto_with_shashi](https://t.me/crypto_with_shashi)
