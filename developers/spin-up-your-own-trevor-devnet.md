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

You're going to be spinning up your Trevor devnet directly from source code instead of using a container system like [Docker](https://www.docker.com/). Although this adds a few extra steps, it means you'll have an easier time modifying the behavior of the stack if you'd like to do so.

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
./packages/contracts-bedrock/scripts/trevor-sepolia/versions.sh
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

1. Clone op-geth

```sh
cd ~
git clone https://github.com/ethereum-optimism/op-geth.git
```

2. Enter op-geth

```bash
cd op-geth
```

3. Build op-geth

```bash
make geth
```

## Fill Out Environment Variables

You'll need to fill out a few environment variables before you can start deploying your chain.

1. Enter the Optimism Monorepo

```bash
cd ~/optimism
```

2\. Duplicate the sample environment variable file

```bash
cp .envrc.example .envrc
```

3. Fill out the environment variable file

Open up the environment variable file and fill out the following variables:

| Variable Name | Description                                                                                                                                                                                                  |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `L1_RPC_URL`  | URL for your L1 node (a Sepolia node in this case).                                                                                                                                                          |
| `L1_RPC_KIND` | Kind of L1 RPC you're connecting to, used to inform optimal transactions receipts fetching. Valid options: `alchemy`, `quicknode`, `infura`, `parity`, `nethermind`, `debug_geth`, `erigon`, `basic`, `any`. |

## Generate Addresses

You'll need four addresses and their private keys when setting up the chain:

* The `Admin` address has the ability to upgrade contracts.
* The `Batcher` address publishes Sequencer transaction data to L1.
* The `Proposer` address publishes L2 transaction results (state roots) to L1.
* The `Sequencer` address signs blocks on the p2p network.

1. Enter the Optimism Monorepo

```bash
cd ~/optimism
```

2. Generate new addresses

```bash
./packages/contracts-bedrock/scripts/trevor-sepolia/wallets.sh
```

3. Check the output

Make sure that you see output that looks something like the following:

```
Copy the following into your .envrc file:
  
# Admin address
export TRV_ADMIN_ADDRESS=0x9625B9aF7C42b4Ab7f2C437dbc4ee749d52E19FC
export TRV_ADMIN_PRIVATE_KEY=0xbb93a75f64c57c6f464fd259ea37c2d4694110df57b2e293db8226a502b30a34

# Batcher address
export TRV_BATCHER_ADDRESS=0xa1AEF4C07AB21E39c37F05466b872094edcf9cB1
export TRV_BATCHER_PRIVATE_KEY=0xe4d9cd91a3e53853b7ea0dad275efdb5173666720b1100866fb2d89757ca9c5a
  
# Proposer address
export TRV_PROPOSER_ADDRESS=0x40E805e252D0Ee3D587b68736544dEfB419F351b
export TRV_PROPOSER_PRIVATE_KEY=0x2d1f265683ebe37d960c67df03a378f79a7859038c6d634a61e40776d561f8a2
  
# Sequencer address
export TRV_SEQUENCER_ADDRESS=0xC06566E8Ec6cF81B4B26376880dB620d83d50Dfb
export TRV_SEQUENCER_PRIVATE_KEY=0x2a0290473f3838dbd083a5e17783e3cc33c905539c0121f9c76614dda8a38dca
```

4. Save the addresses

Copy the output from the previous step and paste it into your `.envrc` file as directed.

5. Fund the addresses

**You will need to send ETH to the `Admin`, `Batcher`, and `Proposer` addresses.** The exact amount of ETH required depends on the L1 network being used. **You do not need to send any ETH to the `Sequencer` address as it does not send transactions.**

It's recommended to fund the addresses with the following amounts when using Sepolia:

* `Admin` — 5 Sepolia ETH, to deploy a series of contracts on Sepolia. Do not need Need replenishment
* `Batcher` — 10 Sepolia ETH, to continually send transactions on Sepolia after the devnet is on. Need replenishment later on.
* `Proposer` — 10 Sepolia ETH, to continually send transactions on Sepolia after the devnet is on. Need replenishment later on.

## Load Environment variables

Now that you've filled out the environment variable file, you need to load those variables into your terminal.

1. Enter the Optimism Monorepo

```bash
cd ~/optimism
```

