---
id: op-stack
title: How to Use the OP Stack with Avail
sidebar_label: Avail-Powered OP Stack Optimium
description: 'Discover how to utilize Avail as a data availability layer.'
keywords:
  - documentation
  - avail
  - develop
  - build
  - data availability
  - da
image: https://docs.availproject.org/img/avail/AvailDocs.png
---

## Introduction

Embark on setting up your own Avail-Optimism chain. This guide targets Ethereum's Goerli testnet and Avail Goldberg testnet. For a detailed understanding, review the [<ins>Optimism Documentation</ins>](https://stack.optimism.io/docs/build/getting-started/#know-before-you-go).

In this guide, you will conduct the following:

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Compile the Core Codebase](#compile-the-core-codebase)
  - [Build the Adapter Source](#build-the-adapter-source)
  - [Build the Optimism Geth Source](#build-the-optimism-geth-source)
  - [Get Access to a Goerli Node](#get-access-to-a-goerli-node)
- [Generate and Secure Keys](#generate-and-secure-keys)
- [Network Configuration and Setup](#network-configuration-and-setup)
- [Core Contract Deployment](#core-contract-deployment)
- [Setting Up L2 Configuration](#setting-up-l2-configuration)
- [Initialize and Configure Geth](#initialize-and-configure-geth)
- [Launch and Monitor Nodes](#launch-and-monitor-nodes)
  - [Running op-geth](#running-op-geth)
    - [Why Archive Mode?](#why-archive-mode)
    - [Reinitializing op-geth](#reinitializing-op-geth)
  - [Running op-node](#running-op-node)
    - [P2P Synchronization](#p2p-synchronization)
  - [Running op-batcher](#running-op-batcher)
    - [Controlling Batcher Costs](#controlling-batcher-costs)
  - [Running op-proposer](#running-op-proposer)
- [Acquire Goerli ETH for Layer 2](#acquire-goerli-eth-for-layer-2)
- [Conduct Test Transactions](#conduct-test-transactions)

## Prerequisites

Ensure you have installed the following software.

> Installation commands are based on Ubuntu 20.04 LTS:

<table>
  <tr>
    <td valign="top">

| Software                                                                 | Version                                                                                |
| ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| [<ins>Git</ins>](https://git-scm.com/)                                   | OS default                                                                             |
| [<ins>Go</ins>](https://go.dev/)                                         | 1.20                                                                                   |
| [<ins>Node</ins>](https://nodejs.org/en/)                                | 16.19.0                                                                                |
| [<ins>Pnpm</ins>](https://pnpm.io/installation)                          | 8.5.6                                                                                  |
| [<ins>Make</ins>](https://linux.die.net/man/1/make)                      | OS default                                                                             |
| [<ins>jq</ins>](https://github.com/jqlang/jq)                            | OS default                                                                             |
| [<ins>direnv</ins>](https://direnv.net/)                                 | Latest                                                                                 |
| [<ins>Foundry</ins>](https://github.com/foundry-rs/foundry#installation) | Foundry will be installed locally within the project's environment, via `package.json` |

</td>
<td valign="top">

```bash
# Install Git
sudo apt install -y git curl make jq

# Install Go
wget https://go.dev/dl/go1.20.linux-amd64.tar.gz
tar xvzf go1.20.linux-amd64.tar.gz
sudo cp go/bin/go /usr/bin/go
sudo mv go /usr/lib
echo export GOROOT=/usr/lib/go >> ~/.bashrc

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs npm

# Install Pnpm
sudo npm install -g pnpm

# Install Make
sudo apt install -y make

# Install jq
sudo apt install -y jq

# Install direnv
sudo apt install -y direnv
```

</td>
  </tr>
</table>

## Compile the Core Codebase

Setting up the EVM Rollup requires compiling code from two critical repositories: the [<ins>avail-op-stack-adapter monorepo</ins>](https://github.com/availproject/avail-op-stack-adapter) and the [<ins>op-geth repository</ins>](https://github.com/ethereum-optimism/op-geth).

### Build the Adapter Source

1. Clone and navigate to the Avail adapter:

   ```bash
   git clone https://github.com/availproject/avail-op-stack-adapter.git
   cd avail-op-stack-adapter
   ```

2. Install modules (simultaneously, you may start [<ins>building `op-geth`</ins>](#building-op-geth)):

   ```bash
   pnpm install
   ```

3. Compile the necessary packages:
   ```bash
   make op-node op-batcher op-proposer
   pnpm build
   ```

### Build the Optimism Geth Source

1. Clone and navigate to `op-geth`:

   ```bash
   git clone https://github.com/ethereum-optimism/op-geth.git
   cd op-geth
   ```

2. Compile `op-geth`:
   ```bash
   make geth
   ```

### Get Access to a Goerli Node

For deploying to Goerli, access an L1 node using a provider like [<ins>Alchemy</ins>](https://www.alchemy.com/) or [<ins>run your own Goerli node</ins>](https://notes.ethereum.org/@launchpad/goerli).

## Generate and Secure Keys

Create four essential accounts with private keys:

- `Admin` (contract upgrade authority)
- `Batcher` (publishes Sequencer data to L1)
- `Proposer` (publishes L2 results to L1)
- `Sequencer` (signs blocks on the p2p network)

You can use `cast wallet` in the `contracts-bedrock` package for key generation:

1. Navigate to `contracts-bedrock`:

   ```bash
   cd avail-op-stack-adapter/packages/contracts-bedrock
   ```

2. Generate accounts:

   ```bash
   echo "Admin:"
   cast wallet new
   echo "Proposer:"
   cast wallet new
   echo "Batcher:"
   cast wallet new
   echo "Sequencer:"
   cast wallet new
   ```

   You should see an output similar to:

   ```bash
   Admin:
   Successfully created new keypair.
   Address:     0xc4A01194958DE0D90A876e8A5fc9D7B530072148
   Private key: 0xb8e39bd94a210e410c4024e1cc91014de45a5eb1e42f3aa99a368b5a5ac19b45
   Proposer:
   Successfully created new keypair.
   Address:     0xFC0374Ae658e46cA4022acA179d3cb6D8e1A4934
   Private key: 0xa9bc1b3f5deb1e00251df68bf86e3493b25bc5430665433546f2f9aacc748d1a
   Batcher:
   Successfully created new keypair.
   Address:     0xD6857B5BE9468Be67d64ABaB48459378d5329b96
   Private key: 0xe9cd8960fc7984a301d567b819e0c62871eb2c7239c2e66b8f319eaa45c3cbd5
   Sequencer:
   Successfully created new keypair.
   Address:     0x33348817E4B1192D576C4f157e9a5EC93dc5392D
   Private key: 0xd98b49e11e4e0be9931017831395e6644a50c36285d08e14d1a479af5ee08675
   ```

   Record and securely store these key details. You'll need to fund `Admin`, `Proposer`, and `Batcher` with Goerli ETH (2 ETH for `Admin`, 5 ETH for `Proposer`, 10 ETH for `Batcher`).

   :::tip Note for Production

   Use secure hardware for key management in production environments.
   `cast wallet` is not designed for production deployments.

   :::

## Network Configuration and Setup

After building the repositories, configure your chain settings in the [<ins>contracts-bedrock package</ins>](https://github.com/availproject/avail-optimism/tree/129032f15b76b0d2a940443a39433de931a97a44/packages/contracts-bedrock).

1. Navigate to `contracts-bedrock`:

   ```bash
   cd ~/avail-op-stack-adapter/packages/contracts-bedrock
   ```

2. Copy the environment file:

   ```bash
   cp .envrc.example .envrc
   ```

3. Edit `.envrc` with necessary values:

   - `ETH_RPC_URL`: URL for your L1 node.
   - `PRIVATE_KEY`: Private key of the Admin account.
   - `DEPLOYMENT_CONTEXT`: Name of the network; should be "avail-optimism"

4. Activate the environment with `direnv`:

   > If you need to install `direnv`, ensure you also [<ins>modify the shell configuration</ins>](https://direnv.net/docs/hook.html).

   ```bash
   direnv allow .
   ```

5. Choose an L1 block as a starting point using `cast` command:

   > Using a finalized L1 block as our starting block is best.

   ```bash
   cast block finalized --rpc-url $ETH_RPC_URL | grep -E "(timestamp|hash|number)"
   ```

   You should get a response that looks like:

   ```bash
   hash                 0x784d8e7f0e90969e375c7d12dac7a3df6879450d41b4cb04d4f8f209ff0c4cd9
   number               8482289
   timestamp            1676253324
   ```

6. Next, create and modify `deploy-config/avail-optimism.json` based on `deploy-config/getting-started.json`. Retain the default settings provided in the configuration and apply the following modifications:

   - Replace `ADMIN` with the address of the Admin account you generated earlier.
   - Replace `PROPOSER` with the address of the Proposer account you generated earlier.
   - Replace `BATCHER` with the address of the Batcher account you generated earlier.
   - Replace `SEQUENCER` with the address of the Sequencer account you generated earlier.
   - Replace `BLOCKHASH` with the blockhash you got from the cast command.
   - Replace `TIMESTAMP` with the timestamp you got from the cast command. Note that although all the other fields are strings, this field is a number! Don’t include the quotation marks.

7. Configure `enableDA` in `avail-optimism.json` (`true` for Avail chain as DA, `false` for Ethereum).

8. Enter `op-avail` module:

   ```bash
   cd ~/avail-op-stack-adapter/op-avail
   ```

9. Create `config.json` with necessary variables (`seed`, `api_url`, `app_id`).

   ```bash
   {
     "seed": "test test test test test test test test test test test test",
     "api_url": "wss://goldberg.avail.tools:443/ws",
     "app_id": 1
   }
   ```

## Core Contract Deployment

Deploy essential L1 contracts for the chain’s functionality:

1. Create `avail-optimism` directory:

   ```bash
   mkdir deployments/avail-optimism
   ```

2. Deploy contracts (can take up to 15 minutes):
   ```bash
   forge script scripts/Deploy.s.sol:Deploy --private-key $PRIVATE_KEY --broadcast --rpc-url $ETH_RPC_URL
   forge script scripts/Deploy.s.sol:Deploy --sig 'sync()' --private-key $PRIVATE_KEY --broadcast --rpc-url $ETH_RPC_URL
   ```

## Setting Up L2 Configuration

After configuring the L1 layer, focus shifts to establishing the L2 infrastructure. This involves generating three key files:

- `genesis.json` for the genesis block
- `rollup.json` for rollup configurations
- `jwt.txt` for secure communication between `op-node` and `op-geth`

1. Navigate to the `op-node` directory:

   ```bash
   cd ~/avail-op-stack-adapter/op-node
   ```

2. Run the following command, ensuring you replace `<RPC>` with your specific L1 RPC URL. This generates the `genesis.json` and `rollup.json` files:

   ```bash
   go run cmd/main.go genesis l2 \
       --deploy-config ../packages/contracts-bedrock/deploy-config/avail-optimism.json \
       --deployment-dir ../packages/contracts-bedrock/deployments/avail-optimism/ \
       --outfile.l2 genesis.json \
       --outfile.rollup rollup.json \
       --l1-rpc=$L1_RPC
   ```

   You'll find the newly created `genesis.json` and `rollup.json` in the `op-node` package.

3. Generate a `jwt.txt` file, which is crucial for the secure interaction between nodes:

   ```bash
   openssl rand -hex 32 > jwt.txt
   ```

4. To get `op-geth` ready, move the `genesis.json` and `jwt.txt` files into its directory:

   ```bash
   cp genesis.json ~/op-geth
   cp jwt.txt ~/op-geth
   ```

These steps ensure the L2 layer is correctly configured and ready for integration with the L1 components, paving the way for a fully functional EVM Rollup on the Avail-OP Stack.

## Initialize and Configure Geth

Prepare `op-geth` for running the chain:

1. Navigate to `op-geth`:

   ```bash
   cd ~/op-geth
   ```

2. Create a data directory:

   ```bash
   mkdir datadir
   ```

3. Initialize with the genesis file:
   ```bash
   build/bin/geth init --datadir=datadir genesis.json
   ```

## Launch and Monitor Nodes

Running `op-geth` and `op-node` is essential for every node. `op-batcher` and `op-proposer` are exclusive to the sequencer.

Set the following environment variables:

| Variable       | Value                                     |
| -------------- | ----------------------------------------- |
| `SEQ_KEY`      | Sequencer private key                     |
| `BATCHER_KEY`  | Batcher private key (minimum 1 ETH)       |
| `PROPOSER_KEY` | Proposer private key                      |
| `L1_RPC`       | L1 node URL                               |
| `RPC_KIND`     | L1 server type (e.g., alchemy, quicknode) |
| `L2OO_ADDR`    | L2OutputOracleProxy address               |

### Running op-geth

To initiate `op-geth`, navigate to its directory and execute the following commands:

```bash
cd ~/op-geth
./build/bin/geth \
  --datadir ./datadir \
  --http \
  --http.corsdomain="*" \
  --http.vhosts="*" \
  --http.addr=0.0.0.0 \
  --http.port=9545 \
  --http.api=web3,debug,eth,txpool,net,engine \
  --ws \
  --ws.addr=0.0.0.0 \
  --ws.port=9546 \
  --ws.origins="*" \
  --ws.api=debug,eth,txpool,net,engine \
  --syncmode=full \
  --gcmode=archive \
  --nodiscover \
  --maxpeers=0 \
  --networkid=42069 \
  --authrpc.vhosts="*" \
  --authrpc.addr=0.0.0.0 \
  --authrpc.port=9551 \
  --authrpc.jwtsecret=./jwt.txt \
  --rollup.disabletxpoolgossip=true
```

`op-geth` is now active, but block creation will begin once `op-node` is operational.

#### Why Archive Mode?

Archive mode, requiring more disk space than full mode, is essential for:

1. `op-proposer` to access the full state history.
2. The explorer's functionality.

#### Reinitializing op-geth

In cases of database corruption indicated by `op-node` errors or failure to find L2 heads, follow these steps:

1. Stop `op-geth`.
2. Remove the existing data:
   ```bash
   cd ~/op-geth
   rm -rf datadir/geth
   ```
3. Reinitialize:
   ```bash
   build/bin/geth init --datadir=datadir genesis.json
   ```
4. Restart `op-geth` and then `op-node`.

### Running op-node

To launch `op-node`, which acts as a consensus client, run:

```bash
cd ~/optimism/op-node
./bin/op-node \
  --l2=http://localhost:9551 \
  --l2.jwt-secret=./jwt.txt \
  --sequencer.enabled \
  --sequencer.l1-confs=3 \
  --verifier.l1-confs=3 \
  --rollup.config=./rollup.json \
  --rpc.addr=0.0.0.0 \
  --rpc.port=9547 \
  --p2p.disable \
  --rpc.enable-admin \
  --p2p.sequencer.key=$SEQ_KEY \
  --l1=$L1_RPC \
  --l1.rpckind=$RPC_KIND
```

Block creation will commence once `op-node` starts processing L1 information and interfaces with `op-geth`.

#### P2P Synchronization

To optimize synchronization and avoid network resource waste:

- Disable p2p sync (`--p2p.disable`) by default.
- Use specific command line parameters for synchronization among multiple nodes.

### Running op-batcher

`op-batcher` is crucial in publishing transactions from the Sequencer to L1. Ensure it has at least 1 Goerli ETH for operational continuity.

```bash
cd ~/optimism/op-batcher
./bin/op-batcher \
  --l2-eth-rpc=http://localhost:9545 \
  --rollup-rpc=http://localhost:9547 \
  --poll-interval=10s \
  --sub-safety-margin=6 \
  --num-confirmations=1 \
  --safe-abort-nonce-too-low-count=3 \
  --resubmission-timeout=30s \
  --rpc.addr=0.0.0.0 \
  --rpc.port=9548 \
  --rpc.enable-admin \
  --max-channel-duration=1 \
  --l1-eth-rpc=$L1_RPC \
  --private-key=$BATCHER_KEY
```

#### Controlling Batcher Costs

Adjust the `--max-channel-duration=n` setting to balance transaction frequency on L1 and the operational costs of the batcher.

### Running op-proposer

Finally, start `op-proposer` to propose new state roots:

```bash
cd ~/optimism/op-proposer
./bin/op-proposer \
  --poll-interval=12s \
  --rpc.port=9560 \
  --rollup-rpc=http://localhost:9547 \
  --l2oo-address=$L2OO_ADDR \
  --private-key=$PROPOSER_KEY \
  --l1-eth-rpc=$L1_RPC
```

## Acquire Goerli ETH for Layer 2

To obtain ETH on your Rollup:

1. Go to `contracts-bedrock`:

   ```bash
   cd ~/optimism/packages/contracts-bedrock
   ```

2. Find the L1 standard bridge contract address:

   ```bash
   cat deployments/avail-optimism/L1StandardBridgeProxy.json | jq -r .address
   ```

3. Send Goerli ETH to the bridge contract address.

## Conduct Test Transactions

You now have a fully operational Avail-Powered Optimism-based EVM Rollup. Experiment with it as you would with any other test blockchain.

<!-- Add test transactions -->

**Congratulations on setting up your chain!**
