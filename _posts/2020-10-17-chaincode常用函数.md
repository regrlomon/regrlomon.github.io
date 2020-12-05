---
title: go编写chaincode用到的函数
author: S1rumAA
tag: ["go", "hyperledger fabric"]
---
### 写在前面
fabric的最新版本是2.2，现在从github上git下来的fabric里面找不到shim和peer这两个包了  
解决办法就是从`https://github.com/hyperledger/fabric/releases`上面下载1.4及以前版本就能找到shim和peer包
### 链码结构
```
package main

import (
	"github.com/hyperledger/fabric/core/chaincode/shim"
	pb "github.com/hyperledger/fabric/protos/peer"
	"fmt"
)

type SimpleChaincode struct {
}

func (t* SimpleChaincode) Init(stubshim.ChaincodeStubInterface) pb.Response  {
}

func (t* SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {

}

func main(){
	err:=shim.Start(new(SimpleChaincode))
	if err!=nil{
		fmt.Printf("error starting SimpleChaincode:%s",err)
	}
}
```
## 链码相关的api
### 参数解析api
```
//返回调用链码时提供的参数列表（以字符串数组形式返回）
GetStringArgs() []string
//返回调用链码时在交易提案中指定的被调用的函数名称以及函数的参数列表
GetFunctionAndParameters() (function string, params []string)
//返回调用链码时在交易提案中提供的被调用的函数名称及函数的参数列表
GetArgs() [][]byte
```
### 账本数据操作api
```
//查询账本，返回指定键对应的值
GetState(key string) ([]byte, error)
//尝试添加进账本一对键值
PutState(key string, value []byte) error
//尝试删除账本中一对键值
DelState(key string) error
//查询指定范围的键值，
GetStateByRange(startKey, endKey string) (StateQueryIteratorInterface, error)
//返回指定键的所有历史值。该方法的使用需要在节点配置中打开历史数据库特性（ledger.history.enableHistoryDatabase=true）
GetHistoryForKey(key string) (HistoryQueryIteratorInterface, error)
//给定一组属性，将这些属性组合起来构造返回一个复合键
CreateCompositeKey(objectType string, attributes []string) (string, error)
//将指定的复合键进行分割，划分成构造复合键时所用的属性
SplitCompositeKey(compositeKey string) (string, []string, error)
```
### 交易相关的api
```

```
存储管理相关的方法
PutState
func PutState(key string,value []byte)error;
Info Text.

peer chaincode instantiate -o orderer.example.com:7050 --tls -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "or ('Org1MSP.peer','Org2MSP.peer')" \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp \
CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/fabcar/go
```

peer chaincode instantiate -o orderer.example.com:7050 --tls -C $CHANNEL_NAME -n mygo -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer')"  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

export CHANNEL_NAME=mychannel

peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

peer channel join -b mychannel.block

peer chaincode install -n mygo -v 1.0 -p github.com/chaincode/fabcar/go

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp \
CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
peer channel join -b mychannel.block



peer chaincode instantiate -o orderer.example.com:7050 -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer')"

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp \ CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" \ CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \ peer chaincode install -n mygo -v 1.0 -p github.com/chaincode/fabcar/go




初始化账本数据
peer chaincode invoke -n mygo  -c  '{"Args":["initLedger"]}' -C $CHANNEL_NAME --tls  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
查询账本全部汽车的信息：
peer chaincode query  -n mygo  -c  '{"Args":["queryAllCars"]}' -C $CHANNEL_NAME --tls  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem