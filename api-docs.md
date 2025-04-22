# KryptoExpress API Documentation

* [1.Enums](#1-enums)
    + [1.1 Fiat currency Enum. ](#11-fiat-currency-enum)
    + [1.2 Cryptocurrency Enum.](#12-cryptocurrency-enum)
    + [1.3 Payment Type Enum.](#13-payment-type-enum)
    + [1.4 Withdraw Type Enum.](#14-withdraw-type-enum)
* [2. Payment](#2-payment)
    + [2.1 Create Payment](#21-create-payment)
    + [2.2 Get Payment](#22-get-payment)
* [3. Wallet](#3-wallet)
    + [3.1 Wallet](#31-wallet)
    + [3.2 Withdrawal](#32-withdrawal)
* [4.0 Callbacks](#4-callbacks)
    + [4.1 Callback attempts](#41-callback-attempts)
    + [4.2 Callback HMAC security](#42-hmac-security)

### Base URL

https://KryptoExpress.pro/api

### Request Headers

| Header       | Value              | Required |
|--------------|--------------------|----------|
| Content-Type | `application/json` | Yes      |
| X-Api-Key    | `YOUR_API_KEY`	    | Yes      |

## 1. Enums

### 1.1 Fiat Currency ENUM

| Currency value |
|----------------|
| USD            |
| EUR            |
| GBP            |
| JPY            |

### 1.2 Cryptocurrency ENUM

| Cryptocurrency value |
|----------------------|
| BTC                  |
| LTC                  |
| ETH                  |
| BNB                  |
| SOL                  |

### 1.3 Payment Type ENUM

Currently, the service supports two payment types:

1. **DEPOSIT**
    - **No `fiatAmount` Required**: The system accepts any cryptocurrency transaction to the provided payment address.
    - Process:
        - Only the **first transaction** is counted.
        - The received cryptocurrency amount is automatically converted to its fiat equivalent.
        - The fiat value is sent to the specified `callbackUrl` for further processing.

2. **PAYMENT**
    - **Required Field**: `fiatAmount` must be specified.
    - Process:
        - The provided fiat amount is converted into its cryptocurrency equivalent.
        - The user must send this exact crypto amount for the payment to be completed.
    - **Partial Payments**: The payment can be fulfilled via multiple transactions (aggregated until the full amount is
      reached).

### 1.4 Withdraw Type ENUM

| Withdraw Type value |
|---------------------|
| ALL                 |
| SINGLE              |

## 2. Payment

### 2.1 Create payment

**Request URL**: `/payment`

**Request Method**: `POST`

**Request Headers**:

- `Content-Type`: `application/json`
- `X-Api-Key`: YOUR_API_KEY

**Request Body for DEPOSIT**:

```
{
    "fiatCurrency":"USD", // str | Fiat Currency ENUM
    "paymentType":"DEPOSIT", // str | Payment Type ENUM 
    "cryptoCurrency":"BTC", // str | Cryptocurrency ENUM
    "callbackUrl":"https://httpbin.org/post", // Required | str 
    "callbackSecret":"1234567890" // Optional | str 
}
```

**Request Body for PAYMENT**:

```
{
  "fiatCurrency": "USD", // str | Fiat Currency ENUM
  "paymentType": "PAYMENT", // str | Payment Type ENUM
  "fiatAmount": 100, // int | float | required for PAYMENT paymentType
  "cryptoCurrency": "BTC", // str | Cryptocurrency ENUM
  "callbackUrl": "https://httpbin.org/post", // Required | str 
  "callbackSecret": "1234567890" // Optional | str 
}
```

**Response Example (PAYMENT)**:

```json
{
  "id": 634,
  "paymentType": "PAYMENT",
  "fiatCurrency": "USD",
  "fiatAmount": 1.0,
  "cryptoAmount": 0.000012,
  "userId": "7afca8a2-9262-4844-be82-107d7d3bcd72",
  "cryptoCurrency": "BTC",
  "expireDatetime": 1745241933410,
  "createDatetime": 1745234733410,
  "address": "bc1q6guwyuy6h0astkys3u7auput0wzcpxtad4dgwg",
  "isPaid": false,
  "isWithdrawn": false,
  "hash": "079fbfc747c726204154b102412a1a17fdb399c20d9acc18a2760c2984d26a67",
  "callbackUrl": "https://google.com"
}
```

**Response Example (DEPOSIT)**:

```json
{
  "id": 635,
  "paymentType": "DEPOSIT",
  "fiatCurrency": "USD",
  "userId": "7afca8a2-9262-4844-be82-107d7d3bcd72",
  "cryptoCurrency": "BTC",
  "expireDatetime": 1745242017757,
  "createDatetime": 1745234817757,
  "address": "bc1qg000j838y9q2shxp5adykzv8qzc445zqyrdcwd",
  "isPaid": false,
  "isWithdrawn": false,
  "hash": "cff9214406d728e35cd09104eb9c50716a183ccb54b610961ffe456400e9121e",
  "callbackUrl": "https://google.com"
}
```

### 2.2 Get payment

> Note  
> Authorization is not required for this endpoint.

**Request URL**: `/payment`

**Request Method**: `GET`

**Request Params**: `hash | str`

**Example**: `https://KryptoExpress.pro/api/payment?hash=079fbfc747c726204154b102412a1a17fdb399c20d9acc18a2760c2984d26a67`

**Response Example**:

```json
{
  "id": 634,
  "paymentType": "PAYMENT",
  "fiatCurrency": "USD",
  "fiatAmount": 1.0,
  "cryptoAmount": 0.000012,
  "userId": "7afca8a2-9262-4844-be82-107d7d3bcd72",
  "cryptoCurrency": "BTC",
  "expireDatetime": 1745241933410,
  "createDatetime": 1745234733410,
  "address": "bc1q6guwyuy6h0astkys3u7auput0wzcpxtad4dgwg",
  "isPaid": false,
  "isWithdrawn": false,
  "hash": "079fbfc747c726204154b102412a1a17fdb399c20d9acc18a2760c2984d26a67",
  "callbackUrl": "https://google.com"
}
```

## 3. Wallet

### 3.1 Wallet

**Request URL**: `/wallet`

**Request Method**: `GET`

**Response Example**:

```json
{
  "BTC": 0.1,
  "LTC": 0.2,
  "SOL": 0.3,
  "ETH": 0,
  "NB": 0
}
```

### 3.2 Withdrawal

### Withdrawal Types

The cryptocurrency withdrawal endpoint accepts two possible values for `withdrawType`:

1. **ALL**

- Withdraws all successful payments to your specified address
    - No additional parameters required

2. **SINGLE**

- Withdraws a specific payment to your address
    - **Required Parameter**:
        - `paymentId` (The identifier of the specific payment to withdraw)

### Additional Parameters

* `onlyCalculate` (Boolean)

- `false` :
    - Performs the actual withdrawal transaction
- `true`:
    - Performs a dry-run calculation without executing the withdrawal
    - Returns:
        - Service fee (KryptoExpress commission)
        - Blockchain network fee
        - Final receivable amount
    - **Use Case**: Useful for estimating costs, especially when blockchain fees might be unexpectedly high

**Request URL**: `/wallet/withdrawal`

**Request Method**: `POST`

**Request Body Example (ALL)**:

```json
{
  "withdrawType": "ALL", // str | WithdrawTypeEnum
  "cryptoCurrency": "LTC", // str | CryptocurrencyEnum
  "toAddress": "ltc1qh7zzth73czm9mhw3a6d6uss6slyywlzy1cxlnu", // str
  "onlyCalculate": false // bool 
}
```

**Request Body Example (SINGLE)**:

```json
{
  "withdrawType": "SINGLE", // str | WithdrawTypeEnum
  "cryptoCurrency": "LTC", // str | CryptocurrencyEnum
  "paymentId": "123", // int | REQUIRED for SINGLE withdrawType
  "toAddress": "ltc1qh7zzth73czk9mhw3a6d6uss6slyywlzy1cxlnu", // str
  "onlyCalculate": false // bool 
}
```

**Response Example**:

```json
{
  "id": 250, // int
  "withdrawType": "ALL", // WithdrawTypeEnum
  "cryptoCurrency": "LTC", // CryptocurrencyEnum
  "toAddress": "ltc1qh7zzth73czm9mhw3a6d6uss6slyywlzy1cxlnu", // str
  "txIdList": [
    "10f257168228ca26cf98aea7b3c2f097b9bd47d2c3a7a87c3672d7f6da490249" // str
  ],
  "receivingAmount": 0.68907986, // decimal
  "blockchainFeeAmount": 0.0000049, // decimal
  "serviceFeeAmount": 0.0, // decimal
  "onlyCalculate": false, // bool
  "totalWithdrawalAmount": 0.68908476, // decimal
  "createDatetime": 1745237332414 // long
}
```

### 4. Callbacks

#### 4.1 Callback attempts

The **KryptoExpress** service uses callback to notify you of payment events, successful and unsuccessful.
A total of 5 callbacks can be sent with this delay.

| Callback Attempt | Seconds delay |
|------------------|---------------|
| 1                | `0`           |
| 2                | `120`	        |
| 3                | `240`	        |
| 4                | `480`	        |
| 5                | `960`	        |

#### 4.2 HMAC Security

When creating a payment request, if you provide a `callbackSecret` parameter, the service will sign all callback requests using HMAC-SHA512 for verification purposes.

#### Verification Process:
1. **Signature Header**:  
   The generated signature will be included in the `X-Signature` header of the callback request.

2. **Signed Content**:  
   The HMAC signature is computed using:
    - Your provided `callbackSecret` as the key
    - The raw JSON body (without beatify and indentation) of the callback as the message

3. **Verification Recommendation**:  
   Always verify the signature by:
    - Computing HMAC-SHA512 using your `callbackSecret` and the received raw request body
    - Comparing your computed signature with the `X-Signature` header value

#### Example Verification (Python):
```python
import hmac
import hashlib
import json

def verify_signature(json_str, secret, signature):
    compact_json = json.dumps(json.loads(json_str), separators=(',', ':'))
    expected = hmac.new(
        secret.encode(),
        compact_json.encode(),
        hashlib.sha512
    ).hexdigest()
    return hmac.compare_digest(expected, signature)


json_str = """{
  "id": 638,
  "paymentType": "DEPOSIT",
  "fiatCurrency": "USD",
  "fiatAmount": 16.31,
  "cryptoAmount": 0.00018486,
  "userId": "fa92a3e0-48c0-44db-9d59-213ca94a14c3",
  "cryptoCurrency": "BTC",
  "expireDatetime": 1745252847543,
  "createDatetime": 1745245647543,
  "address": "bc1qxquy5w247rw0natp4ts4nz9agm4s9z24p0tl7u",
  "isPaid": true,
  "isWithdrawn": false,
  "hash": "adaeca287275d5589623a397fb4e35fac3aad7ba2ba9f7111e6df809df1e7474",
  "callbackUrl": "https://google.com/event"
}"""
secret = "1234567890"
header = "d5633ea456a3fd3e47798f7effe0082ad9d98e667a6c159e1e5eafa06e72e3ddaa0b847859221a4f6fc86b6af322da2ff3869db32cc545f1f91cc300ae3b05db"

is_valid = verify_signature(json_str, secret, header)
print(is_valid)
## True
```
