# 为 Trevor Rollup 搭建 Blockscout 浏览器

## 参考文档

blockscout 官方教程：https://docs.blockscout.com/for-developers/deployment/docker-compose-deployment

optimism 教程：https://docs.optimism.io/builders/chain-operators/tools/explorer

## 依赖
- Docker
- docker-compose

## 安装与启动

```bash
git clone https://github.com/blockscout/blockscout.git
cd blockscout/docker-compose

# 按 L2 教程，services/user-ops-indexer.yml 中 8545 应改为 8546

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

### 外部 ip 访问

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

# 顺便把网络名称改好
NEXT_PUBLIC_NETWORK_NAME=Trevor chain
NEXT_PUBLIC_NETWORK_SHORT_NAME=Trevor chain
NEXT_PUBLIC_NETWORK_ID=689388

NEXT_PUBLIC_APP_HOST=1.2.3.4:8000
NEXT_PUBLIC_VISUALIZE_API_HOST=http://1.2.3.4:8081

# 如果是主网，这里改 false
NEXT_PUBLIC_IS_TESTNET=true 
```

改动 `proxy/default.conf.template`。该文件改动之后 `docker-compose -f geth.yml up -d` 命令并不会重启 proxy 容器，可以在此之前先运行 `docker kill proxy` 停掉容器。

```bash
server {
    listen       8080;
    add_header 'Access-Control-Allow-Origin' 'http://1.2.3.4:8000' always;
...
```

### 外部域名 + https 访问

