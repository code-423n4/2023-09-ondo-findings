I was only able to audit rUSDY.sol for a limited time.
I started by laying out all the functions, then noting which functions are different from the standard ERC20 and following them accordingly.
I understand the need for a blocklist due to law restrictions.

The contract (rUSDY) is basically a non-standard ERC20 token which extends USDY.
The amount of rUSDY held is determined by the amount of USDY (which can be swapped into shares) and the price of USDY in the formula shares*USDY.price.

The functions are all over the place and not well ordered. They mix the standard ERC20 tokens with their added functionality. 

They indicate that the token is a rebasing token while also indicating that the price moves up in a linear way where the slope can only be slowed down but not turned negative. To me this is not a rebasing token but merely an interest bearing token. 

One thing that really bothers me is not necessarily the power to restrict someone to send or receive but more the power to burn any tokens at will (admin power). This is an accepted risk which I will never understand. What if the admin gets fired, what if the admin gets their laptop stolen, what if the admin gets hacked. 

### Time spent:
5 hours