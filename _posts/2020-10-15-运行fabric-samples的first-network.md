---
title: 运行fabric-samples的first-network
tag: hyperledger fabric
---
首先说一下我的配置环境：Ubuntu20.04、go、docker、docker-compose、fabric1.1、fabric-ca1.1。
<!--more-->
## 使用自动化工具运行first-network
>cd fabric/fabric-samples/first-network

进入到`first-network`文件夹中，有一个byfn.sh的文件，有下面几种用法：
```
./byfn.sh generate          
生成first-network需要的所有证书、创世区块和三个配置交易。会在当前目录创建一个叫crypto-config的文件夹，生成的证书都会放在这个文件夹内。一般地，先执行byfn.sh generate，再执行byfn.sh up
./byfn.sh up                
启动first-network网络，该命令会检查当前目录是否有crypto-config这个文件夹，如果该文件夹不存在，会先执行与byfn.sh generate等效的所有操作
./byfn.sh down           
停止first-network，删除所有docker容器、证书、创世区块和三个配置交易，删除由byfn.sh up生成的链码镜像
```
执行./byfn.sh up  
![start](https://i.loli.net/2020/10/15/Ajho7St1UniaXBl.png){:.rounded}  
>启动了6个docker容器，其中orderer是fabric的排序节点，cli是客户端节点，剩下四个是peer节点然后程序执行，直到出现

![end](https://i.loli.net/2020/10/15/eLqXzRn9GvYF2Ef.png){:.rounded}  
>这样整个first-network就结束了，通过输入`docker ps`可以查看正在运行的docker容器

![docker ps](https://i.loli.net/2020/10/15/5uLlpxKFDkZ369b.png){:.rounded}  
>最下面的6个是上面说的，最上面的3个是在三个peer节点上实例的链码  
使用`docker exec -it cli bash`进入cli客户端，在客户端输入`peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'`即可查询a账户的余额

![peer query](https://i.loli.net/2020/10/15/eCYtXFSm3lxhyPc.png){:.rounded}  
以上就是使用自动化工具执行first-network，但是执行完了啥也没学到，因为在执行过程终端输出了一大堆都不知道啥意思，能看懂的就是最开始的start跟最后的end以及a账户的余额为90，所以有必要学习一下怎么手动执行这个first-network
## 手动运行first-network
和上面一样，cd到first-network这个文件夹中，首先需要设置一下环境变量
```
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}
```
这里说一下第一条，${PWD}表示的是当前目录，..表示的是上一级的目录，也就是说这个`${PWD}/../bin`指的是当前目录的上级目录下的bin目录，有点绕。export这个命令是作用于当前shell的，如果退出当前shell，export设置的环境变量就都会没有了，**如果想设置永久的环境变量，可在根目录下的.bashrc导入上面的内容即可。**
### 生成证书及相关文件
第一个需要使用的是cryptogen工具，其可执行程序位于`fabric-samples/bin`下，可以用来生成证书及相关文件
```
cryptogen generate --config=./crypto-config.yaml
```
注意：执行命令前要检查当前目录是否存在 crypto-config 目录，如果存在，请先删除。  
这个`crypto-config.yaml`以后会经常使用，它定义了这个网络中有一个orderer组织，该组织的域名是example.com，还定义了两个peer组织，域名分别是org1.example.com，org2.example.com，其中每个peer组织都有两个peer节点和一个用户。  
执行成功后，会在当前目录下增加一个新目录 crypto-config，包含有这个网络节点的证书、私钥等加密、签名相关元素，使用tree命令即可查看该文件目录
<img src="https://i.loli.net/2020/10/16/FsgdjzWthqiJaQu.png" width="90%" height="90%" />  
这个图太长了。在这个网络中，有一个ca，在它的目录下有一个证书文件`ca.example.com-cert.pem`，输入
```
openssl x509 -in ca.example.com-cert.pem -text
```
即可查看到该证书的内容
![ca.example.com-cert.pem](https://i.loli.net/2020/10/15/HEi9OwSrDfQWcj1.png){:.rounded}  
其中第六行显示使用的是sha256的加密算法
### 生成创世区块Genesis Block
```
configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```
生成的文件位于目录 channel-artifacts 下，可以通过以下命令将 Block 详细内容导入到 json 文件方便查看：
```
configtxgen -inspectBlock channel-artifacts/genesis.block > genesis.block.json
```
### 生成其他文件
创建一个channel名为mychannel（上面环境变量已配置）
```
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```
为org1创建锚节点
```
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
```
为org2创建锚节点
```
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
```
### 使用docker-compose启动network并打开cli
```
docker-compose -f docker-compose-cli.yaml up -d
docker exec -it cli bash
```
### 创建并加入channel
这个和上面那个创建channel不一样，上面那个是生成了一个channel的文件，这个是引用这个文件创建真正的channel
```
//插入环境变量
export CHANNEL_NAME=mychannel

//创建channel
peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

//加入channel
peer channel join -b $CHANNEL_NAME.block
```
由于创建channel会在当前路径生成一个mychannel.block的区块，所以可以讲当前cli指向	`peer0.org2.example.com`，让其加入该channel
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel join -b mychannel.block
```
之所以修改环境变量的原因是cli自身的环境变量是指向的`peer0.org1.example.com`的，然后我们现在需要让其指向`peer0.org2.example.com`，所以需要修改环境变量，而且修改的环境变量仅在当前命令有效
```
//查看当前节点加入的channel
peer channel list
```
### 更新锚节点
```
//更新org1的锚节点
peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

//更新org2的锚节点
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```
### 安装链码
```
//安装链码，其中-n后面是表示链码的名字，这里是mycc，-v表示版本号，-p表示链码的路径
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/

//安装完链码后需要对链码进行初始化，也就是指定背书策略，其中"AND ('Org1MSP.peer','Org2MSP.peer')"表示需要org1和org2的peer节点背书才能使交易通过
peer chaincode instantiate -o orderer.example.com:7050 --tls -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')" --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```
一个链码只需要初始化一次即可。节点`peer0.org2.example.com`加入该链码
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```
```
//查看安装的链码
peer chaincode list --installed
```
