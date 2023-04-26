# Decoding & Encoding Monero Subscription Payment Strings

This document explains how to decode and encode Monero subscription payment strings using gzip compression and Base64 encoding.

## Decoding Monero Subscription Payment Strings
To decode a Monero subscription payment string, follow these steps:

1. Remove the prefix: `MONERO_SUBSCRIPTION::::`
1. Decode the string from Base64 to obtain the compressed data.
2. Decompress the compressed data using gzip to get the JSON string.
3. Parse the JSON string to extract the field values.

## Encoding Monero Subscription Payment Strings
To encode a Monero subscription payment string, follow these steps:

1. Convert the payment details to a JSON object.
2. Compress the JSON object using gzip compression.
3. Encode the compressed data in Base64 format.
4. Add the Monero Subscription identifier `MONERO_SUBSCRIPTION::::` to the encoded string.

## Example Code

```python
# Decoding a Monero Subscription Payment String
import base64
import gzip
import json

monero_subscription = 'MONERO_SUBSCRIPTION::::H4sIAAABiVYC/x1Sx67DQBiEX4VwbQ3lZPAAZQFQWcaKfZumXblJu7p3q3esq5K5sf5J5e/7pvlq/bFWZkU/q8heKYXfgiU6KQ2/LGw75zcPSBvUsOja7TIS39iZjWlI8r2rY0FmSMdKxuWg8zddv1s23/NnJxcZ/HnmrWttGvM9rJy/e6/hjlvdszW8p+bo1Ja09vc3p3q3euqB8mr9vx9XUHUN6VqM3bJcxdVevj+63RJgYuL0/z/kHj7VZorElK49Xh6U5O6OyKjQr1D8h1VzvxwAAAP//ks/vAgAA'

# Decode the string from Base64 format to get the compressed data.
compressed_data = base64.b64decode(monero_subscription[len('MONERO_SUBSCRIPTION::::'):])

# Decompress the compressed data using gzip to get the JSON string.
json_str = gzip.decompress(compressed_data).decode('utf-8')

# Parse the JSON string to extract the field values.
payment_data = json.loads(json_str)

print(payment_data)

# Encoding a Monero Subscription Payment String
payment_data = {
    "custom_label": "Developer Donation",
    "sellers_wallet": "4At3X5rvVypTofgmueN9s9QtrzdRe5BueFrskAZi17BoYbhzysozzoMFB6zWnTKdGC6AxEAbEE5czFR3hbEEJbsm4hCeX2S",
    "currency": "USD",
    "namount": 19.99,
    "payment_id": "EyRTrKDsbkH6GwdL",
    "start_date": "2023-04-25",
    "billing_cycle_days": 30
}

# Convert the payment details to a JSON object.
json_str = json.dumps(payment_data)

# Compress the JSON object using gzip compression.
compressed_data = gzip.compress(json_str.encode('utf-8'))

# Encode the compressed data in Base64 format.
encoded_str = base64.b64encode(compressed_data).decode('ascii')

# Add the Monero Subscription identifier to the encoded string.
monero_subscription = 'MONERO_SUBSCRIPTION::::' + encoded_str

print(monero_subscription)
