# Destinations

As with sources, there are several options when it comes to deciding where should funds in a financial transaction come from. The send statement provides the following ways of defining destinations:

## Single destination

```numscript
send [COIN 100] (
  source = @world
  destination = @users:001
)
```

---

## Allocation destinations

Similar to portioned sources, destination can be defined as a sequence of fractions to split the monetary onto multiple accounts.

In any case, the summed total of fractions in a block needs to be equal to 1 and the `remaining` keyword can be used to reach that total:

```numscript
send [COIN 100] (
  source = @world
  destination = {
    90/100 to @users:001
    remaining to @fees
  }
)
```

---

## Ordered destinations with maximum caps

Ordered destinations route funds to multiple destinations in sequence, with maximum caps for each destination:

Syntax:

```numscript
{
  destination = {
    max [value] to @destination_account1
    remaining to @destination_account2
  }
}
```

```numscript
send [COIN 100] (
  source = @world
  destination = {
    max [COIN 20] to @users:001
    max [COIN 50] to @users:002
    remaining to @users:003
  }
)
```

This sends up to the specified maximum to each destination in order, with any remaining amount going to the final destination.

---

## Nested destinations

Finally, as with sources, destination blocks can be nested:

```numscript
send [COIN 100] (
  source = @world
  destination = {
    80% to @users:001
    20% to {
      70% to @platform
      15% to @taxes
      remaining to @charity
    }
  }
)
```
