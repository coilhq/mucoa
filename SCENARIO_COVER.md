# Settlement Model Scenarios
The motivation behind this document stems from [MJL - Ledgers in the Hub](https://docs.mojaloop.io/mojaloop-business-docs/HubOperations/Settlement/ledgers-in-the-hub.html)
The purpose is to illustrate the chart of accounts (COA) for covering each of the required Mojaloop Settlement Models.

## TODO @jason, furthermore explain that the purpose of the document is to cover the journey of funds moving in MJL, and the tracking of those funds.

## Deposit
```
DR Bank                                     120
    CR Participant A Collateral                         120
DR Participant A Collateral                 120
    CR Participant A Liquidity                          120
```


## Definitions
| Definition  | Description                                                                          |
|-------------|--------------------------------------------------------------------------------------|
| Participant | A provider who is a member of a payment scheme, and subject to that scheme's rules.  |
| Hub         | The Mojaloop operator                                                                |
| Transfer    | A debit/credit from one account to another account                                   |


## Bilateral Immediate Gross Settlement (B-IGS)
Settlements are gross if each transfer is settled separately. 
*Gross* settlements may be immediate or deferred. 
They are deferred if approval for settlement from outside the Hub is required, and immediate if the Hub can proceed to settlement of a transfer without requiring any external approval. 
At present, Mojaloop only supports immediate gross settlements.
This is useful for SME environments where quick payment turnarounds are often desirable in order to maximize liquidity. 
IGS _(Immediate Gross Settlement)_ is also known as Real Time Gross Settlement (RTGS).

### Activity
To follow are the transfer and journal activities for IGS.

#### Transfer 1 [A to B]
> DR on Participant A Liquidity account, followed by a CR on Participant B account of 100 units.

# TODO @jason Remove the jump from Collateral to Liquidity (not necessary)
# TODO @jason account name `Payable`, should be `Cleared`

```
DR Participant A Liquidity                  100
    CR A Clearing B                                     100
DR A Clearing B                             100
    CR Participant B Liquidity                          100
```
#### Transfer 2 [B to A]
> DR on Participant B Liquidity account, followed by a CR on Participant A account of 10 units.
```
DR Participant B Liquidity                  10
    CR B Clearing A                                     10
DR B Clearing A                             10
    CR Participant A Liquidity                          10
```
#### Transfer 3 [B to C]
> DR on Participant B Liquidity account, followed by a CR on Participant C account of 20 units.
```
DR Participant B Liquidity                  20
    CR B Clearing C                                     20
DR B Clearing C                             20
    CR Participant C Liquidity                          20
```

#### Account Balances
| Account                         | Debits   | Credits  | Balance   |
|---------------------------------|----------|----------|-----------|
| `Opening Balances`              |          |          |           |
| **Participant A Collateral**    | `0`      | `0`      | `0`       |------> Opening Balance <-------
| **Participant B Collateral**    | `0`      | `0`      | `0`       |------> Opening Balance <-------
| **Participant C Collateral**    | `0`      | `0`      | `0`       |------> Opening Balance <-------
| **Participant A Liquidity**     | `0`      | `100`    | `100`     |------> Opening Balance <-------
| **Participant B Liquidity**     | `0`      | `0`      | `0`       |------> Opening Balance <-------
| **Participant C Liquidity**     | `0`      | `0`      | `0`       |------> Opening Balance <-------
| **A Payable to B**              | `0`      | `0`      | `0`       |------> Opening Balance <-------
| **B Payable to A**              | `0`      | `0`      | `0`       |------> Opening Balance <-------
| **B Payable to C**              | `0`      | `0`      | `0`       |------> Opening Balance <-------
| `Transfer 1`                    |          |          |           |
| Participant A Liquidity         | `100`    |          | `0`       |------> Transfer 1 <-------
| A Payable to B                  | `100`    | `100`    | `0`       |--> Bilateral (Recording)
| Participant B Collateral        | `100`    | `100`    | `0`       |--> Bilateral (Recording)
| Participant B Liquidity         |          | `100`    | `100`     |--> A Credit B
| `Transfer 2`                    |          |          |           |
| Participant B Liquidity         | `10`     |          | `90`      |------> Transfer 2 <-------
| B Payable to A                  | `10`     | `10`     | `0`       |--> Bilateral (Recording)
| Participant A Collateral        | `10`     | `10`     | `0`       |--> Bilateral (Recording)
| Participant A Liquidity         |          | `10`     | `10`      |--> B Credit A
| `Transfer 3`                    |          |          |           |
| Participant B Liquidity         | `20`     |          | `70`      |------> Transfer 3 <-------
| B Payable to C                  | `20`     | `20`     | `0`       |--> Bilateral (Recording)
| Participant C Collateral        | `20`     | `20`     | `0`       |--> Bilateral (Recording)
| Participant C Liquidity         |          | `20`     | `20`      |--> B Credit C
> `Liquidity` - Accounts for participants balance. 
> 
> `[X] Payable to [Y]` - Accounts to cover bilateral gross. 

#### Settlement Report - Immediate Gross Bilateral:
* A paid to B = 100
* B paid to A = 10
* B paid to C = 20

---

## Bilateral Deferred Net Settlement (B-DNS)
Settlements are deferred net if a number of transfers are settled together. 
Net Settlements _(in which a number of transfers are settled together)_ are by definition deferred _(since it takes time to construct a batch)_.
B-DNS's are deferred due to the clearing occurring at a later stage. 

### Activity
To follow are the transfer and journal activities for Bilateral DNS _(Deferred Net Settlement)_.

#### Transfer 1 [A to B]
> DR on Participant A Liquidity account, followed by a CR on Participant B account of 100 units.
```
DR Participant A Liquidity                  100
    CR A Payable to B                                   100
DR A Payable to B                           100
    CR Participant B Collateral                         100
DR Participant B Collateral                 100
    CR Participant B Liquidity                          100
```
#### Transfer 2 [B to A]
> DR on Participant B Liquidity account, followed by a CR on Participant A account of 10 units.
```
DR Participant B Liquidity                  10
    CR B Payable to A                                   10
DR B Payable to A                           10
    CR Participant A Collateral                         10
DR Participant A Collateral                 10
    CR Participant A Liquidity                          10
```
#### Transfer 3 [B to C]
> DR on Participant B Liquidity account, followed by a CR on Participant C account of 20 units.
```
DR Participant B Liquidity                  20
    CR B Payable to C                                   20
DR B Payable to C                           20
    CR Participant C Collateral                         20
DR Participant C Collateral                 20
    CR Participant C Liquidity                          20
```

#### Account Balances - Transfers
> At this point, the transfers are complete, but settlement reservation has not yet occurred. 
> All Settlement accounts have a `0` unit balance. 
> 
| Account                      | Debits   | Credits    | Balance |
|------------------------------|----------|------------|---------|
| `Opening Balances`           |          |            |         |
| **Participant A Collateral** | `0`      | `0`        | `0`     |------> Opening Balance <-------
| **Participant B Collateral** | `0`      | `0`        | `0`     |------> Opening Balance <-------
| **Participant C Collateral** | `0`      | `0`        | `0`     |------> Opening Balance <-------
| **Participant A Liquidity**  | `0`      | `100`      | `100`   |------> Opening Balance <-------
| **Participant B Liquidity**  | `0`      | `0`        | `0`     |------> Opening Balance <-------
| **Participant C Liquidity**  | `0`      | `0`        | `0`     |------> Opening Balance <-------
| `Transfer-01`                |          |            |         |
| Participant A Liquidity      | `100`    |            | `0`     |------> Transfer 1 <-------
| A Payable to B               | `100`    | `100`      | `0`     |--> Bilateral (Recording)
| Participant B Collateral     | `100`    | `100`      | `0`     |--> Recording
| Participant B Liquidity      |          | `100`      | `100`   |--> A Credit B
| `Transfer-02`                |          |            |         |
| Participant B Liquidity      | `10`     |            | `90`    |------> Transfer 2 <-------
| B Payable to A               | `10`     | `10`       | `0`     |--> Bilateral (Recording)
| Participant A Collateral     | `10`     | `10`       | `0`     |--> Recording
| Participant A Liquidity      |          | `10`       | `10`    |--> B Credit A
| `Transfer-03`                |          |            |         |
| Participant B Liquidity      | `20`     |            | `70`    |------> Transfer 3 <-------
| B Payable to C               | `20`     | `20`       | `0`     |--> Bilateral (Recording)
| Participant C Collateral     | `20`     | `20`       | `0`     |--> Bilateral (Recording)
| Participant C Liquidity      |          | `20`       | `20`    |--> B Credit C
> `Liquidity` = accounts for participants balance.
> `[X] Payable to [Y]` = accounts to cover bilateral gross.


#### Clearing Reservation - [A to B], [B to A] and [B to C] 
> The clearing reservation occurs later during the day as a batch process.
> B-DNS will usually occur as a batch process.
> The clearing reservation and commit will form part of a 2-phase Transfer process.

# TODO @jason A Payable to B should be an [Settlement/Bank] Account
# [A Clearing B] and [A Settlement B]

```
DR Participant A Collateral                 100
    CR A Payable to B                                   100
DR Participant B Collateral                 10
    CR B Payable to A                                   10
DR Participant B Collateral                 20
    CR B Payable to C                                   20
```

#### Account Balances - Clearing Reservation
> The Payable/Receivable accounts are utilized to track clearing reservation.  
> 
| Account                       | Debits  | Credits | Balance |
|-------------------------------|---------|---------|---------|
| `Opening Balances`            |         |         |         |
| **Participant A Collateral**  | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Participant B Collateral**  | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **Participant C Collateral**  | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **A Payable to B**            | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **B Payable to A**            | `0`     | `0`     | `0`     |------> Opening Balance <-------
| **B Payable to C**            | `0`     | `0`     | `0`     |------> Opening Balance <-------
| `Clearing-Reserve 1`          |         |         |         |------> Reserve 1 <-------
| Participant A Collateral      | `100`   |         | `-100`  |--> Payer
| A Payable to B                |         | `100`   | `100`   |--> Payee
| `Clearing-Reserve 2`          |         |         |         |------> Reserve 2 <-------
| Participant B Collateral      | `10`    |         | `-10`   |--> Payer
| B Payable to A                |         | `10`    | `10`    |--> Payee
| `Clearing-Reserve 3`          |         |         |         |------> Reserve 3 <-------
| Participant B Collateral      | `20`    |         | `-30`   |--> Payer
| B Payable to C                |         | `20`    | `20`    |--> Payee
> `[X] Payable to [Y]`    = accounts to cover bilateral gross for payer/payee.
> 
> Risk is managed by the `Participant [X] Collateral` debit balance (Debits may not exceed Credits).

#### Clearing Commit - [A to B], [B to A] and [B to C]
> One may only be able to commit a previously reserved clearing batch.
> The act of committing the clearing batch, will transition the `pending` units to `posted` units.


#### Settlement Reservation - [A to B], [B to A] and [B to C]
> The order of the clearing/settlement is not important.
> Once settlement occurs, the Payable units are debited and collateral is credited.
                
# TODO @jason A Payable to B should be an [Settlement/Bank/Clearing] Account

```
DR A Payable to B                           100
    CR Participant A Collateral                         100
DR B Payable to A                           10
    CR Participant B Collateral                         10
DR B Payable to C                           20
    CR Participant B Collateral                         20
```

#### Account Balances - Settlement Reservation
> The Payable/Receivable accounts are utilized to track settlement reservation.
>
| Account                      | Debits | Credits | Balance |
|------------------------------|--------|---------|---------|
| `Opening Balances`           |        |         |         |
| **Participant A Collateral** | `0`    | `0`     | `-100`  |------> Opening Balance <-------
| **Participant B Collateral** | `0`    | `0`     | `-30`   |------> Opening Balance <-------
| **Participant C Collateral** | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Payable to B**           | `0`    | `0`     | `100`   |------> Opening Balance <-------
| **B Payable to A**           | `0`    | `0`     | `10`    |------> Opening Balance <-------
| **B Payable to C**           | `0`    | `0`     | `20`    |------> Opening Balance <-------
| `Settlement-Reserve 1`       |        |         |         |------> Reserve 1 <-------
| Participant A Collateral     |        | `100`   | `0`     |--> Payer
| A Payable to B               | `100`  |         | `0`     |--> Payee
| `Settlement-Reserve 2`       |        |         |         |------> Reserve 2 <-------
| Participant B Collateral     |        | `10`    | `-20`   |--> Payer
| B Payable to A               | `10`   |         | `0`     |--> Payee
| `Settlement-Reserve 3`       |        |         |         |------> Reserve 3 <-------
| Participant B Collateral     |        | `20`    | `0`     |--> Payer
| B Payable to C               | `20`   |         | `0`     |--> Payee
> `[X] Payable to [Y]`    = accounts to cover bilateral gross for payer/payee.

#### Settlement Commit - [A to B], [B to A] and [B to C]
> One may only be able to commit a previously reserved settlement batch.
> The act of committing the settlement batch, will transition the `pending` units to `posted` units.
> All Settlement accounts (`[X] Payable to [Y]`) have a `0` unit balance.

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


#### Transfer 1 [A to B]
> DR on Participant A Liquidity account, followed by a CR on Participant B account of 100 units.
```
DR Participant A Liquidity                  100
    CR A Clearing Hub                                   100
DR A Clearing Hub                           100
    CR A Clearing B                                     100
DR A Clearing B                             100
    CR Participant B Liquidity                          100
```
#### Transfer 2 [B to A]
> DR on Participant B Liquidity account, followed by a CR on Participant A account of 10 units.
```
DR Participant B Liquidity                  10
    CR B Clearing Hub                                   10
DR B Clearing Hub                           10
    CR B Clearing A                                     10
DR B Clearing A                             10
    CR Participant A Liquidity                          10
```
#### Transfer 3 [B to C]
> DR on Participant B Liquidity account, followed by a CR on Participant C account of 20 units.
```
DR Participant B Liquidity                  20
    CR B Clearing Hub                                   20
DR B Clearing Hub                           20
    CR B Clearing C                                     20
DR B Clearing C                             20
    CR Participant C Liquidity                          20
```

#### Account Balances - Transfers
> At this point, the transfers are complete, but settlement reservation has not yet occurred.
> All Settlement accounts have a `0` unit balance.
> 
| Account                     | Debits | Credits | Balance |
|-----------------------------|--------|---------|---------|
| `Opening Balances`          |        |         |         |
| **Participant A Liquidity** | `0`    | `100`   | `100`   |------> Opening Balance <-------
| **Participant B Liquidity** | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **Participant C Liquidity** | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Payable to Hub**        | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **B Payable to Hub**        | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Payable to B**          | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **B Payable to A**          | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **B Payable to C**          | `0`    | `0`     | `0`     |------> Opening Balance <-------
| `Transfer-01`               |        |         |         |
| Participant A Liquidity     | `100`  |         | `0`     |------> Transfer 1 <-------
| A Payable to Hub            | `100`  | `100`   | `0`     |--> Multilateral
| A Payable to B              | `100`  | `100`   | `0`     |--> Bilateral
| Participant B Collateral    | `100`  | `100`   | `0`     |--> Recording
| Participant B Liquidity     |        | `100`   | `100`   |--> A Credit B
| `Transfer-02`               |        |         |         |
| Participant B Liquidity     | `10`   |         | `90`    |------> Transfer 2 <-------
| B Payable to Hub            | `10`   | `10`    | `0`     |--> Multilateral
| B Payable to A              | `10`   | `10`    | `0`     |--> Bilateral
| Participant A Collateral    | `10`   | `10`    | `0`     |--> Recording
| Participant A Liquidity     |        | `10`    | `10`    |--> B Credit A
| `Transfer-03`               |        |         |         |
| Participant B Liquidity     | `20`   |         | `70`    |------> Transfer 3 <-------
| B Payable to Hub            | `20`   | `20`    | `0`     |--> Multilateral
| B Payable to C              | `20`   | `20`    | `0`     |--> Bilateral
| Participant C Collateral    | `20`   | `20`    | `0`     |--> Recording
| Participant C Liquidity     |        | `20`    | `20`    |--> B Credit C
> `Liquidity` accounts for participants balance. `Payable to Hub` to cover multilateral model.
> `[X] Payable to [Y]` accounts to cover bilateral model.


#### Clearing Reservation - [A to B], [B to A] and [B to C]
> The clearing reservation occurs later during the day as a batch process.
> Since the clearing is deferred, the clearing will occur as a batch process.
> The clearing reservation and commit will form part of a 2-phase Transfer process.

```
DR Participant A Collateral                 100
    CR A Clearing B                                     100
DR Hub Settlement                           100
    CR A Clearing Hub                                   100
DR Participant B Collateral                 10
    CR B Clearing A                                     10
DR Hub Settlement                           10
    CR B Clearing Hub                                   10
DR Participant B Collateral                 20
    CR B Clearing C                                     20
DR Hub Settlement                           20
    CR B Clearing Hub                                   20
```

#### Account Balances - Clearing Reservation
> The Payable/Receivable accounts are utilized to track settlement reservation.
>
| Account                        | Debits | Credits | Balance |
|--------------------------------|--------|---------|---------|
| `Opening Balances`             |        |         |         |
| **Participant A Collateral**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **Participant B Collateral**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **Participant C Collateral**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Payable to Hub**           | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **B Payable to Hub**           | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **Hub Recon**                  | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Payable to B**             | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **B Payable to A**             | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **B Payable to C**             | `0`    | `0`     | `0`     |------> Opening Balance <-------
| `Clearing-Reserve 1`           |        |         |         |------> Reserve 1 <-------
| Participant A Collateral       | `100`  |         | `-100`  |--> Payer
| A Payable to B                 |        | `100`   | `100`   |--> Bilateral
| A Payable to Hub               |        | `100`   | `100`   |--> Multilateral
| Hub Recon                      | `100`  |         | `-100`  |--> Multilateral
| `Clearing-Reserve 2`           |        |         |         |------> Reserve 2 <-------
| Participant B Collateral       | `10`   |         | `-10`   |--> Payer
| B Payable to A                 |        | `10`    | `10`    |--> Bilateral
| B Payable to Hub               |        | `10`    | `10`    |--> Multilateral
| Hub Recon                      | `10`   |         | `-110`  |--> Multilateral
| `Clearing-Reserve 3`           |        |         |         |------> Reserve 3 <-------
| Participant B Collateral       | `20`   |         | `-30`   |--> Payer
| B Payable to C                 |        | `20`    | `20`    |--> Bilateral
| B Payable to Hub               |        | `20`    | `30`    |--> Multilateral
| Hub Recon                      | `20`   |         | `-130`  |--> Multilateral
> `[X] Payable to [Y]`    = accounts to cover bilateral gross for payer/payee.
> `[X] Payable to Hub`    = accounts to cover multilateral gross for payer/payee.

#### Clearing Commit - [A to B], [B to A] and [B to C]
> One may only be able to commit a previously reserved clearing batch.
> The act of committing the clearing batch, will transition the `pending` units to `posted` units.

#### Settlement Reservation - [A to B], [B to A] and [B to C]
> The order of the clearing/settlement is not important.
> Once settlement occurs, the Payable units are debited and collateral is credited.
        
# TODO @jason - [Hub Recon] should be [Clearing Scheme]

```
DR A Clearing B                             100
    CR Participant A Collateral                         100
DR A Clearing Hub                           100
    CR Hub Recon                                        100
DR B Clearing A                             10
    CR Participant B Collateral                         10
DR B Clearing Hub                           10
    CR Hub Recon                                        10
DR B Clearing C                             20
    CR Participant B Collateral                         20
DR B Clearing Hub                           20
    CR Hub Recon                                        20
```

#### Account Balances - Settlement Reservation
> The Payable/Receivable accounts are utilized to track settlement reservation.
> The Payable/Receivable accounts are utilized to track settlement reservation.
>
| Account                       | Debits | Credits | Balance |
|-------------------------------|--------|---------|---------|
| `Opening Balances`            |        |         |         |
| **Participant A Collateral**  | `0`    | `0`     | `-100`  |------> Opening Balance <-------
| **Participant B Collateral**  | `0`    | `0`     | `-30`   |------> Opening Balance <-------
| **Participant C Collateral**  | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Payable to Hub**          | `0`    | `0`     | `100`   |------> Opening Balance <-------
| **B Payable to Hub**          | `0`    | `0`     | `30`    |------> Opening Balance <-------
| **Hub Recon**                 | `0`    | `0`     | `-130`  |------> Opening Balance <-------
| **A Payable to B**            | `0`    | `0`     | `100`   |------> Opening Balance <-------
| **B Payable to A**            | `0`    | `0`     | `10`    |------> Opening Balance <-------
| **B Payable to C**            | `0`    | `0`     | `20`    |------> Opening Balance <-------
| `Settlement-Reserve 1`        |        |         |         |------> Reserve 1 <-------
| Participant A Collateral      |        | `100`   | `0`     |--> Payer
| A Payable to B                | `100`  |         | `0`     |--> Bilateral
| A Payable to Hub              | `100`  |         | `0`     |--> Multilateral
| Hub Recon                     |        | `100`   | `-30`   |--> Multilateral
| `Settlement-Reserve 2`        |        |         |         |------> Reserve 2 <-------
| Participant B Collateral      |        | `10`    | `-20`   |--> Payer
| B Payable to A                | `10`   |         | `0`     |--> Bilateral
| B Payable to Hub              | `10`   |         | `20`    |--> Multilateral
| Hub Recon                     |        | `10`    | `-20`   |--> Multilateral
| `Settlement-Reserve 3`        |        |         |         |------> Reserve 3 <-------
| Participant B Collateral      |        | `20`    | `0`     |--> Payer
| B Payable to C                | `20`   |         | `0`     |--> Bilateral
| B Payable to Hub              | `20`   |         | `0`     |--> Multilateral
| Hub Recon                     |        | `20`    | `0`     |--> Multilateral
> `[X] Payable to [Y]`    = accounts to cover bilateral gross for payer/payee.
> `[X] Payable to Hub`    = accounts to cover multilateral gross for payer/payee.

#### Settlement Commit - [A to B], [B to A] and [B to C]
> One may only be able to commit a previously reserved settlement batch.
> The act of committing the settlement batch, will transition the `pending` units to `posted` units.
> All Settlement accounts (`[X] Payable to [Y]`) have a `0` unit balance.

#### Settlement Report - Multilateral Deferred Net Settlement (M-DNS)
> At this point we have deferred (*immediate also possible through configuration*) gross multilateral settlement for reservation is:

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


> A settlement model specifies a way in which a Mojaloop Hub will settle a set of transfers. 
> In the simple case, there is only one settlement model and it settles all the transfers that are processed by the Hub. 
> However, Mojaloop supports more than one settlement model for a single scheme. 
> This allows, for instance, a scheme to define different settlement models for different currencies, or for different ledger account types.
