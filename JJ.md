
A deposits 100

DR Bank           100
CR A Collateral      100

--

DR A Collateral   100
CR A Liquidity       100

A is owed = collateral (0) + liquidity (100) = 100

--

CLEARING (creating obligations between participants)

DR A Liquidity     100
CR B Collateral      100
--
DR B Collateral    100
CR B Liquidity       100
--

A can spend no further!!! Liquidity = 0

100 of A's collateral is now backing A's obligation to B of 100
100 from A's OTHER credit card goes to B
when A's obligation to B of 100 is discharged, A's collateral must become free again

SETTLEMENT (discharging obligations between participants)

you pay another 100 (to B)

2x EFTS (100 each = 200) - obligation of 100 = collateral of 100

200 - 100 = 100

-- two-phase commit transfer:
DR A Settlement B  100
CR A Collateral      100
--

A's collateral balance is now 100
A's liquidity balance is 0

--

DR A Collateral   100
CR A Liquidity       100