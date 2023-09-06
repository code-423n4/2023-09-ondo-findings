## [G-1] In the ```_rpow``` function the n is never gonna be 0 so it is useless to run the switch case to check if n==0

### Code line

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L354


### Details

In ```_rpow``` there is a switch case which checks for if n==0, this is the condition never meet because  ``_rpow``` is only called inside the ```derivePrice``` function where the value is always summed by 1 so it never gonna be 0. minimum it can be 1. it is useless to check for this and waste some gas.

```diff
 - case 0 {
 -         switch n
 -          case 0 { z := base }
 -         default { z := 0 }
            }


 + case 0 {
 +        s := 0
 +      }
 
          
```

It is reccomended to removed the unused code to save some gas.