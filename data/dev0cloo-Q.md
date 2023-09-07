# QA Report 

## Low Severity Findings

### Range Start and End times must be in Unix format to prevent errors in `RWADynamicOracle`
- The ranges set in the DynamicOracle contract must follow the Unix format since the calculations they are involved in use block.timestamp, which is also in Unix format, to prevent unexpected results. This is especially so when no visible checks exist for these ranges to be in the required format. Here are some instances:
```
function getPrice() public view whenNotPaused returns (uint256 price) {
        uint256 length = ranges.length;
        for (uint256 i = 0; i < length; ++i) {
            Range storage range = ranges[(length - 1) - i];
            if (range.start <= block.timestamp) {
                if (range.end <= block.timestamp) {
                    return derivePrice(range, range.end - 1);
                } else {
                    return derivePrice(range, block.timestamp); //@audit - block.timestamp returns unix values
                }
            }
        }
    }

    function derivePrice(
        Range memory currentRange,
        uint256 currentTime
    ) internal pure returns (uint256 price) {
        uint256 elapsedDays = (currentTime - currentRange.start) / DAY; //@audit - if range.start is not in unix format, this will lead to serious errors
        return
            roundUpTo8(
                _rmul(
                    _rpow(currentRange.dailyInterestRate, elapsedDays + 1, ONE),
                    currentRange.prevRangeClosePrice
                )
            );
    }
```
This is marked as low severity because the `overrideRange` function easily allows a fix for this if it occurs. 