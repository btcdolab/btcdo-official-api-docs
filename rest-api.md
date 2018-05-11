

# Public Rest API for Btcdo

## **域名**

**https://api.btcdo.com**

## **接口列表**

| **接口分类** | **请求类型** | **请求地址**                       | **接口描述**           |
| :----------- | ------------ | ---------------------------------- | ---------------------- |
| 公共类       | GET          | /v1/common/currencies              | 获取所有币种信息       |
| 公共类       | GET          | /v1/common/currencies/{currency}   | 获取指定币种信息       |
| 公共类       | GET          | /v1/common/errorCodes              | 获取错误信息           |
| 公共类       | GET          | /v1/common/symbols                 | 获取所有交易对         |
| 公共类       | GET          | /v1/common/symbols/{symbol}        | 获取指定交易对         |
| 公共类       | GET          | /v1/common/timestamp               | 获取系统时间           |
| 市场类       | GET          | /v1/market/bars/{symbol}/{type}    | 获取指定交易对行情数据 |
| 市场类       | GET          | /v1/market/depth/{symbol}          | 获取指定交易对深度     |
| 市场类       | GET          | /v1/market/prices                  | 获取所有交易对成交信息 |
| 交易类       | GET          | /v1/trade/orders                   | 获取最近交易记录       |
| 交易类       | POST         | /v1/trade/orders                   | 创建订单               |
| 交易类       | GET          | /v1/trade/orders/{orderId}         | 获取订单详情信息       |
| 交易类       | GET          | /v1/trade/orders/{orderId}/matches | 获取订单撮合信息       |
| 用户类       | GET          | /v1/user/accounts                  | 获取用户对应账户信息   |
| 用户类       | GET          | /v1/user/deposits/{currency}       | 获取充值记录           |

## **API签名**

btcdo的交易类、用户类API请求需要签名，以确认用户身份，并防止重放攻击。

### 签名算法：

针对一个HTTP的API请求，有如下信息： 

###### 方法名称：`GET`或`POST`

###### 请求域名：例如`www.btcdo.com`

###### 请求路径：以/开头的URI，例如：`/v1/trade/orders`

###### 请求参数：以key1=value2&key2=value2形式的参数，例如：`id=123456&sort=DESC&from=2017-09-10`

###### 请求Header：例如，`Accept: */*`

###### 请求Body：二进制表示的JSON字符串，仅针对POST有效

然后，我们需要构造一个字符串，按如下格式填入： 

```
GET\n

www.btcdo.com\n

/v1/trade/orders\n

from=2017-09-10&id=123456&sort=DESC\n

API-KEY: xyz123456\n

API-SIGNATURE-METHOD: HmacSHA256\n

API-SIGNATURE-VERSION: 1\n

API-TIMESTAMP: 12300000000\n

API-UNIQUE-ID: uni-123-abc-xyz\n

<json body data>
```

**注意：每行以换行符`\n`结束。最后添加的Body结尾不要添加`\n`。** 

第一行是方法名称，全大写的`GET`或`POST`；

第二行是请求域名，全小写，例如：`www.btcdo.com`；

第三行是请求路径，必须以`/`开头，严格区分大小写，最后不要写`?`；

第四行是请求参数，严格区分大小写，并按照字母顺序排序，即排序后的字符串：`from=2017-09-10`，`id=123456`，`sort=DESC`，再用`&`连接起来。如果没有请求参数，第四行是空行，注意空行也需要以`\n`结束；

紧接着把以`API-`开头的Header排序后以`HEADER: Value`的格式每行一个，Header名称全大写，冒号后面有一个空格，Value严格区分大小写，每个Header一行，以`\n`结束；

最后，如果是`POST`请求，且包含Body，把Body以JSON字符串形式添加到末尾，注意没有`\n`。如果是GET请求，或者POST请求没有Body，就跳过这一步。

以上字符串按UTF-8编码，得到一个二进制byte数组，然后使用API Secret计算Signature：

```
signature = HmacSHA256(payload.encode("UTF-8"), "my-api-secret") 
```

将所得签名以十六进制小写字符串形式添加到Header： 

```
API-Signature: a1b2c3ff001234500900dd01ff 
```

### **说明** ：

参数使用原始字符串计算签名，例如`a=1/5`，不要使用`a=1%2F5`；

Header在计算签名时使用全大写，发送时大小写均可；

