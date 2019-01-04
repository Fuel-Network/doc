---
title: Shipl API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>

search: true
---

# Introduction

Welcome to the Shipl API! You can use our API to process, to verify users, create users proxy-contract, process meta-tx and pay ethereum gas cost on behalf of the users of your dApp.

The Shipl API is organized around REST. Our API has predictable, resource-oriented URLs, and uses HTTP response codes to indicate API errors. We use built-in HTTP features, like HTTP authentication and HTTP verbs, which are understood by off-the-shelf HTTP clients. JSON is returned by all API responses, including errors. 
 
To explore the API as much as possible, we offer testnets environment on rinkeby, kovan and ropsten in addition to the mainnet (mainnet not available yet). Any transfers made in the test environment will never hit the mainnet network or incur any fees. Both testnets and mainnet rely on API keys for authentication.

Be sure to subscribe to Shipl's API announce mailing list to receive information on new additions and changes to Shipl's API and language libraries.

<aside class="notice">
The API is in alpha stage, so features like developer accounts or users management are not yet available but should come before the end of January 2019.
</aside>

# Authentication

> Make sure to replace `AUTH_TOKEN` with your JWT auth token.

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: Bearer AUTH_TOKEN"
```


```javascript
const Shipl = require('fuel-web3-provider');
const readline = require('readline-sync') // Callback to ask for the user phone number if the user isn't verified

const shipl = new Shipl({ privateKey, network: 'rinkeby' })
const { identity, deviceKey } = await shipl.login(readline.question)
```

Shipl uses JWT token to allow access to the API. Shipl expects to auth token to be included in all secured API requests to the server in a header that looks like the following:

`Authorization: Bearer AUTH_TOKEN `


The JWT token is generated from the following control flow:

1. The user submits their phone number and recieves a text with a secret

2. Upon successful completion, a JWT token is created and then signed by the nisaba service (becoming a 'nisaba token' as referenced in the comments in the code

3. This token is associated with the user and allows them to use the methods of the API and make calls


The login method of the Shipl SDK handle the creation/verifcation of the user and the usage of the auth token. You just have to pass a callback function to ask for the phone number and then the code verifcation.

# Errors

Shipl uses conventional HTTP response codes to indicate the success or failure of an API request. In general: Codes in the 2xx range indicate success. Codes in the 4xx range indicate an error that failed given the information provided (e.g., a required parameter was omitted, a charge failed, etc.). Codes in the 5xx range indicate an error with Shipl's servers (these are rare).

Some 4xx errors that could be handled programmatically include an error code that briefly explains the error reported.


Error Code | Meaning
---------- | -------
200 | OK -- Everything worked as expected.
400 | Bad Request -- The request was unacceptable, often due to missing a required parameter or not JSON
401 | Unauthorized -- No nisaba token
403 | Forbidden -- Nisaba token missing or invalid
404 | Not Found -- The specified path could not be found.
405 | Method Not Allowed -- You tried to access an endpoint with an invalid method.
429 | Too Many Requests -- You're requesting too many transactions! Slow down!
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.


# General

## Request phone verification

Starts a verification for a deviceKey and a phoneNumber. Sends a code thru SMS or Call

### HTTP Request

`POST https://api.shipl.co/nisaba/verify`

### Body Parameters

Parameter | Description
--------- | ------- 
deviceKey | The ethereum address of the user to verify
phoneNumber | The phone number of the user to verify

```shell
curl -X POST  https://api.shipl.co/nisaba/verify \
-d deviceKey = <device key> \
-d phoneNumber = <phone number>
```

```javascript
const Shipl = require('fuel-web3-provider');
const readline = require('readline-sync') // Callback to ask for the user phone number if the user isn't verified

const shipl = new Shipl({ privateKey, network: 'rinkeby' })
const { identity, deviceKey } = await shipl.login(readline.question)
```

> The above command returns JSON structured like this:

```json
{
  "status": "success"
}
```

## Verify and Request Token

With the code (which was sent thru SMS) the app can verify it and request the pseudo-attestation token

### HTTP Request

