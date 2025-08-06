## GMX Synthetics采用**严格的两步执行模式**

### **第一步：createOrder (用户发起)**
```260:271:contracts/router/ExchangeRouter.sol
function createOrder(
    IBaseOrderUtils.CreateOrderParams calldata params
) external override payable nonReentrant returns (bytes32) {
    address account = msg.sender;

    return orderHandler.createOrder(
        account,
        0, // srcChainId is the current block.chainId
        params,
        false
    );
}
```

### **第二步：executeOrder (Keeper执行)**
```240:247:contracts/exchange/OrderHandler.sol
function executeOrder(
    bytes32 key,
    OracleUtils.SetPricesParams calldata oracleParams
) external
    globalNonReentrant
    onlyOrderKeeper
    withOraclePrices(oracleParams)
```

## 关键设计理念

### **反前端运行保护**

代码注释明确说明了这种设计的原因：

```33:39:contracts/router/ExchangeRouter.sol
* To avoid front-running issues, most actions require two steps to execute:
*
* - User sends transaction with request details, e.g. deposit / withdraw liquidity,
* swap, increase / decrease position
* - Keepers listen for the transactions, include the prices for the request then
* send a transaction to execute the request
```

### **价格安全机制**

这种两步设计确保：
1. **用户提交订单时**：不包含价格信息，只有订单参数
2. **Keeper执行时**：提供经过验证的链下签名价格数据

### **执行流程**

**步骤1 - 用户 `createOrder`**：
- 用户发送订单参数（市场、大小、抵押品等）
- 支付执行费用
- 订单存储在链上，等待执行
- 返回订单key

**步骤2 - Keeper `executeOrder`**：
- Keeper监听新订单
- 获取最新价格并签名
- 调用`executeOrder`提供价格数据
- 执行实际的交易逻辑

## 无即时执行选项

**重要发现**：即使是**市场订单**也必须经过两步流程！

从`Order.isMarketOrder(order.orderType())`的检查可以看出，市场订单在取消时有特殊验证，但**创建和执行仍然是分离的**。

## 特殊情况：原子操作

唯一的例外是**LP操作的原子提取**（`executeAtomicWithdrawal`），但这不适用于trader订单。

## 总结

**GMX Synthetics中的trader订单必须分成两步**：
1. **`createOrder`** - 用户提交订单请求
2. **`executeOrder`** - Keeper提供价格并执行

这种设计：
- ✅ 防止前端运行攻击
- ✅ 确保价格数据的安全性和实时性
- ✅ 分离用户意图和价格确认
- ❌ 增加了执行延迟（通常几秒到几分钟）
- ❌ 依赖Keeper网络的可用性

这是一个安全性优先的设计，牺牲了一些用户体验来确保交易的安全性和公平性。
