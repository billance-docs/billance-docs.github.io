## Basic Information

- This article lists the baseurl of the REST interface: https://openapi.billance.com/spot/
- It is recommended to modify and add your own server export IP after creating the API to further enhance the API security verification
- All interface responses are in JSON format
- All times and timestamps are UNIX time in milliseconds
- HTTP 4XX error codes are used to indicate wrong request content, behavior, format
- HTTP 429 error code indicates warning access frequency exceeding the limit
- HTTP 5XX error codes are used to indicate problems on the Billance service side
- HTTP 504 means that the API server has submitted a request to the business core but failed to get a response. It is important to note that the 504 code does not mean that the request failed, but is unknown. It is very likely that it has been executed, or it may fail to execute, and further confirmation is required
- Each interface may throw an exception. The format of the exception response is as follows:

```
{
  "code": 10001,
  "msg": "Invalid Paramater."
}
```

- The specific error codes and their explanations are summarized in the error codes
- The interface of the GET method, the parameters must be sent in the query string
- The interface of the POST method, the parameters are sent in the request body (content type application/json)
- The order of parameters is not required

## Access Restriction Instructions

- When the access interface exceeds the frequency limit, the status 429 will be returned: Request too frequently
- Limitation rules, if a valid API key is passed in, the user id will be used to limit the speed; if not, the user's public IP will be used to limit the speed

## Interface Type

- There are mainly two types of interfaces, public interface and private interface
- The public interface can be called without authentication
- Private interface for user orders and accounts. Each private request must be signed using the canonical form of verification. Private interfaces require authentication with your API key

## Signature method

All interface request headers must contain the following:

- ACCESS-KEY API key of string type
- ACCESS-SIGN uses Hex to generate a string to return. For the specific code, please refer to the following Java version code
- ACCESS-TIMESTAMP The timestamp when the request was made
- All requests should contain application/json type content and be valid JSON

Value generation rules for ACCESS-SIGN:

- According to timestamp + method + requestPath + body string (+ means string connection), and secret, use HMAC SHA256 method to encrypt, and finally convert the byte array of the encrypted string into a hexadecimal string and return it
- The value of timestamp is the same as the ACCESS-TIMESTAMP request header and must be the decimal seconds of the Unix timestamp in the UTC time zone or the ISO8601 standard time format, accurate to milliseconds
- Method is the request method, all uppercase letters: GET/POST
- requestPath is the request interface path, for example: /api/v1/spot/account/list
- body is a string that refers to the body of the request. GET requests have no body information and can be omitted; POST requests have body information JSON strings, such as {"instrument_id":"BTC/USDT","price":"3000.0","quantity":"1","direction":"1"}
- The secret is generated when the user applies for the API

Sample interface request:

- There are two cases of GET protocol interface:

```
1. Without parameters
		the signature string: 2019-06-13T11:18:29.009ZGET/api/exchange/v2/market/tradePair/list
2. With parameters
		signature string: 2022-02-23T10:07:36.749ZGET/api/v1/spot/account/list
```


- POST protocol interface situation:

```
Signature string:
2022-02-23T10:09:25.519ZPOST/api/v1/spot/order{"instrument_id":"BTC/USDT","price":"3000.0","quantity":"1","direction":" 1"}
```

- Signature Algorithm Verification:


```
Sample data:
Accept: application/json
ACCESS-KEY: 0748978dd3e78e9f68b0e4d4e2712fb4
ACCESS-SIGN: f6c6fb7f43fdad3cdbb4c51ef601a893affbee6ae2d306a1b0125578d2e7b5f8
ACCESS-TIMESTAMP: 2022-02-23T10:10:36.304Z
Content-Type: application/json; charset=UTF-8
secret: 90d48d98ce244f2abb6f6979ddd957b4

Sample code (Java version)
		/**
    * Generate signature
    * @param timeStamp timestamp
    * @param method Request method: POST or GET
    * @param requestUrl url
    * @param requestBody request content, no null
    * @param secret key
    */
  public static String sign(String timestamp, String method, String requestPath, String queryString, String body, String secretKey) throws Exception {
        if (StringUtils.isEmpty(secretKey) || StringUtils.isEmpty(method)) {
            return APIConstants.EMPTY;
        }
        String preHash = preHash(timestamp, method, requestPath, queryString, body);

        Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
        byte[] secretKeyBytes = secretKey.getBytes(CharsetEnum.UTF_8.charset());
        SecretKeySpec secret_key = new SecretKeySpec(secretKeyBytes, "HmacSHA256");
        sha256_HMAC.init(secret_key);
        byte[] bytes = sha256_HMAC.doFinal(preHash.getBytes(CharsetEnum.UTF_8.charset()));
        return byteArrayToHexString(bytes);

    }
    
    /////////////////////
    
    /**
     * the prehash string = timestamp + method + requestPath + body <br/>
     *
     * @param timestamp   the number of seconds since Unix Epoch in UTC. Decimal values are allowed.
     *                    eg: 2018-03-08T10:59:25.789Z
     * @param method      eg: POST
     * @param requestPath eg: /api/v1/spot/account/one
     * @param queryString eg: asset=USDT
     * @param body        json string, eg: {"instrument_id":"BTC/USDT","price":"3000.0","quantity":"1","direction":"1"}
     */
    public static String preHash(String timestamp, String method, String requestPath, String queryString, String body) {
        StringBuilder preHash = new StringBuilder();
        preHash.append(timestamp);
        preHash.append(method.toUpperCase());
        preHash.append(requestPath);
        if (StringUtils.isNotEmpty(queryString)) {
            preHash.append("?").append(queryString);
        }
        if (StringUtils.isNotEmpty(body)) {
            preHash.append(body);
        }
        return preHash.toString();
    }
    
    /////////////////////
    
    private static String byteArrayToHexString(byte[] b) {
        StringBuilder hs = new StringBuilder();
        String stmp;
        for (int n = 0; b!=null && n < b.length; n++) {
            stmp = Integer.toHexString(b[n] & 0XFF);
            if (stmp.length() == 1)
                hs.append('0');
            hs.append(stmp);
        }
        return hs.toString().toLowerCase();
    }
    
```

