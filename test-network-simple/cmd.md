
## env

```bash
# Admin
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export FABRIC_CFG_PATH=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/configtx


# User1
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export FABRIC_CFG_PATH=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/configtx


export ORDERER_TLS_ROOT_CERT=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

export ORG1_TLS_ROOT_CERT=/home/ubuntu/go/src/github.com/hyperledger/fabric-samples/test-network-simple/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

## 调用合约

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_TLS_ROOT_CERT -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles $ORG1_TLS_ROOT_CERT -c '{"function":"InitLedger","Args":[]}'

peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'

peer channel fetch config config_block_sys.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile $ORDERER_TLS_ROOT_CERT

peer channel fetch config config_block_sys.pb -c mychannel
```

## native

```bash
docker network create docker_test
```

### 清理残留数据

```bash
# test-network-simple
./network.sh down
sudo rm -rf debug/fabric-data/*
```

## tmp


orderer deliver:

- 从 envelope 构造 SignedData，返回的切片长度是 1，里面有 Identity，差 deserializer

    ```go
    signedData, err := protoutil.EnvelopeAsSignedData(env)

    &SignedData{
        Data:      bytes.Join([][]byte{configSig.SignatureHeader, ce.ConfigUpdate}, nil),
        Identity:  sigHeader.Creator,
        Signature: configSig.Signature,
    }

    func (ds deliverSupport) GetChain(chainID string) deliver.Chain {
        chain := ds.Registrar.GetChain(chainID)
        if chain == nil {
            return nil
        }
        return chain
    }
    ```

- policy 里有 deserializer，负责解析身份的 []byte，它的 deserializer 从 policyProvider 中获得，policyProvider 在 new 初始化 deserializer。

    ```go
    channelConfig, err := NewChannelConfig(config.ChannelGroup, bccsp)
    // channelConfig.MSPManager() 实现了 msp.IdentityDeserializer
    policyProviderMap[pType] = cauthdsl.NewPolicyProvider(channelConfig.MSPManager())
        type MSPManager interface {
            // IdentityDeserializer interface needs to be implemented by MSPManager
            IdentityDeserializer
            // Setup the MSP manager instance according to configuration information
            Setup(msps []MSP) error
            // GetMSPs Provides a list of Membership Service providers
            GetMSPs() (map[string]MSP, error)
        }

    // [ ] !!! chain.(channelconfig.Resources) 为什么可以，找源头
    ```

peer deliver:

```go
aclProvider := aclmgmt.NewACLProvider(
    aclmgmt.ResourceGetter(peerInstance.GetStableChannelConfig),
    policyChecker,
)

    func NewACLProvider(rg ResourceGetter, policyChecker policy.PolicyChecker) ACLProvider {
        return &aclMgmtImpl{
            rescfgProvider: newResourceProvider(rg, newDefaultACLProvider(policyChecker)),
        }
    }

policyCheckerProvider := func(resourceName string) deliver.PolicyCheckerFunc {
    return func(env *cb.Envelope, channelID string) error {
        return aclProvider.CheckACL(resourceName, channelID, env)
    }
}

func (s *DeliverServer) Deliver(srv peer.Deliver_DeliverServer) (err error) {
	logger.Debugf("Starting new Deliver handler")
	defer dumpStacktraceOnPanic()
	// getting policy checker based on resources.Event_Block resource name
	deliverServer := &deliver.Server{
		PolicyChecker: s.PolicyCheckerProvider(resources.Event_Block),
		Receiver:      srv,
		ResponseSender: &blockResponseSender{
			Deliver_DeliverServer: srv,
		},
	}
	return s.DeliverHandler.Handle(srv.Context(), deliverServer)
}
```


chain := ds.Registrar.GetChain(chainID)


func (d DeliverChainManager) GetChain(chainID string) deliver.Chain {
	if channel := d.Peer.Channel(chainID); channel != nil {
		return channel
	}
	return nil
}

func (ds deliverSupport) GetChain(chainID string) deliver.Chain {
	chain := ds.Registrar.GetChain(chainID)
	if chain == nil {
		return nil
	}
	return chain
}

找 defaultACLProviderImpl 的 policyChecker 的值
    d.policyChecker.CheckPolicyBySignedData(channelID, policy, typedData)

policyManager 可以拿到
    policyManager := p.channelPolicyManagerGetter.Manager(channelID)
    // policyName:  "/Channel/Application/Readers"
    policy, _ := policyManager.GetPolicy(policyName)

func (p *policy) EvaluateSignedData(signatureSet []*protoutil.SignedData) error {
	if p == nil {
		return errors.New("no such policy")
	}

	ids := policies.SignatureSetToValidIdentities(signatureSet, p.deserializer)

	return p.EvaluateIdentities(ids)
}

localMSP := mgmt.GetLocalMSP(factory.GetDefault())
pp := cauthdsl.NewPolicyProvider(localMSP)

cauthdsl.NewPolicyProvider(channelConfig.MSPManager())


policyManager, err := policies.NewManagerImpl(RootGroupKey, policyProviderMap, config.ChannelGroup)