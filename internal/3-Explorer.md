# 为 Trevor Rollup 搭建 Blockscout 浏览器

## 参考文档

blockscout 官方教程：https://docs.blockscout.com/for-developers/deployment/docker-compose-deployment

optimism 教程：https://docs.optimism.io/builders/chain-operators/tools/explorer

后端环境变量：https://docs.blockscout.com/for-developers/information-and-settings/env-variables

前端环境变量：
https://github.com/blockscout/frontend/blob/main/docs/ENVS.md

## 依赖
- Docker
- docker-compose

## 安装与启动

```bash
git clone https://github.com/blockscout/blockscout.git
cd blockscout/docker-compose

# 适配之前的搭建的 L2，services/user-ops-indexer.yml 中 8545 应改为 8546
vi services/user-ops-indexer.yml

# 适配之前搭建的 L2，services/backend.yml 中的镜像应改为 blockscout/blockscout-optimism:latest
vi services/backend.yml

# 启动，或重启以使配置改动生效
docker-compose -f geth.yml up -d

# 查看
docker ps
docker logs -f backend
curl localhost

# 停止
docker-compose -f geth.yml down
```

## 配置与定制

### 基本身份

envs/common-blockscout.env

```bash
NETWORK=Trevor Sepolia

# 实践证明这个环境变量必须设置正确，否则后续的合约验证服务不能正常运行
CHAIN_ID=689388
```

envs/common-frontend.env

```bash
NEXT_PUBLIC_NETWORK_NAME=Trevor Sepolia
NEXT_PUBLIC_NETWORK_SHORT_NAME=Trevor Sepolia
NEXT_PUBLIC_NETWORK_ID=689388
```


### 外部访问

#### IP 访问

按默认配置，只能在部署 docker 服务的本机使用浏览器访问 http://localhost 才能正常显示页面。

为了方便外部访问，首先需要在公网路由器做端口映射。假设路由器公网 IP 为1.2.3.4。

注意：在中国，如果没有备案，公网路由器的 80, 443, 8080 端口均无法使用。

```bash
# 路由器端口 -> 内部端口
8000 -> 80
8180 -> 8080
8081 -> 8081
```

改动 `envs/common-frontend.env`。

```bash
NEXT_PUBLIC_API_HOST=1.2.3.4:8000
NEXT_PUBLIC_STATS_API_HOST=http://1.2.3.4:8180

NEXT_PUBLIC_APP_HOST=1.2.3.4:8000
NEXT_PUBLIC_VISUALIZE_API_HOST=http://1.2.3.4:8081
```

改动 `proxy/default.conf.template`。该文件改动之后 `docker-compose -f geth.yml up -d` 命令并不会重启 proxy 容器，可以在此之前先运行 `docker kill proxy` 停掉容器再重启服务。

```bash
server {
    listen       8080;
    add_header 'Access-Control-Allow-Origin' 'http://1.2.3.4:8000' always;
...
```

到这里，已经可以用路由器的公网 IP 正常访问浏览器页面了。

#### 域名和 HTTPS 访问
如果想要域名以及 HTTPS 访问，可首先申请域名，然后在公有云上配置反向代理和证书。下面是 Caddyfile 的参考配置：
```bash
dev-explorer.trevorgames.xyz {
  reverse_proxy http://1.2.3.4:8000
}

dev-stats.trevorgames.xyz {
  reverse_proxy http://1.2.3.4:8180
}

dev-visualize.trevorgames.xyz {
  reverse_proxy http://1.2.3.4:8081
}

# 下面的反向代理是为后了方便后续的合约验证服务，且需要海外公有云
eth-bytecode-db.trevorgames.xyz {
  reverse_proxy https://eth-bytecode-db.services.blockscout.com {
    header_up Host {http.reverse_proxy.upstream.hostport}
  }
}
```

