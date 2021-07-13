
## env

```bash
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export FABRIC_CFG_PATH=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/configtx


# Admin
export CORE_PEER_MSPCONFIGPATH=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

# User1
export CORE_PEER_MSPCONFIGPATH=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp



export ORDERER_TLS_ROOT_CERT=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

export ORG1_TLS_ROOT_CERT=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

## 调用合约

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_TLS_ROOT_CERT -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles $ORG1_TLS_ROOT_CERT -c '{"function":"InitLedger","Args":[]}'

peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

## native

```bash
docker network create docker_test

peer chaincode install -n basic -v 1.0 -p github.com/hyperledger/fabric-samples/asset-transfer-basic/chaincode-go

peer chaincode instantiate -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls true --cafile $ORDERER_TLS_ROOT_CERT -C mychannel -n basic -l golang -v 1.0 -c '{"Args":[]}' -P 'OR ('\''Org1MSP.peer'\'')'
```

### 清理残留数据

```bash
# test-network-simple
./network.sh down
sudo rm -rf debug/fabric-data/*
```
