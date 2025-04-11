# Monero Payment Request Standard
> Formerly known as the [Monero Subscription Code Standard](https://github.com/lukeprofits/Monero_Subscription_Code_Standard)

## Implementations

This standard has been implemented in several programming languages to facilitate its use across different platforms and applications. Here are the available libraries:

- Python: [monerorequest](https://github.com/lukeprofits/monerorequest) `pip install monerorequest`
- JavaScript: [monerorequest-js](https://github.com/moshmage/monerorequest-js) `npm install -S monerorequest-js`
- Kotlin: [monerorequest-kotlin](https://github.com/lukeprofits/monerorequest-kotlin) (coming soon to Maven Central)
- Java: [monerorequest-java](https://github.com/lukeprofits/monerorequest-java) (coming soon to Maven Central)
- Ruby: [monerorequest-ruby](https://github.com/snex/monerorequest-ruby) `gem install monerorequest`
- Rust: [monerorequest-rust](https://github.com/NotGovernor/MoneroRequest-Rust) (in progress)


## Table of Contents
1. [Introduction](#introduction)
2. [How It Works](#how-it-works)
    1. [For Merchants](#for-merchants)
    2. [For Consumers](#for-consumers)
3. [Advantages](#advantages)
4. [Mission](#mission)
5. [Tools For Creating `monero-request` codes](#tools-for-creating-monero-request-codes)
6. [Latest Version](#latest-version)
    1. [Decoding A Monero Payment Request](#decoding-a-monero-payment-request)
    2. [Encoding A Monero Payment Request](#encoding-a-monero-payment-request)
7. [Field Explanations](#field-explanations)
    1. [Custom Label](#custom-label)
    2. [Sellers Wallet](#sellers-wallet)
    3. [Currency](#currency)
    4. [Amount](#amount)
    5. [Payment ID](#payment-id)
    6. [Start Date](#start-date)
    7. [Schedule](#schedule)
    8. [Number of Payments](#number-of-payments)
    9. [Change Indicator URL](#change-indicator-url)
8. [Old Versions](#old-versions)

## Introduction
The Monero Payment Request Standard aims to address the complexities and limitations associated with traditional and existing cryptographic payment methods. It facilitates a straightforward and decentralized way to handle one-time, recurring, and scheduled Monero payments.

## How It Works
### For Merchants
1. **Generate A Code**: Create a unique payment code (called a "Payment Request") encapsulating the payment information — billing frequency, amount, etc.
2. **Modification Requests**: Merchants can request to update previously sent Payment Requests, accommodating changing business needs. The customer retains the ability to accept these changes, or remove the Payment Request.

### For Consumers
1. **Input The Code**: Insert the merchant-provided Payment Request into your Monero wallet, review the Payment Request details, and click to confirm.
2. **Retain Control**: Upon confirmation, the payment proceeds according to the agreed-upon schedule. Buyers retain full control over their funds and can cancel Payment Requests at any time.

## Advantages
- **Efficiency**: The protocol eliminates redundant steps, enabling users to establish payment conditions with a singular code.
- **No Intermediaries**: The protocol operates without the need for a middle man, enabling a direct transactional relationship between the buyer and merchant.
- **Multi-Currency Flexibility**: The protocol supports pricing in various currencies, providing flexibility for merchants whose pricing may be based on fiat currencies.
- **Privacy Preserving**: Merchants can confirm the origin of payments without collecting any information from the buyer.

## Mission
To simplify all kinds of payments on Monero, allowing buyers to retain full control of funds, automate payments, and keep transactions private.

## Tools For Creating `monero-request` codes
- [Monero Payment Request Code Creator Website](https://monerosub.tux.pizza/)
- [Monero Payment Request Code Creator CLI Tool](https://github.com/lukeprofits/Monero_Payment_Request_Code_Creator)
- [Monero Payment Request Code Creator Pip Package](https://github.com/lukeprofits/monero_payment_request)
- More `monero-request` integration tools coming soon...

## Latest Version
[Version 2](https://github.com/lukeprofits/Monero_Payment_Request_Standard/blob/main/versions/v2.md)

### Decoding A Monero Payment Request
To decode a Monero Payment Request, follow these steps:

1. Remove the Monero Payment Request identifier `monero-request:` and version identifier, and extract the encoded string.
   - Python: `prefix, version, encoded_str = monero_payment_request.split(':')`
   - JavaScript: `const [prefix, version, encoded_str] = monero_payment_request.split(':');`
2. Decode the string from [Base64](https://en.wikipedia.org/wiki/Base64) to obtain the compressed data.
   - Python: `compressed_data = base64.b64decode(encoded_str)`
   - JavaScript: `const compressed_data = atob(encoded_str)`
3. Decompress the compressed data using [gzip](https://docs.python.org/3/library/gzip.html) to get the JSON string.
   - Python: `json_str = gzip.decompress(compressed_data)`
   - JavaScript: `const json_str = pako.inflate(compressed_data, { to: 'string' })` *(using [pako library](https://github.com/nodeca/pako))*
4. Parse the JSON string to extract the field values.
   - Python: `json.loads(json_str)`
   - JavaScript: `JSON.parse(json_str)`



### Example Python Function To Decode A Monero Payment Request:
```python
import base64
import gzip
import json

def decode_monero_payment_request(monero_payment_request):
    # Extract prefix, version, and Base64-encoded data
    prefix, version, encoded_str = monero_payment_request.split(':')

    # Decode the Base64-encoded string to bytes
    encoded_str = base64.b64decode(encoded_str.encode('ascii'))

    # Decompress the bytes using gzip decompression
    decompressed_data = gzip.decompress(encoded_str)

    # Convert the decompressed bytes into to a JSON string
    json_str = decompressed_data.decode('utf-8')

    # Parse the JSON string into a Python object
    monero_payment_request_data = json.loads(json_str)

    return monero_payment_request_data


monero_payment_request = 'monero-request:2.0.0:H4sIAAAAAAACAy1P207DMAz9lSmP0y69jrVv3diQQEOCDRh7idLEWyOapORC1yL+nRTxYNk+PufY/kZEKCctysNslmUTRCsiL4C5ZJwSqzR2ukY5att2BlcimhpmVIk5afhcKAlaTTV8OjAWea3TGiTtPP9lf/sHGKsErkkJg8muG+1daajmjeVKeoJ0ogSN1Rk3pBMgrUF5MEH/DebMq7IzXS6DZcBCljIaZF5maAXM1eCn49F4FPoYDzDUNWiDW+KzfwklhY2Pqf567ZqDOl+Eg8fMZE9W9+wZ0pWDrTYfxYmHNyv1XlZ9Z1Tfq912tejf5OGB3a0XxXVTlJtNSvvtc1z56r40IqnWcIz2w0pLtMWM2OGWKIjiaZBMo8UhjPMkzeP4hH5+AZhorYxjAQAA'

decoded_data = decode_monero_payment_request(monero_payment_request)

print(decoded_data)
```

### Encoding A Monero Payment Request
To encode a Monero Payment Request, follow these steps:

1. Convert the payment details to a JSON object. Minimize whitespace and sort the keys.
   - Python: `json.dumps(data, separators=(',', ':'), sort_keys=True)`
   - JavaScript: `JSON.stringify(data, Object.keys(data).sort())`
   
2. Compress the JSON object using gzip compression. Set the modification time to a constant value to ensure consistency.
   - Python: `gzip.compress(data, mtime=0)`
   - JavaScript: `pako.gzip(data, { level: 9, windowBits: 31 })`  *(using pako library)*

3. Encode the compressed data in Base64 format.
   - Python: `base64.b64encode(data).decode('ascii')`
   - JavaScript: `btoa(String.fromCharCode.apply(null, new Uint8Array(data)))`

4. Add the Monero Payment Request identifier `monero-request:` and version number `1:` to the encoded string.
   - Python/JavaScript: `'monero-request:1:' + encodedString`


### Example Python Function To Create A Monero Payment Request:
```python
import base64
import gzip
import json


def make_monero_payment_request(json_data, version=1):
    # Convert the JSON data to a string
    json_str = json.dumps(json_data, separators=(',', ':'), sort_keys=True)

    # Compress the string using gzip compression
    compressed_data = gzip.compress(json_str.encode('utf-8'), mtime=0)

    # Encode the compressed data into a Base64-encoded string
    encoded_str = base64.b64encode(compressed_data).decode('ascii')

    # Add the Monero Subscription identifier & version number
    monero_payment_request = 'monero-request:' + str(version) + ':' + encoded_str

    return monero_payment_request


json_data = {
    "custom_label": "My Subscription",  # This can be any text
    "sellers_wallet": "4At3X5rvVypTofgmueN9s9QtrzdRe5BueFrskAZi17BoYbhzysozzoMFB6zWnTKdGC6AxEAbEE5czFR3hbEEJbsm4hCeX2S",
    "currency": "USD",  # Currently supports "USD" or "XMR"
    "amount": 19.99,
    "payment_id": "9fc88080d1d5dc09",  # Unique identifier so the merchant knows which customer the payment relates to
    "start_date": "2023-04-26T13:45:33Z",  # Start date in RFC3339 timestamp format
    "schedule": '* * 1 * *',  # How often it should be billed
    "number_of_payments": 0,  # 1 for one-time, >1 for set-number, 0 for recurring until canceled
    "change_indicator_url": "www.example.com/api/monero-request",  # Optional. Small merchants should leave this blank.
}

monero_payment_request = make_monero_payment_request(json_data=json_data)

print(monero_payment_request)
```

## Field Explanations
### Custom Label
The `custom_label` is a string field allowing users to attach a descriptive label to the payment request. This label can be any text that helps identify or categorize the payment for the user.

- **Data Type**: String
- **Details**: While there's no strict character limit, labels longer than 80 characters are likely to be truncated in most interfaces.
- **Examples**: 
    - "Monthly Subscription"
    - "Donation to XYZ"
    - "Invoice #12345" *(For one-time payments)*

### Sellers Wallet
The `sellers_wallet` field holds the Monero wallet address where payments will be sent. This address must not be a subaddress since integrated addresses (which combine a Monero address and a payment ID) don't support subaddresses.

- **Data Type**: String *(Monero wallet address format)*
- **Limitations**: Must be a valid main Monero wallet address.
- **Examples**: 
    - "4At3X5rvVypTofgmueN9s9QtrzdRe5BueFrskAZi17BoYbhzysozzoMFB6zWnTKdGC6AxEAbEE5czFR3hbEEJbsm4hCeX2S"

### Currency
All payments are made in Monero. The `currency` field is used to specify the currency in which the payment amount is denominated. This allows merchants to base their prices on a variety of currencies.

- **Data Type**: String
- **Details**: While payments are always in Monero, the amount can be specified in any recognized currency or cryptocurrency ticker (e.g., USD, AUD, GBP, BTC). The exact amount of Monero sent will be based on the current exchange rate of the specified currency to Monero.
- **Examples**: 
    - "USD"
    - "GBP"
    - "XMR"

### Amount
The `amount` field specifies the quantity of the specified currency to be transferred. The actual Monero amount sent will be based on this value and the current exchange rate.

- **Data Type**: String
- **Examples**: 
    - "19.99" *(for 19.99 USD worth of Monero — assuming "Currency" was set to USD)*
    - "0.5" *(for 0.5 XMR  — assuming "Currency" was set to XMR)*

### Payment ID
The `payment_id` is a unique identifier generated for the payment request. It is used when generating an integrated address for Monero payments. Merchants can identify which customer made a payment based on this ID, ensuring privacy for the customer.

- **Data Type**: String *(Monero payment ID format)*
- **Details**: Typically in hexadecimal format, it's recommended to generate a unique ID for each customer.
- **Examples**: 
    - "9fc88080d1d5dc09"

### Start Date
The `start_date` field indicates when the first payment or subscription should commence.

- **Data Type**: String *(truncated RFC3339 timestamp format)*
- **Python**: 
    ```python
    from datetime import datetime, timezone
    current_time = datetime.now(timezone.utc).strftime('%Y-%m-%dT%H:%M:%S.%f')[:-3] + 'Z'
    ```
- **JavaScript**:
    ```javascript
    const current_time = new Date().toISOString().slice(0, -1) + 'Z';
    ```
- **Examples**: 
    - 2023-04-26T13:45:33.123Z
    - 2023-10-26T04:55:37.443Z

### Schedule
The `schedule` field defines the frequency of payments for recurring payments using a cron-style syntax.

Minute (`*`, 0-59), Hour (`*`, 0-23), Day Of Month (`*`, 1-31), Month (`*`, 1-12), Day of Week (`*`, 0-7).

See [crontab(5)](https://man7.org/linux/man-pages/man5/crontab.5.html) for the main reference on supported syntax.
The symbol `L` is also supported for the Day of Month entry to represent the last day of the month.

- **Data Type**: String
- **Examples**:
    ```plaintext
    * * * * *    # For payments every minute
    * * * * 0    # For weekly payments
    ```

### Number of Payments
The `number_of_payments` field indicates how many times a payment will occur.

- **Data Type**: Integer
- **Examples**:
    - `1` (for a one-time payment)
    - `6` (for six scheduled payments)
    - `0` (for payments that will recur until canceled)
      
### Change Indicator URL
The `change_indicator_url` is a field designed for large merchants who wish to have the flexibility to request modifications to an existing payment request. **It's important to note that the merchant cannot enforce these changes.** When a change is requested, all related automatic payments are paused until the customer reviews and either confirms or rejects the changes (canceling the payment request).

#### Key Features and Constraints

- **Merchant Requests, Not Commands**: The main utility of this feature is for merchants to request changes, such as updating a wallet address or changing the price. The merchant cannot force these changes on the customer.
  
- **Automatic Pause on Changes**: The wallet will query the `change_indicator_url` before making any payment. If it detects a change, automatic payments are paused and the customer is notified of the requested change (customer can choose to accept the changes, or cancel the payment request).

- **Customer Consent Required**: Payments remain paused until the customer actively confirms or rejects the proposed changes. If confirmed, the payment request is updated and payments resume; if rejected, the payment request is canceled.

#### How it Works

1. **URL Formation**: For the `change_indicator_url`, provide only the base URL of the endpoint that can be checked for a Payment Request changes. The system will automatically append the unique `payment_id` associated with the payment request as a query parameter to this URL.
    - Base URL you provide: `www.mysite.com/api/monero-request`
    - Final URL the customer will query: `www.mysite.com/api/monero-request?payment_id=9fc88080d1d5dc09`

2. **Merchant Changes**: To request a change, the merchant updates the information at the `change_indicator_url`. These changes remain in the status of "requested" until approved or declined by the customer.

3. **Customer Notification and Confirmation**: Upon detecting a change, the wallet notifies the customer who then must make a decision to accept the changes to the payment request, or cancel the payment request. Payments stay paused until this decision is made.

#### Merchant Guidelines

- **Endpoint Setup**: Merchants should create a REST endpoint capable of handling GET requests for the `change_indicator_url`.
  
- **Initiating Changes**: To request changes, the merchant updates the content at the REST endpoint according to the Monero Payment Request Protocol format.

- **Cancellation Request**: If a merchant wishes to cancel the payment request entirely, they can specify this at the `change_indicator_url` (e.g., `"status": "cancelled"`).

- **Supplemental Customer Notification**: Though the `change_indicator_url` should trigger automatic notifications, merchants are encouraged to also notify customers through other channels as a best practice.

#### JSON Structure for Merchant Changes

Merchants can send JSON data with the following fields to initiate different types of changes:

- **To Cancel Subscription**
    ```json
    {
        "action": "cancel",
        "note": "We are going out of business."
    }
    ```

- **To Update Any Payment Request Fields**
    ```json
    {
        "action": "update",
        "fields": {
            "amount": "25.99",
            "currency": "USD"
        },
        "note": "Price has changed due to increased costs."
    }
    ```

#### Bulk Updates

Merchants can ignore the `payment_id` query parameter to initiate blanket updates for all payment_ids associated with the `change_indicator_url`.

This optional `change_indicator_url` feature enhances the protocol's flexibility, enabling merchants to request changes while ensuring customers maintain full control over their payment options.

- **Data Type**: String
- **Examples**:
    - "" *(for small merchants who do not want to use this feature)*
    - "https://www.example.com/api/monero-request"
    - "https://mywebsite.com/update-monero-payments"

## Old Versions
[Version 0](https://github.com/lukeprofits/Monero_Payment_Request_Standard/blob/main/versions/v0.md)
[Version 1](https://github.com/lukeprofits/Monero_Payment_Request_Standard/blob/main/versions/v1.md)
