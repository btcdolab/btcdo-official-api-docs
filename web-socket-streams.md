# Web Socket Streams for Btcdo

## 域名

###### **wss://onli-quotation.btcdo.com**

## 路径

###### **/v1/market**

## 请求类型

###### **websocket**

### **开启订阅** **socket** **推送**

#### 订阅Key：

```
subscribe 
```

#### 订阅参数：

| **名称** | **类型** | **必输** | **描述**            |
| -------- | -------- | -------- | ------------------- |
| symbol   | String   | YES      | 交易对，例：BDB_BTC |

#### 例：

```
socket = new WebSocket(“wss://onli-quotation.btcdo.com");
socket.io.on(‘open', function () {
    socket.emit('subscribe', { symbol: ‘BDB_BTC’ });
});
```

### 价格推送

#### 订阅Key：

```
 topic_prices 
```

#### 订阅返回：

| **名称**     | **类型** | **必输** | **描述**       |
| ------------ | -------- | -------- | -------------- |
| 交易对       | String   | NO       | KEY            |
| 价格信息     | Price[]  | NO       | VALUE          |
| — Price      |          | NO       |                |
| — — Price[0] | Long     | NO       | K线时间（UTC） |
| — — Price[1] | Float    | NO       | 开盘价         |
| — — Price[2] | Float    | NO       | 最高价         |
| — — Price[3] | Float    | NO       | 最低价         |
| — — Price[4] | Float    | NO       | 收盘价         |
| — — Price[5] | Float    | NO       | 成交量         |

```
{
	BDB_BTC: [1523330996424, 0.00000677, 0.00000677, 0.00000675, 0.00000676, 679418],
	BDB_ETH: [1523330996424, 0.00011645, 0.00011645, 0.00010294, 0.00011597, 2028162],
	DVT_ETH: [1523330996424, 0.00007658, 0.000077, 0.00006501, 0.00004515, 436062]
}
```

### 当前交易对Tick

#### 订阅Key：

```
topic_tick 
```

#### 订阅参数：

| **名称** | **类型** | **必输** | **描述**            |
| -------- | -------- | -------- | ------------------- |
| symbol   | String   | YES      | 交易对，例：BDB_BTC |

#### 订阅返回：

| **名称**       | **类型** | **必输** | **描述**                  |
| -------------- | -------- | -------- | ------------------------- |
| 当前交易对Tick | Ticker[] | YES      |                           |
| — Ticker       |          | YES      |                           |
| — —  id        | Long     | YES      |                           |
| — —  amount    | Float    | YES      | 数量                      |
| — —  createdAt | Long     | YES      | 创建时间（UTC）           |
| — —  direction | boolean  | YES      | 涨跌描述，true涨，false跌 |
| — —  price     | Float    | YES      | 当前价格                  |
| — —  symbol    | String   | YES      | 当前币对                  |

```
[{
		amount: 2403,
		 createdAt: 1523405631866,
		 direction: true,
		 id: 114524,
		 price: 0.00000676,
		 symbol: "BDB_BTC" 
	}
]
```

### **当前交易对深度图**

#### 订阅Key：

```
topic_snapshot 
```

#### 订阅参数：

| **名称** | **类型** | **必输** | **描述**            |
| -------- | -------- | -------- | ------------------- |
| symbol   | String   | YES      | 交易对，例：BDB_BTC |

#### 订阅返回：

| **名称**   | **类型** | **必返** | **描述**      |
| ---------- | -------- | -------- | ------------- |
| price      | Float    | YES      | 当前实价      |
| symbol     | String   | YES      | 当前币对      |
| timestamp  | Long     | YES      | 时间戳（UTC） |
| buyOrders  | Depth[]  | YES      | 买入列表      |
| —  price   | Float    | YES      | 价格          |
| — amount   | Float    | YES      | 数量          |
| sellOrders | Depth[]  | YES      | 卖出列表      |
| —  price   | Float    | YES      | 价格          |
| — amount   | Float    | YES      | 数量          |

```
{
	price: 0.076851，
	symbol: "ETH_BTC",
	timestamp: 1523405387779,
	buyOrders: [{
		price: 0.05727,
		amount: 0.312
	}],
	sellOrders: [{
		price: 0.082514,
		amount: 0.021
	}]
}
```