## Interface specification

### Public interface - Get currency pair list

```
Interface rate limit rule: 1 times/1 HTTP GET /spot/api/spot/instruments/trade_pair_list
```

Request parameters: None

Return field description:

| Name              | Type   | Description                          |      |
| ----------------- | ------ | ------------------------------------ | ---- |
| trade_pair_name   | string | currency pair name, such as BTC/USDT |      |
| base_asset        | string | underlying assets, such as BTC       |      |
| quote_asset       | string | valuation assets, such as USDT       |      |
| price_precision   | string | price precision                      |      |
| amount_precision  | string | quantity precision                   |      |
| taker_fee_rate    | string | taker                                |      |
| maker_fee_rate    | string | maker                                |      |
| min_amount        | string | minimum number of delegates          |      |
| price_fluctuation | string | price fluctuation limit              |      |
| min_notional      | string |                                      |      |


```
Request(2022-01-21 19:36:29:627):
		Url: https://openapi.billance.com/spot/api/spot/instruments/trade_pair_list
		Method: GET
		Headers: 
			Accept: application/json
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-21T11:36:29.617ZGET/api/spot/instruments/trade_pair_list
		
Response:
{
    "code":200,
    "data":[
        {
            "trade_pair_name":"BCH/USDT",
            "base_asset":"BCH",
            "quote_asset":"USDT",
            "price_precision":"2",
            "amount_precision":"4",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"0.01",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"BSV/USDT",
            "base_asset":"BSV",
            "quote_asset":"USDT",
            "price_precision":"2",
            "amount_precision":"4",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"0.01",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"BTC/USDT",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "price_precision":"2",
            "amount_precision":"4",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"0.0001",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"DOGE/USDT",
            "base_asset":"DOGE",
            "quote_asset":"USDT",
            "price_precision":"6",
            "amount_precision":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"50",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"EOS/USDT",
            "base_asset":"EOS",
            "quote_asset":"USDT",
            "price_precision":"4",
            "amount_precision":"2",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"1",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"ETH/USDT",
            "base_asset":"ETH",
            "quote_asset":"USDT",
            "price_precision":"2",
            "amount_precision":"4",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"0.001",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"LTC/USDT",
            "base_asset":"LTC",
            "quote_asset":"USDT",
            "price_precision":"2",
            "amount_precision":"4",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"0.01",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"SHIB/USDT",
            "base_asset":"SHIB",
            "quote_asset":"USDT",
            "price_precision":"10",
            "amount_precision":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"100000",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"SOL/USDT",
            "base_asset":"SOL",
            "quote_asset":"USDT",
            "price_precision":"3",
            "amount_precision":"2",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"0.2",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"TRX/USDT",
            "base_asset":"TRX",
            "quote_asset":"USDT",
            "price_precision":"5",
            "amount_precision":"2",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"10",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"USDC/USDT",
            "base_asset":"USDC",
            "quote_asset":"USDT",
            "price_precision":"4",
            "amount_precision":"4",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"1",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        },
        {
            "trade_pair_name":"XRP/USDT",
            "base_asset":"XRP",
            "quote_asset":"USDT",
            "price_precision":"5",
            "amount_precision":"2",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"1",
            "price_fluctuation":"0.20",
            "min_notional":"10"
        }
    ]
}
```

### Public interface - Single currency pair information

```
Interface rate limit rule 1 times/1 HTTP GET /spot/api/spot/instruments/trade_pair_one
```

Request parameters:

| Name          | Type   | Required | Description                          |
| ------------- | ------ | -------- | ------------------------------------ |
| instrument_id | string | yes      | currency pair name, such as BTC/USDT |

Return field description:

| Name             | Type   | Description                          |
| ---------------- | ------ | ------------------------------------ |
| trade_pair_name  | string | currency pair name, such as BTC/USDT |
| base_asset       | string | underlying assets, such as BTC       |
| quote_asset      | string | valuation assets, such as USDT       |
| price_precision  | string | price precision                      |
| amount_precision | string | quantity precision                   |
| taker_fee_rate   | string | taker rate                           |
| maker_fee_rate   | string | maker rate                           |
| min_amount       | string | minimum number of delegates          |
| price_luctuation | string | price fluctuation limit              |
| min_notional     | string |                                      |

```
Request(2022-01-21 19:39:02:174):
		Url: https://openapi.billance.com/spot/api/spot/instruments/trade_pair_one?instrument_id=BTC%2FUSDT
		Method: GET
		Headers: 
			Accept: application/json
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-21T11:39:02.164ZGET/api/spot/instruments/trade_pair_one?instrument_id=BTC%2FUSDT
		
Response:
{
    "code":200,
    "data":{
        "trade_pair_name":"BTC/USDT",
        "base_asset":"BTC",
        "quote_asset":"USDT",
        "price_precision":"2",
        "amount_precision":"4",
        "taker_fee_rate":"0.001",
        "maker_fee_rate":"0.001",
        "min_amount":"0.0001",
        "price_fluctuation":"0.20",
        "min_notional":"10"
    }
}
```