只有以API-开头的Header才被列入并计算签名（API-Signature除外，因为最后才能计算出API-Signature并附加到请求）；

API-Signature-Method必须为`HmacSHA256`；

API-Signature-Version必须为`1`；

API-Timestamp为当前时间戳，单位为毫秒整数，不支持小数，误差不得超过1分钟；

API-Unique-ID为可选，如果提供，则客户端需要提供一个唯一字符串标识，参考[API Unique Id](unique-id)；

带Body的请求，需要先序列化为JSON字符串，然后把Body列入计算签名。不要对一个对象使用两次序列化，因为某些语言的实现可能导致两次序列化的JSON不一样，例如，`{"a":true, "b":1}`和`{"b":1, "a":true}`，两者JSON内容一致但字符串不同，将导致验证签名失败。

## 接口说明

### **获取所有币种信息**

```
GET 	/v1/common/currencies 
```

#### 应答参数：

| **名称**       | **类型**    | **必返** | **描述**                                                     |
| -------------- | ----------- | -------- | ------------------------------------------------------------ |
| currencies     | Currency[ ] | YES      |                                                              |
| — Currency     |             | YES      |                                                              |
| ——name         | String      | YES      | 币种                                                         |
| ——description  | String      | YES      | 描述                                                         |
| ——auditDeposit | boolean     | YES      | 充值审核 <br> true-充值交易所上账需要人工审核<br> false-充值自动上账 |
| — — virtual    | boolean     | YES      | 是否为虚拟货币                                               |

```
{

  "currencies": [

    {

      "name": "BDB",

      "description": "Btcdo",

      "auditDeposit": "false",

      "virtual": true

    },

    {

      "name": "BTC",

      "description": "Bitcoin",

      "auditDeposit": false,

      "virtual": true

    }  ]

}
```

### **获取所有币种信息**

```
GET 	/v1/common/currencies/{currency} 
```

#### **请求参数：**

| **名称** | **类型** | **必输** | **描述** |
| -------- | -------- | -------- | -------- |
| currency | String   | YES      | 币种     |

#### **应答参数：**

| **名称**     | **类型** | **必返** | **描述**                                                     |
| ------------ | -------- | -------- | ------------------------------------------------------------ |
| name         | String   | YES      | 币种                                                         |
| description  | String   | YES      | 描述                                                         |
| auditDeposit | boolean  | YES      | 充值审核 <br> true-充值交易所上账需要人工审核<br> false-充值自动上账 |
| virtual      | boolean  | YES      | 是否为虚拟货币                                               |

```
{

  "name": "BDB",

  "description": "Btcdo",

  "auditDeposit": false,

  "virtual": true

}
```

### **获取错误信息**

```
GET 	/v1/common/errorCodes 
```

**应答参数：**

```
Response Body
{
  "AUTH_USER_NOT_ACTIVE": "Authenticate error: user not active.",
  "RETRY_LATER": "This operation cannot be done but can retry later.",
  "OPERATION_FAILED": "The requested operation cannot be done.",
  "USER_EMAIL_EXIST": "Registration error: user email already exist.",
  "ORDER_NOT_FOUND": "Order error: the specific order not found.",
  "ADDRESS_MAXIMUM": "Cannot add more address.",
  "AUTH_BAD_OLD_PWD": "",
  "AUTH_SIGNIN_REQUIRED": "",
  "USER_CANNOT_WITHDRAW": "Permission error: user cannot withdraw.",
  "USER_NOT_FOUND": "",
  "AUTH_CANNOT_CHANGE_PWD": "",
  "ADDRESS_INVALID": "Invalid address.",
  "AUTH_IP_FORBIDDEN": "Authenticate error: IP forbidden.",
  "ADDRESS_CHECK_FAILED": "Address failed to check.",
  "USER_CANNOT_SIGNIN": "Permission error: user cannot signin.",
  "ADDRESS_NOT_ALLOWED": "Address is not allowed.",
  "AUTH_CANNOT_SIGNIN": "",
  "AUTH_USER_FORBIDDEN": "Authenticate error: user forbidden to access the resource.",
  "USER_CANNOT_TRADE": "Permission error: user cannot trade.",
  "AUTH_AUTHORIZATION_INVALID": "Authenticate error: Authorization header is invalid.",
  "ACCOUNT_ADD_BALANCE_FAILED": "",
  "AUTH_AUTHORIZATION_EXPIRED": "Authenticate error: Authorization header is expired.",
  "ORDER_CANNOT_CANCEL": "",
  "AUTH_SIGNIN_FAILED": "",
  "HEADER_INVALID": "Header error: the request header is invalid.",
  "ACCOUNT_FREEZE_FAILED": "Account error: cannot freeze asset.",
  "AUTH_SIGNATURE_INVALID": "Authenticate error: API signature is invalid.",
  "PARAMETER_INVALID": "Parameter error: the request parameter is invalid.",
  "INTERNAL_SERVER_ERROR": "Internal error: internal server error.",
  "AUTH_APIKEY_DISABLED": "Authenticate error: API key is disabled.",
  "AUTH_APIKEY_INVALID": "Authenticate error: API key is invalid.",
  "ACCOUNT_UNFREEZE_FAILED": "Account error: unfreeze failed.",
  "DEPOSIT_FAILED": "",
  "WITHDRAW_DISABLED": "Withdraw is disabled.",
  "WITHDRAW_INVALID_STATUS": "Invalid withdraw status.",
  "REQUEST_BODY_TOO_LARGE": "The request body is too large."
}
```

