OpenAPI REST Interface for Funds Account

## Basic information
- This article lists the baseurl http://openapi-capital.coinbene.com of REST interface.
- It is suggested that after the API is created, modify and add your own server exit IP to further enhance API security verification.
- All interfaces respond in JSON format
- All time and timestamps are UNIX time in milliseconds
- HTTP 4XX error codes are used to indicate the content, behavior, and format of requests for errors
- HTTP 429 Error Code Represents Warning Access Frequency Overrun
- HTTP 5XX Error Code Used to Indicate Problems on the Coinbene Service Side
- HTTP 504 indicates that the API server has submitted a request to the business core but failed to obtain a response. In particular, it should be noted that the 504 code does not mean that the request failed, but is unknown. It is likely that it has been implemented, or that it may fail, and further confirmation is needed.
- It is possible for each interface to throw an exception. The exception response format is as follows:

```
{
"Code": 10001,
"Msg": "Invalid Paramater."
}
```
- Specific error codes and their explanations are summarized in error codes.
- GET method interface, parameters must be sent in query string.
- The interface of the POST method and the parameters are sent in the request body (content type application / json).
- The order of parameters is not required.
## Access restrictions
- When the access interface exceeds the frequency limit, 429 status is returned: requests are too frequent.
- The restriction rule is to limit the speed with user ID if a valid API key is passed in; if not, limit the speed with the user's public network IP. Each interface has a separate description.
## Interface type
- There are mainly two kinds of interfaces, public interface and private interface.
- Public interfaces can be invoked without authentication.
- Private interface user orders and accounts. Each private request must be signed using a standard form of verification. Private interfaces need to be validated using your API key.
## Signature method
All interface request headers must contain the following:
- API key of ACCESS-KEY string type
- ACCESS-SIGN uses Base64 transcoding signature
- Time stamp of the request initiated by ACCESS-TIMESTAMP
- All requests should contain application / JSON type content and be valid JSON.

ACCESS-SIGN Value Generation Rules:
- According to the timestamp + method + requestPath + body string (+indicating string connection), and secret, the method of HMAC SHA256 is used to encrypt, and finally the byte array of the encrypted string is converted into a string to return.
- Among them, the value of timestamp is the same as that of the ACCESS-TIMESTAMP request header. It must be the decimal seconds of the Unix timestamp in UTC time zone or the time format of ISO8601 standard, accurate to milliseconds.
- Method is the request method, all capitalized: GET/POST
- Request Path is the request interface path, such as: / API / swap / V2 / market / orderBook
- Body refers to the string of the requesting body. GET requests have no body information to omit; POST requests have body information JSON strings, such as {"symbol": "BTCUSDT", "order_id", "7440"}
- Secket is generated when a user applies for an API.

Sample interface request:
- GET protocol interface has two cases:
```
PreHash String: 2019-10-12T09:33:43.126ZGET/api/capital/v1/deposit/address/list?Asset=XRP
```


```
Url: http://domain name/api/capital/v1/deposit/address/list?Asset=XRP
Method: GET
Headers:
Accept: application/json
ACCESS-KEY: 03a0a94d6bb16c81f133a4fc3d2c8790
ACCESS-SIGN: 0a0c59665b296e09b7d5e07b9c12e274e797eb2f78a67fc0cc68577467c3328a
ACCESS-TIMESTAMP: 2019-10-12T09:33:43.126Z
Content-Type: application/json; charset = UTF-8
Cookie: locale = en_US
Body:
PreHash: 2019-10-12T09:33:43.126ZGET/api/capital/v1/deposit/address/list?Asset=XRP

```


- POST protocol interface:
```
PreHash String: 2019-10-12T09:40:19.683ZPOST/api/capital/v1/withdraw/apply {amount":"1","asset","BTC","address":"rHyS9xSwQUBqm5KwprUXDWxZcwEMZYQMJ","tag":"10000737"}

```