### Public interface - Depth

```
Interface rate limit rule: 2 times/1 HTTP GET /spot/api/spot/instruments/depth
```

Request parameters:

| Name          | Type   | Required | Description                                        |
| ------------- | ------ | -------- | -------------------------------------------------- |
| instrument_id | string | yes      | is the name of a currency pair, such as BTC/USDT   |
| depth         | string | yes      | is the depth gear with values of 5, 10, 50 and 100 |

Return field description:

| Name      | Type   | Description                     |
| --------- | ------ | ------------------------------- |
| ask       | array  | Seller depth, [price, quantity] |
| bid       | array  | Buyer depth, [price, quantity]  |
| timestamp | string | utc timestamp                   |

```
Request(2022-01-21 19:40:33:943):
		Url: https://openapi.billance.com/spot/api/spot/instruments/depth?instrument_id=BTC%2FUSDT&depth=5
		Method: GET
		Headers: 
			Accept: application/json
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-21T11:40:33.932ZGET/api/spot/instruments/depth?instrument_id=BTC%2FUSDT&depth=5

Response:
{
    "code":200,
    "data":{
        "asks":[
            [
                "38974.2",
                "0.3121"
            ],
            [
                "38987.33",
                "0.0056"
            ],
            [
                "39051.83",
                "0.3132"
            ],
            [
                "39129.47",
                "0.1273"
            ],
            [
                "39142.02",
                "0.0077"
            ]
        ],
        "bids":[
            [
                "38730.32",
                "0.005"
            ],
            [
                "38711.3",
                "0.002"
            ],
            [
                "38710.89",
                "0.03"
            ],
            [
                "38675.44",
                "0.0052"
            ],
            [
                "38664.92",
                "0.0102"
            ]
        ],
        "timestamp":1642765235009
    }
}
```

### Public interface - Get ticker list information

```
Interface rate limit rule: 2 times/1 HTTP GET /spot/api/spot/instruments/ticker_list
```

Request parameters:  None


Return field description:

| Name               | Type   | Description                          |
| ------------------ | ------ | ------------------------------------ |
| trade_ pair_name   | string | currency pair name, such as BTC/USDT |
| last_ price        | string | latest price                         |
| lowest_ ask        | string | one price                            |
| highest_ bid       | string | buy a price                          |
| highest_ price_24h | string | 24h highest price                    |
| lowest_ price_24h  | string | 24h lowest price                     |
| volume24h          | string | 24h trading volume                   |
| chg24h             | string | 24h price change rate                |
| chg0h              | string | 0h price change rate                 |
| amount24h          | string | 24h trading volume                   |

```
Request(2022-01-22 10:21:46:461):
		Url: https://openapi.billance.com/spot/api/spot/instruments/ticker_list
		Method: GET
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: 2d76c96336c25cb27c89470c0e5f1b03a6024e096c1c6689405343156e9a2f29
			ACCESS-TIMESTAMP: 2022-01-22T02:21:46.415Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-22T02:21:46.415ZGET/api/spot/instruments/ticker_list
		
Response:
{
    "code":200,
    "data":[
        {
            "trade_pair_name":"SHIB/USDT",
            "last_price":"0.0000223772",
            "highest_bid":"0",
            "lowest_ask":"0",
            "highest_price_24h":"0.0000260420",
            "lowest_price_24h":"0.0000217039",
            "volume24h":"2950560",
            "chg24h":"-0.1210",
            "chg0h":"0.0049",
            "amount24h":"2950560"
        },
        {
            "trade_pair_name":"SOL/USDT",
            "last_price":"112.281",
            "highest_bid":"112.19",
            "lowest_ask":"112.41",
            "highest_price_24h":"126.834",
            "lowest_price_24h":"106.400",
            "volume24h":"6457711.14",
            "chg24h":"-0.0950",
            "chg0h":"-0.0010",
            "amount24h":"6457711.14"
        },
        {
            "trade_pair_name":"BCH/USDT",
            "last_price":"313.86",
            "highest_bid":"313.8300",
            "lowest_ask":"314.4600",
            "highest_price_24h":"344.58",
            "lowest_price_24h":"308.79",
            "volume24h":"7355649.7253",
            "chg24h":"-0.0700",
            "chg0h":"0.0000",
            "amount24h":"7355649.7253"
        },
        {
            "trade_pair_name":"BTC/USDT",
            "last_price":"36567.62",
            "highest_bid":"36468.9300",
            "lowest_ask":"36715.4200",
            "highest_price_24h":"39406.97",
            "lowest_price_24h":"35648.44",
            "volume24h":"8616829.7790",
            "chg24h":"-0.0500",
            "chg0h":"0.0000",
            "amount24h":"8616829.7790"
        },
        {
            "trade_pair_name":"ETH/USDT",
            "last_price":"2605.19",
            "highest_bid":"2601.4800",
            "lowest_ask":"2606.6800",
            "highest_price_24h":"2899.77",
            "lowest_price_24h":"2446.41",
            "volume24h":"10805116.9453",
            "chg24h":"-0.0800",
            "chg0h":"0.0100",
            "amount24h":"10805116.9453"
        },
        {
            "trade_pair_name":"XRP/USDT",
            "last_price":"0.64805",
            "highest_bid":"0.64",
            "lowest_ask":"0.64",
            "highest_price_24h":"0.69923",
            "lowest_price_24h":"0.61381",
            "volume24h":"4957970.88",
            "chg24h":"-0.0615",
            "chg0h":"0.0162",
            "amount24h":"4957970.88"
        },
        {
            "trade_pair_name":"BSV/USDT",
            "last_price":"101.31",
            "highest_bid":"101.3100",
            "lowest_ask":"101.3100",
            "highest_price_24h":"101.31",
            "lowest_price_24h":"101.31",
            "volume24h":"0.0000",
            "chg24h":"0.0000",
            "chg0h":"0.0000",
            "amount24h":"0.0000"
        },
        {
            "trade_pair_name":"DOGE/USDT",
            "last_price":"0.144180",
            "highest_bid":"0",
            "lowest_ask":"0",
            "highest_price_24h":"0.154525",
            "lowest_price_24h":"0.140136",
            "volume24h":"20943064",
            "chg24h":"-0.0370",
            "chg0h":"0.0141",
            "amount24h":"20943064"
        },
        {
            "trade_pair_name":"EOS/USDT",
            "last_price":"2.4152",
            "highest_bid":"2.41",
            "lowest_ask":"2.42",
            "highest_price_24h":"2.5916",
            "lowest_price_24h":"2.3149",
            "volume24h":"2945445.81",
            "chg24h":"-0.0476",
            "chg0h":"0.0196",
            "amount24h":"2945445.81"
        },
        {
            "trade_pair_name":"LTC/USDT",
            "last_price":"115.07",
            "highest_bid":"115.0300",
            "lowest_ask":"115.2600",
            "highest_price_24h":"125.96",
            "lowest_price_24h":"110.20",
            "volume24h":"1983110.2431",
            "chg24h":"-0.0500",
            "chg0h":"0.0000",
            "amount24h":"1983110.2431"
        },
        {
            "trade_pair_name":"TRX/USDT",
            "last_price":"0.06082",
            "highest_bid":"0.06",
            "lowest_ask":"0.06",
            "highest_price_24h":"0.06731",
            "lowest_price_24h":"0.05990",
            "volume24h":"4745204.31",
            "chg24h":"-0.0777",
            "chg0h":"-0.0024",
            "amount24h":"4745204.31"
        },
        {
            "trade_pair_name":"USDC/USDT",
            "last_price":"0.9994",
            "highest_bid":"0.9989",
            "lowest_ask":"1.0009",
            "highest_price_24h":"1.0011",
            "lowest_price_24h":"0.9989",
            "volume24h":"1776635.8520",
            "chg24h":"-0.0009",
            "chg0h":"0.0000",
            "amount24h":"1776635.8520"
        }
    ]
}
```