### **获取所有交易对**

```
GET 	/v1/common/symbols 
```

#### 应答参数：

| **名称**           | **类型** | **必返** | **描述**                                                     |
| ------------------ | -------- | -------- | ------------------------------------------------------------ |
| symbols            | Symbol[] | YES      |                                                              |
| — Symbol           |          |          |                                                              |
| — — base           | Map      |          |                                                              |
| — — — name         | String   | YES      | 币种                                                         |
| — — — description  | String   | YES      | 描述                                                         |
| — — — auditDeposit | boolean  | YES      | 充值审核 <br> true-充值交易所上账需要人工审核<br> false-充值自动上账 |
| — — — virtual      | boolean  | YES      | 是否为虚拟货币                                               |
| — —  baseScale     | float    | YES      | 交易币精度（小数点后n位）                                    |
| — —  baseMinimum   | float    | YES      | 交易币最小挂单数量                                           |
| — —  quote         | Map      | YES      | 计价币                                                       |
| — — — name         | String   | YES      | 币种                                                         |
| — — — description  | String   | YES      | 描述                                                         |
| — — — auditDeposit | boolean  | YES      | 充值审核 <br> true-充值交易所上账需要人工审核<br> false-充值自动上账 |
| — — — virtual      | boolean  | YES      | 是否为虚拟货币                                               |
| — —  quoteScale    | float    | YES      | 计价币精度（小数点后n位）                                    |
| — —  quoteMinimum  | float    | YES      | 计价币最小挂单数量                                           |
| — —  startTime     | Long     | YES      | 交易对开启时间（UTC）                                        |
| — —  endTime       | Long     | YES      | 交易对关闭时间（UTC）                                        |
| — —  name          | String   | YES      | 交易对                                                       |
| — —  quoteName     | String   | YES      | 计价币种                                                     |
| — —  baseName      | String   | YES      | 交易币种                                                     |

```
{
  "symbols": [
    {
      "base": {
        "name": "BDB",
        "description": "Btcdo",
        "auditDeposit": true,
        "virtual": true
      },
      "baseScale": 0,
      "baseMinimum": 1,
      "quote": {
        "name": "BTC",
        "description": "Bitcoin",
        "auditDeposit": false,
        "virtual": true
      },
      "quoteScale": 8,
      "quoteMinimum": 1e-8,
      "startTime": 1514779200000,
      "endTime": 1609430400000,
      "name": "BDB_BTC",
      "quoteName": "BTC",
      "baseName": "BDB"
    },
    {
      "base": {
        "name": "BDB",
        "description": "Btcdo",
        "auditDeposit": true,
        "virtual": true
      },
      "baseScale": 0,
      "baseMinimum": 1,
      "quote": {
        "name": "ETH",
        "description": "Ethereum",
        "auditDeposit": false,
        "virtual": true
      },
      "quoteScale": 8,
      "quoteMinimum": 1e-8,
      "startTime": 1514779200000,
      "endTime": 1609430400000,
      "name": "BDB_ETH",
      "quoteName": "ETH",
      "baseName": "BDB"
    }
  ]
}
```

### **获取指定交易对**

```
GET 	/v1/common/symbols/{symbol} 
```

#### 请求参数:

| **名称** | **类型** | **必输** | **描述**                             |
| -------- | -------- | -------- | ------------------------------------ |
| symbol   | String   | YES      | 交易对 交易币种_计价币种 例：BDB_ETH |