`POST https://api.shipl.co/nisaba/check`

### Body Parameters

Parameter | Description
--------- | ------- 
deviceKey | The ethereum address of the user to verify
code | The code number received by the user at the previous step

```shell
curl -X POST  https://api.shipl.co/nisaba/check \
-d deviceKey = <device key> \
-d code = <verifcation code>
```

```javascript
const Shipl = require('fuel-web3-provider');
const readline = require('readline-sync') // Callback to ask for the user phone number if the user isn't verified

const shipl = new Shipl({ privateKey, network: 'rinkeby' })
const { identity, deviceKey } = await shipl.login(readline.question)
```

> The above command returns JSON structured like this:

```json
{
  "status": "success",
  "data": <jwt>
}
```

## Create proxy contract for you users

Calls the MetaIdentityManager contract to create the initial controller and proxy contract. The proxy contract is configured to be controlled by the deviceKey. The identity factory is called in the blockchain specified by blockchain body data. If Shipl does not know how to access the blockchain it returns a 404 status.

The endpoint is private, only valid "nisaba tokens" are allowed.


### HTTP Request

`POST https://api.shipl.co/unnu/createidentity`

### Header

`Authorization: Bearer <jwt nisaba token>`

### Body Parameters

Parameter | Description
--------- | ------- 
deviceKey | The ethereum address of the user to verify
recoveryKey | By default it's a Shipl key
blockchain | Rinkeby, ropsten or kovan
managerType | MetaIdentityManager

```shell
curl -X POST  https://api.shipl.co/unnu/createidentity \
-H "Authorization: Bearer AUTH_TOKEN"\
-d deviceKey = <device key> \
-d recoveryKey = <recovery key> \
-d blockchain = <blockchain name> \
-d managerType = "MetaIdentityManager"
```

```javascript
const Shipl = require('fuel-web3-provider');
const readline = require('readline-sync') // Callback to ask for the user phone number if the user isn't verified

const shipl = new Shipl({ privateKey, network: 'rinkeby' })
const { identity, deviceKey } = await shipl.login(readline.question)
```

> The above command returns JSON structured like this:

```json
{
  "managerType": "MetaIdentityManager",
  "managerAddress": <address>,
  "txHash": <tx hash>
}

```


## Lookup for user proxy contracts

Look for a proxy contract created by a deviceKey.

### HTTP Request

`POST https://api.shipl.co/unnu/lookup`

### Header

`Authorization: Bearer <jwt nisaba token>`

### Body Parameters

Parameter | Description
--------- | ------- 
deviceKey | The ethereum address of the user to verify

```shell
curl -X POST  https://api.shipl.co/unnu/lookup \
-H "Authorization: Bearer AUTH_TOKEN"\
-d deviceKey = <device key>
```

```javascript
const Shipl = require('fuel-web3-provider');
const readline = require('readline-sync') // Callback to ask for the user phone number if the user isn't verified

const shipl = new Shipl({ privateKey, network: 'rinkeby' })
const { identity, deviceKey } = await shipl.login(readline.question)
```

> The above command returns JSON structured like this:

```json
{
  "managerType": "MetaIdentityManager",
  "managerAddress": <address>,
  "identity": <identity address>,
  "blockchain": <blockchain name>
}
```

## Relay meta-transaction

### HTTP Request

`POST https://api.shipl.co/sensui/relay`

#### Header
```
Authorization: Bearer <jwt nisaba token>
```

#### Body Parameters

Parameter | Description
--------- | ------- 
metaSignedTx | The meta signed tx of the user
blockchain | Rinkeby, Ropsten or Kovan

```shell
curl -X POST  https://api.shipl.co/sensui/relay \
-H "Authorization: Bearer AUTH_TOKEN"\
-d metaSignedTx = <signed meta-transaction>
-d blockchain = <blockchain name>
```

