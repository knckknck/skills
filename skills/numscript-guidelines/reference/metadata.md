# Metadata

> Learn how to use metadata in Numscript to store and retrieve additional information

Numscript transactions can interact with metadata, both on transactions and accounts.

## Accounts metadata

### Reading metadata to initialize a variable

Structured account metadata can be injected in Numscript variables during initialization. In the example below, we inject the monetary value stored under the metadata key `"coupon_value"` from the `coupon` account:

```numscript
vars {
  account  $coupon
  account  $wallet
  monetary $value = meta($coupon, "coupon_value")
}

send $value (
  source = $coupon
  destination = $wallet
)
```

Metadata injected in variables need to be typed, and its type is directly read from the value of the key `type` of the object stored under the metadata key. Its value is read from the value of the key `value`. Here are all the available types:

### Number

```json
{
  "amount": {
    "type": "number",
    "value": 1000
  }
}
```

### String 

```json
{
  "reference": {
    "type": "string",
    "value": "82HHON80ILP"
  }
}
```

### Asset

```json
{
  "currency": {
    "type": "asset",
    "value": "USD/2"
  }
}
```

### Monetary

```json
{
  "currency": {
    "type": "asset",
    "value": "USD/2"
  }
}
```

### Account

```json
{
  "merchant": {
    "type": "account",
    "value": "platform:merchant"
  }
}
```

### Portion

```json
{
  "commission": {
    "type": "portion",
    "value": "15.5%"
  }
}
```

### Writing account metadata during a transaction

Metadata can be written to an account using the `set_account_meta(_account_, "key", _value_)` statement. The statement takes a string-type key and a value which can be of any type, either as a variable or a literal.

### Writing metadata to a transaction

Metadata can be written to a transaction using the `set_tx_meta("key", _value_)` statement. The statement takes a string-type key and a value which can be of any type, either as a variable or a literal.

```numscript
set_tx_meta("order_fee", [USD/2 100])
set_tx_meta("tax", 20/100)
set_tx_meta("collection_account", @platform:commission)
set_tx_meta("commission", $commission)
```

### Wrap-up example

```numscript
vars {
  account $order
  account $merchant = meta($order, "merchant")
  monetary $fee
  portion $commission
  string $ref
}

send $fee (
  source = @orders:1234
  destination = @platform:fees
)

send [USD/2 *] (
  source = @orders:1234
  destination = {
    $commission to @platform:fees
    remaining to $merchant
  }
)

set_account_meta($order, "reference", $ref)
set_tx_meta("order_fee", $fee)
set_tx_meta("tax", 20/100)
set_tx_meta("commission", $commission)
```

```json
{
  "script": {
    "vars": {
      "order": "orders:186HH78UH",
      "fee": {
        "amount": 1000,
        "asset": "USD/2"
      },
      "commission": "15.5%",
      "reference": "108IUYGI"
    }
  }
}
```
