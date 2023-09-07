https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

setRange function
low
1)Array Length Check: The code assumes that there is at least one existing range in the ranges array. You should ensure that ranges.length > 0 before accessing ranges[ranges.length - 1] to prevent potential runtime errors.

function setRange(
    uint256 endTimestamp,
    uint256 dailyInterestRate
) external onlyRole(SETTER_ROLE) {
    // Check if there are any existing ranges
    if (ranges.length > 0) {
        Range memory lastRange = ranges[ranges.length - 1];

        // Check that the endTimestamp is greater than the last range's end time
        require(endTimestamp > lastRange.end, "InvalidRange");

        uint256 prevClosePrice = derivePrice(lastRange, lastRange.end - 1);
        ranges.push(
            Range(lastRange.end, endTimestamp, dailyInterestRate, prevClosePrice)
        );

        emit RangeSet(
            ranges.length - 1,
            lastRange.end,
            endTimestamp,
            dailyInterestRate,
            prevClosePrice
        );
    } else {
        // Handle the case where there are no existing ranges (first range)
        // You may need to decide what to do in this case, e.g., create the first range.
        // For demonstration purposes, we'll create a range starting at block.timestamp.
        require(endTimestamp > block.timestamp, "InvalidRange");
        uint256 trueStart = (rangeStartPrice * ONE) / dailyIR;
        ranges.push(Range(block.timestamp, endTimestamp, dailyInterestRate, trueStart));

        emit RangeSet(
            0,
            block.timestamp,
            endTimestamp,
            dailyInterestRate,
            trueStart
        );
    }
}
In this code, we first check if there are any existing ranges in the ranges array. If there are, we perform the regular range addition logic as before. If there are no existing ranges, we have added a placeholder logic to handle the case (e.g., creating the first range) based on your requirements.

2)InvalidRange Revert: The code contains revert InvalidRange();, which seems to be an attempt to revert the transaction if the new range's end timestamp is not greater than the last range's end time. However, the correct way to revert with a custom error message in Solidity is as follows:
revert("InvalidRange");
