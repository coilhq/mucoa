# Chart of Accounts

## The list of supported high level events include;
* [Participant Joins the Scheme](#participant-joins-the-scheme)
* [Participant Deposit funds into an account](#participant-deposit-funds-into-an-account)
* [Payer Transfer funds to a Payee 1Phase Commit](#payer-transfer-funds-to-a-payee-1phase-commit)
* [Payer Transfer funds to a Payee 2Phase Commit](#payer-transfer-funds-to-a-payee-2phase-commit)
* [Settlement Reservation](#settlement-reservation)
* [Settlement Commit](#settlement-commit)
* [Participant Withdraw funds from an account](#participant-withdraw-funds-from-an-account)
* [Participant Close Account](#participant-close-account)
* 



## Participant Joins the Scheme
A new participant joins the scheme and the necessary accounts and configurations are provisioned in the system.
At this time the participant has no liquidity _(a current position of zero and a net debit cap of zero)_.

### Accounts
* Position
* Settlement
* Corresponding Position
* ~~Reconciliation~~ *(Already created as part of Mojaloop initial setup)*

### Players
* Scheme - S (Mojaloop operator)
* Participant - P (Participant on the Mojaloop switch that may be a Payer/Payee)

### Transfers
> No transfers at this stage. Only account creation.



## Participant Deposit funds into an account
A new participant joins the scheme and the necessary accounts and configurations are provisioned in the system. 
At this time the participant has no liquidity _(a current position of zero and a net debit cap of zero)_.

### Accounts
* Settlement
* Reconciliation

### Players
* Bank (External)
* Scheme - S (Mojaloop operator)
* Participant - P (Payer on the Mojaloop switch)

### Transfers
A cash deposit is made at a bank that supports loading of assets into the Mojaloop hub. 
The hub is notified of the payment (which the bank has already taken the risk for).
```
Bank 
    DR:100       (-)
Participant - P
    CR:100       Settlement:Accepted
Scheme - S
    DR:100       Reconciliation:Accepted    
```



## Payer Transfer funds to a Payee 1Phase Commit
An existing participant (Payer) he/she would like to transfer funds to another participant (Payee).
At this time the Payer participant has liquidity _(a current settlement of 100 and a gross debit cap of 100)_
while the Payee has no liquidity _(a current position of zero)_.

### Accounts
* Position
* Corresponding Position

### Players
* Payer - A (Participant on the Mojaloop switch)
* Payee - B (Participant on the Mojaloop switch)

### Transfers
#### Transfer
```
Payer - A
    DR:100       Position:Accepted
    CR:100       Corresponding Position:Accepted
Payee - B
    CR:100       Position:Accepted
    DR:100       Corresponding Position:Accepted
```
#### Rollback
In the event of a rollback (timeout/force), the transfer will be rolled back.
```
Payer - A
    DR:-100       Position:Accepted
    CR:-100       Corresponding Position:Accepted
Payee - B
    CR:-100       Position:Accepted
    DR:-100       Corresponding Position:Accepted
```



## Payer Transfer funds to a Payee 2Phase Commit
An existing participant (Payer) he/she would like to transfer funds to another participant (Payee).
At this time the Payer participant has liquidity _(a current settlement of 100 and a net debit cap of 100)_ 
while the Payee has no liquidity _(a current position of zero)_.

### Accounts
* Position
* Corresponding Position

### Players
* Payer - A (Participant on the Mojaloop switch)
* Payee - B (Participant on the Mojaloop switch)

### Transfers
#### Phase 1 - Prepare
```
Payer - A
    DR:100       Position:Reserved
    CR:100       Corresponding Position:Reserved
Payee - B
    CR:100       Position:Reserved
    DR:100       Corresponding Position:Reserved
```
#### Phase 2 - Commit
```
Payer - A
    DR:100       Position:Accepted
    DR:-100      Position:Reserved
    CR:100       Corresponding Position:Accepted
    CR:-100      Corresponding Position:Reserved
Payee - B
    CR:100       Position:Accepted
    CR:-100      Position:Reserved
    DR:100       Corresponding Position:Accepted
    DR:-100      Corresponding Position:Reserved
```
#### Rollback
In the event of a rollback (timeout/force), the 1st phase will be rolled back.
```
Payer - A
    DR:-100       Position:Reserved
    CR:-100       Corresponding Position:Reserved
Payee - B
    CR:-100       Position:Reserved
    DR:-100       Corresponding Position:Reserved
```



## Settlement Reservation
An existing transfer between Payer and Payee has been completed successfully. 
The purpose of the Settlement reservation is the instruction to reserve the transfer settlement for the Payer 
(_On the previously successful transfer_).
The reservation restricts the Payer from any Funds-Out (_Withdraw of funds from Payer account_) operations.

> //TODO Determine for what duration a settlement reservation may be kept for. 

> //TODO When would the reservation of payer take place??
> 
>       Immediately for Gross Net Settlement?
>       At the time of a callback from the Payer to indicate settlement may take place (minutes or hours after the transfer)? 

### Accounts
* Settlement
* Reconciliation

### Players
* Payer - A (Participant on the Mojaloop switch)
* Reconciliation - R (Mojaloop Hub Recon account)

### Settlement Transfers
#### Settlement Reserve for Payer
```
Payer - A
    DR:100       Settlement:Reserved
Reconciliation - R
    CR:100       Reconciliation:Reserved
```
#### Rollback
In the event of a rollback (timeout/force), the reservation will be rolled back.
> The rollback will only be allowed if the settlement has not been committed??? 
> 
```
Payer - A
    DR:100       Settlement:Reserved
Reconciliation - R
    CR:100       Reconciliation:Reserved
```


## Settlement Commit
An existing settlement reservation between Payer and Payee has been completed successfully 
(a current reserved settlement of 100).
The purpose of the Settlement commit is the instruction to complete the transfer settlement between 
Payer, Payee and the Mojaloop Hub.
> The settlement commit completes the lifecycle of ownership for the digital/physical asset.

### Accounts
* Settlement
* Reconciliation

### Players
* Payer - A (Participant on the Mojaloop switch)
* Payee - B (Participant on the Mojaloop switch)
* Reconciliation - R (Mojaloop Hub Recon account)

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
* Participant - P (Participant on the Mojaloop switch)

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
* Participant - P (Mojaloop Hub participant)




## NOTES

* How do we deal with Fees, would we simply create another group of accounts;
  * Participant Group of Accounts
  * Fee Group of Accounts

* How do we link the accounts in TB;
  * Create accounts with the same `user_data` per Participant, then mark them as linked?
  * For each account type, we use `code`, do we leave it up to Mojaloop to deal with the multiple currencies, or treat is as part of the `code` makeup.
  
* Do we care about account state in TB, or leave it up to Mojaloop;
* 