### Public interface - Get the specified ticker information

```
Interface rate limit rule: 3 times/1 HTTP GET /spot/api/spot/instruments/ticker_one
```

Request parameters: 

| Name          | Type   | Required | Description                                      |
| ------------- | ------ | -------- | ------------------------------------------------ |
| instrument_id | string | yes      | is the name of a currency pair, such as BTC/USDT |

Return field description:

| field name         | type   | description           |
| ------------------ | ------ | --------------------- |
| trade_ pair_name   | string | coin pair name        |
| last_price         | string | latest price          |
| lowest_ask         | string | one price             |
| highest_bid        | string | buy a price           |
| highest_ price_24h | string | 24h highest price     |
| lowest_ price_24h  | string | 24h lowest price      |
| volume24h          | string | 24h trading volume    |
| chg24h             | string | 24h price change rate |
| chg0h              | string | 0h price change rate  |
| amount24h          | string | 24h trading volume    |

```
Request(2022-01-22 11:31:22:818):
		Url: https://openapi.billance.com/spot/api/spot/instruments/ticker_one?instrument_id=SHIB%2FUSDT
		Method: GET
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: 1f223586a80a8a6ce3a6f8e5b2ce3761bd5c3beee817caaca1d43b55de30f8d2
			ACCESS-TIMESTAMP: 2022-01-22T03:31:22.775Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-22T03:31:22.775ZGET/api/spot/instruments/ticker_one?instrument_id=SHIB%2FUSDT
		
Response:
{
    "code":200,
    "data":{
        "trade_pair_name":"SHIB/USDT",
        "last_price":"0.0000222350",
        "highest_bid":"0",
        "lowest_ask":"0",
        "highest_price_24h":"0.0000260420",
        "lowest_price_24h":"0.0000217039",
        "volume24h":"2957978",
        "chg24h":"-0.1266",
        "chg0h":"-0.0014",
        "amount24h":"2957978"
    }
}
```

### Public interface - Get K-line data

```
Interface rate limit rule: 4 times/1 HTTP GET /spot/api/spot/instruments/candles
```

Request parameters:

| Name          | Type   | Required | Description                                                  |      |
| ------------- | ------ | -------- | ------------------------------------------------------------ | ---- |
| instrument_id | string | yes      | currency pair, such as BTC/USDT                              |      |
| start_time    | string | No       | is the start time, and the iso8601 format timestamp is seconds |      |
| end_time      | string | No       | is the deadline. The iso8601 format timestamp is seconds     |      |
| period        | string | yes      | Kline. Please refer to the description for the value range   |      |