```
Url: http://domain name/api/capital/v1/withdraw/apply
Method: POST
Headers:
Accept: application/json
ACCESS-KEY: 03a0a94d6bb16c81f133a4fc3d2c8790
ACCESS-SIGN: af0270d6a7d80da7dabbebbcc8e7b2d31fc929489b1f6bdd1e611d5da9
ACCESS-TIMESTAMP: 2019-10-12T09:40:19.683Z
Content-Type: application/json; charset = UTF-8
Cookie: locale = en_US
Body: {"amount": "1", "asset", "BTC", "address": "rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ", "tag": "10000737"}
PreHash: 2019-10-12T09:40:19.683ZPOST/api/capital/v1/withdraw/apply {"amount": "1", "asset", "BTC", "address": "rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ", "tag": "10000737"}
```
- Signature algorithm verification:


```
Source string: 2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info
Secret: 9daf13ebd76c4f358fc885ca6ede5e27
Generate sign string: a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2

Sample code (Java version):
* *
* Generate signature
*
*@ Param timeStamp timestamp
*@ Param method request method: POST or GET
*@ Param request Url URL
*@ Param requestBody request content, no null passed
*@ param secret key
* /
Private String sign ForContractOpenApi (String time Stamp, String method, String request Url, String request Body, String secret){
String shaResource = timeStamp + method + requestUrl + (requestBody = null?": requestBody);
System. out. println (shaResource);
String signStr = sha256_HMAC (shaResource, secret);
Return signStr;
}


* *
* Sha256_HMAC Encryption
*
*@ Param resource signature source string
*@ param secret key
*@ return encrypted string
* /
Private static String sha256_HMAC (String resource, String secret){
String hash = "";
Try {
Mac sha256_HMAC = Mac. getInstance ("HmacSHA256");
SecretKeySpec secret_key = new SecretKeySpec (secret. getBytes ("UTF-8"), "HmacSHA256");
Sha256_HMAC.init (secret_key);
Byte [] bytes = sha256_HMAC. doFinal (resource. getBytes ("UTF-8");
Hash = byteArray ToHexString (bytes);
} catch (Exception){
System. out. println ("Error HmacSHA256 =============="+e. getMessage ()));
}
Return hash;
}

* *
* Converting an encrypted byte array to a string
*
*@ Param bytes byte array
*@ return string
* /
Private static String by teArray ToHexString (byte [] bytes){
StringBuffer buffer = new StringBuffer ();
String STMP;
For (int index = 0; bytes!= null & & index < bytes. length; index ++){
STMP = Integer. toHexString (bytes [index] & 0XFF);
If (stmp. length () == 1){
// One-digit Zero Padding
Buffer. append ('0');
}
Buffer. append (stmp);
}
Return buffer. toString (). toLowerCase ();
}

Sample code (Python version):

Import hashlib
Import HMAC
Import unittest

Def sign (message, secret):
""
Gen sign
: param message: Message wait sign
Param secret: secret key
Return:
""
Secret = secret. encode ('utf-8')
Message = message. encode ('utf-8')
Sign = hmac. new (secret, message, digestmod = hashlib. sha256). hexdigest ()
Return sign

Class TestUtil (unittest. TestCase):
Def test_sign (self):
Sn = sign ("2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info", "9daf13ebd76c4f358fc885ca6ede5e27")
Self. assertEqual (sn, "a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2")


```


### Private Interface - Application for Currency Drawing Interface

```
Used for API users to apply for withdrawal of money
Speed Limitation Number: 1/1 second
HTTP POST/api/capital/v1/withdraw/apply
```
Request parameters:

Name | Type | Is it mandatory | Description
---------|--------|---------|--------
Asset | string | is the | asset name, such as BTC
Amount | string | is | withdrawal amount
Address | string | is | withdrawal address
Tag | string | no | notes on withdrawal address, according to the actual address
Chain | string | no | chain


Return result parameters

Name | Type | Description
---------|--------|---------
ID | string | withdrawal application ID returned after success
Asset | string | asset name, such as BTC
Amount | string | Number of withdrawals
Address | string | withdrawal address
Tag | string | Note on the address of withdrawal
Chain | string | chain


