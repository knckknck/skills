# Numscript specs format

The Numscript specs format is a conventional way to express unit tests about Numscript, using JSON. It can be used to define assertions over the results of a numscript run, given certain inputs. You can execute the tests using the [`numscript test command`](./cli#test)

A JSON schema is available [online](https://raw.githubusercontent.com/direktly/numscript/refs/heads/main/specs.schema.json), so that you can have autocomplete and diagnostics in your editor. In many editors such as vscode, you can enable it adding it to the json this way:

```json
{
  "$schema": "https://raw.githubusercontent.com/direktly/numscript/refs/heads/main/specs.schema.json"
}
```

Here's the schema (using typescript notation):

```typescript
type Specs = {
  balances?: Balances;
  variables?: Vars;
  metadata?: AccountsMetadata;

  featureFlags?: Array<string>;
  testCases: Array<TestCase>
};

type TestCase = {
  balances?: Balances;
  variables?: Vars;
  metadata?: AccountsMetadata;

  it: string;
  "expect.missingFunds"?: boolean;
  "expect.postings"?: Array<Posting>;
  "expect.txMetadata"?: TxMetadata;
  "expect.metadata"?: AccountsMetadata;
  "expect.volumes"?: Balances;
  "expect.movements"?: Movements;
}
```

```typescript
type Balances = {
  [account: string]: {
    [asset: string]: number
  }
};
type Vars = {
  [name: string]: string
};
type AccountsMetadata = {
  [account: string]: {
    [key: string]: string
  }
};
type TxMetadata = {
  [key: string]: string
};
type Posting = {
  source: string;
  destination: string;
  asset: string;
  amount: number
};
type Movements = {
  [source: string]: {
    [destination: string]: {
      [asset: string]: number
    }
  }
};
```

### Example

Say we have the following numscript:

```numscript
  vars {
    monetary $cap
    account $source
    account $destination
  }

  send [EUR/2 *] (
    source = max $cap from $source
    destination = $destination
  )
  ```

And we want to test that we never send more than `$cap`. We can express the relevant test cases in the following way:

```json
{
  "$schema": "https://raw.githubusercontent.com/direkly/numscript/specs.schema.json",
  "variables": {
    "source": "alice",
    "destination": "bob"
  },
  "balances": {
    "alice": { "EUR/2": 500 }
  },
  "testCases": [
    {
      "it": "sends all the available balance when it doesn't exceed the cap and @alice has enough balance",
      "variables": {
        "cap": "EUR/2 9999"
      },
      "expect.postings": [
        {
          "source": "alice",
          "destination": "bob",
          "amount": 500,
          "asset": "EUR/2"
        }
      ]
    },
    {
      "it": "caps the sent amt to $cap when lower than available balance",
      "variables": {
        "cap": "EUR/2 10"
      },
      "expect.postings": [
        {
          "source": "alice",
          "destination": "bob",
          "amount": 10,
          "asset": "EUR/2"
        }
      ]
    }
  ]
}
```

## Preconditions

The inputs of each test cases. You can set the preconditions top-level (in the outer object), and/or in each `testCase` . The preconditions in a testCase will be merged to the top-level preconditions (with the precedence being given to the inner preconditions).

For example, in the following specs:

```json
{
  "balances": {
    "alice": { "EUR/2": 100,  "USD/2": 100  },
	"bob":   { "EUR/2": -2 }
  },
 "testCases": [
   {
	 "it": "example specs",
     "balances": {
	   "alice": { "EUR/2": 999 }
     }
   }
 ]
}
```

The inner preconditions will only override `@alice` 's `EUR/2` balance, resulting in:

```json
{
  "alice": { "EUR/2": 999,  "USD/2": 100  },
  "bob":   { "EUR/2": -2 }
}
```

### `variables`

The (stringified) value of each variable

```json
{
  "variables": {
     "amount": "USD/2 100"
  }
}
```

### `balances`

The initial accounts' balances.

```json
{
  "balances": {
    "alice": { "USD/2" : 200 },
	"bob": { "USD/2" : -42 },
  }
}
```

### `metadata`

The initial accounts' metadata.

```json
{
  "metadata": {
	"alice": {
		"id": "1234"
	}
  }
}
```

## Assertions

Assertions are only run if explicitly defined.

The recommended assertion to use by default are `expect.postings` or `expect.missingFunds` , but there are also a few weaker assertion that might be useful when the exact postings are an implementation detail of your business logic.

### `expect.error.missingFunds`

Assert that the script failed because of missing funds. Even if this is set to true, the test will still fail if the script outputs a different error.

Defaults to `false`.

<Note>
  Note: this was called `expect.error` in earlier releases
</Note>

### `expect.error.negativeAmount`

Assert that the script failed because of a send statement using a negative amount.

Defaults to `false`.

### `expect.postings`

Assert against the exact postings emitted by the script. To assert that there are no postings, you can use the empty array. To assert that no postings are produced because of a failure due to missing funds, you can use the `expect.missingFunds`  assertion instead.

```json
{
  "expect.postings": [
     { "source": "world", "destination": "user:001", "asset": "EUR/2", "amount": 100 }
  ]
}
```

### `expect.txMetadata`

Assert against the transaction meta emitted by the script (using `set_tx_meta`). It's a map from the metadata key to its (stringified) value.

```json
{
  "expect.txMetadata": {
	"senderAccount": "user:5829"
  }
}
```

### `expect.metadata`

Assert against the accounts metadata at the end of script execution (using `set_account_meta`). It's a nested map from an account, to the metadata key to its (stringified) value.

```json
{
  "expect.metadata": {
	"alice": {
		"id": "1234"
	}
  }
}
```

Note that it takes into account the values defined with `accountsMeta`  as well.

### `expect.endBalances`

Assert against the balances at the end of the script

For example:

```json
{
  "expect.endBalances": {
	"alice": { "EUR/2": 100 }
  }
}
```

means that `@alice`  has `[EUR/2 100]`  balance after the script is applied.

You might consider using this assertion when you only care about the end balance of an account, for example if you need to bring an account to a certain value (not less, not more), so you don't care about how the postings are composed exactly.

<Note>
  Note: this was called `expect.volumes` in earlier releases
</Note>

### `expect.endBalances.includes`

A weaker version of `expect.endBalances` that allows defining a subset of the balances we assert against.

For example, the following:

```json
{
  "expect.endBalances.includes": {
	"alice": { "EUR/2": 100 }
  }
}
```

passes even if there are more accounts in the involved balances, and if `alice` emit postings involving other currencies.

### `expect.movements`

Assert against the resulting movements. A movement is a nested map from a source account, to destination account, to the sent amounts.

For example, this assertion:

```json
{
  "expect.movements": {
 	"alice": {
	  "bob": { "EUR/2": 100 }
	}
  }
}
```

means that `@alice`  sent  `[EUR/2 100]` to `@bob`

You might consider using this assertion when you care about the movements graph from-to accounts, and you don't care about the order of the postings or the way they are split.

## Focus mode

You can select a subset of test to run by using the `focus` and `skip` modifiers on a test case definition. They are only meant to be used while developing, and will produce an error status code so that they aren't commited by mistake thus producing false positive tests.

### `focus`

If at least a test has a `focus` modifier, all the tests without the `focus` modifier will be skipped.

```json
{
  "testCases": [
	{
		"it": "only run this test!",
		"focus": true,
		"expect.postings": // ..
	},
	{
		"it": "this test is skipped",
		"expect.postings": // ..
	}
  ]
}
```

### `skip`

If a test is marked with the `skip` modifier, it will not be run.

```json
{
  "testCases": [
	{
		"it": "skip this test",
		"skip": true,
		"expect.postings": // ..
	}
  ]
}
```
