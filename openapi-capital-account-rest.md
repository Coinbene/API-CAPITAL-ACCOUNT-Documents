
# 资金账户相关OpenAPI REST接口
# [English](https://github.com/Coinbene/API-CAPITAL-ACCOUNT-Documents/blob/master/openapi-capital-account-rest-en.md)
   * [资金账户相关OpenAPI REST接口](#资金账户相关openapi-rest接口)
      * [基本信息](#基本信息)
      * [访问限制](#访问限制)
      * [接口类型](#接口类型)
      * [签名方式](#签名方式)
         * [私有接口-申请提币接口](#私有接口-申请提币接口)
         * [私有接口-查询充币地址列表](#私有接口-查询充币地址列表)
	 * [私有接口-资产划转接口](#私有接口-资产划转接口)
      * [错误代码汇总](#错误代码汇总)

## 基本信息
- 本篇列出REST接口的baseurl http://openapi-exchange.coinbene.com 或 https://openapi-exchange.coinbene.com
- 建议创建完API后，修改添加上自己服务器出口IP，进一步增强API安全性校验
- 国内用户建议机器绑定host，104.16.127.19 openapi-exchange.coinbene.com
- 所有接口的响应都是JSON格式
- 所有时间、时间戳均为UNIX时间，单位为毫秒
- HTTP 4XX 错误码用于指示错误的请求内容、行为、格式
- HTTP 429 错误码表示警告访问频次超限
- HTTP 5XX 错误码用于指示Coinbene服务侧的问题
- HTTP 504 表示API服务端已经向业务核心提交了请求但未能获取响应，特别需要注意的是504代码不代表请求失败，而是未知。很可能已经得到了执行，也有可能执行失败，需要做进一步确认。
- 每个接口都有可能抛出异常，异常响应格式如下：

```
{
  "code": 10001,
  "msg": "Invalid Paramater."
}
```
- 具体的错误码及其解释在错误代码汇总。
- GET方法的接口, 参数必须在query string中发送。
- POST方法的接口, 参数在 request body中发送(content type application/json)。
- 对参数的顺序不做要求。
## 访问限制
- 当访问接口超过频率限制时，将返回429状态：请求太频繁。
- 限制规则，如果传入有效的API key则用user id限速；如果没有则拿用户的公网IP限速。各个接口上有单独的说明。
## 接口类型
- 主要为两类接口，公共接口和私有接口。
- 公共接口无需认证即可调用。
- 私有接口用户订单和账户。每个私有请求必须使用规范的验证形式进行签名。私有接口需要使用您的API key进行验证。
## 签名方式
所有接口请求头必须包含以下内容：
- ACCESS-KEY 字符串类型的API key
- ACCESS-SIGN 使用Base64转码签名
- ACCESS-TIMESTAMP 发起请求的时间戳
- 所有请求都应该含有application/json类型内容，并且是有效的JSON。

ACCESS-SIGN的值生成规则：
- 按照timestamp + method + requestPath + body字符串（+表示字符串连接），以及secret，使用HMAC SHA256方法加密，最后把加密串的字节数组转成字符串返回。
- 其中，timestamp的值与ACCESS-TIMESTAMP请求头相同，必须是UTC时区Unix时间戳的十进制秒数或ISO8601标准的时间格式，精确到毫秒。
- Method是请求方法，字母全部大写：GET/POST
- requestPath是请求接口路径，例如：/api/swap/v2/market/orderBook
- body是指请求主体的字符串。GET请求没有body信息可省略；POST请求有body信息JSON串，例如{"symbol":"BTCUSDT","order_id":"7440"}
- secket为用户申请API时所生成的。

接口请求样例：
- GET协议接口两种情况: 
```
preHash String：2019-10-12T09:33:43.126ZGET/api/capital/v1/deposit/address/list?asset=XRP
```


```
Url: http://域名/api/capital/v1/deposit/address/list?asset=XRP
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 03a0a94d6bb16c81f133a4fc3d2c8790
	ACCESS-SIGN: 0a0c59665b296e09b7d5e07b9c12e274e797eb2f78a67fc0cc68577467c3328a
	ACCESS-TIMESTAMP: 2019-10-12T09:33:43.126Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=en_US
Body: 
preHash: 2019-10-12T09:33:43.126ZGET/api/capital/v1/deposit/address/list?asset=XRP

```


- POST协议接口情况：
```
preHash String：2019-10-12T09:40:19.683ZPOST/api/capital/v1/withdraw/apply{"amount":"1","asset":"BTC","address":"rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ","tag":"10000737"}

```


```
Url: http://域名/api/capital/v1/withdraw/apply
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: 03a0a94d6bb16c81f133a4fc3d2c8790
	ACCESS-SIGN: af0270d6a7d80da7dabbebbcc8e7b2d31fc929489b1f6fb4165bdd1e611d5da9
	ACCESS-TIMESTAMP: 2019-10-12T09:40:19.683Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=en_US
Body: {"amount":"1","asset":"BTC","address":"rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ","tag":"10000737"}
preHash: 2019-10-12T09:40:19.683ZPOST/api/capital/v1/withdraw/apply{"amount":"1","asset":"BTC","address":"rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ","tag":"10000737"}
```
- 签名算法验证：


```
源串：2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info
secret：9daf13ebd76c4f358fc885ca6ede5e27
生成sign串：a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2

样例代码（Java版本）：
**
   * 生成签名
   *
   * @param timeStamp   时间戳
   * @param method      请求方法：POST或者GET
   * @param requestUrl  url
   * @param requestBody 请求内容，没有传null
   * @param secret      密钥
   */
  private String signForContractOpenApi(String timeStamp, String method, String requestUrl, String requestBody, String secret) {
    String shaResource = timeStamp + method + requestUrl + (requestBody == null ? "" : requestBody);
    System.out.println(shaResource);
    String signStr = sha256_HMAC(shaResource, secret);
    return signStr;
  }

  /**
   * sha256_HMAC加密
   *
   * @param resource 签名源字符串
   * @param secret   秘钥
   * @return 加密后字符串
   */
  private static String sha256_HMAC(String resource, String secret) {
    String hash = "";
    try {
      Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
      SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256");
      sha256_HMAC.init(secret_key);
      byte[] bytes = sha256_HMAC.doFinal(resource.getBytes("UTF-8"));
      hash = byteArrayToHexString(bytes);
    } catch (Exception e) {
      System.out.println("Error HmacSHA256 ===========" + e.getMessage());
    }
    return hash;
  }

  /**
   * 将加密后的字节数组转换成字符串
   *
   * @param bytes 字节数组
   * @return 字符串
   */
  private static String byteArrayToHexString(byte[] bytes) {
    StringBuffer buffer = new StringBuffer();
    String stmp;
    for (int index = 0; bytes != null && index < bytes.length; index++) {
      stmp = Integer.toHexString(bytes[index] & 0XFF);
      if (stmp.length() == 1) {
        // 一位补零
        buffer.append('0');
      }
      buffer.append(stmp);
    }
    return buffer.toString().toLowerCase();
  }

样例代码（Python版本）：

import hashlib
import hmac
import unittest

def sign(message, secret):
    """
    gen sign
    :param message: message wait sign
    :param secret:  secret key
    :return:
    """
    secret = secret.encode('utf-8')
    message = message.encode('utf-8')
    sign = hmac.new(secret, message, digestmod=hashlib.sha256).hexdigest()
    return sign

class TestUtil(unittest.TestCase):
    def test_sign(self):
        sn = sign("2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info", "9daf13ebd76c4f358fc885ca6ede5e27")
        self.assertEqual(sn, "a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2")


```



### 私有接口-申请提币接口

```
用于API用户申请提币
限速次数：1次/1秒
HTTP POST /api/capital/v1/withdraw/apply
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|--------|---------|--------
asset   | string | 是 | 提币的币种
amount   | string| 是  | 提币数量
address   | string| 是  | 提币地址
addressTag   | string | 否 | 部分币种充值需用到此标签，比如XRP，EOS。否则输入空字符串
chain   | string | 否 |  部分币种会用此字段来标识不同链。如USDT，这里值为“ETH”，“BTC”


返回结果参数

名称   | 类型  | 说明
---------|--------|---------
id   | string | 成功后返回的提币申请id
asset   | string |  提币的币种
amount   | string|  提币数量
address   | string|  提币地址
addressTag   | string |  部分币种充值需用到此标签，比如XRP，EOS。
chain   | string | 部分币种会用此字段来标识不同链。如USDT，这里值为“ETH”，“BTC”


```
Request:
Url: http://域名/api/capital/v1/withdraw/apply
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: 03a0a94d6bb16c81f133a4fc3d2c8790
	ACCESS-SIGN: af0270d6a7d80da7dabbebbcc8e7b2d31fc929489b1f6fb4165bdd1e611d5da9
	ACCESS-TIMESTAMP: 2019-10-12T09:40:19.683Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=en_US
Body: {"amount":"1","asset":"BTC","address":"rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ","tag":"10000737"}
preHash: 2019-10-12T09:40:19.683ZPOST/api/capital/v1/withdraw/apply{"amount":"1","asset":"BTC","address":"rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ","tag":"10000737"}

Response:
{
  "code": 200, 
  "data": {
    "id": "123", 
    "amount": "10", 
    "asset": "BTC", 
    "address": "rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ", 
    "tag": "", 
    "chain": ""
  }
}
```
### 私有接口-查询充币地址列表


```
获取所有合约的持仓信息
限速规则：1次/1秒
HTTP GET /api/capital/v1/deposit/address/list
```
```
注意：需要在网页或者移动端创建充值地址之后，这里才能获取到充值地址
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|--------|---------|--------
asset      | string | 是 | 充值的币种

返回字段说明：

名称   | 类型  | 说明
---------|--------|---------
asset   | string | 充值的币种
chain  | string | 部分币种会用此字段来标识不同链。如USDT，这里值为“ETH”，“BTC”
address   | string | 充币地址
addressTag   | string | 部分币种充值需用到此标签，比如XRP，EOS。
depositLimit   | string | 最小充值量，小于此数目的充值将不会被到账
blockNumber   | string | 充值区块确认数


```
Request:
Url: http://域名/api/capital/v1/deposit/address/list?asset=XRP
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 03a0a94d6bb16c81f133a4fc3d2c8790
	ACCESS-SIGN: 56113c528d4050ad7b45eb6880accdb3803d0d88dbb7cc6b56558439cb047303
	ACCESS-TIMESTAMP: 2019-10-12T09:40:20.546Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=en_US
Body: 
preHash: 2019-10-12T09:40:20.546ZGET/api/capital/v1/deposit/address/list?asset=XRP

Response:
{
    "code":200,
    "data":[
        {
            "asset":"XRP",
            "chain":"XRP",
            "address":"rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ",
            "addressTag":"10000737",
            "depositLimit":"25",
            "blockNumber":"2"
        }
    ]
}
```

### 私有接口-资产划转接口


```
获取所有合约的持仓信息
限速规则：2次/1秒
HTTP GET /api/capital/v1/asset/transfer
```
请求参数：
名称  | 类型  | 是否必填  | 说明
---|---|---|---
asset      | string | 是 | 划转的资产名称
amount      | string | 是 | 划转数量
from      | string | 是 | 转出业务账户，枚举值：币币:spot，btc合约:btc-contract，usdt合约:usdt-contract，杠杆:margin
to      | string | 是 | 转入业务账户，枚举值：币币:spot，btc合约:btc-contract，usdt合约:usdt-contract，杠杆:margin
fromInstrumentId      | string |  | 转出杠杆币对名称，只有从杠杆币对中转出才需要填写
toInstrumentId      | string |  | 转入杠杆币对名称，只有转入到杠杆币对中才需要填写


返回字段说明：

名称   | 类型  | 说明
---|---|---
transferId   | string | 划转ID
asset  | string | 资产名称
amount   | string | 划转数量
from   | string | 转出业务账户，枚举值：币币:spot，btc合约:btc-contract，usdt合约:usdt-contract，杠杆:margin
to   | string | 转入业务账户，枚举值：币币:spot，btc合约:btc-contract，usdt合约:usdt-contract，杠杆:margin
result   | string | SUCCESS 表示划转成功


```
Request:
Url: http://域名/api/capital/v1/asset/transfer
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: 2c8b514c28b6404f0d0333b958379484
	ACCESS-SIGN: 98566ac857ef292bba71a7c5ffcab4647e40864bef2462886f25f72dc1f8f77c
	ACCESS-TIMESTAMP: 2019-11-11T04:02:41.628Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=en_US
Body: {"amount":"1","asset":"BTC","from":"spot","to":"margin","fromInstrumentId":"","toInstrumentId":"BTC/USDT"}
preHash: 2019-11-11T04:02:41.628ZPOST/api/capital/v1/asset/transfer{"amount":"1","asset":"BTC","from":"spot","to":"margin","fromInstrumentId":"","toInstrumentId":"BTC/USDT"}

Response:
{
    "code":200,
    "data":{
        "transferId":"643420339740823552",
        "asset":"BTC",
        "amount":"1.00000000",
        "from":"spot",
        "to":"margin",
        "result":"SUCCESS"
    }
}
```
 

## 错误代码汇总

错误代码     |  message
---|---
12001 | "ACCESS_KEY"不能为空
12002 | "ACCESS_SIGN"不能为空
12003 | "ACCESS_TIMESTAMP"不能为空
12005 | 无效的ACCESS_TIMESTAMP
12006 | 无效的ACCESS_KEY
12007 | 无效的Content_Type，请使用“application/json”格式
12008 | 请求时间戳过期
12009 | 系统错误
120011 |无效的sign
120012 |api 校验失败
429 |请求太频繁
11000|参数值为空
11001|参数值无效
11002|参数值超过最大限度
11003|第三方接口暂时不返回数据
11004|价格精度无效
11005|无效的数量精度
11006|未知异常
11007|币对与资产不匹配
11008|未获取用户ID
11009|未获取用户的Site
11010|未获取用户所属的Bank
11011|资产暂停充值
11012|资产已下架
11013|资产不存在
2000|账户余额不足
2001|账户被冻结
2003|用户无提币权限
2006|申请提币失败
2010|低于最小提币额度
2016|提币精度不合法
2024|资产不存在
2035|已超过24小时内最大提币量
2038|提币数量需大于0
2049|请完成实名认证提高提币额度
2050|请先去提币地址添加此地址
2051|此提币地址创建时间小于24小时
2060|禁止提转
2071|新用户24H内禁止提币
10001|地址无效
10002|地址已被禁用
10003|禁止向合约地址提币
10004|禁止向平台内提币
10005|{0}无效
10006|缺少{0}
10007|此币单次提币量最大不能超过{0}
10008|未找到资产对应配置
10009|暂时无法获取充值地址
10007|此币单次提币量最大不能超过{0}
10008|未找到资产对应配置
13002|未找到资产对应配置
