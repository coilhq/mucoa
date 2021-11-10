# Chart of Accounts

## Events
* [Participant Joins the Scheme](#participant-joins-the-scheme)
* [Participant Deposit Collateral to Scheme](#participant-deposits-collateral-to-scheme)
* [Payer Transfer units to a Payee](#payer-transfer-units-to-a-payee)
* [Settlement Reservation](#settlement-reservation)
* [Settlement Commit](#settlement-commit)
* [Participant Withdraw Collateral from Scheme](#participant-withdraw-collateral-from-scheme)
* [Participant Close Account](#participant-close-account)



# Moja vs Doc (TODO REMOVE)
Position = Liquidity
Settlement = Collateral
Scheme Reconciliation = `Reconciliation Account`
HMLNS  = `Hub Multilateral Net Settlement`


# ---

#
## Participant Joins the Scheme
A new participant joins the scheme and the necessary accounts and configurations are provisioned in the system.
At this time the participant has no liquidity _(a current position of zero and a net debit cap of zero)_.

### Entities
* Scheme S - (Mojaloop operator)
* Participant A - (Participant on the Mojaloop switch that may be a Payer/Payee)

### Transfers
> No transfers at this stage. Only account creation.


#
## Participant Deposits Collateral to Scheme
A participant decides to deposit 100 units collateral into the Mojaloop hub Scheme.

### Entities
* Bank (External)
* Scheme S Collateral - Mojaloop operator collateral
* Scheme S Liquidity - Mojaloop operator liquidity
* Scheme Fees SF Collateral - Mojaloop operator entity used for Scheme Fee collateral
* Participant Liquidity A - The participants liquidity on the Scheme
* Participant Collateral A - Deposit collateral into the Scheme

### Transfers
A cash deposit is made at a bank that supports loading of assets into the Mojaloop hub.
The hub is notified of the payment.

###
#### Participant A Deposits Collateral (Bank to Scheme, Scheme to Participant):
> The following event occurs outside the Mojaloop hub.
```
DR Bank                               120
    CR Participant A Collateral                 120
```

###
#### Scheme decides to charge Fees on Deposit of Collateral:
> (where fee is a function of collateral deposit amount = 120 * 8.333% = 10)
```
DR Participant A Collateral           10
    CR Participant A Fees                       10
DR Participant A Fees                 10
    CR Scheme Fees                              10
-OLD--DR Participant A Collateral                           10
-OLD--    CR Scheme Fees SF Collateral                                10
```
###


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


###
#### Scheme extends Credit to A as a function of A's collateral deposit event amount:
> (A has a net debit cap of 150 according to rules of scheme)

```
DR Scheme S Liquidity               50
    CR Participant A Liquidity                  50
```

#
### Summary
* A's Liquidity has a CR balance of; `100 + 10 + 50 = 160`
  * `100` units for initial release, plus additional `10`, final additional `50` as credit line to participant
  * `50` of the `160` units were made available by S's credit line
* A's Collateral has a CR balance of; `120 - 10 - 100 - 10 = 0`
  * `120` units for deposit, minus `10` for fee, minus `100` then `10` for liquidity release
* S's Liquidity has a DR balance of; `0 + 50 = 50`
  * Scheme extends `50` units of liquidity to A
  * Note that the scheme limit is set at 150 units liquidity cap, current usage is at 33%
* S's Collateral has a CR balance of; `0 + 120 - 120 = 0`
  * `120` units from Bank deposit, minus `120` units to A's collateral
* SF's Collateral has a CR balance of; `0 + 10`
  * `10` units Fee charge for A's deposit

### Invariants
> //TODO


#
## Transfer
An existing participant A (Payer) would like to transfer funds to another participant B (Payee).
At this time the Payer participant A has liquidity of 160 units _(and a gross debit cap of 200)_
Participant A would like to transfer `100` units to Participant B.

### Entities
* Scheme Fees SF Liquidity - Fees being charged by the Scheme
* Participant A Liquidity - Pay Participant - B (Payer)
* Participant B Liquidity - Receive payment from Participant - A (Payee)
* Corresponding Liquidity A1 - Participant A Corresponding account
  * Participant A Payable to B (TODO Refactor) - [Immediate Gross]
    * Bilateral Net      - Already A to B
    * Multilateral Net   - 
* Corresponding Liquidity B1 - Participant B Corresponding account


>>TODO Write down examples of the above for each.


###
#### 1. `Participant A` Transfer units to `Participant B` (Payer to Payee direct):
> Scheme allows for direct liquidity to Payee.
```
DR Participant A Liquidity            100
    CR Participant B Liquidity                  100
DR Corresponding B1 Liquidity         100
    CR Corresponding A1 Liquidity               100
```

```
DR Participant A Liquidity            100
    CR Participant A Payable to B                  100
DR Participant A Payable to B         100    
    CR Participant B Liquidity                     100
    
    
    
DR Corresponding B1 Liquidity         100
    CR Corresponding A1 Liquidity               100
```

###
#### 2 `Participant A` is charged with a `10%` Transfer fee (100 * 0.10 = 10):
```
DR Participant A Liquidity            10
    CR Scheme Fees SF Liquidity                 10
```

###### Summary
* A's Liquidity has a DR balance of;  `0 + 10 + 100 = 110`
  * `10` units for fee and `100` units for transfer
* A's Liquidity has a CR balance of;  `160 - 100 - 10 = 50`
  * `160` units minus `100` for transfer and `10` for the SF's fee
* A's Corresponding A1 Liquidity has a net CR balance of;  `0 + 100 = 100`
* B's Liquidity has a CR balance of; `0 + 100 = 100`
  * `100` units received from A
* B's Corresponding B1 Liquidity has a net DR balance of;  `0 + 100 = 100`
* SF's Scheme Fees SF Liquidity has a net CR balance of;  `0 + 10 = 10`

### Invariants
* A transfer will typically be performed as a 2phase commit 
  * The liquidity will be reserved until a commit on the transfer is performed
  * If no commit is received within the provided timeout, the units will be reversed
* In the event of a rollback (timeout/force), the transfer will be rolled back


#
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

### Entities
* Participant A Settlement - Payer settlement reserve  
* Scheme Recon SR - Scheme Reconciliation (Mojaloop operator)

###
#### 1. `Transfer A` Settlement Reservation (On Payer) and Scheme Recon:
>> An external event or Mojaloop trigger will instruct the Settlement Reservation event. 
```
DR Participant A Settlement                     100
    CR Scheme Recon SR                                    100
```

###### Summary
* A's Settlement Liquidity has a net DR balance of;  `0 + 100 = 100`
  * `100` units debited for settlement
* SR has a CR balance of;  `0 + 100 = 100`
  * `100` units as credit to the recon
  * A positive balance indicates outstanding settlements

### Invariants
>> Do we want to have a timeout set with 2phase commit for the DR on Payer? <br/> How long will we wait for settlement?
>  

#
## Settlement Commit
An existing settlement reservation for a Payer has been completed successfully (a current reserved settlement of 100 units).
The purpose of the Settlement commit is the instruction to complete the transfer settlement between 
Payer, Payee and the Scheme.
The settlement commit would likely be process as part of a batch.

> The settlement commit completes the lifecycle of ownership for the digital/physical asset.

### Entities
* Participant B Settlement  - Payee settlement reserve
* Scheme Recon SR - Scheme Reconciliation (Mojaloop operator)
* Corresponding Liquidity A1 - Participant A Corresponding account
* Corresponding Liquidity B1 - Participant B Corresponding account

###
#### 1. Committing transfer amount for Payee and Scheme Recon:
```
DR Scheme Recon SR                              100
    CR Participant B Settlement Liquidity                 100
DR Corresponding Liquidity A1                   100
    CR Corresponding Liquidity B1                         100
```

###### Summary
* B's Settlement Liquidity has a net CR balance of;  `0 + 100 = 100`
  * `100` unit settlement instruction received
* SR has a CR balance of;  `100 - 100 = 0`
  * `100` units as credit to the recon
  * `0` units indicate that all transfers have been settled
* A1's Corresponding Liquidity has a net CR balance of; `100 - 100 = 0`
  * At time of transfer the units were `100`, which is now set to `0`
* B1's Corresponding Liquidity has a net DR balance of; `100 - 100 = 0`
  * At time of transfer the units were `100`, which is now set to `0`

### Invariants
- No commit would result in a corrupted state for the transaction. 
- 
>>>> TODO
> 

#
## Participant Withdraw Collateral from Scheme
An existing participant A would like to withdraw `units` from the Scheme S. 

### Entities
* Bank (External)
* Scheme S Collateral - Mojaloop operator collateral
* Participant Liquidity A - The participants liquidity on the Scheme

###
#### 1. Transfer the Participant Liquidity to the Scheme Collateral: 
```
DR Participant Liquidity A                      100
    CR Scheme S Collateral                                100
```

#### 2. Transfer the Scheme S Collateral to the Bank:
> The Scheme to the Bank transfer occurs outside of Mojaloop Hub
```
DR Scheme S Collateral                          100
    CR Bank                                               100
```

###### Summary
* A's Liquidity has a net CR balance of;  `100 - 100 = 0`
  * `100` units debited to the bank where A will receive 'cash'
* S's Collateral has a net CR balance of;  `0 + 100 - 100 = 0`
  * `100` units from A, then `100` units to bank
* Bank settles with Scheme outside of Mojaloop hub


#
## Participant Close Account
An account may only be closed when the DR/CR for a participant is `0` units.

### Entities
>>>> ???


## NOTES

* How do we link the accounts in TB;
  * Create accounts with the same `user_data` per Participant, then mark them as linked?
  * For each account type, we use `code`, do we leave it up to Mojaloop to deal with the multiple currencies, or treat is as part of the `code` makeup.
  
* Do we care about account state in TB, or leave it up to Mojaloop?
* 