```
The value of period can only be "1", "3", "5", "15", "30", "60", "120", "240", "360", "720", "D", "W", "M"], otherwise the request will be rejected,
Corresponding to [1min, 3min, 5min, 15min, 30min, 1 hour, 2 hour, 4 hour, 6 hour, 12 hour, 1 day, 1 week, 1 month]

Format Description: [timestamp,open,high,low,close,volume]

Request(2022-01-21 19:47:34:225):
		Url: https://openapi.billance.com/spot/api/spot/instruments/candles?instrument_id=BTC%2FUSDT&period=1&start_time=&end_time=
		Method: GET
		Headers: 
			Accept: application/json
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-21T11:47:34.215ZGET/api/spot/instruments/candles?instrument_id=BTC%2FUSDT&period=1&start_time=&end_time=
		
Response:
{
    "code":200,
    "data":[
        [
            "1642765620000",
            "38969.61",
            "38969.61",
            "38897.14",
            "38897.14",
            "0.1034"
        ],
        [
            "1642765560000",
            "38912.89",
            "38969.62",
            "38912.89",
            "38969.61",
            "0.2745"
        ],
        [
            "1642765500000",
            "38893.31",
            "38937.57",
            "38893.31",
            "38912.89",
            "0.004"
        ],
        [
            "1642765440000",
            "38904.65",
            "38918.23",
            "38893.31",
            "38893.31",
            "0.1192"
        ],
        [
            "1642765380000",
            "38892.98",
            "38917.72",
            "38892.98",
            "38904.65",
            "0.2129"
        ],
        [
            "1642765320000",
            "38888.91",
            "38906.85",
            "38880",
            "38892.98",
            "0.2201"
        ],
        [
            "1642765260000",
            "38870.47",
            "38894.31",
            "38870.47",
            "38888.91",
            "0.1558"
        ],
        [
            "1642765200000",
            "38827.62",
            "38895.81",
            "38827.62",
            "38870.47",
            "0.1206"
        ],
        [
            "1642765140000",
            "38782.24",
            "38835.7",
            "38782.24",
            "38827.62",
            "0.2082"
        ],
        [
            "1642765080000",
            "38908.77",
            "38918.15",
            "38763.27",
            "38782.24",
            "0.1312"
        ]
    ]
}
```


### Public interface - Get the latest trade list information

```
Interface rate limit rule: 1 times/1 HTTP GET /spot/api/spot/instruments/trade_list
```

Request parameters:

| Name          | Type   | Required | Description                                      |
| ------------- | ------ | -------- | ------------------------------------------------ |
| instrument_id | string | yes      | is the name of a currency pair, such as BTC/USDT |

Return field description:


```
Request(2022-01-21 19:50:00:653):
		Url: https://openapi.billance.com/spot/api/spot/instruments/trade_list?instrument_id=BTC%2FUSDT
		Method: GET
		Headers: 
			Accept: application/json
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-21T11:50:00.641ZGET/api/spot/instruments/trade_list?instrument_id=BTC%2FUSDT

Format Description: [trade_pair_name,price,volume,side,timestamp]

Response:
{
    "code":200,
    "data":[
        [
            "BTC/USDT",
            "38839.07",
            "0.0581",
            "sell",
            "1642765788000"
        ],
        [
            "BTC/USDT",
            "38865.89",
            "0.0088",
            "sell",
            "1642765772000"
        ],
        [
            "BTC/USDT",
            "38861.36",
            "0.0056",
            "sell",
            "1642765760000"
        ],
        [
            "BTC/USDT",
            "38870.11",
            "0.0354",
            "sell",
            "1642765746000"
        ],
        [
            "BTC/USDT",
            "38870.11",
            "0.1418",
            "sell",
            "1642765734000"
        ],
        [
            "BTC/USDT",
            "38855.92",
            "0.0058",
            "sell",
            "1642765719000"
        ],
        [
            "BTC/USDT",
            "38855.92",
            "0.0536",
            "sell",
            "1642765705000"
        ],
        [
            "BTC/USDT",
            "38855.93",
            "0.0056",
            "sell",
            "1642765688000"
        ],
        [
            "BTC/USDT",
            "38877.31",
            "0.0079",
            "buy",
            "1642765673000"
        ],
        [
            "BTC/USDT",
            "38877.31",
            "0.0767",
            "buy",
            "1642765659000"
        ]
    ]
}

```

### Private interface - Get all account information

```
Interface rate limit rule: 1 times/1 HTTP GET /spot/api/v1/spot/account/list
```

Request parameter: None

Return result parameters:

| Name           | type   | description                           |
| -------------- | ------ | ------------------------------------- |
| asset          | string | asset name                            |
| available      | string | available balance                     |
| frozen_balance | string | freeze balance                        |
| total_balance  | string | total balance,include freezes balance |

```
Request(2022-01-21 19:54:51:14):
		Url: https://openapi.billance.com/spot/api/v1/spot/account/list
		Method: GET
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: b4cf211599cbb57ee3a6ca56b2d106704d9f0f0d11aca18c679780800546a115
			ACCESS-TIMESTAMP: 2022-01-21T11:54:50.982Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-21T11:54:50.982ZGET/api/v1/spot/account/list
		
Response:
{
    "code":200,
    "data":[
        {
            "asset":"USDT",
            "available":"10000",
            "frozen_balance":"0",
            "total_balance":"0"
        },
        {
            "asset":"SHIB",
            "available":"0",
            "frozen_balance":"0",
            "total_balance":"0"
        },
        {
            "asset":"ETH",
            "available":"0",
            "frozen_balance":"0",
            "total_balance":"0"
        },
        {
            "asset":"BTC",
            "available":"0",
            "frozen_balance":"0",
            "total_balance":"0"
        }
    ]
}
```

### Private interface - Get asset information of specified account

```
Interface rate limit rule: 1 times/1 HTTP GET /spot/api/v1/spot/account/one
```

Request parameters

| Name  | Type   | Required | Description                                          |
| ----- | ------ | -------- | ---------------------------------------------------- |
| asset | string | yes      | is the name / abbreviation of an asset, such as USDT |