当然之前的相关配置也做如下改动：
```bash
NEXT_PUBLIC_API_HOST=dev-explorer.trevorgames.xyz
NEXT_PUBLIC_API_PROTOCOL=https
NEXT_PUBLIC_STATS_API_HOST=https://dev-stats.trevorgames.xyz

NEXT_PUBLIC_APP_HOST=dev-explorer.trevorgames.xyz
NEXT_PUBLIC_APP_PROTOCOL=https

NEXT_PUBLIC_VISUALIZE_API_HOST=https://dev-visualize.trevorgames.xyz

NEXT_PUBLIC_API_WEBSOCKET_PROTOCOL=wss

server {
    listen       8080;
    add_header 'Access-Control-Allow-Origin' 'https://dev-explorer.trevorgames.xyz' always;
```

### 合约交互

#### 识别预部署合约
首先，浏览器不能将所有预部署的合约地址（比如很多以 0x4200 开头）正常识别为合约。解决办法是向后端提供 L2 链的 `genesis.json` 文件。

services/backend.yml 

```yml
services:
  backend:
    volumes:
      - /nvme/op-geth/genesis.json:/genesis.json
```

envs/common-blockscout.env

```bash
CHAIN_SPEC_PATH=/genesis.json
```

重启服务大概10分钟之后，浏览器就能识别预部署合约了。

#### 合约验证

在浏览器上进行合约交互的前提是向浏览器验证合约字节码对应的源码。但是 “Verify & publish ” 页面加载良久，且编译器下拉列表为空。查看后端日志输出：Error while sending request to verification microservice url: https://eth-bytecode-db.services.blockscout.com/api/v2/verifier/solidity/versions

原来是这个网址在国内访问不到，可以替换成之前配置的反向代理地址。

envs/common-blockscout.env

```bash
MICROSERVICE_SC_VERIFIER_URL=https://eth-bytecode-db.trevorgames.xyz/
```

重启之后合约验证服务就正常了，更方便的是预部署的合约还会匹配数据库而自动验证通过。

至于用户部署的合约，在浏览器上操作验证还是有些麻烦，更方便的办法是通过 foundry 等命令行工具进行验证合约。

```bash
# 部署单个合约时验证，用文件传输空白分割的构造参数。缺点是无法输入带空格的字符串，即使用引号包起来
forge create --rpc-url $L2_RPC_URL --private-key $DEPLOY_PRIVATE_KEY --verify --verifier blockscout --verifier-url $L2_EXPLORER_URL/api? Contract.sol:Contract --constructor-args-path args.txt

# 部署单个合约时验证，在最后用命令行参数传入构造参数
forge create --rpc-url $L2_RPC_URL --private-key $DEPLOY_PRIVATE_KEY --verify --verifier blockscout --verifier-url $L2_EXPLORER_URL/api? Contract.sol:Contract  --constructor-args argA "arg B" argc

# 运行脚本部署多个合约时验证
forge script --rpc-url $L2_RPC_URL --private-key $DEPLOY_PRIVATE_KEY --verify --verifier blockscout --verifier-url $L2_EXPLORER_URL/api? script.s.sol

# 验证已部署合约
forge verify-contract --chain-id 689388 --verifier=blockscout --verifier-url=$L2_EXPLORER_URL/api? $CONTRACT_ADDRESS Contract.sol:Contract
```

#### 合约读写

合约一经验证，就可在浏览器上进行读的操作。然而写的操作还需要与钱包插件的交互。

envs/common-frontend.env

```bash

# 从 https://cloud.walletconnect.com/ 获取 Project ID
NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID=
NEXT_PUBLIC_NETWORK_RPC_URL=https://dev-rpc.trevorgames.xyz/
```

这样既可以通过浏览器直接将网络添加至 Metamask 等钱包插件，也能连接钱包之后直接在浏览器中进行合约的写操作。


### 跨链

显示 L1 与 L2 之间的资产流动。

envs/common-frontend.env
```bash
NEXT_PUBLIC_ROLLUP_TYPE=optimistic
NEXT_PUBLIC_ROLLUP_L1_BASE_URL=https://eth-sepolia.blockscout.com

# 自我实现的跨链桥的前端
NEXT_PUBLIC_ROLLUP_L2_WITHDRAWAL_URL=https://bridge.trevorgames.xyz/withdraw
```

