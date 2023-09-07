Divide Before Multiply

Performing integer division before multiplication truncates the low bits, losing the precision of calculation.

In `RWADynamicOracle`: 

```solidity=383
          x := div(xxRound, base)
```

```solidity=385
            let zx := mul(z, x)
```

Consider applying multiplication before division to avoid loss of precision.