Return result parameters

| Name           | type   | description               |
| -------------- | ------ | ------------------------- |
| asset          | string | asset name / abbreviation |
| available      | string | available balance         |
| frozen_balance | string | freeze balance            |
| total_balance  | string | total, frozen + balance   |

```
Request(2022-01-21 19:52:38:332):
		Url: https://openapi.billance.com/spot/api/v1/spot/account/one?asset=USDT
		Method: GET
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: bffecee55130be478a686e9cb69df249760908e4c991d56d1f71a9d5642abda3
			ACCESS-TIMESTAMP: 2022-01-21T11:52:38.301Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-21T11:52:38.301ZGET/api/v1/spot/account/one?asset=USDT
		
Response:
{
    "code":200,
    "data":{
        "asset":"USDT",
        "available":"10000",
        "frozen_balance":"0",
        "total_balance":"0"
    }
}
```

### Private interface - Place order

```
Interface rate limit rule: 1 times/1 HTTP POST /spot/api/v1/spot/order
```

Request parameters:

| Name          | Type   | Required | Description                                               |
| ------------- | ------ | -------- | --------------------------------------------------------- |
| instrument_id | string | yes      | currency pair, such as BTC/USDT                           |
| direction     | string | yes      | is the direction, direction = 1: buy, direction = 2: sell |
| price         | string | yes      | is the order price                                        |
| quantity      | string | yes      | is the entrusted quantity                                 |

Return field description:

| Name     | Type   | Description        |
| -------- | ------ | ------------------ |
| order_id | string | generated order ID |

```
Request(2022-01-21 19:56:09:374):
		Url: https://openapi.billance.com/spot/api/v1/spot/order
		Method: POST
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: 52f6c855357b648e53f0b00ed69eddc30fab1766959ca6c3315294d17b1509c6
			ACCESS-TIMESTAMP: 2022-01-21T11:56:09.330Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: {"instrument_id":"BTC/USDT","price":"38000.0","quantity":"1","direction":"1"}
		preHash: 2022-01-21T11:56:09.330ZPOST/api/v1/spot/order{"instrument_id":"BTC/USDT","price":"38000.0","quantity":"1","direction":"1"}
		
Response:
{
    "code":200,
    "data":{
        "order_id":"2257615041262632960"
    }
}
```

### Private interface - Batch place order

```
Interface rate limit rule: 1 times/1 HTTP POST /spot/api/v1/spot/batch_order
```

The request parameter is an array object that contains the following parameters

| Name          | Type   | Required | Description                                                  |
| ------------- | ------ | -------- | ------------------------------------------------------------ |
| instrument_id | string | yes      | currency pair, such as BTC/USDT                              |
| direction     | string | yes      | order direction = 1 means buy direction = 2 means sell direction |
| price         | string | yes      | is the order price                                           |
| quantity      | string | yes      | is the entrusted quantity                                    |

Return field description:

| Name     | type   | description        |
| -------- | ------ | ------------------ |
| order_id | string | generated order ID |

```
Request(2022-01-21 20:04:32:565):
		Url: https://openapi.billance.com/spot/api/v1/spot/batch_order
		Method: POST
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: ac3fff8aab9075e8812851a46bb3ac352fce3d61fec8b74e2e73aa61f0239667
			ACCESS-TIMESTAMP: 2022-01-21T12:04:32.525Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: [{"instrument_id":"BTC/USDT","price":"3994.13","quantity":"0.1","direction":"1","orderType":"1"},{"instrument_id":"BTC/USDT","price":"3894.13","quantity":"0.2","direction":"1","orderType":"1"}]
		preHash: 2022-01-21T12:04:32.525ZPOST/api/v1/spot/batch_order[{"instrument_id":"BTC/USDT","price":"3994.13","quantity":"0.1","direction":"1","orderType":"1"},{"instrument_id":"BTC/USDT","price":"3894.13","quantity":"0.2","direction":"1","orderType":"1"}]
    
Response:
{
    "code":200,
    "data":[
        {
            "order_id":"2257617165715357696",
            "code":"200",
            "message":""
        },
        {
            "order_id":"2257617165786660864",
            "code":"200",
            "message":""
        }
    ]
}
```

### Private interface - Get the list of the current delegated orders

```
Interface rate limit rule: 1 times/1 HTTP GET /spot/api/v1/spot/open_order_list
```

Request parameters:

| Name          | Type   | Required | Description                                                  |
| ------------- | ------ | -------- | ------------------------------------------------------------ |
| instrument_id | string | yes      | is the name of a currency pair, such as BTC/USDT             |
| next_order_id | string | no       | order ID, which is used in pagination. The default value is empty. The latest 20 pieces of data are returned and displayed in reverse order by order ID. Get the last order Id-1, take the next page of data |

Return field description:

| Name            | Type   | Description                                                  |
| --------------- | ------ | ------------------------------------------------------------ |
| order_id        | string | order ID                                                     |
| base_asset      | string | transaction currency, such as BTC                            |
| quote_asset     | string | valuation currency, such as USDT                             |
| direction       | string | single direction                                             |
| quantity        | string | order quantity                                               |
| filled_quantity | string | number of transactions                                       |
| amount          | string | order amount                                                 |
| filled_amount   | string | transaction amount                                           |
| trade_pair_name | string | coin pair name                                               |
| price           | string | order price                                                  |
| status          | string | order status, unsettled: open, complete, cancelled, partially cancelled |
| order_time      | string | order time                                                   |
| fee             | string | handling fee                                                 |
| taker_fee_rate  | string | taker                                                        |
| taker_fee_rate  | string | maker                                                        |