```javascript
const Web3 = require('web3')
const Shipl = require('fuel-web3-provider');
const readline = require('readline-sync') // Callback to ask for the user phone number if the user isn't verified

const shipl = new Shipl({ privateKey, network: 'rinkeby' })
const { identity, deviceKey } = await shipl.login(readline.question)
const web3 = new Web3(shipl.start())

const dappContract = new web3.eth.Contract(DAPP_CONTRACT_ADDRESS_ABI, DAPP_CONTRACT_ADDRESS)
const txHash = await dappContract.register(10, { from: deviceKey })

dappContract.methods.register(RANDOM_NUMBER).send({ from: deviceKey })
  .on('transactionHash', async (transactionHash) => {
    await shipl.getInternalTransactionsData(abi, txHash)
  })
```

> The above command returns JSON structured like this:

```json
{
  "txHash": <tx hash>,
}
```

# Shipl SDK

## Getting Started

The Shipl SDK is an easy to embed module aimed to handle on-boarding for you and provide feesless transactions on the ethereum network. We’ve tried to keep things as simple/lightweight as possible.

This document is an implementation example for developers to understand end-user’s onboarding flow through the Shipl SDK.

## Parties involved

Shipl - Shipl is a service provider to you, our partner. We verify Users off-chain through a phone number verifcation. We serve meta-tx to the Users of the Business. We do not disclose their personal information unless in the event of a regulatory intervention as requested (e.g. subpoena, etc...).

Business - DEX, Dapps, etc. Likely this is you. You have users, some of which are verified via Shipl, some of which are not.

Users - DEX, Dapp end users. They are the ones getting verified and getting the experience from you. Shipl's part in this is purely to onboard them, and facilitate the ethereum transaction processing.

## Installation

You can install the library via `npm`:

```sh
> npm i shipl
```

## Usage

The package need to be configured with an ethereum private key or a web3 wallet instance (eg. Metamask)

```javascript
import Shipl from 'shipl';

const shipl = new Shipl({
    privateKey: "YOUR_PRIVATE_KEY", // You can provide a private key to shipl
    web3Provider: window.web3, // Or a web3 compatilbe wallet like Metamask
    network: 'rinkeby' // Can be rinkeby, ropsten or kovan (mainnet not available yet)
})
```

Then login into shipl. You have to pass an input callback to make the shipl SDK capable to ask for the phone number and then for the code verifcation send by SMS. For example on node you can use readline-sync. 

```javascript
const { identity, deviceKey } = await shipl.login(readline.question)
```

Then pass you can pass the shipl sdk into any web3 compatible library. Don't forget to execute the start function to launch the web3 provider.

```javascript
const web3 = new Web3(shipl.start());
```
Then you can call a contract in the regular web3 way

```javascript
const targetContract = new web3.eth.Contract(abi, contractAddress);

targetContract.methods.register('0x' + config.address, 1)
  .send({
    from: '0x' + config.address
  })
  .on('transactionHash', transactionHash => {
    console.log('This the transactionHash', transactionHash);
  });
```

Finnaly you can get the internal transaction data by passing the txHash and contract abi to the method below.

```javascript
const internalTxDatas = await shipl.getInternalTransactionsData(abi, txHash)
```

### Browser Window Quick Start

For use directly in the browser you can reference the shipl distribution files from a number of places. They can be found in our npm package in the 'dist' folder or you can build them locally from this repo.

For a quick setup you may also request a remote copy from unpkg CDN as follows:

```html
<!-- The most recent version  -->
<script src="https://unpkg.com/shipl/dist/shipl.js"></script>
<!-- The most recent minified version  -->
<script src="https://unpkg.com/shipl/dist/shipl.min.js"></script>
<!-- You can also fetch specific versions by specifying the version, files names may differ for past versions -->
<script src="https://unpkg.com/shipl@<version>/dist/shipl.js"></script>
```

To see all available dist files on unpkg, vist [unpkg.com/shipl/dist](https://unpkg.com/shipl/dist)

Then to instantiate the shipl object from the browser window object:

```javascript
const Shipl = window.shipl
const shipl = new Shipl({ privateKey, network })
```

## Examples

For a more in depth guide, check out our documentation site or clone this repository and check out the sample apps in the [/examples](https://github.com/shiplco/shipl-sdk/tree/master/examples/node-integration-tutorial) folder.