```
Request:
Url: http://domain name/api/capital/v1/withdraw/apply
Method: POST
Headers:
Accept: application/json
ACCESS-KEY: 03a0a94d6bb16c81f133a4fc3d2c8790
ACCESS-SIGN: af0270d6a7d80da7dabbebbcc8e7b2d31fc929489b1f6bdd1e611d5da9
ACCESS-TIMESTAMP: 2019-10-12T09:40:19.683Z
Content-Type: application/json; charset = UTF-8
Cookie: locale = en_US
Body: {"amount": "1", "asset", "BTC", "address": "rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ", "tag": "10000737"}
PreHash: 2019-10-12T09:40:19.683ZPOST/api/capital/v1/withdraw/apply {"amount": "1", "asset", "BTC", "address": "rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ", "tag": "10000737"}

Response:
{
"Code": 200,
"Data": {
"Id": "123",
"Amount": "10".
"Asset": "BTC"
"Address": "rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ".
"Tag":
"Chain": "
}
}
```
### Private Interface - Query the List of Currency Addresses


```
Get hold information of all contracts
Speed limit rule: 1 time / 1 second
HTTP GET/api/capital/v1/deposit/address/list
```
Request parameters:

Name | Type | Is it mandatory | Description
---------|--------|---------|--------
Asset | string | is the | asset name, such as BTC

Return field description:

Name | Type | Description
---------|--------|---------
Asset | string | asset name
Chain | string | chain
Address | string | currency address
AddressTag | string|
DepositLimit | string | Minimum Currency Exchange Limit
BlockNumber | string | Confirm the number of blocks
Status | string | 1 is available, 0 is unavailable


```
Request:
Url: http://domain name/api/capital/v1/deposit/address/list?Asset=XRP
Method: GET
Headers:
Accept: application/json
ACCESS-KEY: 03a0a94d6bb16c81f133a4fc3d2c8790
ACCESS-SIGN: 56113c528d4050ad7b45eb6880accdb3803d0d88dbb7cc6b56558439cb047303
ACCESS-TIMESTAMP: 2019-10-12T09:40:20.546Z
Content-Type: application/json; charset = UTF-8
Cookie: locale = en_US
Body:
PreHash: 2019-10-12T09:40:20.546ZGET/api/capital/v1/deposit/address/list?Asset=XRP

Response:
{
"Code": 200,
"Data":
{
"Asset": "XRP"
"Chain": "XRP"
"Address": "rHyS9xSwQUBqm5KjwprUXDWxZcwEMZYQMJ".
"AddressTag": "10000737".
"Deposit Limit": "25".
"Block Number": "2".
"Status": "1"
}
]
}
```


## Error code summary

Error code | message
---|---
12001 | "ACCESS_KEY" cannot be empty
12002 | "ACCESS_SIGN" cannot be empty
12003 | "ACCESS_TIMESTAMP" cannot be empty
12005 | Invalid ACESS_TIMESTAMP
12006 | Invalid ACESS_KEY
12007 | Invalid Content_Type, use the "application/json" format
12008 | Request timestamp expiration
12009 | System Error
120011 | invalid sign
120012 | API verification failure
429 | Requests are too frequent
11000 |{0} mandatory parameter is blank
11001 |{0} parameter value is not valid
11002 |{0} parameter values exceed the maximum limit
11003 | No data is returned to the third-party interface for the time being
2000 | Inadequate account balance
2001 | Accounts frozen
2003 | Users have no right to withdraw money
2006 | Failure to apply for withdrawal of currency
2010 | less than the minimum withdrawal amount
2016 | Illegal precision of withdrawal
2024 | Assets do not exist
2035 | Maximum withdrawal in more than 24 hours
2038 | The amount of money withdrawn should be greater than 0
2049 | Please complete the real-name certification and increase the amount of money withdrawn
2050 | Please add this address to the withdrawal address first.
2051 | This withdrawal address takes less than 24 hours to create
2060 | No transfer
2071 | No withdrawal within 24H for new users
10001 | address invalid
10002 | Address has been disabled
10003 | No withdrawal of money from the contract address
10004 | No withdrawal of money from the platform
10005 | {0} is invalid
10006 | Lack of {0}
10007 | The maximum amount of money withdrawn per time should not exceed {0}
10008 | No asset allocation found
10009 | temporarily unable to obtain a replenishment address
10007 | The maximum amount of money withdrawn per time should not exceed {0}
10008 | No asset allocation found
13002 | No asset allocation found