```
Request(2022-01-21 20:00:45:572):
		Url: https://openapi.billance.com/spot/api/v1/spot/open_order_list?instrument_id=BTC%2FUSDT&latestOrderId=
		Method: GET
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: 1e45a4bb4dd81299141271bf0ac3696aaf896b72095d751c889af813b8653074
			ACCESS-TIMESTAMP: 2022-01-21T12:00:45.539Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-21T12:00:45.539ZGET/api/v1/spot/open_order_list?instrument_id=BTC%2FUSDT&latestOrderId=
		
Response:
{
    "code":200,
    "data":[
        {
            "order_id":"2257615454510661632",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"0.2",
            "filled_quantity":"0.0056",
            "amount":"7798.826",
            "filled_amount":"218.33252",
            "average_price":"",
            "status":"Open",
            "order_time":1642766269000,
            "fee":"0.21833252",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"38994.13",
            "order_type":"limit"
        },
        {
            "order_id":"2257615041262632960",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"1",
            "filled_quantity":"0",
            "amount":"38000",
            "filled_amount":"0",
            "average_price":"",
            "status":"Open",
            "order_time":1642766170000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"38000",
            "order_type":"limit"
        }
    ]
}
```

### Private interface - Get historical delegation list

```
Interface rate limit rule: 1 times/1 HTTP GET /spot/api/v1/spot/closed_order_list
```

Request parameters:

| Name          | Type   | Required | Description                                                  |
| ------------- | ------ | -------- | ------------------------------------------------------------ |
| instrument_id | string | yes      | is the name of a currency pair, such as BTC/USDT             |
| next_order_id | string | no       | order ID, which is used in pagination. The default value is empty. The latest 20 pieces of data are returned and displayed in reverse order by order ID. Get the last order Id-1, take the next page of data |

Return field description:

| Name            | type   | description                                                  |
| --------------- | ------ | ------------------------------------------------------------ |
| order_id        | string | order ID                                                     |
| base_asset      | string | transaction currency, such as BTC                            |
| quote_asset     | string | valuation currency, such as usdt                             |
| direction       | string | single direction                                             |
| quantity        | string | order quantity                                               |
| filled_quantity | string | number of transactions                                       |
| amount          | string | order amount                                                 |
| filled_amount   | string | transaction amount                                           |
| trade_pair_name | string | coin pair name                                               |
| price           | string | order price                                                  |
| status          | string | order status unsettled: Open, Filled, Cancelled, Partially cancelled |
| order_time      | string | order time                                                   |
| fee             | string | handling fee                                                 |
| taker_fee_rate  | string | taker                                                        |
| taker_fee_rate  | string | maker                                                        |

```
Request(2022-01-21 20:07:56:435):
		Url: https://openapi.billance.com/spot/api/v1/spot/closed_order_list?instrument_id=BTC%2FUSDT&latestOrderId=
		Method: GET
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: 9b2c34e31028b6bd177280f317ea44ef986a4c030634ea7ff061a5b9468b808b
			ACCESS-TIMESTAMP: 2022-01-21T12:07:56.398Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-21T12:07:56.398ZGET/api/v1/spot/closed_order_list?instrument_id=BTC%2FUSDT&latestOrderId=
		
Response:
{
    "code":200,
    "data":[
        {
            "order_id":"2257617165786660864",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"0.2",
            "filled_quantity":"0",
            "amount":"778.826",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642766677000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"3894.13",
            "order_type":"limit"
        },
        {
            "order_id":"2257617165715357696",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"0.1",
            "filled_quantity":"0",
            "amount":"399.413",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642766677000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"3994.13",
            "order_type":"limit"
        },
        {
            "order_id":"2257616624297820160",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"0.2",
            "filled_quantity":"0",
            "amount":"7798.826",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642766548000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"38994.13",
            "order_type":"limit"
        },
        {
            "order_id":"2257615454510661632",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"0.2",
            "filled_quantity":"0.0056",
            "amount":"7798.826",
            "filled_amount":"218.33252",
            "average_price":"",
            "status":"Partially cancelled",
            "order_time":1642766269000,
            "fee":"0.21833252",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"38994.13",
            "order_type":"limit"
        },
        {
            "order_id":"2257615041262632960",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"1",
            "filled_quantity":"0",
            "amount":"38000",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642766170000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"38000",
            "order_type":"limit"
        },
        {
            "order_id":"2257541740263751680",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"1",
            "filled_quantity":"0",
            "amount":"38000",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642748694000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"38000",
            "order_type":"limit"
        },
        {
            "order_id":"2257538740703084544",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"0.1",
            "filled_quantity":"0",
            "amount":"3833.813",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642747979000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"38338.13",
            "order_type":"limit"
        },
        {
            "order_id":"2257538716472590336",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"1",
            "filled_quantity":"0",
            "amount":"38000",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642747973000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"38000",
            "order_type":"limit"
        },
        {
            "order_id":"2257535910088916992",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"1",
            "filled_quantity":"0",
            "amount":"38000",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642747304000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"38000",
            "order_type":"limit"
        },
        {
            "order_id":"2257533663791661056",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"1",
            "filled_quantity":"0",
            "amount":"38000",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642746768000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"38000",
            "order_type":"limit"
        },
        {
            "order_id":"2257528716287164416",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"sell",
            "quantity":"0.0011",
            "filled_quantity":"0",
            "amount":"49.5",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642745589000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"45000",
            "order_type":"limit"
        },
        {
            "order_id":"2257528646372311040",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"0.1",
            "filled_quantity":"0",
            "amount":"3500",
            "filled_amount":"0",
            "average_price":"",
            "status":"Cancelled",
            "order_time":1642745572000,
            "fee":"0",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"35000",
            "order_type":"limit"
        },
        {
            "order_id":"2257527522894397440",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"0",
            "filled_quantity":"0.0128",
            "amount":"500",
            "filled_amount":"498.200445",
            "average_price":"",
            "status":"Filled",
            "order_time":1642745304000,
            "fee":"0.498200445",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "trade_pair_name":"BTC/USDT",
            "price":"0",
            "order_type":"market"
        }
    ]
}
```