#### 应答参数:

| **名称**       | **类型** | **必返** | **描述**                                                     |
| -------------- | -------- | -------- | ------------------------------------------------------------ |
| base           | Map      |          |                                                              |
| — name         | String   | YES      | 币种                                                         |
| — description  | String   | YES      | 描述                                                         |
| — auditDeposit | boolean  | YES      | 充值审核 <br> true-充值交易所上账需要人工审核<br> false-充值自动上账 |
| — virtual      | boolean  | YES      | 是否为虚拟货币                                               |
| baseScale      | float    | YES      | 交易币精度（小数点后n位）                                    |
| baseMinimum    | float    | YES      | 交易币最小挂单数量                                           |
| quote          | Map      | YES      | 计价币                                                       |
| — name         | String   | YES      | 币种                                                         |
| — description  | String   | YES      | 描述                                                         |
| — auditDeposit | boolean  | YES      | 充值审核 <br> true-充值交易所上账需要人工审核<br> false-充值自动上账 |
| — virtual      | boolean  | YES      | 是否为虚拟货币                                               |
| quoteScale     | float    | YES      | 计价币精度（小数点后n位）                                    |
| quoteMinimum   | float    | YES      | 计价币最小挂单数量                                           |
| startTime      | Long     | YES      | 交易对开启时间（UTC）                                        |
| endTime        | Long     | YES      | 交易对关闭时间（UTC）                                        |
| name           | String   | YES      | 交易对                                                       |
| quoteName      | String   | YES      | 计价币种                                                     |
| baseName       | String   | YES      | 交易币种                                                     |

```
{
  "base": {
    "name": "BDB",
    "description": "Btcdo",
    "auditDeposit": true,
    "virtual": true
  },
  "baseScale": 0,
  "baseMinimum": 1,
  "quote": {
    "name": "BTC",
    "description": "Bitcoin",
    "auditDeposit": false,
    "virtual": true
  },
  "quoteScale": 8,
  "quoteMinimum": 1e-8,
  "startTime": 1514779200000,
  "endTime": 1609430400000,
  "name": "BDB_BTC",
  "quoteName": "BTC",
  "baseName": "BDB"
}
```

### **获取系统时间**

```
GET 	/v1/common/timestamp 
```

#### 应答参数：

| **名称**  | **类型** | **必返** | **描述**        |
| --------- | -------- | -------- | --------------- |
| timestamp | Long     | YES      | 系统时间（UTC） |

{

  "timestamp": 1523261306835

}

### **获取指定交易对行情数据**

```
GET 	/v1/market/bars/{symbol}/{type} 
```

#### 请求参数：

