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
curl "https://api.shipl.co/nisaba/verify"
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
curl "http://api.shipl.co/nisaba/check"
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
blockchain | Rinkeby, Ropsten or Kovan
managerType | MetaIdentityManager

```shell
curl "https://api.shipl.co/unnu/createidentity"
  -H Authorization: Bearer <jwt nisaba token>
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
curl "https://api.shipl.co/unnu/lookup"
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
curl "https://api.shipl.co/sensui/relay"
  -H Authorization: Bearer <jwt nisaba token>
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