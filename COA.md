# Chart of Accounts

## The list of supported high level events include;
* [Participant Joins the Scheme](#participant-joins-the-scheme)
* [Participant Deposit Collateral to Scheme](#participant-deposit-collateral-to-scheme)
* [Payer Transfer funds to a Payee 1Phase Commit](#payer-transfer-funds-to-a-payee-1phase-commit)
* [Payer Transfer funds to a Payee 2Phase Commit](#payer-transfer-funds-to-a-payee-2phase-commit)
* [Settlement Reservation](#settlement-reservation)
* [Settlement Commit](#settlement-commit)
* [Participant Withdraw funds from an account](#participant-withdraw-funds-from-an-account)
* [Participant Close Account](#participant-close-account)



# Moja vs Doc
Position = Liquidity
Settlement = Collateral
Scheme Reconciliation = `Reconciliation Account`
HMLNS  = `Hub Multilateral Net Settlement`


# ---


## Participant Joins the Scheme
A new participant joins the scheme and the necessary accounts and configurations are provisioned in the system.
At this time the participant has no liquidity _(a current position of zero and a net debit cap of zero)_.

### Entities
* Scheme - S (Mojaloop operator)
* Participant - A (Participant on the Mojaloop switch that may be a Payer/Payee)

### Transfers
> No transfers at this stage. Only account creation.

### Invariants
> //TODO


## Participant Deposit Collateral to Scheme
A participant decides to deposit 100 units collateral into the Mojaloop hub scheme.

### Entities
* Bank (External)
* Scheme S - Scheme Reconciliation (Mojaloop operator)
* Scheme Fees SF - Fees being charged by the Scheme
* Participant Liquidity A - Deposit collateral into the Scheme 
* Participant Collateral A - Deposit collateral into the Scheme 

### Transfers
A cash deposit is made at a bank that supports loading of assets into the Mojaloop hub.
The hub is notified of the payment (which the bank has already taken the risk for).

###
#### 1. `Participant A` Deposits Collateral (Bank to Scheme):
>> The following event occurs outside the Mojaloop hub.
```
DR Bank                               120
    CR Scheme - S Collateral                    120
```

###
#### 2. `Participant A` Deposits Collateral (Scheme to Participant):
```
DR Scheme - S Collateral              120
    CR Participant A Collateral                 120
```

###
#### 3.1. `Scheme` decides to charge Fees on Deposit of Collateral:
>> (where fee is a function of collateral deposit amount = 120 * 8.333% = 10)
```
DR Participant A Collateral           10
    CR Scheme Fees SF Collateral                10
```
###

---
#### `(Repeatable)` 4. Participant A decides to make 100 collateral available as liquidity:
>> (either all collateral they have, or at most N units, in this case at most 100 units)
```
DR Participant - A Collateral           100
    CR Participant - A Liquidity                100
```
#### `(Repeatable)` 4. Participant A decides to make 10 collateral available as liquidity:
>> (Secondary event debits the last 10 units of collateral from the Participant A)
> At this point the collateral is `0`
```
DR Participant A Collateral           10
    CR Participant A Liquidity                  10
```

---

###
#### 5. Scheme extends Credit to A as a function of A's collateral deposit event amount:
>> (A has a net debit cap of 50 according to rules of scheme)

```
DR Scheme Line Liquidity to A         50
    CR Participant A Liquidity                  50
```

#
###### Summary
* A's Liquidity has a net CR balance of;  `100 + 10 + 50 = 160`
* A's Collateral has a net balance of;    `120 - 10 - 100 - 10 = 0`

>> very important to clarify that collateral and liquidity are both liability accounts

>> 
> 
> s

Scheme owes A the sum of liability accounts = 160 + 0 = 160
less what A owes the scheme = 50
160 - 50 = 110
120 - 10 = 110

### Invariants
> //TODO


#
## Payer Transfer units to a Payee
An existing participant (Payer) would like to transfer funds to another participant (Payee).
At this time the Payer participant has liquidity of 160 units _(and a gross debit cap of 200)_
Participant A would like to transfer `100` units to Participant B.

### Entities
* Scheme Fees SF - Fees being charged by the Scheme
* Corresponding A1 - Participant A Corresponding account
* Corresponding B1 - Participant B Corresponding account
* Participant A - Pay Participant - B (Payer)
* Participant B - Receive payment from Participant - A (Payee)

###
#### 1. `Participant A` Transfer units to `Participant B` (Payer to Payee):
```
DR Participant A Liquidity            100
    CR Participant B Liquidity                  100
DR Corresponding B1 Liquidity         100
    CR Corresponding A1 Liquidity               100
```

###
#### 2.1 `Participant A` is charged with a `10%` Transfer fee (100 * 0.10 = 10):
```
DR Participant A Liquidity            10
    CR Scheme Fees SF Collateral                10
```

###### Summary
* A's Liquidity has a net DR balance of;  `0 + 10 + 100 = 110`
* B's Corresponding B1 Liquidity has a net DR balance of;  `0 + 100 = 100`
* A's Liquidity has a net CR balance of;  `160 - 100 - 10 = 50`
* A's Corresponding A1 Liquidity has a net CR balance of;  `0 + 100 = 100`
* SF's Scheme Fees SF Collateral has a net CR balance of;  `0 + 10 = 10`

### Invariants
* A transfer will typically be performed as a 2phase commit 
  * The liquidity will be reserved until a commit on the transfer is performed
  * If no commit is received within the provided timeout, the units will be reversed
* 
  In the event of a rollback (timeout/force), the transfer will be rolled back.
>>>> TODO

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
* Participant A Settlement Liquidity - Reserve 
* Scheme S - Scheme Reconciliation Liquidity (Mojaloop operator)

###
#### 1. `Transfer A` Settlement Reservation:
```
DR Participant A Settlement Liquidity       100
    CR Scheme S Liquidity                             100
```

###### Summary
* A's Settlement Liquidity has a net DR balance of;  `0 + 100 = 100`
* S's Liquidity has a net CR balance of;  `0 + 100 = 100`

### Invariants
>>>> TODO


#
## Settlement Commit
An existing settlement reservation for a Payer has been completed successfully (a current reserved settlement of 100 units).
The purpose of the Settlement commit is the instruction to complete the transfer settlement between 
Payer, Payee and the Scheme.
> The settlement commit completes the lifecycle of ownership for the digital/physical asset.

### Entities
* Participant B Settlement Liquidity
* Scheme S - Scheme Reconciliation Liquidity (Mojaloop operator)

###
#### 1. Committing transfer amount for recipient and scheme:
```
DR Scheme S Liquidity                       100
    CR Participant B Settlement Liquidity             100
```

### Transfers
#### Payee/Payer Settlement Commit
```
Payee - B
    CR:100       Settlement:Accepted
Payer - A
    DR:100       Settlement:Accepted
Reconciliation - R
    DR:100       Reconciliation:Accepted (Payee)
    CR:100       Reconciliation:Accepted (Payer)
```
#### Rollback
> No Rollback allowed???



## Participant Withdraw funds from an account
--TODO

### Accounts
* Settlement
* Reconciliation

### Players
* Bank (External)
* Scheme - S (Mojaloop operator)
* Participant - A (Participant on the Mojaloop switch)

### Transfers
A cash deposit is made at a bank that supports loading of assets into the Mojaloop hub.
The hub is notified of the payment (which the bank has already taken the risk for).
```
Bank 
    CR:100       (-)
Payee - A
    DR:100       Settlement:Accepted
Scheme - S
    DR:100       Reconciliation:Accepted    
```



## Participant Close Account
An account may only be closed when the DR/CR for

### Accounts
* Settlement

### Players
* Participant - A (Mojaloop Hub participant)




## NOTES

* How do we deal with Fees, would we simply create another group of accounts;
  * Participant Group of Accounts
  * Fee Group of Accounts

* How do we link the accounts in TB;
  * Create accounts with the same `user_data` per Participant, then mark them as linked?
  * For each account type, we use `code`, do we leave it up to Mojaloop to deal with the multiple currencies, or treat is as part of the `code` makeup.
  
* Do we care about account state in TB, or leave it up to Mojaloop;
* 
