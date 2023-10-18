# Monero Payment Request Standard
> Formerly the [Monero Subscription Code Standard](https://github.com/lukeprofits/Monero_Subscription_Code_Standard)

# Tools For Creating `monero-request` codes (these will be updated soon)

- [Monero Payment Request Code Creator Website](https://monerosub.tux.pizza/)
- [Monero Payment Request Code Creator CLI Tool](https://github.com/lukeprofits/Monero_Payment_Request_Code_Creator)
- [Monero Payment Request Code Creator Pip Package](https://github.com/lukeprofits/monero_payment_request)
- More `monero-request` integration tools coming soon...

# Current Version: [Version 1](https://github.com/lukeprofits/Monero_Payment_Request_Standard/blob/main/versions/v1.md)
## Decoding & Encoding Monero Payment Requests 
This document explains how to decode and encode `monero-request:` payment requests using gzip compression and Base64 encoding.

# Decoding A Monero Payment Request
To decode a Monero Payment Request, follow these steps:

1. Remove the Monero Payment Request identifier: `monero-request:` and version identifier `1:`.
2. Decode the string from Base64 to obtain the compressed data.
3. Decompress the compressed data using gzip to get the JSON string.
4. Parse the JSON string to extract the field values.

## Example Python Function To Decode A Monero Payment Request:
```
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


monero_payment_request = 'monero-request:1:H4sIAAAAAAAC/y2P207DMAyGXwXleocex9q7bmxIoCGxDRi7idLEWyNyKDnQtYh3J0Nc2f79+7P9jYjUXjlUxsWkKEaINkSdAXPFOCVOG+yNQCXqum4CFyJbAROq5ZS0fCq1AqPHBj49WIfCrDcGFO2D/2V39ydYpyUWpIYrZNPf7HxtqeGt41oFAyO9xS0YXHMhuDpj2lMBqEyjEVJe1qGjT7glvQTlLCqD/F9gzgKxONH5PJpHLGY5o1ERkBaEAGNxR0IMf6GscukhN1+vfbvXp7P08FTY4tmZgW0hX3hYG/tRHXl8u9DvdTP0Vg+D3qwXs+FN7R/Z/XJWXVZVvVrldFhv0yZkD7WVWbOEQ7K7rnTEOMyIC5ejJErScZSNk9k+TsssL9P0iH5+AS8IcFpoAQAA'

decoded_data = decode_monero_payment_request(monero_payment_request)

print(decoded_data)
```


# Encoding A Monero Payment Request
To encode a Monero Payment Request, follow these steps:

1. Convert the payment details to a JSON object. Minimize whitespace and sort the keys.
   - Python: `json.dumps(data, separators=(',', ':'), sort_keys=True)`
   - JavaScript: `JSON.stringify(data, Object.keys(data).sort())`
   
2. Compress the JSON object using gzip compression. Set the modification time to a constant value to ensure consistency.
   - Python: `gzip.compress(data, mtime=0)`
   - JavaScript: `pako.gzip(data, {mtime: 0})`  *(using pako library)*

3. Encode the compressed data in Base64 format.
   - Python: `base64.b64encode(data).decode('ascii')`
   - JavaScript: `btoa(String.fromCharCode.apply(null, new Uint8Array(data)))`

4. Add the Monero Payment Request identifier `monero-request:` and version number `1:` to the encoded string.
   - Python/JavaScript: `'monero-request:1:' + encodedString`


## Example Python Function To Create A Monero Payment Request:
```
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
    "days_per_billing_cycle": 30,  # How often it should be billed
    "number_of_payments": 0,  # 1 for one-time, >1 for set-number, 0 for recurring until canceled
    "change_indicator_url": "www.example.com/api/monero-request",  # Optional. Small merchants should leave this blank.
}

monero_payment_request = make_monero_payment_request(json_data=json_data)

print(monero_payment_request)
```


# Using the Optional `change_indicator_url`
The `change_indicator_url` is an optional field designed for merchants who wish to have the flexibility to request modifications to an existing payment request. **It's important to note that the merchant cannot enforce these changes.** When a change is requested, all related automatic payments are paused until the customer reviews and either confirms or rejects the changes.

#### Key Features and Constraints

- **Merchant Requests, Not Commands**: The main utility of this feature is for merchants to request changes, such as updating a wallet address or changing the price. The merchant cannot force these changes on the customer.
  
- **Automatic Pause on Changes**: The wallet will query the `change_indicator_url` before any scheduled payment. If it detects a change, automatic payments are paused and the customer is notified.

- **Customer Consent Required**: Payments remain paused until the customer actively confirms or rejects the proposed changes. If confirmed, the payment request is updated and payments resume; if rejected, the payment request is canceled.

#### How it Works

1. **URL Formation**: For the `change_indicator_url`, provide only the base URL of the endpoint that should be notified upon payment status changes. The system will automatically append the unique `payment_id` associated with the payment request as a query parameter to this URL.
    - Base URL you provide: `www.mysite.com/api/monero-request`
    - Final URL used by the system: `www.mysite.com/api/monero-request?payment_id=9fc88080d1d5dc09`

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
            "amount": 25.99,
            "currency": "USD"
        },
        "note": "Price has changed due to increased costs."
    }
    ```

#### Bulk Updates

Merchants can ignore the `payment_id` query parameter to initiate blanket updates for all payment_ids associated with the `change_indicator_url`.

This optional `change_indicator_url` feature enhances the protocol's flexibility, enabling merchants to request changes while ensuring customers maintain full control over their payment options.


# Old Versions: 
[Version 0](https://github.com/lukeprofits/Monero_Payment_Request_Standard/blob/main/versions/v0.md)
