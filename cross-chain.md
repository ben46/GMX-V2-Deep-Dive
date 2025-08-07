

## 2. 跨链支持 (Cross-chain Support)


```mermaid
flowchart LR
   
    
    subgraph "跨链支持 (Cross-chain Support)"
        P[用户在源链<br/>Source Chain] --> Q[签名跨链请求]
        Q --> R[MultichainRouter接收]
        R --> S[验证srcChainId]
        S --> T[处理跨链逻辑]
        
        T --> U{操作类型}
        U -->|存款| V[跨链存款到MultichainVault]
        U -->|订单| W[跨链创建订单]
        U -->|提取| X[跨链提取资金]
        
        V --> Y[LayerZero/Bridge协议]
        W --> Z[目标链执行]
        X --> Y
        
        Y --> AA[资金到达目标链]
        Z --> BB[订单在目标链创建]
        AA --> CC[更新用户多链余额]
        BB --> CC
    end
    
 
    
    subgraph "多链架构层次"
        QQ[BaseGelatoRelayRouter<br/>基础Gelato中继]
        QQ --> RR[MultichainRouter<br/>多链路由器]
        RR --> SS[MultichainOrderRouter<br/>多链订单路由]
        RR --> TT[MultichainSubaccountRouter<br/>多链子账号路由]
        RR --> UU[MultichainGmRouter<br/>多链GM路由]
        RR --> VV[MultichainGlvRouter<br/>多链GLV路由]
    end
    
    %% 样式设置
    classDef subaccount fill:#e3f2fd
    classDef crosschain fill:#f3e5f5
    classDef workflow fill:#e8f5e8
    classDef architecture fill:#fff3e0
    
    class A,B,C,D,E,F,G,H,I,J,K,L,M,N,O subaccount
    class P,Q,R,S,T,U,V,W,X,Y,Z,AA,BB,CC crosschain
    class DD,EE,FF,GG,HH,II,JJ,KK,LL,MM,NN,OO,PP workflow
    class QQ,RR,SS,TT,UU,VV architecture
``` 
### 设计架构

**withRelay修饰器的跨链逻辑**：
```solidity
modifier withRelay(
    IRelayUtils.RelayParams calldata relayParams,
    address account,
    uint256 srcChainId,  // 关键：源链ID
    bool isSubaccount
) {
    // 1. 验证跨链参数
    if (_isMultichain()) {
        if (!dataStore.getBool(Keys.isSrcChainIdEnabledKey(srcChainId))) {
            revert Errors.InvalidSrcChainId(srcChainId);
        }
    }
    // 2. 执行跨链逻辑
    _;
}
```

### 跨链流程

**1. 同链操作 (srcChainId == block.chainid)**:
- 直接在当前链执行
- 使用标准的Gelato Relay

**2. 跨链操作 (srcChainId != block.chainid)**:
- 用户在源链签名请求
- 通过LayerZero等跨链协议传输
- 在目标链执行操作
- 资金在MultichainVault中管理

### 多链架构层次

**继承关系**：
```
BaseGelatoRelayRouter
    ↓
MultichainRouter (添加跨链支持)
    ↓
具体多链路由器:
- MultichainOrderRouter (订单)
- MultichainSubaccountRouter (子账号)
- MultichainGmRouter (GM代币)
- MultichainGlvRouter (GLV代币)
```

### 资金管理

**MultichainVault**：
- 管理用户在不同链上的余额
- 支持跨链资金转移
- 记录跨链交易历史

**LayerZero集成**：
- 使用LayerZero协议进行跨链通信
- 支持Stargate进行资金桥接
- 处理跨链费用和滑点
 
