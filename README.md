# Chart of Accounts
The COA (Chart of Accounts) consists of all the accounts required to operate the Mojaloop Hub.
> TODO add more intro....

## Events
* [Participant Joins the Hub](#participant-joins-the-hub)
* [Participant Deposit Collateral to Hub](#participant-deposits-collateral-to-scheme)
* [Payer Transfer units to a Payee](#payer-transfer-units-to-a-payee)
* [Settlement Reservation](#settlement-reservation)
* [Settlement Commit](#settlement-commit)
* [Participant Withdraw Collateral from Hub](#participant-withdraw-collateral-from-scheme)
* [Participant Close Account](#participant-close-account)

## Definitions
| Definition  | Description                                                                          |
|-------------|--------------------------------------------------------------------------------------|
| Participant | A provider who is a member of a payment scheme, and subject to that scheme's rules.  |
| Hub         | The Mojaloop operator                                                                |
| Transfer    | A debit/credit from one account to another account                                   |


## Participant Joins the Hub
A new participant joins the scheme and the necessary accounts and configurations are provisioned in the system.
At this time the participant has no liquidity _(a current position of zero and a net debit cap of zero)_.

### Entities
* Hub - (Hub operator)
* Participant A - (Participant on the Hub that may be a Payer/Payee)

### Activity
Participant with relevant information is captured, but no accounts have been created for the participant.

## Participant Deposits Collateral to Hub
A participant decides to deposit 100 units collateral into the Hub.

### Accounts
* **Deposit** - Account recording the deposit of units
* **Collateral** - Account recording the collateral deposited by the participant
* **Liquidity** - Account recording the liquidity available for the participant
* **Fee** - Account recording fees charged by the hub

### Activity
A cash deposit is made at a bank that supports loading of assets into the Hub. The hub is notified of the payment.
The necessary accounts are created  

#### Participant A Deposits Collateral (Bank to Hub, then Hub to Participant):
> The handing of assets occur outside the hub. A notification event is sent to the hub to inform the hub of the deposit.
> (either all collateral they have, or at most N units, in this case at most 100 units)

```
DR A Deposit                                    110
    CR A Collateral                                       110
DR A Collateral                                 110
    CR A Liquidity                                        110
```

#### Hub charges Fees on Deposit of Collateral:
> (where fee is a function of collateral deposit amount = 120 * 9.1% = 10)
```
DR A Liquidity                                  10
    CR A Fees                                             10
```

### Account Balances
| Account            | Debits | Credits | Balance |
|--------------------|--------|---------|---------|
| `Opening Balances` |        |         |         |
| **A Deposit**      | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Collateral**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Liquidity**    | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Fees**         | `0`    | `0`     | `0`     |------> Opening Balance <-------
| `Deposit 1`        |        |         |         |------> Deposit 1 <-------
| A Deposit          | `110`  |         | `-110`  |--> Debit for Hub
| A Collateral       | `110`  | `110`   | `0`     |--> Collateral (Recording)
| A Liquidity        |        | `110`   | `110`   |--> Liquidity Available
| A Liquidity        | `10`   |         | `100`   |--> Fee Charge on A
| A Fees             |        | `10`    | `10`    |--> Fee Recorded for Hub 


### Summary
* A Deposit DR balance of; `0 + 110`
  * `110` units deposit at the bank; `0 - 110 = -110`
* A's Collateral has a CR balance of; `0 + 110 - 110 = 0`
  * `110` units for deposit, minus `110` to be made available for liquidity
* A's Liquidity has a CR balance of; `0 + 110 - 10 = 100`
    * `100` units for initial release, minus `10` for fee
* A's Fees has a CR balance of; `0 + 10 = 10`
  * `10` units fee charge for deposit on liquidity

## Transfer (Clearing)

Participant A (Payer) would like to transfer funds to participant B (Payee, these are linked transfers).
At this time the Payer participant A has liquidity of `100` units liquidity.
Participant A would like to transfer `100` units to Participant B.

The liquidity from A to B is immediately available to B, however, 
the settlement reservation and commit is a separate action that is between A to Hub and B from Hub.

### Accounts
* Participant A Liquidity - Pay Participant - B (Payer)
* Participant B Liquidity - Receive payment from Participant - A (Payee)
* Participant A Payable to Hub - Existing or on demand account to record the transfer from A to B
* Participant A Fees - Participant fee debit

### Activity
A pays B = 70, these are linked transfers

#### Participant A Transfer units to Participant B (Payer to Payee direct):
> Hub allows for direct liquidity to Payee.
```
DR A Liquidity                                  70
    CR A Clearing (B)                                     70
DR A Clearing (B)                               70
    CR B Liquidity                                        70
```

### Summary
* A's Liquidity has a CR balance of;  `100 - 70 = 30`
  * `70` units transferred to B
* A's Clearing B has a DR/CR balance of;  `0 + 70 - 70 = 0`
  * `160` units minus `100` for transfer and `10` for the Hub fee


### Invariants
* A transfer will typically be performed as a 2phase commit
  * The liquidity will be reserved until a commit on the transfer is performed
  * If no commit is received within the provided timeout, the units will be reversed (B back to scheme and scheme back to A)
* In the event of a rollback (timeout/force), the transfer will be rolled back


## Fee Settlement
Fee's charged to Participant from Hub needs to be settled. The settling of fees may be performed as an 
immediate or deferred activity.
The Fee is debited from the Participant's liquidity to ensure out-of-pocket spending is not possible. 

### Accounts
* Participant A Fees - The account tracking the transfer fees charged by the scheme against A
* Participant A Payable to Hub - Hub operator entity used for Hub Fee collateral

### Activity
A batch job is ran at the end of day to settle all outstanding fees.

#### Fees for Participant A is Settled
```
DR Participant A Payable to Hub           10
    CR Participant A Fees                           10
DR Participant A Payable to Hub           10
    CR Participant A Fees                           10
```

### Summary
* Participant A Payable to Hub has a balance of; `20 - (10 + 10) = 0`
  * `10` units Fee charge for A's deposit (existing)
  * `10` units Fee charge for A's transfer to B (existing)
  * `20` units settled against A Fee account
* `Do we want to settle against another Participant A Fees account? @jorangreef`

## Settlement - Reservation
An existing transfer between Payer and Payee has been completed successfully. The purpose of the Settlement
reservation is the instruction to reserve the transfer settlement for the Payer and Payee as collateral (_On the previously successful transfer_).
The reservation restricts the Payer from any Funds-Out (_Withdraw of funds from Payer account_) operations.

The settlement reservation may be performed on a per-transaction basis, or as a batch operation (immediate or deferred).

### Accounts
* Participant A Payable To Hub - Payer settlement reserve
* Participant B Receivable From Hub - Hub Reconciliation for Payee (Hub operator)
      
### Activity
#### Transfer Settlement Reservation (On Payer):
> An external event or Hub trigger will instruct the Settlement Reservation event.
```
DR Participant A Payable To Hub                 100
    CR Participant B Receivable From Hub                    100
```

### Summary
* A's Payable to Hub DR balance of;  `0 + 100 = 100`
  * `100` units debited for settlement
  * A negative balance indicates outstanding settlements from A (payable)
* B's Receivable from Hub CR balance of;  `0 + 100 = 100`
  * `100` units as credit to the recon
  * A positive balance indicates outstanding settlements from B (receivable)

### Invariants
> Do we want to have a timeout set with 2phase commit for the DR on Payer? <br/> How long will we wait for settlement?


## Settlement - Commit
An existing settlement reservation for a Payer has been completed successfully (a current reserved settlement of `100` units).
The purpose of the Settlement commit is the instruction to finalise the transfer settlement between Payer and the Hub Recon.
The settlement commit would likely be process as part of a batch.
> The settlement commit completes the lifecycle of ownership for the digital/physical asset.

### Accounts
* Participant A Payable To Hub - Payer settlement reserve
* Participant B Receivable From Hub - Hub Reconciliation for Payee (Hub operator)

### Activity
#### 1. Committing Transfer for Payee and Hub Recon:
```
DR Participant B Receivable From Hub            100
    CR Participant A Payable To Hub                         100
```

### Summary
* A's Payable to Hub DR balance of;  `100 - 100 = 0`
  * `100` units credited for settlement commit
  * A balance of `0` indicates no outstanding settlements from A (payable)
* B's Receivable from Hub CR balance of;  `100 - 100 = 0`
  * `100` units as debit to the recon
  * A balance of `0` indicates no outstanding settlements from Hub (receivable)

### Invariants
- No commit would result in a positive debit balance for A to Hub
- No commit would result in a positive credit balance for Hub to B

## Participant Withdraw Collateral from Hub
An existing participant A would like to withdraw `units` from the Hub.

### Entities
* Bank (External)
* Hub Collateral - Hub operator collateral
* Participant Liquidity A - The participants liquidity on the Hub

#### 1. Transfer the Participant Liquidity to the Hub Collateral:
```
DR Participant Liquidity A                      100
    CR Hub Collateral                                     100
```

#### 2. Transfer the Hub Collateral to the Bank:
> The Hub to the Bank transfer occurs outside of Hub Hub
```
DR Hub Collateral                               100
    CR Bank                                               100
```

### Summary
* A's Liquidity has a net CR balance of;  `100 - 100 = 0`
    * `100` units debited to the bank where A will receive 'cash'
* S's Collateral has a net CR balance of;  `0 + 100 - 100 = 0`
    * `100` units from A, then `100` units to bank
* Bank settles with Hub outside of Hub hub


## Participant Close Account
An account may only be closed when the DR/CR for a participant is `0` units.

### Entities
>>>> TODO


## NOTES

* How do we link the accounts in TB;
    * Create accounts with the same `user_data` per Participant, then mark them as linked?
    * For each account type, we use `code`, do we leave it up to Hub to deal with the multiple currencies, or treat is as part of the `code` makeup.

* Do we care about account state in TB, or leave it up to Hub?
* 
