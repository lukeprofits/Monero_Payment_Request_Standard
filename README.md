# Monero Payment Request Standard
> Formerly the [Monero Subscription Code Standard](https://github.com/lukeprofits/Monero_Subscription_Code_Standard)

# Version 1:
## Decoding & Encoding Monero Payment Requests 
This document explains how to decode and encode `monero-request:` payment requests.

## Decoding A Monero Payment Request
To decode a Monero Payment Request, follow these steps:

1. Remove the Monero Payment Request identifier: `monero-request:`
2. Remove the version identifier `1:`
3. Decode the string from Base64 to obtain the compressed data.
4. Decompress the compressed data using gzip to get the JSON string.
5. Parse the JSON string to extract the field values.

## Example Python Function To Decode A Monero Payment Request
```
import base64
import gzip
import json

def decode_monero_payment_request(monero_payment_request):
    code_parts = monero_payment_request.split('monero-request:')
    if len(code_parts) == 2:
        monero_request_data = code_parts[1]
    else:
        monero_request_data = code_parts[0]
    encoded_str = monero_request_data
    compressed_data = base64.b64decode(encoded_str.encode('ascii'))
    json_bytes = gzip.decompress(compressed_data)
    json_str = json_bytes.decode('utf-8')
    request_data_as_json = json.loads(json_str)
    return request_data_as_json
```

## Encoding A Monero Payment Request
To encode a Monero Payment Request, follow these steps:

1. Convert the payment details to a JSON object.
2. Compress the JSON object using gzip compression.
3. Encode the compressed data in Base64 format.
4. Add the Monero Payment Request identifier `monero-request:` and version number `1:` to the encoded string.

## Example Python Function To Encode A Monero Payment Request
```
import base64
import gzip
import json

def make_monero_payment_request(json_data, version=1):
    json_str = json.dumps(json_data, separators=(',', ':'), sort_keys=True)
    compressed_data = gzip.compress(json_str.encode('utf-8'), mtime=0)
    encoded_str = base64.b64encode(compressed_data).decode('ascii')
    monero_payment_request = 'monero-request:' + f'{str(version)}:' + encoded_str
    return monero_payment_request
```

## Using the Optional `change_indicator_url`
The `change_indicator_url` is an optional field designed for merchants who wish to have the flexibility to request modifications to an existing payment request. This could be a one-time payment, a payment with a set number, or a recurring subscription. **It's important to note that the merchant cannot enforce these changes unilaterally.** When a change is requested, all related automatic payments are paused until the customer reviews and either confirms or rejects the changes.

#### Key Features and Constraints

- **Merchant Requests, Not Commands**: The main utility of this feature is for merchants to request changes, such as updating a wallet address or changing the price. The merchant cannot force these changes on the customer.
  
- **Automatic Pause on Changes**: The wallet will query the `change_indicator_url` before any scheduled payment. If it detects a change, automatic payments are paused and the customer is notified.

- **Customer Consent Required**: Payments remain paused until the customer actively confirms or rejects the proposed changes. If confirmed, payments resume; if rejected, the payment schedule is canceled.

#### How it Works

1. **URL Formation**: The `change_indicator_url` is constructed by appending the unique `payment_id` to a base URL specified by the merchant.
    - Example: `www.mysite.com/api/monero-request?payment_id=9fc88080d1d5dc09`

2. **Merchant Changes**: To request a change, the merchant updates the information at the `change_indicator_url`. These changes remain in the status of "requested" until approved or declined by the customer.

3. **Customer Notification and Confirmation**: Upon detecting a change, the wallet notifies the customer who then must make a decision to accept or decline. Payments stay paused until this decision is made.

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

- **To Update Payment Fields**
    ```json
    {
        "action": "update",
        "fields": {
            "amount": 25.99,
            "currency": "XMR"
        },
        "note": "Price has changed due to increased costs."
    }
    ```

#### Bulk Updates

Merchants can ignore the `payment_id` query parameter to initiate blanket updates for all payment_ids associated with the `change_indicator_url`.

This optional `change_indicator_url` feature enhances the protocol's flexibility, enabling merchants to request changes while ensuring customers maintain full control over their payment options.


# Tools For Creating `monero-request` codes

- [Monero Payment Request Code Creator Website](https://monerosub.tux.pizza/)
- [Monero Payment Request Code Creator CLI Tool](https://github.com/lukeprofits/Monero_Payment_Request_Code_Creator)
- [Monero Payment Request Code Creator Pip Package](https://github.com/lukeprofits/monero_payment_request)
- More `monero-request` integration tools coming soon...
