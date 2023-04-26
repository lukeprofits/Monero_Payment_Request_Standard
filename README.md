# Decoding & Encoding Monero Subscription Payment Strings

This document explains how to decode and encode Monero subscription payment strings using gzip compression and Base64 encoding.

## Decoding Monero Subscription Payment Strings
To decode a Monero subscription payment string, follow these steps:

1. Remove the prefix: `monero_subscription:`
1. Decode the string from Base64 to obtain the compressed data.
2. Decompress the compressed data using gzip to get the JSON string.
3. Parse the JSON string to extract the field values.

## Example Function To Decode Monero Subscription Code

```
import base64
import gzip
import json


def decode_monero_subscription_code(monero_subscription_code):
    # Catches user error. Code can start with "monero_subscription:", or ""
    code_parts = monero_subscription_code.split('_subscription:')
    if len(code_parts) == 2:
        monero_subscription_data = code_parts[1]
    else:
        monero_subscription_data = code_parts[0]
        
    # Extract the Base64-encoded string from the second part of the code
    encoded_str = monero_subscription_data
    
    # Decode the Base64-encoded string into bytes
    compressed_data = base64.b64decode(encoded_str.encode('ascii'))
    
    # Decompress the bytes using gzip decompression
    json_bytes = gzip.decompress(compressed_data)
    
    # Convert the decompressed bytes into a JSON string
    json_str = json_bytes.decode('utf-8')
    
    # Parse the JSON string into a Python object
    subscription_data_as_json = json.loads(json_str)
    
    return subscription_data_as_json


monero_subscription_code = 'monero_subscription:H4sIAGOfSWQC/x2O206DQBBAf4XwbBvKzeAbtKCxqYnQCvpCdpexoAtL9lK7a/x32b7NnDnJmV+XKCHZ2FKEgboPjtscSqdS2KkRpSCdHVzW7p3jClhWLtqfG7ZimMqgifjlTc9H9nkeFbwkInmV3HQlRJmCgovv9GPY3GfsHfdGC2YMOxRZbOrpuO8et3F6zVOc5xExRRn0y/SMxRj2W2j8ykaJ4hwmom3uVO0sQiNTk+1vknWSLGBGeoRJtkNnra+m8b35SKo623dPp+vtdYm4bDskwRq+5wcrL1z5sb3hgdJhOrdEEwqLo8XiBN7fP3qScGsYAQAA'

decoded_data = decode_monero_subscription_code(monero_subscription_code)

print(decoded_data)
```

## Encoding Monero Subscription Payment Strings
To encode a Monero subscription payment string, follow these steps:

1. Convert the payment details to a JSON object.
2. Compress the JSON object using gzip compression.
3. Encode the compressed data in Base64 format.
4. Add the Monero Subscription identifier `monero_subscription:` to the encoded string.


## Example Function To Create A Monero Subscription Code

```
import base64
import gzip
import json


def make_monero_subscription_code(json_data):
    # Convert the JSON data to a string
    json_str = json.dumps(json_data)
    
    # Compress the string using gzip compression
    compressed_data = gzip.compress(json_str.encode('utf-8'))
    
    # Encode the compressed data into a Base64-encoded string
    encoded_str = base64.b64encode(compressed_data).decode('ascii')
    
    # Add the Monero Subscription identifier
    monero_subscription = 'monero_subscription:' + encoded_str
    
    return monero_subscription
    

json_data = {
     "custom_label": "My Subscription",  # This can be any text
     "sellers_wallet": "4At3X5rvVypTofgmueN9s9QtrzdRe5BueFrskAZi17BoYbhzysozzoMFB6zWnTKdGC6AxEAbEE5czFR3hbEEJbsm4hCeX2S",
     "currency": "USD",                  # Currently supports "USD" or "XMR"
     "amount": 19.99,                    
     "payment_id": "EyRTrKDsbkH6GwdL",   # Unique identifier so the merchant knows which customer the payment relates to
     "start_date": "2023-04-26",         # If you want it to start the day of, you can use: datetime.now().strftime("%Y-%m-%d")
     "billing_cycle_days": 30            # How often it should be billed
     }
     
monero_subscription_code = make_monero_subscription_code(json_data)

print(monero_subscription_code)

```

