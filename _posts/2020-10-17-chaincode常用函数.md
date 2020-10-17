---
title: go编写chaincode用到的函数
author: S1rumAA
tag: ["go", "hyperledger fabric"]
---

## 链码结构
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
//
```

存储管理相关的方法
PutState
func PutState(key string,value []byte)error;
Info Text.



```

```