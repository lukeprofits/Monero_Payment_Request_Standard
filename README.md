# Monero Payment Request Standard

# Version 1:

## Decoding & Encoding `monero-request:` Payment Codes

This document explains how to decode and encode `monero-request:` payment codes using gzip compression and Base64 encoding.

## Decoding Monero Payment Requests

To decode a Monero Payment Request, follow these steps:

1. Remove the Monero Payment Request identifier: `monero-request:`
2. Remove the version identifier `1:`
3. Decode the string from Base64 to obtain the compressed data.
4. Decompress the compressed data using gzip to get the JSON string.
5. Parse the JSON string to extract the field values.

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

## Encoding Monero Payment Requests

To encode a Monero Payment Request, follow these steps:

1. Convert the payment details to a JSON object.
2. Compress the JSON object using gzip compression.
3. Encode the compressed data in Base64 format.
4. Add the Monero Payment Request identifier `monero-request:` and version number to the encoded string.

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

The `change_indicator_url` is an optional field designed for merchants who wish to have the flexibility to request modifications to an existing payment request. This could be a one-time payment, a payment with a set number, or a recurring subscription.

### Key Features and Constraints
- Merchant Requests, Not Commands
- Automatic Pause on Changes
- Customer Consent Required

### How it Works

1. URL Formation
2. Merchant Changes
3. Customer Notification and Confirmation

### Merchant Guidelines

- Endpoint Setup
- Initiating Changes
- Cancellation Request
- Supplemental Customer Notification

### JSON Structure for Merchant Changes

- To Cancel Subscription
- To Update Payment Fields

### Bulk Updates

# Tools For Creating `monero-request` codes

- [Monero Payment Request Code Creator Website](https://monerosub.tux.pizza/)
- [Monero Payment Request Code Creator CLI Tool](https://github.com/lukeprofits/Monero_Payment_Request_Code_Creator)
- [Monero Payment Request Code Creator Pip Package](https://github.com/lukeprofits/monero_payment_request)
- More `monero-request` integration tools coming soon...
