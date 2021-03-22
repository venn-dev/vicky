# Monads

## Probabilistic Programming (List monad)

Let's say we are playing a game, and we have a light switch. 

In every turn, if it's on, there's a 50-50% chance to switch it off, but when it's off, there's 2/3 chance to switch it on.

```haskell
let turn x = if x == "on" then ["off", "on"] else ["on", "on", "off"]
```

What happens if we start from an "on" position after one turn?

```haskell
> ["on"] >>= turn
["off","on"]
```

The values in the list signify all the probable current states (and not a particular state nor the states that we passed from), starting from the `on` state and after one turn.

So, 1/2 cases it was off, 1/2 it was off. Okay.

What about after two turns?

```haskell
> ["on"] >>= turn >>= turn
["on","on","off","off","on"]
```

Okay, 2/3 chance to be on, 1/3 off.

```haskell
> ["on"] >>= turn >>= turn >>= turn
["off","on","off","on","on","on","off","on","on","off","off","on"]
```

After three turns it's a bit hard to figure out what's going on so, let's build a function for asking the probability of a state.

```haskell
count = fromIntegral . length :: [a] -> Float

(??) p x = 100 * (count (filter (== x) p)) / (count p)
```

`??` returns the percentage of elements in `p` that are equal to `x`.

`count` is just a wrapper around length that returns a Float instead of an Int.

```haskell
> (["on"] >>= turn >>= turn >>= turn) ?? "off"
41.666666666666664
> (["on"] >>= turn >>= turn >>= turn) ?? "on"
58.333333333333336
```

A more interesting example, let's say that we have a pandemic (cough, cough) and everyday there's:

 - 25% chance to get sick when healthy
 - 17% chance to get hospitalized, 33% chance to recover when sick,
 - and 50-50% chance to die or recover when hospitalized


```haskell
let vprob x = if x == "healthy" then ["healthy", "healthy", "healthy", "sick"] else if x == "sick" then ["sick", "sick", "sick", "hospitalized", "healthy", "healthy"] else if x == "hospitalized" then ["healthy", "dead"] else ["dead"]
```

What's the probability one will die after 10 days?

```haskell
> let p = ["healthy"] >>= gen >>= gen >>= gen >>= gen >>= gen >>= gen >>= gen >>= gen >>= gen >>= gen
> p ?? "dead"
2.2962844
````

Okay, okay, haskell is full of tricks, what do I care. Well, if you understand how this works, you can do this in python too.

We could call `>>=` function `apply` (haskellers: I know apply is a different function, sssh). Apply takes a function that transforms a list and the list and returns the new list.

The transforming function will take one element from the list and return all future states. So, for each previous state, we get a list of new states, therefore we will flatten the result.

```python
def flatten(l): return sum(l, [])
  
def apply(p, f): return flatten([f(x) for x in p])
```

We can know define the previous `vprob` probability function:


```python
def vprob(x): return ["healthy", "healthy", "healthy", "sick"] if x == "healthy" else ["sick", "sick", "sick", "hospitalized", "healthy", "healthy"] if x == "sick" else ["healthy", "dead"] if x == "hospitalized" else ["dead"]```

>>> apply(["healthy"], vprob)
['healthy', 'healthy', 'healthy', 'sick']
>>> apply(apply(["healthy"], vprob), vprob)
['healthy', 'healthy', 'healthy', 'sick', 'healthy', 'healthy', 'healthy', 'sick', 'healthy', 'healthy', 'healthy', 'sick', 'sick', 'sick', 'sick', 'hospitalized', 'healthy', 'healthy']
```

Applying the function multiple times is somewhat ugly in python so we will make a wrapper function `apply_times` that recursively calls apply:

```python
def apply_times(p, f, c): return p if c == 0 else apply_times(apply(p, f), f, c-1)

>>> apply_times(["healthy"], vprob, 2)
['healthy', 'healthy', 'healthy', 'sick', 'healthy', 'healthy', 'healthy', 'sick', 'healthy', 'healthy', 'healthy', 'sick', 'sick', 'sick', 'sick', 'hospitalized', 'healthy', 'healthy']
```

And finally the function for determining the probability of a certain state:

```python
def prob(p, s): return len([x for x in p if x == s])/len(p)

>>> prob(apply_times(["healthy"], gen, 2), "hospitalized")
0.05555555555555555
```

I tried running the apply_times with 10 parameter but python crashed =))