| **名称** | **类型** | **必输** | **描述**                                                     |
| :------- | -------- | -------- | ------------------------------------------------------------ |
| symbol   | String   | YES      | 交易对 交易币种_计价币种 例：BDB_ETH                         |
| type     | String   | YES      | K_1_SEC：秒K（最近1小时）<br> K_1_MIN：分钟K(最近24小时）<br> K_1_HOUR：小时K（最近30天）<br> K_1_DAY：日K |

#### 应答参数：

| **名称**    | **类型**    | **必返** | **描述**       |
| ----------- | ----------- | -------- | -------------- |
| bars        | Float[ ][ ] | YES      |                |
| —bars[0][0] | Long        | NO       | K线时间（UTC） |
| —bars[0][1] | Float       | NO       | 开盘价         |
| —bars[0][2] | Float       | NO       | 最高价         |
| —bars[0][3] | Float       | NO       | 最低价         |
| —bars[0][4] | Float       | NO       | 收盘价         |
| —bars[0][5] | Float       | NO       | 成交量         |

```
{
  "bars": [
    [
      1517788800000,
      0.0001,
      0.00016,
      0.0001,
      0.000145,
      332196
    ],
    [
      1517875200000,
      0.00015,
      0.00019988,
      0.00015,
      0.0001887,
      358926
    ]
  ]
}
```

### **获取指定交易对深度**

```
GET 	/v1/market/depth/{symbol} 
```

#### **请求参数：**

| **名称** | **类型** | **必输** | **描述**                             |
| -------- | -------- | -------- | ------------------------------------ |
| symbol   | String   | YES      | 交易对 交易币种_计价币种 例：BDB_ETH |

#### **应答参数：**

| **名称**     | **类型**     | **必返** | **描述**                             |
| ------------ | ------------ | -------- | ------------------------------------ |
| symbol       | String       | YES      | 交易对 交易币种_计价币种 例：BDB_ETH |
| timestamp    | Long         | YES      | 系统时间（UTC）                      |
| price        | BigDecimal   | YES      | 价格                                 |
| buyOrders    | DepthOrder[] | NO       | 买单                                 |
| — DepthOrder |              | NO       |                                      |
| — — price    | BigDecimal   | NO       | 买单价格                             |
| — — amount   | BigDecimal   | NO       | 买单数量                             |
| sellOrders   | DepthOrder[] | NO       | 卖单                                 |
| — DepthOrder |              | NO       |                                      |
| — — price    | BigDecimal   | NO       | 卖单价格                             |
| — — amount   | BigDecimal   | NO       | 卖单数量                             |

```
{
  "symbol": "BDB_ETH",
  "timestamp": 1523263377379,
  "price": 0.00010103,
  "buyOrders": [
    {
      "price": 0.00010292,
      "amount": 5800
    },
    {
      "price": 0.00010291,
      "amount": 27444
    }
  ],
  "sellOrders": [
    {
      "price": 0.00011792,
      "amount": 2543
    },
    {
      "price": 0.00011793,
      "amount": 9745
    }
  ]
}
```

### **获取所有交易对成交信息**

```
GET 	/v1/market/prices 
```

#### 应答参数：

| **名称**     | **类型** | **必返** | **描述**       |
| ------------ | -------- | -------- | -------------- |
| 交易对       | String   | NO       | KEY            |
| 交易信息     | Float[ ] | NO       | VALUE          |
| —交易信息[0] | Float    | NO       | K线时间（UTC） |
| —交易信息[1] | Float    | NO       | 开盘价         |
| —交易信息[2] | Float    | NO       | 最高价         |
| —交易信息[3] | Float    | NO       | 最低价         |
| —交易信息[4] | Float    | NO       | 收盘价         |
| —交易信息[5] | Float    | NO       | 成交量         |

```
{
  "BDB_BTC": [
    1523154838957,
    0.00000677,
    0.00000677,
    0.00000677,
    0.00000677,
    285992
  ],
  "IOST_ETH": [
    1522979758598,
    0.00008213,
    0.00008213,
    0.00008213,
    0.00008213,
    71396
  ]
}
```

### **获取最近交易记录（默认****100****条记录）**

```
GET 	/v1/trade/orders 
```

#### **请求参数：**

| **名称** | **类型** | **必输** | **描述**                                                     |
| -------- | -------- | -------- | ------------------------------------------------------------ |
| offsetId | long     | NO       | 默认为0<br>订单号，从当前offsetId（订单号）开始倒序向前查找limit条记录<br>当不传或传入offsetId=0，则从该用户最后一个订单开始倒叙向前查找limit条记录 |
| limit    | int      | NO       | 默认为100<br>从指定订单号（offsetId）倒叙向前查找记录数      |
| symbol   | String   | NO       | 默认返回所有交易对记录                                       |

#### **应答参数：**

| **名称**        | **类型**   | **必返** | **描述**                                                     |
| --------------- | ---------- | -------- | ------------------------------------------------------------ |
| orders          | Order[]    | NO       |                                                              |
| — Order         |            | NO       |                                                              |
| — — id          | String     | NO       | 订单号                                                       |
| — — createdAt   | Long       | NO       | 订单创建时间（UTC）                                          |
| — — updatedAt   | Long       | NO       | 订单更新时间（UTC）                                          |
| — — seqId       | Long       | NO       | 订单序列号                                                   |
| — — refOrderId  | Long       | NO       | 关联订单号                                                   |
| — — refSeqId    | Long       | NO       | 关联序列号                                                   |
| — —userId       | Long       | NO       | 用户编号                                                     |
| — —symbol       | String     | NO       | 交易对                                                       |
| — —type         | String     | NO       | 订单类型  <br>BUY_LIMIT ：限价买单；<br>SELL_LIMIT：限价卖单；<br>CANCEL_BUY_LIMIT：限价买单撤单；<br>CANCEL_SELL_LIMIT：限价卖单撤单 |
| — —price        | BigDecimal | NO       | 价格                                                         |
| — —amount       | BigDecimal | NO       | 数量                                                         |
| — —filledAmount | BigDecimal | NO       | 成交额                                                       |
| — —fee          | BigDecimal | NO       | 手续费                                                       |
| — —features     | int        | NO       | 订单特性                                                     |
| — —status       | String     | NO       | 订单状态<br> 限价买单&限价卖单<br> SEQUENCED：已定序；<br> FULLY_FILLED：全部成交；<br> PARTIAL_FILLED：部分成交；<br> PARTIAL_CANCELLED：部分取消；<br> FULLY_CANCELLED：全部取消；<br> 限价买单撤单&限价卖单撤单 <br> CANCELLED_OK：撤单成功；<br> CANCELLED_FAILED：撤单失败 |

```
{
	"orders": [{
		"id": 104098,
		"createdAt": 1523182488682,
		"updatedAt": 1523182488739,
		"seqId": 7947,
		"refOrderId": 0,
		"refSeqId": 0,
		"userId": 100017,
		"symbol": "IOST_BDB",
		"type": "SELL_LIMIT",
		"price": 6.0,
		"amount": 17.0,
		"filledAmount": 17.0,
		"fee": 0.204,
		"features": 65536,
		"status": "FULLY_FILLED"
	}, {
		"id": 104097,
		"createdAt": 1523182483611,
		"updatedAt": 1523182488739,
		"seqId": 7946,
		"refOrderId": 0,
		"refSeqId": 0,
		"userId": 100017,
		"symbol": "IOST_BDB",
		"type": "BUY_LIMIT",
		"price": 6.0,
		"amount": 17.0,
		"filledAmount": 17.0,
		"fee": 0.034,
		"features": 65536,
		"status": "FULLY_FILLED"
	}]
}
```

### 创建订单

```
POST 	/v1/trade/orders 
```

#### 请求参数：

| **名称**       | **类型** | **必输** | **描述**                                                     |
| -------------- | -------- | -------- | ------------------------------------------------------------ |
| amount         | float    | NO       | 金额（限价买/卖单必输）                                      |
| customFeatures | int      | NO       | 订单特性<br>65536：燃烧DBD(使用BDB抵扣手续费)<br>2：做市商-Maker订单 |
| orderType      | String   | YES      | 订单类型<br> BUY_LIMIT：限价买单；<br> SELL_LIMIT：限价卖单；<br> CANCEL_BUY_LIMIT：限价买单撤单；<br> CANCEL_SELL_LIMIT：限价卖单撤单 |
| price          | float    | NO       | 价格（限价买/卖单必输）                                      |
| symbol         | String   | YES      | 交易对儿                                                     |
| targetOrderId  | Long     | NO       | 原订单ID（撤销订单必输）                                     |

#### 应答参数：

| **名称**     | **类型** | **必返** | **描述**                                                     |
| ------------ | -------- | -------- | ------------------------------------------------------------ |
| id           | String   | NO       | 订单号                                                       |
| createdAt    | Long     | NO       | 订单创建时间（UTC）                                          |
| updatedAt    | Long     | NO       | 订单更新时间（UTC）                                          |
| seqId        | Long     | NO       | 订单序列号                                                   |
| refOrderId   | Long     | NO       | 关联订单号                                                   |
| refSeqId     | Long     | NO       | 关联序列号                                                   |
| userId       | Long     | NO       | 用户编号                                                     |
| symbol       | String   | NO       | 交易对                                                       |
| type         | String   | NO       | 订单类型<br>  BUY_LIMIT：限价买单；<br> SELL_LIMIT：限价卖单；<br>CANCEL_BUY_LIMIT：限价买单撤单；<br>CANCEL_SELL_LIMIT：限价卖单撤单 |
| price        | float    | NO       | 价格                                                         |
| amount       | float    | NO       | 数量                                                         |
| filledAmount | float    | NO       | 成交额                                                       |
| fee          | float    | NO       | 手续费                                                       |
| features     | int      | NO       | 订单特性                                                     |
| status       | String   | NO       | 订单状态<br> 限价买单&限价卖单<br> SEQUENCED：已定序；<br> FULLY_FILLED：全部成交；<br> PARTIAL_FILLED：部分成交；<br> PARTIAL_CANCELLED：部分取消；<br> FULLY_CANCELLED：全部取消；<br> 限价买单撤单&限价卖单撤单 <br>CANCELLED_OK：撤单成功；<br>CANCELLED_FAILED：撤单失败 |

```
{
	"id": 104151,
	"createdAt": 1523268534688,
	"updatedAt": 1523268534790,
	"seqId": 8000,
	"refOrderId": 0,
	"refSeqId": 0,
	"userId": 100017,
	"symbol": "BDB_BTC",
	"type": "SELL_LIMIT",
	"price": 0.03500003,
	"amount": 1.0,
	"filledAmount": 1.0,
	"fee": 7.000006e-05,
	"features": 65536,
	"status": "FULLY_FILLED"
}
```

### **获取订单详情信息**

```
GET		/v1/trade/orders/{orderId} 
```

#### 请求参数：

| **名称** | **类型** | **必输** | **描述** |
| -------- | -------- | -------- | -------- |
| orderId  | Long     | YES      | 订单编号 |

#### 应答参数：

| **名称**     | **类型** | **必返** | **描述**                                                     |
| ------------ | -------- | -------- | ------------------------------------------------------------ |
| id           | String   | NO       | 订单号                                                       |
| createdAt    | Long     | NO       | 订单创建时间（UTC）                                          |
| updatedAt    | Long     | NO       | 订单更新时间（UTC）                                          |
| seqId        | Long     | NO       | 订单序列号                                                   |
| refOrderId   | Long     | NO       | 关联订单号                                                   |
| refSeqId     | Long     | NO       | 关联序列号                                                   |
| userId       | Long     | NO       | 用户编号                                                     |
| symbol       | String   | NO       | 交易对                                                       |
| type         | String   | NO       | 订单类型 <br> BUY_LIMIT：限价买单；<br> SELL_LIMIT：限价卖单；<br>CANCEL_BUY_LIMIT：限价买单撤单；<br>CANCEL_SELL_LIMIT：限价卖单撤单 |
| price        | float    | NO       | 价格                                                         |
| amount       | float    | NO       | 数量                                                         |
| filledAmount | float    | NO       | 成交额                                                       |
| fee          | float    | NO       | 手续费                                                       |
| features     | int      | NO       | 订单特性                                                     |
| status       | String   | NO       | 订单状态<br> 限价买单&限价卖单<br> SEQUENCED：已定序；<br> FULLY_FILLED：全部成交；<br> PARTIAL_FILLED：部分成交；<br> PARTIAL_CANCELLED：部分取消；<br> FULLY_CANCELLED：全部取消；<br> 限价买单撤单&限价卖单撤单 <br>CANCELLED_OK：撤单成功；<br>CANCELLED_FAILED：撤单失败 |

```
{
	"id": 104151,
	"createdAt": 1523268534688,
	"updatedAt": 1523268534790,
	"seqId": 8000,
	"refOrderId": 0,
	"refSeqId": 0,
	"userId": 100017,
	"symbol": "BDB_BTC",
	"type": "SELL_LIMIT",
	"price": 0.03500003,
	"amount": 1.0,
	"filledAmount": 1.0,
	"fee": 7.000006e-05,
	"features": 65536,
	"status": "FULLY_FILLED"
}
```

### **获取订单撮合信息**

```
GET 	/v1/trade/orders/{orderId}/matches 
```

#### 请求参数：

| **名称** | **类型** | **必输** | **描述** |
| -------- | -------- | -------- | -------- |
| orderId  | Long     | YES      | 订单编号 |

#### 应答参数：

| **名称**       | **类型** | **必返** | **描述**        |
| -------------- | -------- | -------- | --------------- |
| matches        | Match[ ] | YES      |                 |
| — Match        |          | YES      |                 |
| — — id         | Long     | YES      | 对手订单编号    |
| — — createdAt  | Long     | YES      | 撮合时间        |
| — — type       | String   | YES      | TAKER<br> MAKER |
| — — price      | float    | YES      | 价格            |
| —— amount      | float    | YES      | 数量            |
| — — BigDecimal | float    | YES      | 手续费          |

```
{
	"matches": [{
		"id": 103638,
		"createdAt": 1523269159004,
		"type": "TAKER",
		"price": 0.035,
		"amount": 1.0,
		"fee": 7e-05
	}, {
		"id": 103636,
		"createdAt": 1523269159004,
		"type": "TAKER",
		"price": 0.03500001,
		"amount": 1.0,
		"fee": 7.000002e-05
	}, {
		"id": 103634,
		"createdAt": 1523269159003,
		"type": "TAKER",
		"price": 0.03500002,
		"amount": 1.0,
		"fee": 7.000004e-05
	}]
}
```

### **获取用户对应账户信息**

```
GET 	/v1/user/accounts 
```

#### 应答参数：

| **名称**      | **类型**   | **必返** | **描述**                                                     |
| ------------- | ---------- | -------- | ------------------------------------------------------------ |
| accounts      | Account[ ] | YES      |                                                              |
| —Account      |            | YES      |                                                              |
| — — id        | Long       | YES      | 账户编号                                                     |
| — — createdAt | Long       | YES      | 创建时间                                                     |
| — — updatedAt | Long       | YES      | 更新时间                                                     |
| — — userId    | Long       | YES      | 用户编号                                                     |
| — — currency  | String     | YES      | 币种                                                         |
| — — type      | Long       | YES      | 账户类型 <br>SPOT_AVAILABLE:可用余额 <br>SPOT_FROZEN：冻结余额 |
| — — balance   | float      | YES      | 余额                                                         |

```
{
	"accounts": [{
		"id": 100690,
		"createdAt": 1521430554062,
		"updatedAt": 1523269164075,
		"userId": 100017,
		"currency": "BDB",
		"type": "SPOT_AVAILABLE",
		"balance": 92905.39508336772
	}, {
		"id": 100696,
		"createdAt": 1521431390541,
		"updatedAt": 1523269159050,
		"userId": 100017,
		"currency": "BDB",
		"type": "SPOT_FROZEN",
		"balance": 0.0
	}, {
		"id": 100691,
		"createdAt": 1521430631145,
		"updatedAt": 1523269164076,
		"userId": 100017,
		"currency": "BTC",
		"type": "SPOT_AVAILABLE",
		"balance": 9.24000006
	}, {
		"id": 101052,
		"createdAt": 1523180345697,
		"updatedAt": 1523180345801,
		"userId": 100017,
		"currency": "BTC",
		"type": "SPOT_FROZEN",
		"balance": 0.0
	}]
}
```

### **获取充值记录**

```
GET 	/v1/user/deposits/{currency} 
```

#### 请求参数：

| **名称** | **类型** | **必输** | **描述**                                                  |
| -------- | -------- | -------- | --------------------------------------------------------- |
| currency | String   | YES      | 币种                                                      |
| pending  | String   | NO       | “true”：只查询充值中状态记录<br>“false”：查询所有状态记录 |

#### 应答参数：

| **名称**               | **类型**   | **必返** | **描述**                                                     |
| ---------------------- | ---------- | -------- | ------------------------------------------------------------ |
| deposits               | Deposit[ ] | NO       |                                                              |
| — Deposit              |            |          |                                                              |
| — — DepositLog: status | String     | NO       | 充值状态<br>PENDING：充值中；<br>DEPOSITED ：已成功；<br>CANCELLED ：已取消；<br>WAITING_FOR_APPROVAL ：待审核； <br>DENIED ：已拒绝 |
| — —currency            | String     | NO       | 币种                                                         |
| — —toAddress           | String     | NO       | 充值地址                                                     |
| — —uniqueId            | String     | NO       | 唯一编号                                                     |
| — —amount              | float      | NO       | 数量                                                         |
| — —confirms            | int        | NO       | 上账确认次数                                                 |
| — — deposited          | int        | NO       | 充值次数                                                     |
| — —  cancelled         | int        | NO       | 取消次数                                                     |

```
{
	deposits = [{
		DepositLog: status = DEPOSITED,
		currency = BDB,
		toAddress = 0xd1946970e672d88ef1dcb709495f0ba3767101b1,
		uniqueId = 0x87afd731818cb95ca9fdc2a77dcbd1f6481ebd1e865c2a63e983ecf0b64ad345 #0x46caaee51b35b4b8ca4def0d45455dde0bf1cc8875cb0de4e57fbd82d416bd96, 
	    amount= 20.0000000000000000,
		confirms = 12 / 12,
		deposited = 0,
		cancelled = 0
	}, {
		DepositLog: status = DEPOSITED,
		currency = BDB,
		toAddress = 0xc465c1004b1efa8d86af422e6803aad398d1aab3,
		uniqueId = 0xb86c024cce3dd3df4bd36161456d0eea5aa31cbcef7fb3a5ab68b4e2429738c3 #0xc911cadc5ffe00c8b7c1e23787eb2b6cc737ab5f741ffe596869df24ec064149, 
	    amount= 10.0000000000000000,
		confirms = 13 / 12,
		deposited = 0,
		cancelled = 0
	}]
}
```