2. Load the variables with direnv

```bash
direnv allow
```

WARNING: `direnv` will unload itself whenever your `.envrc` file changes. **You **_**must**_**  rerun the following command every time you change the `.envrc`file.**

3. Confirm that the variables were loaded

After running `direnv allow` you should see output that looks something like the following (the exact output will vary depending on the variables you've set, don't worry if it doesn't look exactly like this):

```bash
direnv: loading ~/optimism/.envrc                                                            
direnv: export +DEPLOYMENT_CONTEXT +ETHERSCAN_API_KEY +TRV_ADMIN_ADDRESS +TRV_ADMIN_PRIVATE_KEY +TRV_BATCHER_ADDRESS +TRV_BATCHER_PRIVATE_KEY +TRV_PROPOSER_ADDRESS +TRV_PROPOSER_PRIVATE_KEY +TRV_SEQUENCER_ADDRESS +TRV_SEQUENCER_PRIVATE_KEY +IMPL_SALT +L1_RPC_KIND +L1_RPC_URL +PRIVATE_KEY +TENDERLY_PROJECT +TENDERLY_USERNAME
```

**If you don't see this output, you likely haven't properly configured `direnv`.** Make sure you've configured `direnv` properly and run `direnv allow` again so that you see the desired output.

## Configure your network

Once you've built both repositories, you'll need to head back to the Optimism Monorepo to set up the configuration file for your chain. Currently, chain configuration lives inside of the [`contracts-bedrock`](https://github.com/trevorgames/optimism/tree/trevor-devnet/packages/contracts-bedrock) package in the form of a JSON file.

1. Enter the Optimism Monorepo

```bash
cd ~/optimism
```

2. Move into the contracts-bedrock package

```bash
cd packages/contracts-bedrock
```

3. Generate the configuration file

Run the following script to generate the `trevor-seplia.json` configuration file inside of the `deploy-config` directory.

```bash
./scripts/trevor-sepolia/config.sh
```

## Deploy the L1 contracts

Once you've configured your network, it's time to deploy the L1 contracts necessary for the functionality of the chain.

1. Deploy the L1 contracts

```bash
forge script scripts/Deploy.s.sol:Deploy --private-key $TRV_ADMIN_PRIVATE_KEY --broadcast --rpc-url $L1_RPC_URL --slow
```

If you see a nondescript error that includes `EvmError: Revert` and `Script failed` then you likely need to change the `IMPL_SALT` environment variable. This variable determines the addresses of various smart contracts that are deployed via [CREATE2](https://eips.ethereum.org/EIPS/eip-1014). If the same `IMPL_SALT` is used to deploy the same contracts twice, the second deployment will fail. **You can generate a new `IMPL_SALT` by running `direnv allow` anywhere in the Optimism Monorepo.**

2. Generate contract artifacts

```bash
forge script scripts/Deploy.s.sol:Deploy --sig 'sync()' --rpc-url $L1_RPC_URL
```

## Generate the L2 config files

Now that you've set up the L1 smart contracts you can automatically generate several configuration files that are used within the Consensus Client and the Execution Client.

You need to generate three important files:

* `genesis.json` includes the genesis state of the chain for the Execution Client.
* `rollup.json` includes configuration information for the Consensus Client.
* `jwt.txt` is a [JSON Web Token](https://jwt.io/introduction) that allows the Consensus Client and the Execution Client to communicate securely (the same mechanism is used in Ethereum clients).

1. Navigate to the op-node package

```bash
cd ~/optimism/op-node
```

2. Create genesis files

Now you'll generate the `genesis.json` and `rollup.json` files within the `op-node` folder:

```bash
go run cmd/main.go genesis l2 \
  --deploy-config ../packages/contracts-bedrock/deploy-config/trevor-sepolia.json \
  --deployment-dir ../packages/contracts-bedrock/deployments/trevor-sepolia/ \
  --outfile.l2 genesis.json \
  --outfile.rollup rollup.json \
  --l1-rpc $L1_RPC_URL
```

3. Create an authentication key

Next you'll create a [JSON Web Token](https://jwt.io/introduction) that will be used to authenticate the Consensus Client and the Execution Client. This token is used to ensure that only the Consensus Client and the Execution Client can communicate with each other. You can generate a JWT with the following command:

```bash
openssl rand -hex 32 > jwt.txt
```

4. Copy genesis files into the op-geth directory

Finally, you'll need to copy the `genesis.json` file and `jwt.txt` file into `op-geth` so you can use it to initialize and run `op-geth`:

```bash
cp genesis.json ~/op-geth
cp jwt.txt ~/op-geth
```

## Initialize `op-geth`

You're almost ready to run your Trevor chain! Now you just need to run a few commands to initialize `op-geth`. You're going to be running a Sequencer node, so you'll need to import the `Sequencer` private key that you generated earlier. This private key is what your Sequencer will use to sign new blocks.

1. Navigate to the op-geth directory

```bash
cd ~/op-geth
```

2. Create a data directory folder

```bash
mkdir datadir
```

3. Initialize op-geth

```bash
build/bin/geth init --datadir=datadir genesis.json
```

4. Import teh Sequencer key

```bash
cd ~/optimism
echo $TRV_SEQUENCER_PRIVATE_KEY | sed s/^..// > ~/op-geth/sequencer_key.txt
cd ~/op-geth
./build/bin/geth account import --datadir ./datadir sequencer_key.txt
```

## Start `op-geth`

Now you'll start `op-geth`, your Execution Client. Note that you won't start seeing any transactions until you start the Consensus Client in the next step.

1. Open up a new terminal

You'll need a terminal window to run `op-geth` in.

2. Navigate to the op-geth directory

```bash
cd ~/op-geth
```

3. Run op-geth

You're using `--gcmode=archive` to run `op-geth` here because this node will act as your Sequencer. It's useful to run the Sequencer in archive mode because the `op-proposer` requires access to the full state. Feel free to run other (non-Sequencer) nodes in full mode if you'd like to save disk space. It's important that you've already initialized the geth node at this point as per the previous section. Failure to do this will cause startup issues between `op-geth` and `op-node`.

```bash
./build/bin/geth \
  --datadir ./datadir \
  --http \
  --http.corsdomain="*" \
  --http.vhosts="*" \
  --http.addr=0.0.0.0 \
  --http.api=web3,debug,eth,txpool,net,engine \
  --ws \
  --ws.addr=0.0.0.0 \
  --ws.port=8546 \
  --ws.origins="*" \
  --ws.api=debug,eth,txpool,net,engine \
  --syncmode=full \
  --gcmode=archive \
  --nodiscover \
  --maxpeers=0 \
  --networkid=689388 \
  --authrpc.vhosts="*" \
  --authrpc.addr=0.0.0.0 \
  --authrpc.port=8551 \
  --authrpc.jwtsecret=./jwt.txt \
  --rollup.disabletxpoolgossip=true
```

## Start `op-node`

Once you've got `op-geth` running you'll need to run `op-node`. Like Ethereum, Trevor has a Consensus Client (`op-node`) and an Execution Client (`op-geth`). The Consensus Client "drives" the Execution Client over the Engine API.

1. Open up a new terminal

You'll need a terminal window to run the `op-node` in.

2. Navigate to the op-node directory

```bash
cd ~/optimism/op-node
```

3. Run op-node

```bash
./bin/op-node \
  --l2=http://localhost:8551 \
  --l2.jwt-secret=./jwt.txt \
  --sequencer.enabled \
  --sequencer.l1-confs=5 \
  --verifier.l1-confs=4 \
  --rollup.config=./rollup.json \
  --rpc.addr=0.0.0.0 \
  --rpc.port=8547 \
  --p2p.disable \
  --rpc.enable-admin \
  --p2p.sequencer.key=$TRV_SEQUENCER_PRIVATE_KEY \
  --l1=$L1_RPC_URL \
  --l1.rpckind=$L1_RPC_KIND
```

Once you run this command, you should start seeing the `op-node` begin to sync L2 blocks from the L1 chain. Once the `op-node` has caught up to the tip of the L1 chain, it'll begin to send blocks to `op-geth` for execution. At that point, you'll start to see blocks being created inside of `op-geth`.

**By default, your `op-node` will try to use a peer-to-peer to speed up the synchronization process.** If you're using a chain ID that is also being used by others, like the default chain ID for this tutorial (42069), your `op-node` will receive blocks signed by other sequencers. These requests will fail and waste time and network resources. **To avoid this, this tutorial starts with peer-to-peer synchronization disabled (`--p2p.disable`).**

Once you have multiple nodes, you may want to enable peer-to-peer synchronization. You can add the following options to the `op-node` command to enable peer-to-peer synchronization with specific nodes:

```
  --p2p.static=<nodes> \
  --p2p.listen.ip=0.0.0.0 \
  --p2p.listen.tcp=9003 \
  --p2p.listen.udp=9003 \
```

You can alternatively also remove the --p2p.static option, but you may see failed requests from other chains using the same chain ID.

## Start `op-batcher`

The `op-batcher` takes transactions from the Sequencer and publishes those transactions to L1. Once these Sequencer transactions are included in a finalized L1 block, they're officially part of the canonical chain. The `op-batcher` is critical!

It's best to give the `Batcher` address at least 1 Sepolia ETH to ensure that it can continue operating without running out of ETH for gas. Keep an eye on the balance of the `Batcher` address because it can expend ETH quickly if there are a lot of transactions to publish.

1. Open up a new terminal

You'll need a terminal window to run the `op-batcher` in.

2. Navigate to the op-batcher directory

```bash
cd ~/optimism/op-batcher
```

3. Run op-batcher

```bash
./bin/op-batcher \
  --l2-eth-rpc=http://localhost:8545 \
  --rollup-rpc=http://localhost:8547 \
  --poll-interval=1s \
  --sub-safety-margin=6 \
  --num-confirmations=1 \
  --safe-abort-nonce-too-low-count=3 \
  --resubmission-timeout=30s \
  --rpc.addr=0.0.0.0 \
  --rpc.port=8548 \
  --rpc.enable-admin \
  --max-channel-duration=1 \
  --l1-eth-rpc=$L1_RPC_URL \
  --private-key=$TRV_BATCHER_PRIVATE_KEY
```

The `--max-channel-duration=n` setting tells the batcher to write all the data to L1 every `n` L1 blocks. When it is low, transactions are written to L1 frequently and other nodes can synchronize from L1 quickly. When it is high, transactions are written to L1 less frequently and the batcher spends less ETH. If you want to reduce costs, either set this value to 0 to disable it or increase it to a higher value.

## Start `op-proposer`

Now start `op-proposer`, which proposes new state roots.

1. Open up a new terminal

You'll need a terminal window to run the `op-proposer` in.

2. Navigate to the op-proposer directory

```bash
cd ~/optimism/op-proposer
```

3. Run op-proposer

```bash
./bin/op-proposer \
  --poll-interval=12s \
  --rpc.port=8560 \
  --rollup-rpc=http://localhost:8547 \
  --l2oo-address=$(cat ../packages/contracts-bedrock/deployments/trevor-sepolia/L2OutputOracleProxy.json | jq -r .address) \
  --private-key=$TRV_PROPOSER_PRIVATE_KEY \
  --l1-eth-rpc=$L1_RPC_URL
```

## Connect Your Wallet to Your Chain

You now have a fully functioning OP Stack Rollup with a Sequencer node running on `http://localhost:8545`. You can connect your wallet to this chain the same way you'd connect your wallet to any other EVM chain.&#x20;

## Get ETH On Your Chain

Once you've connected your wallet, you'll probably notice that you don't have any ETH to pay for gas on your chain. The easiest way to deposit Sepolia ETH into your chain is to send ETH directly to the `L1StandardBridge` contract.

1. Navigate to the contracts-bedrock directory

```bash
cd ~/optimism/packages/contracts-bedrock
```

2. Get the address of the L1StandardBridgeProxy contract

```bash
cat deployments/trevor-sepolia/L1StandardBridgeProxy.json | jq -r .address
```

3. Send some Sepolia ETH to the L1StandardBridgeProxy contract

Grab the L1 bridge proxy contract address and, using the wallet that you want to have ETH on your Rollup, send that address a small amount of ETH on Sepolia (0.1 or less is fine). This will trigger a deposit that will mint ETH into your wallet on L2. It may take up to 5 minutes for that ETH to appear in your wallet on L2.

## See Your Trevor Rollup in Action

You can interact with your Trevor Rollup the same way you'd interact with any other EVM chain. Send some transactions, deploy some contracts, and see what happens!









