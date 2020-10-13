---
title: 运行fabric-samples中的first-network
tag: hyperledger fabric
---

进入到fabric/fabric-samples/first-network目录下，首先需要配置环境变量
```
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}
export CLI_TIMEOUT=10
export CLI_DELAY=3
export CHANNEL_NAME="mychannel"
export COMPOSE_FILE=docker-compose-cli.yaml
export COMPOSE_FILE_COUCH=docker-compose-couch.yaml
export COMPOSE_FILE_ORG3=docker-compose-org3.yaml
export LANGUAGE=golang
export IMAGETAG="latest"
export VERBOSE=true
```
first-network目录中有一个配置文件crypto-config.yaml，这个配置文件中主要定义了：
1个orderer节点：orderer.example.com
4个perr节点：peer0.org1.example.com， peer1.org1.example.com， peer0.org2.example.com， peer1.org2.example.com
这里可以使用cryptogen命令，生成这个区块链网络所需要的一系列证书及相关文件。证书是基于PKI体系的x509格式证书。
在执行前请确认当前hgsfjhhjk
gedrggdr
