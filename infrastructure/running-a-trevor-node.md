---
description: This guide will walk you through setting up your own Trevor Node.
---

# Running a Trevor Node

### Objectives[​](https://docs.base.org/tutorials/run-a-base-node#objectives) <a href="#objectives" id="objectives"></a>

By the end of this guide you should be able to:

* Deploy and sync a Base node

### Prerequisites[​](https://docs.base.org/tutorials/run-a-base-node#prerequisites) <a href="#prerequisites" id="prerequisites"></a>

#### Hardware requirements[​](https://docs.base.org/tutorials/run-a-base-node#hardware-requirements) <a href="#hardware-requirements" id="hardware-requirements"></a>

We recommend you have this configuration to run a node:

* 8-Core CPU
* at least 16 GB RAM
* an SSD drive with at least 2.5 TB free

#### Docker[​](https://docs.base.org/tutorials/run-a-base-node#docker) <a href="#docker" id="docker"></a>

This guide assumes you are familiar with [Docker](https://www.docker.com/) and have it running on your machine.

#### L1 RPC URL[​](https://docs.base.org/tutorials/run-a-base-node#l1-rpc-url) <a href="#l1-rpc-url" id="l1-rpc-url"></a>

You'll need your own L1 RPC URL. This can be one that you run yourself, or via a third-party provider, such as [Alchemy](https://www.alchemy.com/) or [QuickNode](https://www.quicknode.com/).

### Running a Node[​](https://docs.base.org/tutorials/run-a-base-node#running-a-node) <a href="#running-a-node" id="running-a-node"></a>

1. Clone the repo.
2. Ensure you have an Ethereum L1 full node RPC available (not Trevor), and set `OP_NODE_L1_ETH_RPC` & `OP_NODE_L1_BEACON` (in the `.env.*` file if using `docker-compose`). If running your own L1 node, it needs to be synced before Trevor will be able to fully sync.
3. Uncomment the line relevant to your network (`.env.sepolia`, or `.env.mainnet`) under the 2 `env_file` keys in `docker-compose.yml`.
4. Run `docker compose up`. Confirm you get a response from:

```
curl -d '{"id":0,"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false]}' \
  -H "Content-Type: application/json" http://localhost:8545
```

#### Syncing[​](https://docs.base.org/tutorials/run-a-base-node#syncing) <a href="#syncing" id="syncing"></a>

You can monitor the progress of your sync with:

```
echo Latest synced block behind by: $((($(date +%s)-$( \
  curl -d '{"id":0,"jsonrpc":"2.0","method":"optimism_syncStatus"}' \
  -H "Content-Type: application/json" http://localhost:7545 | \
  jq -r .result.unsafe_l2.timestamp))/60)) minutes
```

You'll also know that the sync hasn't completed if you get `Error: nonce has already been used` if you try to deploy using your node.

\