envs/common-blockscout.env
```bash
CHAIN_TYPE=optimism

INDEXER_OPTIMISM_L1_RPC=http://172.16.203.21:8545

# 可设置为在 L1 上部署 L2 相关合约时的高度
INDEXER_OPTIMISM_L1_BATCH_START_BLOCK=

# deploy-config/trevor-sepolia.json 中的 batchInboxAddress
INDEXER_OPTIMISM_L1_BATCH_INBOX=

# deploy-config/trevor-sepolia.json 中的 batchSenderAddress
INDEXER_OPTIMISM_L1_BATCH_SUBMITTER=
 
INDEXER_OPTIMISM_L1_BATCH_BLOCKS_CHUNK_SIZE=4
INDEXER_OPTIMISM_L1_BATCH_BLOCKSCOUT_BLOBS_API_URL=https://eth-sepolia.blockscout.com/api/v2/blobs
INDEXER_OPTIMISM_L2_BATCH_GENESIS_BLOCK_NUMBER=1

# cat /data/opstack/optimism/packages/contracts-bedrock/deployments/getting-started/OptimismPortalProxy.json | jq .address
INDEXER_OPTIMISM_L1_PORTAL_CONTRACT=

# 可设置为在 L1 上部署 L2 相关合约时的高度
INDEXER_OPTIMISM_L1_OUTPUT_ROOTS_START_BLOCK=

# cat deployments/trevor-sepolia/L2OutputOracleProxy.json | jq .address
INDEXER_OPTIMISM_L1_OUTPUT_ORACLE_CONTRACT=

# 可设置为在 L1 上部署 L2 相关合约时的高度
INDEXER_OPTIMISM_L1_WITHDRAWALS_START_BLOCK=

INDEXER_OPTIMISM_L2_WITHDRAWALS_START_BLOCK=1
INDEXER_OPTIMISM_L2_MESSAGE_PASSER_CONTRACT=0x4200000000000000000000000000000000000016

# 可设置为在 L1 上部署 L2 相关合约时的高度
INDEXER_OPTIMISM_L1_DEPOSITS_START_BLOCK=

INDEXER_OPTIMISM_L1_DEPOSITS_BATCH_SIZE=500
```

### 外观定制

envs/common-frontend.env

```bash
# 去广告
NEXT_PUBLIC_AD_BANNER_PROVIDER=none
NEXT_PUBLIC_AD_TEXT_PROVIDER=none

# homepage
NEXT_PUBLIC_HOMEPAGE_PLATE_TEXT_COLOR=rgb(255,255,255)
NEXT_PUBLIC_HOMEPAGE_PLATE_BACKGROUND=linear-gradient(90deg,rgb(57,120,255)0%,rgb(255,81,249)100%)

# sidebar

# 如果是主网，这里改 false
NEXT_PUBLIC_IS_TESTNET=true 
# 以下链接是本地访问，故可使用 file 协议（如果使用，需要做好容器的目录映射）
NEXT_PUBLIC_NETWORK_ICON=file:///custom-static/trevor-icon.png
NEXT_PUBLIC_NETWORK_ICON_DARK=file:///custom-static/trevor-icon.png
NEXT_PUBLIC_NETWORK_LOGO=file:///custom-static/trevor-logo.jpg
NEXT_PUBLIC_NETWORK_LOGO_DARK=file:///custom-static/trevor-logo.jpg

# footer
NEXT_PUBLIC_FOOTER_LINKS=file:///custom-static/trevor-footer.json

# favicon
# 从 https://realfavicongenerator.net/api/ 获取 api key
FAVICON_GENERATOR_API_KEY=8ff4d23b935297e3bbb046eff1676908f9c29893
# 该 URL 不是本地访问，故不能使用 file 协议
FAVICON_MASTER_URL=https://raw.githubusercontent.com/fantasist-crypto/blockscout-frontend-configs/main/configs/network-icons/trevor.png

```
