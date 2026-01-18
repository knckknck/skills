# Overdraft

The `overdraft` directive lets you instruct the Numscript interpreter that an account post-transaction balance is allowed to be less than zero.

## Unbounded

The overdraft directive allows for the account to go below zero without any limit, by using the `unbounded` keyword:

```numscript
send [USD/2 100] (
  source = @foo allowing unbounded overdraft
  destination = @bar
)
```

## Bounded

The exact amount below zero at which the account is allowed to go can be specified by using the `up to` keyword with a monetary value:

```numscript
send [USD/2 100] (
  source = @foo allowing overdraft up to [USD/2 50]
  destination = @bar
)
```
