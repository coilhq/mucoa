The basic journal entries:

-----------------------------------------------------------
DEPOSIT
-----------------------------------------------------------

```
DR A Deposit                                100
    CR A Collateral                                     100
DR A Collateral                             100
    CR A Liquidity                                      100
```

-----------------------------------------------------------
CLEARING (A pays B = 70, these are linked transfers)
-----------------------------------------------------------

The intermediary account here is only a tracing account
to simplify queries (queries then need only one DB index).

> Participant A's clearing account with respect to participant B

```
DR A Liquidity                               70
    CR A Clearing (B)                                    70
DR A Clearing (B)                            70
    CR B Liquidity                                       70
```

-----------------------------------------------------------
SETTLEMENT (scheme is notified that A settled with B)
-----------------------------------------------------------

```
DR A Settlement (B)                          70
    CR A Liquidity                                       70
```

-----------------------------------------------------------

Now let's add C into the mix.
In addition to A's transfer to B above, B pays C 170.

From beginning to end:

A liquidity = 100
B liquidity = 100
C liquidity = 100

A pays B = 70
B pays C = 170
C pays A = 50

A liquidity = 100 - 70 + 50 = 80
B liquidity = 100 + 70 - 170 = 0
C liquidity = 100 + 170 - 50 = 220

Note 1: Total liquidity backed by collateral remains constant = 300
Note 2: B can no longer clear payments because B's liquidity is exhausted.
Note 3: The goal of settlement is to get liquidity back to what it was before clearing.

BILATERAL NET SETTLEMENT:

A owes B less what B owes A = 70
B owes C less what C owes B = 170
C owes A less what A owes C = 50

A liquidity plus settlement = 80 + 70 - 50 = 80 + 20 = 100
B liquidity plus settlement = 0 + 170 - 70 = 0 + 100 = 100
C liquidity plus settlement = 220 + 50 - 170 = 220 - 120 = 100

MULTILATERAL NET SETTLEMENT:

A owes someone who owes someone else, so A should go direct to C:
(there are multiple ways to arrange who pays what, all are valid)

A owes C (up to what A owes B) less what C owes A = 70 - 50 = 20
B owes C less what A owes C on B's behalf = 170 - 70 = 100

```
DR A Settlement (C)        20
    CR A Liquidity              20

DR B Settlement (C)       100
    CR B Liquidity             100
```

A and B's liquidity is now both back up to 100.

This is the most efficient way of doing multilateral net because it
doesn't require a "pot" at all and minimizes the number of
settlement transactions.

Another simpler way to do this is for anyone who owes something to
pay into a "pot" and then for anyone who is owed something to be
paid out of the "pot". This requires a pot and a few more
transactions.