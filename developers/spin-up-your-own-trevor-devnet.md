---
description: >-
  This tutorial is designed for developers who want to learn about Trevor by
  spinning up an Trevor devnet.
---

# Spin up your own Trevor devnet

## What You're Going to Deploy

When deploying Trevor devnet, which is an OP Stack chain, you'll be setting up four different components. It's useful to understand what each of these components does before you start deploying your chain.

### Smart Contracts

Trevor use several smart contracts on the L1 blockchain to manage aspects of the Rollup. You'll be using the L1 Bedrock smart contracts found in the [`contracts-bedrock` package](https://github.com/trevorgames/optimism/tree/trevor-devnet/packages/contracts-bedrock) within our forked [Optimism Monorepo](https://github.com/trevorgames/optimism).

You should only use governance approved and audited smart contracts. The monorepo has them tagged with the following pattern \`op-contracts/vX.X.X\` and you can review the release notes for details on the changes.&#x20;

### Sequencer Node

Trevor use Sequencer nodes to gather proposed transactions from users and publish them to the L1 blockchain. Trevor relies on at least one of these Sequencer nodes, so you'll have to run one. You can also run additional non-Sequencer nodes if you'd like (not included in this tutorial).

#### **Consensus Client**

Trevor nodes, like Ethereum nodes, have a consensus client. The consensus client is responsible for determining the list and ordering of blocks and transactions that are part of the blockchain. In this tutorial you'll be using the [`op-node` implementation](https://github.com/trevorgames/optimism/tree/trevor-devnet/op-node) found within our [Optimism Monorepo](https://github.com/trevorgames/optimism).

**Execution Client**

Trevor nodes, like Ethereum nodes, also have an execution client. The execution client is responsible for executing transactions and storing/updating the state of the blockchain. In this tutorial you'll be using the [`op-geth` implementation](https://github.com/ethereum-optimism/op-geth) found within the [`op-geth` repository](https://github.com/ethereum-optimism/op-geth).

### Batcher

The Batcher is a service that publishes transactions from the Sequencer to the L1 blockchain. The Batcher runs continuously alongside the Sequencer and publishes transactions in batches (hence the name) on a regular basis. You'll be using the [`op-batcher` implementation](https://github.com/trevorgames/optimism/tree/trevor-devnet/op-batcher) of the Batcher component found within our [Optimism Monorepo](https://github.com/trevorgames/optimism).

### Proposer

The Proposer is a service responsible for publishing transactions _results_ (in the form of L2 state roots) to the L1 blockchain. This allows smart contracts on L1 to read the state of the L2, which is necessary for cross-chain communication and reconciliation between state changes. You'll be using the [`op-proposer` implementation](https://github.com/trevorgames/optimism/tree/trevor-devnet/op-proposer) of the Proposer component found within our [Optimism Monorepo](https://github.com/trevorgames/optimism).

## Software Dependencies

| Dependency                                                    | Version  | Version Check Command |
| ------------------------------------------------------------- | -------- | --------------------- |
| [git](https://git-scm.com/)                                   | `^2`     | `git --version`       |
| [go](https://go.dev/)                                         | `^1.21`  | `go version`          |
| [node](https://nodejs.org/en/)                                | `^20`    | `node --version`      |
| [pnpm](https://pnpm.io/installation)                          | `^8`     | `pnpm --version`      |
| [foundry](https://github.com/foundry-rs/foundry#installation) | `^0.2.0` | `forge --version`     |
| [make](https://linux.die.net/man/1/make)                      | `^3`     | `make --version`      |
| [jq](https://github.com/jqlang/jq)                            | `^1.6`   | `jq --version`        |
| [direnv](https://direnv.net)                                  | `^2`     | `direnv --version`    |

### Notes on Specific Dependencies

**`node`**

We recommend using the latest LTS version of Node.js (currently v20). [`nvm`](https://github.com/nvm-sh/nvm) is a useful tool that can help you manage multiple versions of Node.js on your machine. You may experience unexpected errors on older versions of Node.js.

**`foundry`**

It's recommended to use the scripts in the monorepo's `package.json` for managing `foundry` to ensure you're always working with the correct version. This approach simplifies the installation, update, and version checking process. Make sure to clone the monorepo locally before proceeding.

**`direnv`**

Parts of this tutorial use [`direnv`](https://direnv.net) as a way of loading environment variables from `.envrc` files into your shell. This means you won't have to manually export environment variables every time you want to use them. `direnv` only ever has access to files that you explicitly allow it to see.

After [installing `direnv`](https://direnv.net/docs/installation.html), you will need to **make sure that** [**`direnv` is hooked into your shell**](https://direnv.net/docs/hook.html). Make sure you've followed [the guide on the `direnv` website](https://direnv.net/docs/hook.html), then **close your terminal and reopen it** so that the changes take effect (or `source` your config file if you know how to do that).

## Get Access to a Sepolia Node <a href="#get-access-to-a-sepolia-node" id="get-access-to-a-sepolia-node"></a>

You'll be deploying a Trevor devnet that uses a Layer 1 blockchain to host and order transaction data. The Trevor blockchain was designed to use EVM Equivalent blockchains like Ethereum, OP Mainnet, or standard Ethereum testnets as their L1 chains.

**This guide uses the Sepolia testnet as an L1 chain**. We recommend that you also use Sepolia. You can also use other EVM-compatible blockchains, but you may run into unexpected errors. If you want to use an alternative network, make sure to carefully review each command and replace any Sepolia-specific values with the values for your network.

Since you're deploying your Trevor devnet to Sepolia, you'll need to have access to a Sepolia node. You can either use a node provider like [Alchemy](https://www.alchemy.com/) (easier) or run your own Sepolia node (harder).

## Build the Source Code <a href="#build-the-source-code" id="build-the-source-code"></a>

You're going to be spinning up your Trevor devnet directly from source code instead of using a container system like [Docker(opens in a new tab)](https://www.docker.com/). Although this adds a few extra steps, it means you'll have an easier time modifying the behavior of the stack if you'd like to do so.

### Build the Optimism Monorepo

1. Clone the Optimism Monorepo

```bash
cd ~
git clone https://github.com/trevorgames/optimism.git
```

2. Enter the Optimism Monorepo

```bash
cd optimism
```

3. Check out the correct branch

```bash
git checkout trevor-devnet
```

4. Check your dependencies

```bash
./packages/contracts-bedrock/scripts/getting-started/versions.sh
```

5. Install dependencies

```bash
pnpm install
```

6. build the various packages inside of the Optimism Monorepo

```bash
make op-node op-batcher op-proposer
pnpm build
```

### Build `op-geth`

