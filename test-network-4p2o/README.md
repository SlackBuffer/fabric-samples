## Prerequisites

```bash
# pwd: fabric
GO_TAGS=pkcs11 make docker
GO_TAGS=pkcs11 make native
```

## sw

```bash
## fabric/fabric-samples/test-network
./network.sh createChannel -c mychannel
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go


# Environment variables for Org1
export FABRIC_CFG_PATH=$PWD/configtx
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'


# Environment variables for Org2
export FABRIC_CFG_PATH=$PWD/configtx
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051

peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

## gmsm

```bash
## fabric/fabric-samples/test-network
./network.sh createChannel -c mychannel -b GM -x gmsm -g gmsm
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go -b GM -x gmsm -g gmsm


# Environment variables for Org1
export FABRIC_CFG_PATH=$PWD/configtx
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
export CORE_PEER_BCCSP_DEFAULT=GM
export CORE_PEER_X509PLUGINTYPE=gmsm
export CORE_PEER_BCCSP_GM_IMPLTYPE=gmsm

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'


# Environment variables for Org2
export FABRIC_CFG_PATH=$PWD/configtx
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
export CORE_PEER_BCCSP_DEFAULT=GM
export CORE_PEER_X509PLUGINTYPE=gmsm
export CORE_PEER_BCCSP_GM_IMPLTYPE=gmsm

peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```
## mix

```bash
# orderer: gmsm
# org1 peers: 信安 gm
# peer0.org2: 天安 pkcs11 gm
# peer1.org2: 天安 sdf

## fabric/fabric-samples/test-network
./network.sh createChannel -c mychannel -b MIX -x gmsm -g gmsm
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go -b MIX -x gmsm -g gmsm


# Environment variables for Org1
export FABRIC_CFG_PATH=$PWD/configtx
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/crypto-config-gm/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/crypto-config-gm/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_BCCSP_DEFAULT=GM
export CORE_PEER_X509PLUGINTYPE=gmsm
export CORE_PEER_BCCSP_GM_IMPLTYPE=gmsm

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/crypto-config-gm/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles ${PWD}/crypto-config-gm/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles ${PWD}/crypto-config-gm/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'


# Environment variables for Org2
export FABRIC_CFG_PATH=$PWD/configtx
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/crypto-config-gm/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/crypto-config-gm/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:9051
export CORE_PEER_BCCSP_DEFAULT=GM
export CORE_PEER_X509PLUGINTYPE=gmsm
export CORE_PEER_BCCSP_GM_IMPLTYPE=gmsm

peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```
## Running the test network

You can use the `./network.sh` script to stand up a simple Fabric test network. The test network has two peer organizations with one peer each and a single node raft ordering service. You can also use the `./network.sh` script to create channels and deploy chaincode. For more information, see [Using the Fabric test network](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html). The test network is being introduced in Fabric v2.0 as the long term replacement for the `first-network` sample.

Before you can deploy the test network, you need to follow the instructions to [Install the Samples, Binaries and Docker Images](https://hyperledger-fabric.readthedocs.io/en/latest/install.html) in the Hyperledger Fabric documentation.
