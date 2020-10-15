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
输入export -p可查看上面是否export成功
first-network目录中有一个配置文件crypto-config.yaml，这个配置文件中主要定义了：
1个orderer节点：orderer.example.com
4个perr节点：peer0.org1.example.com， peer1.org1.example.com， peer0.org2.example.com， peer1.org2.example.com
这里可以使用cryptogen命令，生成这个区块链网络所需要的一系列证书及相关文件。证书是基于PKI体系的x509格式证书。
执行前请先确认当前目录下没有 crypto-config 目录，如果已经存在，请先删除
生成证书

```
cryptogen generate --config=./crypto-config.yaml
```
> 执行成功后，会在当前目录下增加一个新目录 crypto-config，包含有这个网络节点的证书、私钥等加密、签名相关元素，使用tree命令即可查看该文件目录.在这个网络中，有一个CA，即证书颁发机构，名称为example.com，它给自己颁发了一个自签名根证书ca.example.com-cert.pem，在生成的多个目录中都包含了这个 CA 根证书，而其他节点证书（除了 TLS 证书）都由此 CA 颁发。进入其中一个目录`crypto-config/ordererOrganizations/example.com/ca`发现有两个文
件,其中ca.example.com-cert.pem 是 CA 自签名根证书
7357b1a60b074fedafe077b5881d5340f887bde90c4aacc1a42dccd5002a14f9_sk是根证书对应
的私钥文件
显示CA根证书内容

```
openssl x509 -in ca.example.com-cert.pem -text -noout
```
显示证书内容如下：
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            e5:e2:b6:09:22:02:df:fc:89:ef:db:e3:69:57:98:93
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: C = US, ST = California, L = San Francisco, O = example.com, CN = ca.example.com
        Validity
            Not Before: Oct 13 15:45:16 2020 GMT
            Not After : Oct 11 15:45:16 2030 GMT
        Subject: C = US, ST = California, L = San Francisco, O = example.com, CN = ca.example.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:be:36:35:1b:27:32:36:60:ac:17:0a:8e:34:7d:
                    8c:04:36:da:d3:52:1c:83:fc:a8:9d:04:92:e5:db:
                    04:7c:4c:d8:96:84:cc:a0:f1:7b:da:37:ea:f8:e1:
                    b4:d8:fc:ec:09:01:33:fb:bc:43:43:75:f3:00:ea:
                    36:31:7f:eb:e3
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign, CRL Sign
            X509v3 Extended Key Usage: 
                Any Extended Key Usage
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                73:57:B1:A6:0B:07:4F:ED:AF:E0:77:B5:88:1D:53:40:F8:87:BD:E9:0C:4A:AC:C1:A4:2D:CC:D5:00:2A:14:F9
    Signature Algorithm: ecdsa-with-SHA256
         30:45:02:21:00:a2:bf:79:57:7c:2c:37:90:4b:f6:15:be:a6:
         45:be:8f:24:17:6d:a0:d2:af:75:04:33:fd:15:89:92:89:d9:
         08:02:20:6b:25:a1:19:a0:52:d6:b7:65:13:cc:f9:39:c0:1b:
         be:1f:ca:5e:1d:43:e3:6d:8f:60:01:db:8c:4d:18:cc:f7
```
其中第六行显示使用的是sha256的加密算法
orderer节点管理员在`crypto-config/ordererOrganizations/example.com/msp/admincerts`目录下，进入这个目录输入`openssl x509 -in Admin@example.com-cert.pem -text -noout`显示管理员证书
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            cf:b1:f7:5b:63:b4:e1:c5:ae:d1:d4:30:4d:8b:75:e3
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: C = US, ST = California, L = San Francisco, O = example.com, CN = ca.example.com
        Validity
            Not Before: Oct 13 15:45:16 2020 GMT
            Not After : Oct 11 15:45:16 2030 GMT
        Subject: C = US, ST = California, L = San Francisco, CN = Admin@example.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:ff:28:e4:ec:39:6c:2f:37:7e:8d:13:8c:f3:9b:
                    f6:13:7f:f4:70:e1:88:92:50:0a:b9:c0:aa:6d:d6:
                    60:1a:b4:16:90:3e:fd:10:4e:0a:d0:4b:71:f4:d3:
                    6b:df:7b:79:03:73:98:e6:58:01:48:a0:27:c9:44:
                    40:b1:e2:3c:e2
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier: 
                keyid:73:57:B1:A6:0B:07:4F:ED:AF:E0:77:B5:88:1D:53:40:F8:87:BD:E9:0C:4A:AC:C1:A4:2D:CC:D5:00:2A:14:F9

    Signature Algorithm: ecdsa-with-SHA256
         30:45:02:21:00:90:fc:2e:06:e9:33:02:d0:8e:6f:0c:a2:29:
         df:d3:2b:b6:dc:79:2f:8c:89:6c:c9:00:04:cc:e3:e6:7b:9c:
         f6:02:20:23:ed:19:9e:7b:2e:ce:8b:b5:97:a9:e4:81:55:9b:
         03:da:b6:19:33:0e:90:24:7d:43:da:c3:39:85:82:f5:04
```
用根证书验证 A dmin@example.com 证书：
```
openssl verify -CAfile ../../ca/ca.example.com-cert.pem Admin@example.com-cert.pem
```
##  别的
configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
docker-compose -f docker-compose-cli.yaml up -d
docker exec -it cli bash
export CHANNEL_NAME=mychannel
peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
peer channel join -b mychannel.block

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp \
CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
peer channel join -b mychannel.block

peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp \
CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/

peer chaincode instantiate -o orderer.example.com:7050 --tls -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')" \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer')"
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
peer chaincode invoke -o orderer.example.com:7050  --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'

