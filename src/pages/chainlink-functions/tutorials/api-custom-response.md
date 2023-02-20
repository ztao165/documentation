---
layout: ../../../layouts/MainLayout.astro
section: chainlinkFunctions
date: Last Modified
title: "Return Custom Data Types"
setup: |
  import ChainlinkFunctions from "@features/chainlink-functions/common/ChainlinkFunctions.astro"
---

In this tutorial, you send a request to a Decentralized Oracle Network to call the [Cryptocompare GET /data/pricemultifull API](https://min-api.cryptocompare.com/documentation?key=Price&cat=multipleSymbolsFullPriceEndpoint). After [OCR](/chainlink-functions/resources/concepts/) completes off-chain computation and aggregation, it returns several a response to your smart contract. The response includes an asset price, daily volume, and market name to your smart contract. The example uses query parameters to specify the ETH/USD asset pair, but you can configure HTTP query parameters to make requests for different assets.

:::note[Maximum response size]
You can return any number of responses as long as they are encoded in a `bytes` response. The maximum response size that you can return is 256 bytes.
:::

## Before you begin

:::note[Request Access]
Chainlink Functions is currently in a limited BETA.
Apply [here](http://functions.chain.link/) to add your EVM account address to the Allow List.
:::

1. **[Complete the setup steps in the Getting Started guide](/chainlink-functions/getting-started):** The Getting Started Guide shows you how to set up your environment with the necessary tools for these tutorials. You can re-use the same consumer contract for each of these tutorials.

1. Make sure your subscription has enough LINK to pay for your requests. Read [Get Subscription details](/chainlink-functions/resources/subscriptions#get-subscription-details) to learn how to check your subscription balance. If your subscription runs out of LINK, follow the [Fund a Subscription](/chainlink-functions/resources/subscriptions#fund-a-subscription) guide.

1. **Check out the correct branch before you try this tutorial:** Each tutorial is stored in a separate branch of the [Chainlink Functions Starter Kit](https://github.com/smartcontractkit/functions-hardhat-starter-kit) repository.

   ```bash
   git checkout tutorial-3
   ```

## Tutorial

This tutorial is configured to get the `ETH/USD`, daily volume, and market in a single request. For a detailed explanation of the code example, read the [Explanation](#explanation) section.

- Open `Functions-request-config.js`. Note the `args` value is `["ETH", "USD"]`: We want to fetch the current `ETH/USD` price and daily volume. You can adapt `args` to get the list of supported symbols. Read the [API docs](https://min-api.cryptocompare.com/documentation?key=Price&cat=multipleSymbolsFullPriceEndpoint) to learn more about these configuration variables. For a more detailed explanation about the configuration in this example, read the [request config explanation](#functions-request-configjs) section.
- Open `Functions-request-source.js` to analyze the JavaScript source code. Read the [source code explanation](#functions-request-sourcejs) section for a more detailed explanation of how the source file is written.

### Simulation

The [Chainlink Functions hardhat starter kit](https://github.com/smartcontractkit/functions-hardhat-starter-kit) includes a simulator to test your Functions code on your local machine. The `functions-simulate` command will execute your code in a local runtime environment and simulate an end-to-end fulfillment. Simulation can help you to fix any issues before you submit your requests to the Decentralized Oracle Network.

Run the `functions-simulate` task to run the source code locally and make sure `Functions-request-config.js` and `Functions-request-source.js` are correctly written:

```bash
npx hardhat functions-simulate
```

Example:

```bash
$ npx hardhat functions-simulate
secp256k1 unavailable, reverting to browser version

__Compiling Contracts__
Nothing to compile
Duplicate definition of Transfer (Transfer(address,address,uint256,bytes), Transfer(address,address,uint256))

Executing JavaScript request source code locally...

__Console log messages from sandboxed code__
HTTP GET Request to https://min-api.cryptocompare.com/data/pricemultifull?fsyms=ETH&tsyms=USD
ETH price is: 1606.70 USD. 24h Volume is 658844.24 USD. Market: lmax

__Output from sandboxed source code__
Output represented as a hex string: 0x7b227072696365223a22313630362e3730222c22766f6c756d65223a223635383834342e3234222c226c6173744d61726b6574223a226c6d6178227d
Decoded as a string: {"price":"1606.70","volume":"658844.24","lastMarket":"lmax"}

__Simulated On-Chain Response__
Response returned to client contract represented as a hex string: 0x7b227072696365223a22313630362e3730222c22766f6c756d65223a223635383834342e3234222c226c6173744d61726b6574223a226c6d6178227d
Decoded as a string: {"price":"1606.70","volume":"658844.24","lastMarket":"lmax"}

Estimated transmission cost: 0.000047242892373668 LINK (This will vary based on gas price)
Base fee: 0.0 LINK
Total estimated cost: 0.000047242892373668 LINK
```

Reading the output of the example above, you can see that the `ETH/USD` price is _1606.70 USD_, volume is _658844.24_, and the market is _lmax_. Because the final result is a JSON object, the example converts it to a string and returns the `bytes` encoded value `0x7b227072696365223a22313630362e3730222c22766f6c756d65223a223635383834342e3234222c226c6173744d61726b6574223a226c6d6178227d` in the callback. Read the [source code explanation](#functions-request-sourcejs) for a more detailed explanation.

### Request

Send a request to the Decentralized Oracle Network to fetch the asset price. Run the `functions-request` task with the `subid` (subscription ID) and `contract` parameters. This task passes the JavaScript source code, arguments, and secrets when you call the `executeRequest` function in your deployed `FunctionsConsumer` contract. Read the [functionsConsumer](#functionsconsumersol) section for a more detailed explanation.

```bash
npx hardhat functions-request --subid REPLACE_SUBSCRIPTION_ID --contract REPLACE_CONSUMER_CONTRACT_ADDRESS --network REPLACE_NETWORK
```

Example:

```bash
$ npx hardhat functions-request --subid 6 --contract 0xa9b286E892d579dc727c79D3be9b01949796240A  --network mumbai
secp256k1 unavailable, reverting to browser version
Simulating Functions request locally...

__Console log messages from sandboxed code__
HTTP GET Request to https://min-api.cryptocompare.com/data/pricemultifull?fsyms=ETH&tsyms=USD
ETH price is: 1609.01 USD. 24h Volume is 664907.71 USD. Market: Coinbase

__Output from sandboxed source code__
Output represented as a hex string: 0x7b227072696365223a22313630392e3031222c22766f6c756d65223a223636343930372e3731222c226c6173744d61726b6574223a22436f696e62617365227d
Decoded as a string: {"price":"1609.01","volume":"664907.71","lastMarket":"Coinbase"}


If all 100000 callback gas is used, this request is estimated to cost 0.000052991755396126 LINK
Continue? (y) Yes / (n) No
y

Requesting new data for FunctionsConsumer contract 0xa9b286E892d579dc727c79D3be9b01949796240A on network mumbai
Waiting 2 blocks for transaction 0xb219bced3f395a0320155369006406da1f2c894b21ee59a80ee952898449889b to be confirmed...

Request 0x71e7e87772e5f60e41b945d7fad2c12affa65dba469cd164d27c32cab30806b7 initiated
Waiting for fulfillment...

Transmission cost: 0.00006213736353407 LINK
Base fee: 0.0 LINK
Total cost: 0.00006213736353407 LINK

Request 0x71e7e87772e5f60e41b945d7fad2c12affa65dba469cd164d27c32cab30806b7 fulfilled!
Response returned to client contract represented as a hex string: 0x7b227072696365223a22313630382e3731222c22766f6c756d65223a223636353030342e3930222c226c6173744d61726b6574223a22436f696e62617365227d
Decoded as a string: {"price":"1608.71","volume":"665004.90","lastMarket":"Coinbase"}
```

The output of the example above gives you the following information:

- The `executeRequest` function was successfully called in the `FunctionsConsumer` contract. The transaction in this example is [0xb219bced3f395a0320155369006406da1f2c894b21ee59a80ee952898449889b](https://mumbai.polygonscan.com/tx/0xb219bced3f395a0320155369006406da1f2c894b21ee59a80ee952898449889b).
- The request ID is `0x71e7e87772e5f60e41b945d7fad2c12affa65dba469cd164d27c32cab30806b7`.
- The DON successfully fulfilled your request. The total cost was: `0.00006213736353407 LINK`.
- The consumer contract received a response in `bytes` with a value of `0x7b227072696365223a22313630382e3731222c22766f6c756d65223a223636353030342e3930222c226c6173744d61726b6574223a22436f696e62617365227d`. Decoding it off-chain to `string` gives you the following result: `{"price":"1608.71","volume":"665004.90","lastMarket":"Coinbase"}`.

At any time, you can run the `functions-read` task with the `contract` parameter to read the latest received response.

```bash
npx hardhat functions-read  --contract REPLACE_CONSUMER_CONTRACT_ADDRESS --network REPLACE_NETWORK
```

Example:

```bash
$ npx hardhat functions-read  --contract 0xa9b286E892d579dc727c79D3be9b01949796240A --network mumbai
secp256k1 unavailable, reverting to browser version
Reading data from Functions client contract 0xa9b286E892d579dc727c79D3be9b01949796240A on network mumbai

On-chain response represented as a hex string: 0x7b227072696365223a22313630382e3731222c22766f6c756d65223a223636353030342e3930222c226c6173744d61726b6574223a22436f696e62617365227d
Decoded as a string: {"price":"1608.71","volume":"665004.90","lastMarket":"Coinbase"}
```

Decoding `0x7b227072696365223a22313630382e3731222c22766f6c756d65223a223636353030342e3930222c226c6173744d61726b6574223a22436f696e62617365227d` from `bytes` to `string` gives you `{"price":"1608.71","volume":"665004.90","lastMarket":"Coinbase"}`. Off-chain, you can use [JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse) to convert the JSON string to a JSON object.

## Explanation

### FunctionsConsumer.sol

<ChainlinkFunctions section="functions-consumer" />

### Functions-request-config.js

Read the [Request Configuration](https://github.com/smartcontractkit/functions-hardhat-starter-kit#functions-library) section for a detailed description of each setting. This example uses the following settings:

- `codeLocation: Location.Inline`: The JavaScript code is provided within the request.
- `codeLanguage: CodeLanguage.JavaScript`: The source code is developed in the JavaScript language.
- `source: fs.readFileSync("./Functions-request-source.js").toString()`: The source code must be a script object. This example uses `fs.readFileSync` to read `Functions-request-source.js` and calls `toString()` to get the content as a `string` object.
- `args: ["ETH", "USD"]`: These arguments are passed to the source code. This example requests the `ETH/USD` price and daily volume.
- `expectedReturnType: ReturnType.string`: The response received by the DON is encoded in `bytes`. Because the price, daily volume and market are put in a JSON string, define `ReturnType.string` to inform users how to decode the response received by the DON. Read [source code explanation](#functions-request-sourcejs) for more information.

### Functions-request-source.js

You can check the expected API response. Directly paste the following URL in your browser `https://min-api.cryptocompare.com/data/pricemultifull?fsyms=ETH&tsyms=USD` or run a `curl` command in your terminal:

```bash
curl -X 'GET' \
  'https://min-api.cryptocompare.com/data/pricemultifull?fsyms=ETH&tsyms=USD' \
  -H 'accept: application/json'
```

The response should be similar to the following example:

<!-- prettier-ignore -->
```json
{
  "RAW": {
    "ETH": {
      "USD": {
        "TYPE": "5",
        "MARKET": "CCCAGG",
        "FROMSYMBOL": "ETH",
        "TOSYMBOL": "USD",
        "FLAGS": "2049",
        "PRICE": 2867.04,
        "LASTUPDATE": 1650896942,
        "MEDIAN": 2866.2,
        "LASTVOLUME": 0.16533939,
        "LASTVOLUMETO": 474.375243849,
        "LASTTRADEID": "1072154517",
        "VOLUMEDAY": 195241.78281014622,
        "VOLUMEDAYTO": 556240560.4621655,
        "VOLUME24HOUR": 236248.94641103,
        ...
}
```

The price is located at `RAW,ETH,USD,PRICE`, the volume is at `RAW,ETH,USD,VOLUME24HOUR`, and the market is at `RAW,ETH,USD,LASTMARKET`.

Read the [JavaScript code](https://github.com/smartcontractkit/functions-hardhat-starter-kit#javascript-code) section for a detailed explanation of how to write compatible JavaScript source code. This JavaScript source code uses [Functions.makeHttpRequest](https://github.com/smartcontractkit/functions-hardhat-starter-kit#functions-library) to make HTTP requests. To request the `ETH/USD` price, the source code calls this URL: `https://min-api.cryptocompare.com/data/pricemultifull?fsyms=ETH&tsyms=USD` If you read the documentation for [Functions.makeHttpRequest](https://github.com/smartcontractkit/functions-hardhat-starter-kit#functions-library), you see you must provide the following parameters:

- `url`: `https://min-api.cryptocompare.com/data/pricemultifull`
- `params`: The query parameters object:

  ```
  {
    fsyms: fromSymbol,
    tsyms: toSymbol
  }
  ```

Note `fromSymbol` and `toSymbol` are fetched from `args` (see [request config](#functions-request-configjs)).

The code is self-explanatory and has comments to help you understand all the steps. The main steps are:

- Construct the HTTP object `cryptoCompareRequest` using `Functions.makeHttpRequest`.
- Run the HTTP request.
- Read the asset price, daily volume, and market from the response.
- Construct a JSON object.

  ```javascript
  const result = {
    price: price.toFixed(2),
    volume: volume.toFixed(2),
    lastMarket,
  }
  ```

- Convert the JSON object to a JSON string using `JSON.stringify(result)`. This step is mandatory before encoding `string` to `bytes`.
- Return the result as a [buffer](https://nodejs.org/api/buffer.html#buffer) using the `Functions.string` helper function. **Note**: Read this [article](https://www.freecodecamp.org/news/do-you-want-a-better-understanding-of-buffer-in-node-js-check-this-out-2e29de2968e8/) if you are new to Javascript Buffers and want to understand why they are important.