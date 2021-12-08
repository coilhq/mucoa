# Settlement Model Scenarios
The motivation behind this document stems from [MJL - Ledgers in the Hub](https://docs.mojaloop.io/mojaloop-business-docs/HubOperations/Settlement/ledgers-in-the-hub.html)
The purpose is to illustrate the chart of accounts (COA) for covering each of the required Mojaloop Settlement Models.

## Definitions
| Definition  | Description                                                                          |
|-------------|--------------------------------------------------------------------------------------|
| Participant | A provider who is a member of a payment scheme, and subject to that scheme's rules.  |
| Hub         | The Mojaloop operator                                                                |
| Transfer    | A debit/credit from one account to another account                                   |


## Immediate Gross Bilateral Settlement (IGS)
Settlements are gross if each transfer is settled separately. 
*Gross* settlements may be immediate or deferred. 
They are deferred if approval for settlement from outside the Hub is required, and immediate if the Hub can proceed to settlement of a transfer without requiring any external approval. 
At present, Mojaloop only supports immediate gross settlements.
This is useful for SME environments where quick payment turnarounds are often desirable in order to maximize liquidity. 
IGS is also known as Real Time Gross Settlement (RTGS).

### Activity
To follow are the transfer and journal activities for IGS.

#### Transfer 1 [A to B]
> DR on Participant A Liquidity account, followed by a CR on Participant B account of 100 units.
```
DR Participant A Liquidity                  100
    CR Settlement A Payable to B                        100
DR Settlement A Payable to B                100
    CR Participant B Liquidity                          100
```
#### Transfer 2 [B to A]
> DR on Participant B Liquidity account, followed by a CR on Participant A account of 10 units.
```
DR Participant B Liquidity                  10
    CR Settlement B Payable to A                        10
DR Settlement B Payable to A                10
    CR Participant A Liquidity                          10
```
#### Transfer 3 [B to C]
> DR on Participant B Liquidity account, followed by a CR on Participant C account of 20 units.
```
DR Participant B Liquidity                  20
    CR Settlement B Payable to C                        20
DR Settlement B Payable to C                20
    CR Participant C Liquidity                          20
```

#### Account Balances
| Account                       | Debits  | Credits | Balance |
|-------------------------------|---------|---------|---------|
| `Opening Balances`            |         |         |         |
| **Participant A Liquidity**   | `0`     | `100`   | `100`   |------> Opening Balance <-------
| **Participant B Liquidity**   | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Participant C Liquidity**   | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement A Payable to B** | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement B Payable to A** | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement B Payable to C** | `0`     | `0`     | `0`     |------> Opening Balance <-------
| `Transfer-01`                 |         |         |         |
| Participant A Liquidity       | `100`   |         | `0`     |------> Transfer 1 <-------
| Settlement A Payable to B     | `100`   | `100`   | `0`     |--> Bilateral (Recording)
| Participant B Liquidity       |         | `100`   | `100`   |--> A Credit B
| `Transfer-02`                 |         |         |         |
| Participant B Liquidity       | `10`    |         | `90`    |------> Transfer 2 <-------
| Settlement B Payable to A     | `10`    | `10`    | `0`     |--> Bilateral (Recording)
| Participant A Liquidity       |         | `10`    | `10`    |--> B Credit A
| `Transfer-03`                 |         |         |         |
| Participant B Liquidity       | `20`    |         | `70`    |------> Transfer 3 <-------
| Settlement B Payable to C     | `20`    | `20`    | `0`     |--> Bilateral (Recording)
| Participant C Liquidity       |         | `20`    | `20`    |--> B Credit C
> `Liquidity` = accounts for participants balance. 
> `Settlement [X] Payable to [Y]` = accounts to cover bilateral gross. 

#### Settlement Report - Immediate Gross Bilateral:
* A paid to B = 100
* B paid to A = 10
* B paid to C = 20

---

## Bilateral Deferred Net Settlement (B-DNS)
Settlements are deferred net if a number of transfers are settled together. 
Net Settlements (in which a number of transfers are settled together) are by definition deferred 
(since it takes time to construct a batch.)

### Activity
To follow are the transfer and journal activities for Bilateral DNS.

#### Transfer 1 [A to B]
> DR on Participant A Liquidity account, followed by a CR on Participant B account of 100 units.
```
DR Participant A Liquidity                  100
    CR Settlement A Payable to B                        100
DR Settlement A Payable to B                100
    CR Participant B Liquidity                          100
```
#### Transfer 2 [B to A]
> DR on Participant B Liquidity account, followed by a CR on Participant A account of 10 units.
```
DR Participant B Liquidity                  10
    CR Settlement B Payable to A                        10
DR Settlement B Payable to A                10
    CR Participant A Liquidity                          10
```
#### Transfer 3 [B to C]
> DR on Participant B Liquidity account, followed by a CR on Participant C account of 20 units.
```
DR Participant B Liquidity                  20
    CR Settlement B Payable to C                        20
DR Settlement B Payable to C                20
    CR Participant C Liquidity                          20
```

#### Account Balances Transfers
> At this point, the transfers are complete, but settlement reservation has not yet occurred. 
> All Settlement accounts have a `0` unit balance. 
> 
| Account                      | Debits  | Credits | Balance |
|------------------------------|---------|---------|---------|
| `Opening Balances`           |         |         |         |
| **Participant A Liquidity**  | `0`     | `100`   | `100`   |------> Opening Balance <-------
| **Participant B Liquidity**  | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Participant C Liquidity**  | `0`     | `0`     | `0`     |------> Opening Balance <-------
| `Transfer-01`                |         |         |         |
| Participant A Liquidity      | `100`   |         | `0`     |------> Transfer 1 <-------
| Settlement A Payable to B    | `100`   | `100`   | `0`     |--> Bilateral (Recording)
| Participant B Liquidity      |         | `100`   | `100`   |--> A Credit B
| `Transfer-02`                |         |         |         |
| Participant B Liquidity      | `10`    |         | `90`    |------> Transfer 2 <-------
| Settlement B Payable to A    | `10`    | `10`    | `0`     |--> Bilateral (Recording)
| Participant A Liquidity      |         | `10`    | `10`    |--> B Credit A
| `Transfer-03`                |         |         |         |
| Participant B Liquidity      | `20`    |         | `70`    |------> Transfer 3 <-------
| Settlement B Payable to C    | `20`    | `20`    | `0`     |--> Bilateral (Recording)
| Participant C Liquidity      |         | `20`    | `20`    |--> B Credit C
> `Liquidity` = accounts for participants balance.
> `Settlement [X] Payable to [Y]` = accounts to cover bilateral gross.


#### Settlement Reservation - [A to B], [B to A] and [B to C] 
> The settlement reservation occurs later during the day as a batch process.
> Since the settlement is deferred, the settlement will occur as a batch process.
```
DR Settlement A Payable to B                100
    CR Settlement B Receivable from A                   100
DR Settlement B Payable to A                10
    CR Settlement A Receivable from B                   10
DR Settlement B Payable to C                20
    CR Settlement C Receivable from B                   20
```

#### Account Balances Settlement Reservation
> The Payable/Receivable accounts are utilized to track settlement reservation.  
> 
| Account                              | Debits  | Credits | Balance |
|--------------------------------------|---------|---------|---------|
| `Opening Balances`                   |         |         |         |
| **Settlement A Payable to B**        | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement B Payable to A**        | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement B Payable to C**        | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement B Receivable from A**   | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement A Receivable from B**   | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement C Receivable from B**   | `0`     | `0`     | `0`     |------> Opening Balance <-------
| `Settlement-Reserve-01`              |         |         |         |------> Reserve 1 <-------
| Settlement A Payable to B            | `100`   |         | `-100`  |--> Payer
| Settlement B Receivable from A       |         | `100`   | `100`   |--> Payee
| `Settlement-Reserve-02`              |         |         |         |------> Reserve 2 <-------
| Settlement B Payable to A            | `10`    |         | `-10`   |--> Payer
| Settlement A Receivable from B       |         | `10`    | `10`    |--> Payee
| `Settlement-Reserve-03`              |         |         |         |------> Reserve 3 <-------
| Settlement B Payable to C            | `20`    |         | `-20`   |--> Payer
| Settlement C Receivable from B       |         | `20`    | `20`    |--> Payee
> `Settlement [X] Payable to [Y]`    = accounts to cover bilateral gross for payer.
> `Settlement [X] Receivable to [Y]` = accounts to cover bilateral gross for payee.

#### Settlement Commit - [A to B], [B to A] and [B to C]
> One may only be able to commit a previously reserved settlement.
```
DR Settlement B Receivable from A           100
    CR Settlement A Payable to B                        100
DR Settlement A Receivable from B           10
    CR Settlement B Payable to A                        10
DR CR Settlement C Receivable from B        20
    Settlement B Payable to C                           20
```

#### Account Balances Settlement Commit
> The Payable/Receivable accounts are now reset back to `0` post commit.
>
| Account                            | Debits  | Credits | Balance |
|------------------------------------|---------|---------|---------|
| `Opening Balances`                 |         |         |         |
| **Settlement A Payable to B**      | `100`   | `0`     | `-100`  |------> Opening Balance <-------
| **Settlement B Payable to A**      | `10`    | `0`     | `-10`   |------> Opening Balance <-------
| **Settlement B Payable to C**      | `20`    | `0`     | `-20`   |------> Opening Balance <-------
| **Settlement B Receivable from A** | `0`     | `100`   | `100`   |------> Opening Balance <-------
| **Settlement A Receivable from B** | `0`     | `10`    | `10`    |------> Opening Balance <-------
| **Settlement C Receivable from B** | `0`     | `20`    | `20`    |------> Opening Balance <-------
| `Settlement-Commit-01`             |         |         |         |------> Commit 1 <-------
| Settlement A Payable to B          |         | `100`   | `0`     |--> Payer
| Settlement B Receivable from A     | `100`   |         | `0`     |--> Payee
| `Settlement-Commit-02`             |         |         |         |------> Commit 2 <-------
| Settlement B Payable to A          |         | `10`    | `0`     |--> Payer
| Settlement A Receivable from B     | `10 `   |         | `0`     |--> Payee
| `Settlement-Commit-03`             |         |         |         |------> Commit 3 <-------
| Settlement B Payable to C          |         | `20`    | `0`     |--> Payer
| Settlement C Receivable from B     | `20`    |         | `0`     |--> Payee

> `Settlement [X] Payable to [Y]`    = accounts to cover bilateral gross for payer.
> `Settlement [X] Receivable to [Y]` = accounts to cover bilateral gross for payee.

#### Settlement Report - Immediate Gross Bilateral:
* A paid to B = 100
* B paid to A = 10
* B paid to C = 20

#### Settlement Report - Deferred Net Bilateral:
* A owes B = 100 - 10 = 90
* B owes C = 20

---

## Multilateral Deferred Net Settlement (M-DNS)
Method of deferring payments to enable settlement on multiple batches according a predetermined schedule. 
This is useful for environments involving multiple Participants to a transaction requiring a balance of payment due settlement approach.

### Activity
To follow are the transfer and journal activities for Multilateral DNS.

# Transfer 1 [A to B]
> DR on Participant A Liquidity account, followed by a CR on Participant B account of 100 units.
```
DR Participant A Liquidity                  100
    CR Settlement A Payable to Hub                      100
DR Settlement A Payable to Hub              100
    CR Settlement A Payable to B                        100
DR Settlement A Payable to B                100
    CR Participant B Liquidity                          100
```
# Transfer 2 [B to A]
> DR on Participant B Liquidity account, followed by a CR on Participant A account of 10 units.
```
DR Participant B Liquidity                  10
    CR Settlement B Payable to Hub                      10
DR Settlement B Payable to Hub              10
    CR Settlement B Payable to A                        10
DR Settlement B Payable to A                10
    CR Participant A Liquidity                          10
```
# Transfer 3 [B to C]
> DR on Participant B Liquidity account, followed by a CR on Participant C account of 20 units.
```
DR Participant B Liquidity                  20
    CR Settlement B Payable to Hub                      20
DR Settlement B Payable to Hub              20
    CR Settlement B Payable to C                        20
DR Settlement B Payable to C                20
    CR Participant C Liquidity                          20
```

#### Account Balances Transfers
> At this point, the transfers are complete, but settlement reservation has not yet occurred.
> All Settlement accounts have a `0` unit balance.
> 
| Account                         | Debits | Credits | Balance |
|---------------------------------|--------|---------|---------|
| `Opening Balances`              |        |         |         |
| **Participant A Liquidity**     | `0`    | `100`   | `100`   |------> Opening Balance <-------
| **Participant B Liquidity**     | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **Participant C Liquidity**     | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **Settlement A Payable to Hub** | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **Settlement B Payable to Hub** | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **Settlement A Payable to B**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **Settlement B Payable to A**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **Settlement B Payable to C**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| `Transfer-01`                   |        |         |         |
| Participant A Liquidity         | `100`  |         | `0`     |------> Transfer 1 <-------
| Settlement A Payable to Hub     | `100`  | `100`   | `0`     |--> Multilateral
| Settlement A Payable to B       | `100`  | `100`   | `0`     |--> Bilateral
| Participant B Liquidity         |        | `100`   | `100`   |--> A Credit B
| `Transfer-02`                   |        |         |         |
| Participant B Liquidity         | `10`   |         | `90`    |------> Transfer 2 <-------
| Settlement B Payable to Hub     | `10`   | `10`    | `0`     |--> Multilateral
| Settlement B Payable to A       | `10`   | `10`    | `0`     |--> Bilateral
| Participant A Liquidity         |        | `10`    | `10`    |--> B Credit A
| `Transfer-03`                   |        |         |         |
| Participant B Liquidity         | `20`   |         | `70`    |------> Transfer 3 <-------
| Settlement B Payable to Hub     | `20`   | `20`    | `0`     |--> Multilateral
| Settlement B Payable to C       | `20`   | `20`    | `0`     |--> Bilateral
| Participant C Liquidity         |        | `20`    | `20`    |--> B Credit C
> `Liquidity` accounts for participants balance. `Payable to Hub` to cover multilateral model.
> `[X] Payable to [Y]` accounts to cover bilateral model.


#### Settlement Reservation - [A to B], [B to A] and [B to C]
> The settlement reservation occurs later during the day as a batch process.
> Since the settlement is deferred, the settlement will occur as a batch process.
```
DR Settlement A Payable to Hub              100
    CR Settlement B Receivable from Hub                 100
DR Settlement B Payable to Hub              10
    CR Settlement A Receivable from Hub                 10
DR Settlement B Payable to Hub              20
    CR Settlement C Receivable from Hub                 20
```

#### Account Balances Settlement Reservation
> The Payable/Receivable accounts are utilized to track settlement reservation.
>
| Account                              | Debits  | Credits | Balance |
|--------------------------------------|---------|---------|---------|
| `Opening Balances`                   |         |         |         |
| **Settlement A Payable to Hub**      | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement B Payable to Hub**      | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement A Receivable from Hub** | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement B Receivable from Hub** | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Settlement C Receivable from Hub** | `0`     | `0`     | `0`     |------> Opening Balance <-------
| `Settlement-Reserve-01`              |         |         |         |------> Reserve 1 <-------
| Settlement A Payable to Hub          | `100`   |         | `-100`  |--> Payer
| Settlement B Receivable from Hub     |         | `100`   | `100`   |--> Payee
| `Settlement-Reserve-02`              |         |         |         |------> Reserve 2 <-------
| Settlement B Payable to Hub          | `10`    |         | `-10`   |--> Payer
| Settlement A Receivable from Hub     |         | `10`    | `10`    |--> Payee
| `Settlement-Reserve-03`              |         |         |         |------> Reserve 3 <-------
| Settlement B Payable to Hub          | `20`    |         | `-30`   |--> Payer
| Settlement C Receivable from Hub     |         | `20`    | `20`    |--> Payee
> `Settlement [X] Payable to [Y]`    = accounts to cover bilateral gross for payer.
> `Settlement [X] Receivable to [Y]` = accounts to cover bilateral gross for payee.



#### Settlement Commit - [A to B], [B to A] and [B to C]
> One may only be able to commit a previously reserved settlement.
```
DR Settlement B Receivable from Hub         100
    CR Settlement A Payable to Hub                      100
DR Settlement A Receivable from Hub         10
    CR Settlement B Payable to Hub                      10
DR Settlement C Receivable from Hub         20
    CR Settlement B Payable to Hub                      20
```

#### Account Balances Settlement Commit
> The Payable/Receivable accounts are now reset back to `0` post commit.
>
| Account                               | Debits  | Credits | Balance   |
|---------------------------------------|---------|---------|-----------|
| `Opening Balances`                    |         |         |           |
| **Settlement A Payable to Hub**       | `0`     | `0`     | `-100`    |------> Opening Balance <-------
| **Settlement B Payable to Hub**       | `0`     | `0`     | `-30`     |------> Opening Balance <-------
| **Settlement A Receivable from Hub**  | `0`     | `0`     | `10`      |------> Opening Balance <-------
| **Settlement B Receivable from Hub**  | `0`     | `0`     | `100`     |------> Opening Balance <-------
| **Settlement C Receivable from Hub**  | `0`     | `0`     | `20`      |------> Opening Balance <-------
| `Settlement-Commit-01`                |         |         |           |------> Commit 1 <-------
| Settlement A Payable to Hub           |         | `100`   | `0`       |--> Payer
| Settlement B Receivable from Hub      | `100`   |         | `0`       |--> Payee
| `Settlement-Commit-02`                |         |         |           |------> Commit 2 <-------
| Settlement B Payable to Hub           |         | `10`    | `-20`     |--> Payer
| Settlement A Receivable from Hub      | `10`    |         | `0`       |--> Payee
| `Settlement-Commit-03`                |         |         |           |------> Commit 3 <-------
| Settlement B Payable to Hub           |         | `20`    | `0`       |--> Payer
| Settlement C Receivable from Hub      | `20`    |         | `0`       |--> Payee
> `Settlement [X] Payable to [Y]`    = accounts to cover bilateral gross for payer.
> `Settlement [X] Receivable to [Y]` = accounts to cover bilateral gross for payee.

#### At this point we have deferred (*immediate also possible through configuration*) gross multilateral settlement for reservation is:

* Participant A receivable from Hub    = 10
* Participant A payable to Hub         = 100
* Participant B receivable from Hub    = 100
* Participant B payable to Hub         = 30
* Participant C receivable from Hub    = 20
* Participant C payable to Hub         = 0

#### Deferred net multilateral:

* A owes B      = 100 (gross)
* B owes A      = 10 (gross)
* B owes C      = 20 (gross)
* A owes B      = 90 (net)
* B owes C      = 20 (net)
* A owes Hub    = 100
* B owes Hub    = 30 (10A + 20C)
* C owes Hub    = 0










---
---
---
---
---
---
---
---
---
---
---
---
---
---
---




## Immediate Gross Bilateral:
## Deferred Net multilateral (between a participant and the hub):
## Immediate net multilateral (between a participant and the hub):


- Immediate gross bilateral
- Deferred net bilateral (between two participants)
- 
- Deferred net multilateral (between a participant and the hub)
- 
- Immediate net multilateral (between a participant and the hub)

> TODO Need to split up the layout to have models rather than a continues flow.
> MODEL PER HEADER (Expose the journey), also have an introduction per header.
> ALSO NEED A INITIAL INTRODUCTION.
> SETTLEMENT/SUMMARY REPORT

## Immediate gross bilateral:

# Transfer

# Transfer 1 [A to B]
```
DR Participant A Liquidity                  100
    CR Participant A Payable to Hub                     100
DR Participant A Payable to Hub             100
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


# Insight
TODO Split the headers up to display 

Settlement report (immediate gross bilateral):

* A paid to B = 100
* B paid to A = 10
* B paid to C = 20

Settlement report (deferred net bilateral):

* A owes B = 100 - 10 = 90
* B owes C = 20


Participant A opening balance of 100 units Credit.

| Account                      | Debits | Credits | Balance |
|------------------------------|--------|---------|---------|
| **Participant A Liquidity**  |        | `100`   | `100`   |------> Opening Balance <-------
| **Participant B Liquidity**  |        | `0`     | `0`     |------> Opening Balance <-------
| **Participant C Liquidity**  |        | `0`     | `0`     |------> Opening Balance <-------
| Participant A Liquidity      | `100`  |         | `0`     |------> Transfer 1 <-------
| Participant A Payable to Hub | `100`  | `100`   | `0`     |--> Multilateral
| Participant A Payable to B   | `100`  | `100`   | `0`     |--> Bilateral
| Participant B Liquidity      |        | `100`   | `100`   |--> A Credit B
| Participant B Liquidity      | `10`   |         | `90`    |------> Transfer 2 <-------
| Participant B Payable to Hub | `10`   | `10`    | `0`     |--> Multilateral
| Participant B Payable to A   | `10`   | `10`    | `0`     |--> Bilateral
| Participant A Liquidity      |        | `10`    | `10`    |--> B Credit A
| Participant B Liquidity      | `20`   |         | `70`    |------> Transfer 3 <-------
| Participant B Payable to Hub | `20`   | `20`    | `0`     |--> Multilateral
| Participant B Payable to C   | `20`   | `20`    | `0`     |--> Bilateral
| Participant C Liquidity      |        | `20`    | `20`    |--> B Credit C
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
