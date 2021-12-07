# Settlement models:
- immediate gross bilateral
- deferred net bilateral (between two participants)
- deferred net multilateral (between a participant and the hub)
- immediate net multilateral (between a participant and the hub)

## immediate gross bilateral:

# Transfer
Participant A Liquidity has 100 units.

# Transfer 1 [A to B]
```
DR Participant A Liquidity                  100
    CR Participant A Payable To Hub                     100
DR Participant A Payable To Hub             100
    CR Participant A Payable to B                       100
DR Participant A Payable to B               100
    CR Participant B Liquidity                          100
```

# Transfer 2 [B to A]
```
DR Participant B Liquidity                  10
    CR Participant B Payable to Hub                     10
DR Participant B Payable to Hub             10
    CR Participant B Payable to A                       10
DR Participant B Payable to A               10
    CR Participant A Liquidity                          10
```

# Transfer 3 [B to C]
```
DR Participant B Liquidity                  20
    CR Participant B Payable to Hub                     20
DR Participant B Payable to Hub             20
    CR Participant B Payable to C                       20
DR Participant B Payable to C               20
    CR Participant C Liquidity                          20
```

> insight: we don't need journal entries for deferred net bi/multilateral
> we don't need to change our chart of accounts

Settlement report (immediate gross bilateral):

* A paid to B = 100
* B paid to A = 10
* B paid to C = 20

Settlement report (deferred net bilateral):

* A owes B = 100 - 10 = 90
* B owes C = 20

| Account                      | Debits | Credits | Balance |
|------------------------------|--------|---------|---------|
| **Participant A Liquidity**  | `100`  |         | `0`     |------> Transfer 1 <-------
| Participant A Payable to Hub | `100`  | `100`   | `0`     |--> Multilateral
| Participant A Payable to B   | `100`  | `100`   | `0`     |--> Bilateral
| Participant B Liquidity      |        | `100`   | `100`   |--> A Credit B
| **Participant B Liquidity**  | `10`   |         | `90`    |------> Transfer 2 <-------
| Participant B Payable to Hub | `10`   | `10`    | `0`     |--> Multilateral
| Participant B Payable to A   | `10`   | `10`    | `0`     |--> Bilateral
| **Participant A Liquidity**  |        | `10`    | `10`    |--> B Credit A
| **Participant B Liquidity**  | `20`   |         | `70`    |------> Transfer 3 <-------
| Participant B Payable to Hub | `20`   | `20`    | `0`     |--> Multilateral
| Participant B Payable to C   | `20`   | `20`    | `0`     |--> Bilateral
| **Participant C Liquidity**  |        | `20`    | `20`    |--> B Credit C
> `Liquidity` accounts for participants balance. `Payable to Hub` to cover multilateral model.
> `[X] Payable to [Y]` accounts to cover bilateral model.

# Settlement - Reserve
The reservation may be immediate or deferred.

## Reserve 1 - A to B
```
DR Participant A Payable to Hub                 100
    CR Participant B Receivable from Hub                    100
```
## Reserve 2 - B to A
```
DR Participant B Payable to Hub                 10
    CR Participant A Receivable from Hub                    10
```
## Reserve 3 - B to C
```
DR Participant B Payable to Hub                 20
    CR Participant C Receivable from Hub                    20
```

At this point we have deferred (*immediate also possible through configuration*) gross multilateral settlement for reservation is:

* Participant A receivable from Hub    = 10
* Participant A payable to Hub         = 100
* Participant B receivable from Hub    = 100
* Participant B payable to Hub         = 30
* Participant C receivable from Hub    = 20
* Participant C payable to Hub         = 0

deferred net multilateral:

* A owes B      = 100 (gross)
* B owes A      = 10 (gross)
* B owes C      = 20 (gross)
* A owes B      = 90 (net)
* B owes C      = 20 (net)
* A owes Hub    = 100
* B owes Hub    = 30 (10A + 20C)
* C owes Hub    = 0

| Account                           | Debits | Credits | Balance |
|-----------------------------------|--------|---------|---------|
| Participant A Payable to Hub      | `100`  |         | `-100`  |------> Transfer 1 (A to B) <-------
| Participant B Receivable from Hub |        | `100`   | `100`   |--> Reserve 1
| Participant B Payable to Hub      | `10`   |         | `-10`   |------> Transfer 2 (B to A) <-------
| Participant A Receivable from Hub |        | `10`    | `10`    |--> Reserve 2
| Participant B Payable to Hub      | `20`   |         | `-20`   |------> Transfer 3 (B to C) <-------
| Participant C Receivable from Hub |        | `20`    | `20`    |--> Reserve 3
> The `Payable to Hub` having a negative balance and `Receivable from Hub` having a positive 
> balance indicates there is an outstanding settlement.


# Settlement - Commit
The commit may be immediate or deferred.

## Commit 1.1
```
DR Participant B Receivable from Hub            100
    CR Participant A Payable to Hub                         100
```
## Commit 1.2
```
DR Participant A Receivable from Hub            10
    CR Participant B Payable to Hub                         10
```
## Commit 1.3
```
DR Participant C Receivable from Hub            20
    CR Participant B Payable to Hub                         20
```

deferred net multilateral:

* A settled B      = `90` (100 - 10 = 90)
* A settled Hub    = `100`
* B settled Hub    = `30` (10A + 20C = 30)
* B settled C      = `20`
* C settled Hub    = `0`

| Account                           | Debits | Credits | Balance |
|-----------------------------------|--------|---------|---------|
| Participant B Receivable from Hub | `100`  |         | `0`     |------> Transfer 1 (A to B) <-------
| Participant A Payable to Hub      |        | `100`   | `0`     |--> Commit 1
| Participant A Receivable from Hub | `10`   |         | `0`     |------> Transfer 2 (B to A) <-------
| Participant B Payable to Hub      |        | `10`    | `0`     |--> Commit 2
| Participant C Receivable from Hub | `20`   |         | `0`     |------> Transfer 3 (B to C) <-------
| Participant B Payable to Hub      |        | `20`    | `0`     |--> Commit 2
> Both Hub payable and receivable Accounts are set to `0` after settlement commit (happy path).
> The Participant Liquidity remains unaffected since transfer. 

# Other Notes;

* MJL
* `[Multilateral deferred net settlement]` - Settlements are deferred net if a number of transfers are settled together. Net settements (in which a number of transfers are settled together) are by definition deferred (since it takes time to construct a batch.)
    * Settlements are gross if each transfer is settled separately. 
    * Gross settlements may be immediate or deferred. 
    * They are deferred if approval for settlement from outside the Hub is required, and immediate if the Hub can proceed to settlement of a transfer without requiring any external approval. 
    * At present, Mojaloop only supports immediate gross settlements.
* `[Bilateral deferred net settlement]` - Settlements are bilateral if each pair of participants settles with each other for the net of all transfers between them. 
    * Settlements are multilateral if each participant settles with the Hub for the net of all transfers to which it has been a party, no matter who the other party was.

> A settlement model specifies a way in which a Mojaloop Hub will settle a set of transfers. 
> In the simple case, there is only one settlement model and it settles all the transfers that are processed by the Hub. 
> However, Mojaloop supports more than one settlement model for a single scheme. 
> This allows, for instance, a scheme to define different settlement models for different currencies, or for different ledger account types.
