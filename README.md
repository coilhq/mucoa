# Chart of Accounts

## Events
* [Participant Joins the Scheme](#participant-joins-the-scheme)
* [Participant Deposit Collateral to Scheme](#participant-deposits-collateral-to-scheme)
* [Payer Transfer units to a Payee](#payer-transfer-units-to-a-payee)
* [Settlement Reservation](#settlement-reservation)
* [Settlement Commit](#settlement-commit)
* [Participant Withdraw Collateral from Scheme](#participant-withdraw-collateral-from-scheme)
* [Participant Close Account](#participant-close-account)



## Participant Joins the Scheme
A new participant joins the scheme and the necessary accounts and configurations are provisioned in the system.
At this time the participant has no liquidity _(a current position of zero and a net debit cap of zero)_.

### Entities
* Scheme - (Scheme operator)
* Participant A - (Participant on the Scheme that may be a Payer/Payee)

### Activity
Account is created with a `0` unit balance (DR/CR).

## Participant Deposits Collateral to Scheme
A participant decides to deposit 100 units collateral into the Scheme hub.

### Entities
* Bank (External)

### Accounts
* Scheme Collateral - Scheme operator collateral
* Scheme Liquidity - Scheme operator liquidity
* Scheme Fees - Scheme operator entity used for Scheme Fee collateral
* Participant Liquidity A - The participants liquidity on the Scheme
* Participant Collateral A - Deposit collateral into the Scheme
* Participant A Fees - The account tracking the transfer fees charged by the scheme

### Activity
A cash deposit is made at a bank that supports loading of assets into the Scheme hub.
The hub is notified of the payment.

#### Participant A Deposits Collateral (Bank to Scheme, then Scheme to Participant):
> The handing of assets occur outside the hub. A notification event is sent to the hub to 
> inform the hub of the deposit.
```
DR Bank                               120
    CR Participant A Collateral                 120
```

#### Scheme decides to charge Fees on Deposit of Collateral:
> (where fee is a function of collateral deposit amount = 120 * 8.333% = 10)
```
DR Participant A Collateral           10
    CR Participant A Fees                       10
DR Participant A Fees                 10
    CR Scheme Fees                              10
```

#### `(Repeatable)` Participant A decides to make 100 collateral available as liquidity:
> (either all collateral they have, or at most N units, in this case at most 100 units)
```
DR Participant A Collateral           100
    CR Participant A Liquidity                100
```
#### `(Repeatable)` Participant A decides to make 10 collateral available as liquidity:
> (Secondary event debits the last 10 units of collateral from the Participant A). <br/>
> At this point the collateral is `0`
```
DR Participant A Collateral           10
    CR Participant A Liquidity                  10
```

#### Scheme extends Credit to A as a function of A's collateral deposit event amount:
> (A has a net debit cap of 150 according to rules of scheme)

```
DR Scheme Liquidity                   50
    CR Participant A Liquidity                  50
```


### Summary
* A's Liquidity has a CR balance of; `100 + 10 + 50 = 160`
    * `100` units for initial release, plus additional `10`, final additional `50` as credit line to participant
    * `50` of the `160` units were made available by the Scheme's credit line
* A's Collateral has a CR balance of; `120 - 10 - 100 - 10 = 0`
    * `120` units for deposit, minus `10` for deposit fee, minus `100` then `10` for liquidity release
* A's Fees has a DR balance of; `0 + 10 - 10 = 0`
  * `10` units fee charge for deposit, followed by `10` fee units credited to Scheme Fees 
* Scheme's Liquidity has a DR balance of; `0 + 50 = 50`
    * Scheme extends `50` units of liquidity to A
    * Note that the scheme limit is set at 150 units liquidity cap, current usage is at 33%
* Scheme's Collateral has a CR balance of; `0 + 120 - 120 = 0`
    * `120` units from Bank deposit, minus `120` units to A's collateral
* Scheme Fees has a CR balance of; `0 + 10`
    * `10` units Fee charge for A's deposit

### Invariants
> //TODO

## Transfer
Participant A (Payer) would like to transfer funds to participant B (Payee).
At this time the Payer participant A has liquidity of 160 units _(and a gross debit cap of 200)_.
Participant A would like to transfer `100` units to Participant B.
The Scheme Fee has a CR balance of `10` units (from a previous Participant deposit).

### Accounts
* Scheme Fees - Fees being charged by the Scheme
* Participant A Liquidity - Pay Participant - B (Payer)
* Participant B Liquidity - Receive payment from Participant - A (Payee)
* Participant A Payable to B - Existing or on demand account to record the transfer from A to B
  * [Immediate Gross]  - 
  * [Bilateral Net]    - 
  * [Multilateral Net] - 


### Activity
#### Transfer from Participant A to Participant B
>>>> TODO Write down examples of the above for each.

#### Participant A Transfer units to Participant B (Payer to Payee direct):
> Scheme allows for direct liquidity to Payee.
```
DR Participant A Liquidity            100
    CR Participant A Payable to B                  100
DR Participant A Payable to B         100    
    CR Participant B Liquidity                     100
```

#### Participant A is charged with a `10%` Transfer fee (100 * 0.10 = 10):
```
DR Participant A Liquidity            10
    CR Participant A Fees                          10
DR Participant A Fees                 10
    CR Scheme Fees                                 10
```

### Summary
* A's Liquidity has a DR balance of;  `0 + 10 + 100 = 110`
  * `10` units for fee and `100` units for transfer
* A's Liquidity has a CR balance of;  `160 - 100 - 10 = 50`
  * `160` units minus `100` for transfer and `10` for the Scheme fee
* A's Fees has a DR balance of; `0 + 10 - 10 = 0`
  * `10` units fee charge for transfer, followed by `10` fee units credited to Scheme Fees
* Participant A Payable to B has a CR balance of; `0 + 100`
  * `100` units transferred to B
  * The `100` units will be debited once units are settled
* B's Liquidity has a CR balance of; `0 + 100 = 100`
  * `100` units received from A
* Scheme Fees Liquidity has a net CR balance of;  `10 + 10 = 20`
  * `10` existing, followed by another `10` for transfer fee

### Invariants
* A transfer will typically be performed as a 2phase commit
    * The liquidity will be reserved until a commit on the transfer is performed
    * If no commit is received within the provided timeout, the units will be reversed
* In the event of a rollback (timeout/force), the transfer will be rolled back


## Settlement Reservation
An existing transfer between Payer and Payee has been completed successfully. The purpose of the Settlement
reservation is the instruction to reserve the transfer settlement for the Payer and Payee as collateral (_On the previously successful transfer_).
The reservation restricts the Payer from any Funds-Out (_Withdraw of funds from Payer account_) operations.

The settlement reservation may be performed on a per-transaction basis, or as a batch operation.

> //TODO Determine for what duration a settlement reservation may be kept for.

> //TODO When would the reservation of payer take place??
>
>       Immediately for Gross Net Settlement?
>       At the time of a callback from the Payer to indicate settlement may take place (minutes or hours after the transfer)? 

### Accounts
* Participant A Settlement - Payer settlement reserve
* Scheme Recon - Scheme Reconciliation (Scheme operator)
      
### Activity
#### Transfer Settlement Reservation (On Payer):
> An external event or Scheme trigger will instruct the Settlement Reservation event.
```
DR Participant A Settlement                     100
    CR Scheme Recon                                       100
DR Participant B Collatoral                     100
    CR Participant A Payable to B                         100
```

### Summary
* A's Settlement has a net DR balance of;  `0 + 100 = 100`
  * `100` units debited for settlement
* Scheme Recon has a CR balance of;  `0 + 100 = 100`
  * `100` units as credit to the recon
  * A positive balance indicates outstanding settlements
* B's Collateral DR has a balance of; `0 + 100 = 100`
  * `100` units collateral is reserved as part of the reservation
  * The reservation will be released after the funds are committed
* A Payable to B has a CR balance of; `0 + 100 = 100`
  * `100` units payable credit as part of reservation 

### Invariants
> Do we want to have a timeout set with 2phase commit for the DR on Payer? <br/> How long will we wait for settlement?


## Settlement Commit
An existing settlement reservation for a Payer has been completed successfully (a current reserved settlement of `100` units).
The purpose of the Settlement commit is the instruction to finalise the transfer settlement between Payer and the Scheme Recon.
The settlement commit would likely be process as part of a batch.
> The settlement commit completes the lifecycle of ownership for the digital/physical asset.

### Accounts
* Participant A Settlement  - Payee settlement reserve
* Scheme Recon - Scheme Reconciliation (Scheme operator)
* Participant A Payable to B - Transfer of A(DR) to B(CR)
                   
### Activity
#### 1. Committing Transfer for Payee and Scheme Recon:
```
DR Scheme Recon                                 100
    CR Participant A Settlement                           100
DR Participant A Payable to B                   100
    CR Participant B Collatoral                           100
```

### Summary
* Scheme Recon has a balance of; `100 - 100 = 0`
  * `100` units credited as part of settlement reservation, which is now debited as part of settlement commit
* A's Settlement has a balance of; `100 - 100 = 0`
  * `100` units received from scheme as confirmed settlement
* A Payable to B has a balance of; `100 - 100 = 0`
  * `100` units are now settled
* B's Collateral has a balance of; `100 - 100 = 0`
  * `100` units collateral is restored due to settlement

### Invariants
- No commit would result in a corrupted state for the transaction.
  - This is not the case anymore. We are only keeping the Settlement on the Payer.
- 
>>>> TODO

## Participant Withdraw Collateral from Scheme
An existing participant A would like to withdraw `units` from the Scheme.

### Entities
* Bank (External)
* Scheme Collateral - Scheme operator collateral
* Participant Liquidity A - The participants liquidity on the Scheme

#### 1. Transfer the Participant Liquidity to the Scheme Collateral:
```
DR Participant Liquidity A                      100
    CR Scheme Collateral                                100
```

#### 2. Transfer the Scheme Collateral to the Bank:
> The Scheme to the Bank transfer occurs outside of Scheme Hub
```
DR Scheme Collateral                          100
    CR Bank                                               100
```

### Summary
* A's Liquidity has a net CR balance of;  `100 - 100 = 0`
    * `100` units debited to the bank where A will receive 'cash'
* S's Collateral has a net CR balance of;  `0 + 100 - 100 = 0`
    * `100` units from A, then `100` units to bank
* Bank settles with Scheme outside of Scheme hub


## Participant Close Account
An account may only be closed when the DR/CR for a participant is `0` units.

### Entities
>>>> TODO


## NOTES

* How do we link the accounts in TB;
    * Create accounts with the same `user_data` per Participant, then mark them as linked?
    * For each account type, we use `code`, do we leave it up to Scheme to deal with the multiple currencies, or treat is as part of the `code` makeup.

* Do we care about account state in TB, or leave it up to Scheme?
* 
