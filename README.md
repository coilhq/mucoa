# Chart Of Accounts

The purpose of this document is to guide the process of identifying the COA (Chart of Accounts) for a Mojaloop Scheme and to describe how typical transactions impact the accounts. 

This document can be used as a baseline or for reference, with acknowledgement that particular accounting systems, compliance regulation and Scheme rules, amongst other things, ultimately impact the COA and the account interactions for a Scheme. 

## Contents
* [Definitions](#definitions)
* [Overview](#overview)
* [Participant Joins Scheme](#participant-joins-scheme)
* [Participant Deposits Collateral](#participant-deposits-collateral-to-scheme)
* [Fees](#scheme-charges-fees-on-deposit-of-collateral)
* [Payer Transfer to Payee](#transfer--clearing)
* [Settlement](#settlement)
* [Participant Withdraws Collateral](#participant-withdraw-collateral-from-scheme)
* [Participant Closes Account](#participant-closes-account)
* [References](#references)

## Definitions
| Definition     | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Account        | An account that may receive debit/credit transfers.                                                                                                                                                                                                                                                                                                                                                                                                             |
| API            | Application Programming Interface. A set of clearly defined methods of communication to allow interaction and sharing of data between different software programs.                                                                                                                                                                                                                                                                                              |
| Clearing       | The process of transmitting, reconciling, and, in some cases, confirming transactions prior to settlement, potentially including the netting of transactions and the establishment of final positions for settlement. Sometimes this term is also used (imprecisely) to cover settlement. For the clearing of futures and options, this term also refers to the daily balancing of profits and losses and the daily calculation of collateral requirements.     |
| Clearing House | A central location or central processing mechanism through which financial institutions agree to exchange payment instructions or other financial obligations (for example, securities). The institutions settle for items exchanged at a designated time based on the rules and procedures of the clearinghouse. In some cases, the clearinghouse may assume significant counterparty, financial, or risk management responsibilities for the clearing system. |
| COA            | In financial accounting the Chart of Accounts refers to a list of the accounts that make up the general ledger. The accounts are available for recording transactions in the general ledger.                                                                                                                                                                                                                                                                    |
| CR             | A credit journal record entry for a double-entry accounting system.                                                                                                                                                                                                                                                                                                                                                                                             |
| Deposit        | The exchange of digital/physical funds from a participant to a scheme.                                                                                                                                                                                                                                                                                                                                                                                          |
| DFSP           | Digital Financial Service Provider is an entity that provides financial services, and in this context, is a participant on a Mojaloop hub.                                                                                                                                                                                                                                                                                                                      |
| DR             | A debit journal record entry for a double-entry accounting system.                                                                                                                                                                                                                                                                                                                                                                                              |
| KYC            | Know your customer/client is a financial services requirement or standard used for due diligence to establish characteristics to identify a person or any entity, their activities, the source of their funds, as a measure of authenticating legitimacy and mitigating activities such as fraud and money laundering.                                                                                                                                          |
| Liquidity      | The availability of liquid assets to support an obligation. Banks and non-bank providers need liquidity to meet their obligations. Agents need liquidity to meet cash-out transactions by consumers and small merchants.                                                                                                                                                                                                                                        |
| Participant    | A provider of financial services who is a member of a payment scheme, and subject to that scheme's rules.                                                                                                                                                                                                                                                                                                                                                       |
| Payee          | The recipient of electronic funds in a payment transaction.                                                                                                                                                                                                                                                                                                                                                                                                     |
| Payer          | The payer of electronic funds in a payment transaction.	                                                                                                                                                                                                                                                                                                                                                                                                        |
| Scheme         | A group of entities constitute a payments scheme which defines rules and agreements between payments service providers on how they transact. A Mojaloop hub operator is an administrator of the payments scheme.                                                                                                                                                                                                                                                |
| Settlement     | In financial accounting, the process where one party reimburses another party for financial obligations incurred as a consequence of transaction clearing.                                                                                                                                                                                                                                                                                                      |
| Transfer       | A debit/credit from one account to another account.                                                                                                                                                                                                                                                                                                                                                                                                             |
| Withdraw       | The exchange of digital/physical funds from a scheme to a participant.                                                                                                                                                                                                                                                                                                                                                                                          |

# Overview
The sections that follow illustrate how a Mojaloop Scheme can:
* Map out the general ledger accounts for 6 typical scenarios.
* Identify the impact of the scenarios on the accounts and balances.
* Show how the accounts interact with one another, depending on the events.
* Define the main building blocks for settlement processing and describe how clearing and settlement impacts the accounts of partner settlement entities/banks.

There are 6 scenarios or user stories that are covered:

| Number | User Story                                                               |
|--------|--------------------------------------------------------------------------|
| 1      | A new participant joins an existing Mojaloop Scheme.                     |
| 2      | The new participant deposits collateral into the Mojaloop Scheme.        |
| 3      | Transfers between new and existing participants.                         |
| 4      | Settlement processing to restore liquidity between participants.         |
| 5      | A participant withdraws collateral from the Mojaloop Scheme.             |
| 6      | A participant exits the Mojaloop Scheme.                                 |


## Participant Joins Scheme
A new participant joins the scheme and the necessary participant and configuration data is provisioned in the system.
At this time the participant has no liquidity _(a current position of zero and a net debit cap of zero)_. 
All the necessary KYC requirements will be captured and completed by the DFSP when on-boarding contacts.

### Entities
The following entities are present for a participant joining the Scheme: 
* Scheme
* Participant A - a DFSP on the Scheme, this may be a Payer/Payee

### Events
Participant with relevant KYC information is captured, **but no financial accounts** have been created for the participant.

### Summary
No financial account has been created as yet, but if joining was successful, the participant is now ready to deposit collateral into the Scheme. 

## Participant Deposits Collateral To Scheme
Once a participant successfully joins the scheme, the next step is to provide collateral and track that collateral within the Scheme.
A participant deposits `110` units of collateral into the Scheme, which enables the participant to initiate transfers.

### Entities
The following entities are present for a participant depositing collateral into the Scheme:
* **Deposit** - Account recording the deposit of units
* **Collateral** - Account recording the collateral deposited by the participant
* **Liquidity** - Account recording the liquidity available for the participant
* **Fee** - Account recording fees charged by the scheme
* **Signup Bonus** - Account recording Scheme signup bonuses / credit extension based on initial collateral deposit

### Events
A cash deposit is made at a bank that supports loading of assets into the Scheme. The scheme is notified of the payment.
The necessary accounts are created as part of a chained event.

#### Participant A Deposits Collateral (Bank to Scheme, then Scheme to Participant):
The handing of assets occurs outside the scheme. A notification is sent to the scheme to inform the scheme of the deposit.
(either all collateral they have, or a maximum _N units_, in this case at most 110 units)

```
DR Participant A Deposit                        110
    CR Participant A Collateral                        110
DR Participant A Collateral                     110
    CR Participant A Liquidity                         110
```
At this point, the Participant A CR liquidity balance is `110` units, with initial financial accounts created.

#### Scheme Charges Fees on Deposit of Collateral:
Scheme fees are not mandatory for a deposit, thus they may also be applicable for:
  * Transfers - Charged per individual transaction, based on transaction categories, volumes etc.
  * Accounts - Once-off or recurring account service charges
> Fees will be applied to the participant `Liquidity` accounts. 
> To illustrate, we charge Deposit fees as a function of the collateral deposit amount (i.e. 110 * 18% = 20) and Transfer fees as a fixed fee per transaction. 

```
DR Participant A Liquidity                       20
    CR Participant A Fees                               20
```
At this point, the Participant A liquidity CR balance is `110 - 20 = 90` units.
The Scheme Participant Fee account has been credited with `20` units.

#### Scheme Extends Liquidity as Sign-on Bonus:
The Scheme provides a 9.1% bonus on liquidity for a first time deposit.
The bonus is based on the participant deposit amount.

```
DR Participant A Signup Bonus                    10
    CR Participant A Liquidity                          10
```
At this point, the Participant A liquidity CR balance is `90 + 10 = 100` units.
The Scheme Signup Bonus DR balance is `10` units.

### Account Balances Statement
The starting account balances are `0` units for all participants. 
The table below depicts the impact on account balances when depositing collateral into the Scheme.

| Account                    | Debits (DR) | Credits (CR) | Balance  |
|----------------------------|-------------|--------------|----------|
| Participant A Deposit      | `0`         | `0`          | `0`      |------> Opening Balance <-------
| Participant A Collateral   | `0`         | `0`          | `0`      |------> Opening Balance <-------
| Participant A Liquidity    | `0`         | `0`          | `0`      |------> Opening Balance <-------
| Participant A Fees         | `0`         | `0`          | `0`      |------> Opening Balance <-------
| Participant A Signup Bonus | `0`         | `0`          | `0`      |------> Opening Balance <-------
| **Deposit 1**              |             |              |          |------> Deposit 1 <-------
| Participant A Deposit      | `110`       |              | `DR 110` |--> Debit for Scheme
| Participant A Collateral   | `110`       | `110`        | `0`      |--> Collateral (Recording)
| Participant A Liquidity    |             | `110`        | `CR 110` |--> Liquidity Available
| Participant A Liquidity    | `20`        |              | `CR 90`  |--> Fee Charge on A for Deposit
| Participant A Fees         |             | `20`         | `CR 20`  |--> Fee Recorded for Scheme
| Participant A Signup Bonus | `10`        |              | `DR 10`  |--> Signup Bonus
| Participant A Liquidity    |             | `10`         | `CR 100` |--> Credit extened to A

### Summary
* A's Deposit has a DR balance of `0 - 110 = -110`
  * `110` units deposited at the bank 
* A's Collateral has a CR balance of `0 + 110 - 110 = 0`
  * `110` units deposited as collateral, minus `110` to be made available for liquidity
* A's Liquidity has a CR balance of `0 + 110 - 20 + 10 = 100`
  * `110` units as an initial deposit;
  * deduct `20` as a charge for the deposit
  * add `10` as a first time Sign-on bonus
* A's Fees has a CR balance of `0 + 20 = 20`
  * `20` units charged for deposit on liquidity
* A's Bonus has a DR balance of `0 - 10 = -10`
  * `10` additional units for first time deposit on liquidity

## Transfer & Clearing
When a payment is made on a real-time payments system, the DFSP who is the custodian of the beneficiary’s account (the creditor DFSP) agrees to credit the beneficiary with the funds immediately.
The creditor DFSP has not yet received the funds from the DFSP who is the custodian of the debtor’s account. At this stage, the debtor DFSP has incurred an obligation to reimburse the creditor DFSP, and the obligation has been recorded in the Mojaloop Scheme.

This is a series of linked transfers that constitute a transaction that succeeds or fails as a unit:
* Participant A (Payer) would like to transfer funds to Participant B (Payee). 
* At this time, Participant A has liquidity of `100` units.
* Participant A initiates a transfer of `100` units to Participant B. 
* Real-time clearing immediately makes liquidity available to Participant B.
* The Scheme records an outstanding obligation for settlement from DFSP A to B. 

The settlement reservation and commit are processed at a later stage, as actions from A to Scheme, and from Scheme to B.

### Entities
The following entities are present for a participant transferring liquidity to another participant:
* **Liquidity** - Account recording the liquidity available for the participant
* **Clearing** - Existing or on-demand account to record the clearing from A to B
* **Fee** - Account recording fees charged by the Scheme for the transfer

### Events
A transfer of units is made between two participants of the Scheme (Payer/Payee). 
In the 1st example, the transfer will be from Participant A *(Payer)* to Participant B *(Payee)*.
It is important for Participant A _(Payer)_ to have enough available units to cover Fee charges as part of the transfer. 
The scheme is notified of the transfer.

#### Linked Events
Linked events have the property of succeeding or failing together.
In this instance, a fee is linked to the clearing transfer to ensure that a fee can only be charged in the event of successful transfer clearing.
For this context, linked events are applicable to accounts and transfers.

#### Scheme Charges Fees on Transfer of Liquidity:
The Scheme charges a `10` unit Fee for the Transfer from A to B. Participant A *(Payer)* is liable for the charges.

```
DR Participant A Liquidity                       10
    CR Participant A Fees                               10
```
At this point, the Participant A CR liquidity balance is `90` units *(keeping in mind that the charge and transfer are linked)*.

#### Participant A Transfers Units to Participant B (Payer to Payee Direct):
Scheme allows for direct liquidity to Payee.

```
DR Participant A Liquidity                       70
    CR Participant A Clearing (B)                       70
DR Participant A Clearing (B)                    70
    CR Participant B Liquidity                          70
```
At this point, the Participant A CR liquidity balance is `20` units, while Participant B liquidity balance is `170` units as a result of the transfer.
The clearing and fee are part of a linked transfer that succeeds or fails together.
An example of a failure may be where a Payer exceeds their liquidity limit when the transfer and fee are taken into account.

### Account Balances Statement
Participant A and B's liquidity CR balances start at `100` units *(Initial deposit)*.

**Transfer 1**: Participant A transfers `70` units to Participant B.

| Account                  | Debits (DR) | Credits (CR) | Balance  |
|--------------------------|-------------|--------------|----------|
| Participant A Fees       | `0`         | `0`          | `CR 20`  |------> Opening Balance <-------
| Participant A Liquidity  | `0`         | `0`          | `CR 100` |------> Opening Balance <-------
| Participant B Liquidity  | `0`         | `0`          | `CR 100` |------> Opening Balance <-------
| Participant A Clearing B | `0`         | `0`          | `0`      |------> Opening Balance <-------
| **Transfer 1**           |             |              |          |------> Transer 1 <-------
| Participant A Liquidity  | `10`        |              | `CR 90`  |--> Fee Applied to Payer
| Participant A Fees       |             | `10`         | `CR 30`  |--> Scheme Credited for Transfer
| Participant A Liquidity  | `70`        |              | `CR 20`  |--> Deduct from A
| Participant A Clearing B | `70`        | `70`         | `0`      |--> Clearing (Recording)
| Participant B Liquidity  |             | `70`         | `CR 170` |--> Increase at B

Let's add Participant C into the mix. We will **exclude** any transfer fees for Transfers 2 & 3.

**Transfer 2**: Participant B pays Participant C `170` units.

```
DR Participant B Liquidity                      170
    CR Participant B Clearing (C)                      170
DR Participant B Clearing (C)                   170
    CR Participant C Liquidity                         170
```
At this point, the Participant B CR liquidity balance is `0` units, while Participant C liquidity balance is `270` units.

**Transfer 3**: Participant C decides to pay Participant A `60` units.
```
DR Participant C Liquidity                       60
    CR Participant C Clearing (A)                       60
DR Participant C Clearing (A)                    60
    CR Participant A Liquidity                          60
```
At this point, the Participant A CR liquidity balance is `80` units, 
Participant B CR liquidity balance is `0` units, while Participant C liquidity balance is `210` units.

### Account Balances Statement
The table below illustrates the account balance updates between Transfer 2 & 3.
Transfer 2 is for `170` units from Participant B to Participant C.
Transfer 3 is for `60` units from Participant C to Participant A.

| Account                   | Debits (DR) | Credits (CR) | Balance  |
|---------------------------|-------------|--------------|----------|
| Participant A Liquidity   | `0`         | `0`          | `CR 20`  |------> Opening Balance <-------
| Participant B Liquidity   | `0`         | `0`          | `CR 170` |------> Opening Balance <-------
| Participant C Liquidity   | `0`         | `0`          | `CR 100` |------> Opening Balance <-------
| Participant A Clearing B  | `0`         | `0`          | `0`      |------> Opening Balance <-------
| Participant B Clearing C  | `0`         | `0`          | `0`      |------> Opening Balance <-------
| Participant C Clearing A  | `0`         | `0`          | `0`      |------> Opening Balance <-------
| **Transfer 2**            |             |              |          |------> Transer 2 <-------
| Participant B Liquidity   | `170`       |              | `0`      |--> Deduct from B
| Participant B Clearing C  | `170`       | `170`        | `0`      |--> Clearing (Recording)
| Participant C Liquidity   |             | `170`        | `CR 270` |--> Increase at C
| **Transfer 3**            |             |              |          |------> Transer 3 <-------
| Participant C Liquidity   | `60`        |              | `CR 210` |--> Deduct from C
| Participant C Clearing A  | `60`        | `60`         | `0`      |--> Clearing (Recording)
| Participant A Liquidity   |             | `60`         | `CR 80`  |--> Increase at A


### Summary
* A's Liquidity has a CR balance of  `100 - 10 - 70 + 60 = 80`
  * Opening balance of `100` units 
  * `10` units fee charge
  * `70` units transferred to B
  * `60` units received from C
* B's Liquidity has a balance of  `100 + 70 - 170 = 0`
  * Opening balance of `100` units
  * `70` units received from A
  * `170` units sent to C
* C's Liquidity has a CR balance of  `100 + 170 - 60 = 210`
  * Opening balance of `100` units
  * `170` units received from B
  * `60` units sent to A
* Total liquidity that is backed by collateral remains constant at `330` units (i.e. `110` units for each of the 3 Participants). 
* Participant B can no longer clear payments because B's liquidity is exhausted *(balance of `0`)*.
* The goal of settlement is to restore liquidity positions to what they were prior to clearing.

## Settlement
Settlement is the process by which a debtor DFSP reimburses a creditor DFSP for obligations incurred as a consequence of clearing transfers.

A Settlement reservation is the instruction to reserve the transfer amount for the Payer and Payee as collateral.
The reservation restricts the Payer from any Funds-Out *(Withdraw of funds from Payer account)* operations.
An existing transfer between Payer and Payee has been completed successfully in order to settle.

The settlement reservation may be performed on a per-transaction basis, or as a batch operation *(immediate or deferred)*.

### Entities
The following entities are present for participant settlement:
* **Participant X Settlement Y** - Settlement from Payer (A) to Payee (C) debited 
* **Participant Liquidity** - Participant A,C liquidity is restored

### Events
An external event or Scheme trigger will instruct the Settlement Reservation event.
The Settlement event will occur as a batch process.
```
DR Participant A Settlement (C)                  30
    CR Participant A Liquidity                          30

DR Participant B Settlement (C)                 110
    CR Participant B Liquidity                         110
```
At this point, the Participant A CR liquidity balance is `110` units,
Participant B CR liquidity balance is `110` units, while Participant C liquidity remains at `210` units.

### Account Balances Statement
The table below illustrates the settlement account balance updates between Participants A and B. 
There is no need to settle Participant C account, since the CR balance is above `110` units (`110` Collateral Deposit).

| Account                         | Debits (DR) | Credits (CR) | Balance  |
|---------------------------------|-------------|--------------|----------|
| Participant A Settlement C      | `0`         | `0`          | `0`      |------> Opening Balance <-------
| Participant B Settlement C      | `0`         | `0`          | `0`      |------> Opening Balance <-------
| Participant A Liquidity         | `0`         | `0`          | `CR 80`  |------> Opening Balance <-------
| Participant B Liquidity         | `0`         | `0`          | `0`      |------> Opening Balance <-------
| Participant C Liquidity         | `0`         | `0`          | `CR 210` |------> Opening Balance <-------
| **Settlement 1, 2 & 3**         |             |              |          |------> Settlement 2 & 3 <-------
| Participant A Settlement C      | `30`        |              | `DR 30`  |--> A Settlement to B
| Participant A Liquidity         |             | `30`         | `CR 110` |--> A Liquidity is restored
| Participant B Settlement C      | `110`       |              | `DR 110` |--> B Settlement to C
| Participant B Liquidity         |             | `110`        | `CR 110` |--> B Liquidity is restored


### Bilateral Net Settlement Model:
Settlements are deferred net if a number of transfers are settled together. Net settlements *(in which a number of transfers are settled together)* are by definition deferred *(since it takes time to construct a batch)*.

Settlements are bilateral if each pair of participants settles with each other for the net of all transfers between them.

```
A owes B less what B owes A = 70
B owes C less what C owes B = 170
C owes A less what A owes C = 50

A liquidity plus settlement = 90 - 70 + 60 = 80 + 30      = 110
B liquidity plus settlement = 100 + 70 - 170 = 0 + 110    = 110
C liquidity plus settlement = 100 + 170 - 60 = 210 - 110  = 110
```

### Multilateral Net Settlement Model:
Settlements are deferred net if a number of transfers are settled together. Net settlements *(in which a number of transfers are settled together)* are by definition deferred *(since it takes time to construct a batch)*.

Settlements are multilateral if each participant settles with the Hub for the net of all transfers to which it has been a party, no matter who the other party was.

```
A owes someone who owes someone else, so A should go direct to C:
(there are multiple ways to arrange who pays what, all are valid)

A owes C (up to what A owes B) less what C owes A = 70 - 50 = 20
B owes C less what A owes C on B's behalf = 170 - 60 = 110

After settlement, A and B's liquidity is now both back up to 110, while C remains at 210.

This is the most efficient way of doing multilateral net because it doesn't require a "pot" at all and minimizes the number of settlement transactions.

Another simpler way to do this is for anyone who owes something to pay into a "pot" and then for anyone who is owed something to be paid out of the "pot". 
This requires a pot and a few more transactions.
```

## Participant Withdraw Collateral From Scheme
An existing participant A would like to withdraw the units that it had deposited.

### Entities
The following entities are present when a participant withdraws from the Scheme:
* Bank (External)
* Scheme Collateral - Scheme operator collateral
* Participant Liquidity A - The participant's liquidity on the Scheme

### Events
Participant A would like to withdraw all collateral from the Scheme.
The transfer of funds are:
- Liquidity to Collateral, and then;
- Collateral to Deposit 

#### 1. Transfer The Participant Liquidity To The Scheme Deposit Account:
```
DR Participant A Liquidity                      110
    CR Participant A Collateral                        110
DR Participant A Collateral                     110
    CR Participant A Deposit                           110
```
At this point, the Participant A CR liquidity and collateral balance is `0` units.
The Participant Deposit account CR balance is now recovered back to a balance of `110` units. 

### Account Balances Statement
The table below depicts the events for withdrawing collateral from the Scheme.
The Participant A Deposit account has a DR balance of `110` units, due to the CR Participant A Fees charges incurred during deposit.  

| Account                    | Debits (DR) | Credits (CR) | Balance  |
|----------------------------|-------------|--------------|----------|
| Participant A Liquidity    | `0`         | `0`          | `CR 110` |------> Opening Balance <-------
| Participant A Collateral   | `0`         | `0`          | `0`      |------> Opening Balance <-------
| Participant A Deposit      | `0`         | `0`          | `DR 110` |------> Opening Balance <-------
| **Settlement 2 & 3**       |             |              |          |------> Settlement 2 & 3 <-------
| Participant A Liquidity    | `110`       |              | `0`      |--> A Settlement to B
| Participant A Collateral   | `110`       | `110`        | `0`      |--> Recording Collateral withdraw
| Participant A Deposit      |             | `110`        | `0`      |--> Deposit is recovered

### Summary
* A's Liquidity has a net CR balance of  `110 - 110 = 0`
    * `110` units debited to the bank where A will receive 'cash'
* A's Collateral has a net CR balance of  `0 + 110 - 110 = 0`
    * `110` units from A, then `110` units to bank
* Bank settles with Participant A outside of the Scheme

## Participant Closes Account
An account may only be closed when the DR/CR Liquidity and Collateral balance for a participant is `0` units.
The positive Participant Deposit CR balance indicates the collateral is now out of the scheme.

### Entities
The following entities are present when a participant closes their account (inactive):
* Scheme
* Participant A - Participant on the Scheme that may be a Payer/Payee

## Notes
Please keep the following notes in mind.

- Linking the accounts in TigerBeetle with Mojaloop:
  - Create accounts with the same `user_data` per Participant, then mark them as linked?
  - For each account type, we use `code`
  - For each currency, we use `ledger`
- Indicating linked flags for linked events in TigerBeetle:
  - When the .linked flag is specified, it links an event with the next event in the batch, to create a chain of events, of arbitrary length, which all succeed or fail together.
  - The tail of a chain is denoted by the first event without this flag.
  - The last event in a batch may therefore never have the `.linked` flag set as this would leave a chain open-ended.
  - Multiple chains or individual events may coexist within a batch to succeed or fail independently.
  - Events within a chain are executed within order, or are rolled back on error, so that the effect of each event in the chain is visible to the next, and so that the chain is either visible or invisible as a unit to subsequent events after the chain. 
  - The event that was the first to break the chain will have a unique error result. Other events in the chain will have their error result set to `.linked_event_failed`.

## References
| Description                                                               | Link                                                                                                              |
|---------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| Mojaloop Business Operations Documentation >> Mojaloop Ledgers in the Hub | <code>https://docs.mojaloop.io/mojaloop-business-docs/HubOperations/Settlement/ledgers-in-the-hub.html</code>     |
| Working spreadsheet for Chart of Accounts in Mojaloop                     | <code>https://docs.google.com/spreadsheets/d/19TnECdsKjBcJkIKWqqTUNMa8Ur21668UhxMp_8TJ81I/edit?usp=sharing</code> |
| Business Onboarding of DFSP                                               | <code>https://docs.mojaloop.io/mojaloop-business-docs/HubOperations/Onboarding/business-onboarding.html</code>    |
| vNext Reference Architecture - Accounts & Balances                        | <code>https://mojaloop.github.io/reference-architecture-doc/boundedContexts/accountsAndBalances/</code>           | 
| vNext Miro Board                                                          | <code>https://miro.com/app/board/o9J_lJyA1TA=/</code>                                                             |
 
