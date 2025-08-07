## 1. 子账号系统 (Subaccount System)
```mermaid
flowchart LR
    subgraph "子账号系统 (Subaccount System)"
        A[主账号 Main Account] --> B[添加子账号<br/>addSubaccount]
        B --> C[子账号权限设置]
        C --> D[设置过期时间<br/>expiresAt]
        C --> E[设置最大操作次数<br/>maxAllowedCount] 
        C --> F[设置操作类型<br/>actionType]
        
        D --> G[子账号操作验证]
        E --> G
        F --> G
        
        G --> H{验证通过?}
        H -->|是| I[执行子账号操作]
        H -->|否| J[拒绝操作]
        
        I --> K[自动充值Gas费<br/>autoTopUp]
        
        subgraph "子账号限制"
            L[只能为主账号操作]
            M[不能执行外部调用]
            N[有操作次数限制]
            O[有时间限制]
        end
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

### 设计目的
子账号系统允许第三方应用或服务代表用户执行操作，同时保持安全控制。

### 核心特性

**主要组件**：
- **主账号 (Main Account)**: 资金和权限的所有者
- **子账号 (Subaccount)**: 被授权代表主账号操作的地址
- **SubaccountApproval**: 包含授权信息的结构体

**权限控制**：
```solidity
struct SubaccountApproval {
    address subaccount;        // 子账号地址
    bool shouldAdd;           // 是否添加子账号
    uint256 expiresAt;        // 权限过期时间
    uint256 maxAllowedCount;  // 最大操作次数
    bytes32 actionType;       // 操作类型 (如: ORDER_ACTION)
    uint256 nonce;           // 防重放攻击
    uint256 desChainId;      // 目标链ID
    uint256 deadline;        // 签名截止时间
    bytes32 integrationId;   // 集成标识符
    bytes signature;         // 主账号签名
}
```

**安全限制**：
- 子账号只能为主账号执行操作
- 不能执行外部调用 (防止恶意行为)
- 有操作次数和时间限制
- 需要主账号预先授权

**自动充值**：
- 子账号可以自动从主账号余额中充值Gas费
- 通过 `autoTopUpAmount` 设置