### Private interface - Get specified order information

```
Interface rate limit rule: 1 times/1 HTTP GET /api/v1/spot/order_info?order_id=2120224216451338240
```

Request parameters:

| Name     | Type   | Required | Description     |
| -------- | ------ | -------- | --------------- |
| order_id | string | yes      | is the order ID |

Return field description:

| Name            | Type   | Description                                                  |
| --------------- | ------ | ------------------------------------------------------------ |
| order_id        | string | order ID                                                     |
| base_ asset     | string | transaction currency, such as BTC                            |
| quote_asset     | string | valuation currency, such as usdt                             |
| direction       | string | single direction                                             |
| quantity        | string | order quantity                                               |
| amount          | string | order amount                                                 |
| filled_amount   | string | transaction amount                                           |
| taker_fee_rate  | string | taker                                                        |
| maker_fee_rate  | string | maker                                                        |
| order_type      | string | order type limit indicates price limit order                 |
| price           | string | order price                                                  |
| status          | string | order status, unsettled: open, complete, cancelled, partially cancelled |
| order_time      | string | order time                                                   |
| fee             | string | handling fee                                                 |
| trade_pair_name | string | coin pair name                                               |

```
Request(2022-01-22 14:13:58:165):
		Url: https://openapi.billance.com/spot/api/v1/spot/order_info?order_id=2257824251095171072
		Method: GET
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: ea12373a7f8809bf22fea8639e3b985cf371aa5a5c4be5be1a3b42cbe590338f
			ACCESS-TIMESTAMP: 2022-01-22T06:13:58.124Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: 
		preHash: 2022-01-22T06:13:58.124ZGET/api/v1/spot/order_info?order_id=2257824251095171072
		
Response:
{
    "code":200,
    "data":{
        "order_id":"2257824251095171072",
        "base_asset":"BTC",
        "quote_asset":"USDT",
        "direction":"buy",
        "quantity":"0.1",
        "filled_quantity":"0",
        "amount":"3000",
        "filled_amount":"0",
        "average_price":"0",
        "status":"Cancelled",
        "order_time":1642816050000,
        "fee":"0",
        "taker_fee_rate":"0.001",
        "maker_fee_rate":"0.001",
        "trade_pair_name":"BTC/USDT",
        "price":"30000",
        "order_type":"limit"
    }
}
```

### Private interface - Cancel specified order

```
Interface rate limit rule: 1 times/1 HTTP POST /spot/api/v1/spot/cancel_order
```

Request parameters:

| Name     | Type   | Required | Description     |
| -------- | ------ | -------- | --------------- |
| order_id | string | yes      | is the order ID |

Return field description:

| Name     | Type   | Description        |
| -------- | ------ | ------------------ |
| order_id | string | cancelled order ID |



```
Request(2022-01-21 20:03:03:405):
		Url: https://openapi.billance.com/spot/api/v1/spot/cancel_order
		Method: POST
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: 3c2c6f249a0160c94339cfba6000c194db1dea5c4eb72f53c1b1bb49c7f76f63
			ACCESS-TIMESTAMP: 2022-01-21T12:03:03.363Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: {"order_id":"2257616624297820160"}
		preHash: 2022-01-21T12:03:03.363ZPOST/api/v1/spot/cancel_order{"order_id":"2257616624297820160"}
		
Response:
{
    "code":200,
    "data":{
        "order_id":"2257616624297820160"
    }
}
```

### Private interface - Batch Cancel specified orders

```
Interface rate limit rule: 1 times/1 HTTP POST /spot/api/v1/spot/batch_cancel_order
```

Request parameters:

| Name     | Type | Required | Description     |
| -------- | ---- | -------- | --------------- |
| orderIds | list | yes      | is the order ID |

Return field description:

| Name     | Type   | Description        |
| -------- | ------ | ------------------ |
| order_id | string | cancelled order ID |

```
Request(2022-01-21 20:05:43:965):
		Url: https://openapi.billance.com/spot/api/v1/spot/batch_cancel_order
		Method: POST
		Headers: 
			Accept: application/json
			ACCESS-KEY: e80fbf6a6f5fdd9ed9ddd52aedbd4380
			ACCESS-SIGN: a15838887cac21fac68295fc0fc29cc4c4fc85b3e90fe106f7f2c22b6af072db
			ACCESS-TIMESTAMP: 2022-01-21T12:05:43.927Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=en_US
		Body: {"orderIds":["2257617165715357696","2257617165786660864"]}
		preHash: 2022-01-21T12:05:43.927ZPOST/api/v1/spot/batch_cancel_order{"orderIds":["2257617165715357696","2257617165786660864"]}
		
Response:
{
    "code":200,
    "data":[
        {
            "order_id":"2257617165715357696",
            "code":"200",
            "message":""
        },
        {
            "order_id":"2257617165786660864",
            "code":"200",
            "message":""
        }
    ]
}
```
