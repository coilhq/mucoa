# CHART OF ACCOUNTS
When a payment is made in a real-time payments system like Mojaloop, the DFSP who is the custodian of the beneficiary’s account (the creditor DFSP) agrees to credit the beneficiary with the funds immediately. 
But the creditor DFSP has not yet received the funds from the DFSP who is the custodian of the debtor’s account: all that has happened so far is that the debtor DFSP has incurred an obligation to reimburse the creditor DFSP, and that obligation has been recorded in the Mojaloop Hub.

The process of settlement is the process by which a debtor DFSP reimburses a creditor DFSP for the obligations that the debtor DFSP has incurred as a consequence of transfers.

This guide describes how settlements are managed by the Mojaloop Hub using various accounts (Chart of Accounts) and the partner settlement bank(s), and introduces the main building blocks of settlement processing.

> TODO @jason add Fees for transfer and credit extension.

## INDEX
* [Definitions](#definitions)
* [Participant Joins the Hub](#participant-joins-the-hub)
* [Participant Deposit Collateral to Hub](#participant-deposits-collateral-to-hub)
* [Payer Transfer units to a Payee](#transfer-clearing)
* [Settlement](#settlement)
* [Participant Withdraw Collateral from Hub](#participant-withdraw-collateral-from-hub)
* [Participant Close Account](#participant-close-account)

## DEFINITIONS
| Definition  | Description                                                                         |
|-------------|-------------------------------------------------------------------------------------|
| Participant | A provider who is a member of a payment scheme, and subject to that scheme's rules. |
| Hub         | The Mojaloop operator.                                                              |
| Transfer    | A debit/credit from one account to another account.                                 |
| DFSP        | Digital Financial Service Provider.                                                 |


## PARTICIPANT JOINS THE HUB
A new participant joins the scheme and the necessary accounts and configurations are provisioned in the system.
At this time the participant has no liquidity _(a current position of zero and a net debit cap of zero)_.

### Entities
* Hub - (Hub operator)
* Participant A - (Participant on the Hub that may be a Payer/Payee)

### Activity
Participant with relevant information is captured, but no accounts have been created for the participant.

## PARTICIPANT DEPOSITS COLLATERAL TO HUB
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
    * `100` units for initial release, minus `10` as a Deposit Fee charge
* A's Fees has a CR balance of; `0 + 10 = 10`
  * `10` units fee charge for deposit on liquidity

## TRANSFER (CLEARING)
Participant A (Payer) would like to transfer funds to participant B (Payee, these are linked transfers).
At this time the Payer participant A has liquidity of `100` units liquidity.
Participant A would like to transfer `100` units to Participant B.

The liquidity from A to B is immediately available to B, however, 
the settlement reservation and commit is a separate action that is between A to Hub and B from Hub.

### Accounts
* **Liquidity A** - Pay Participant - B (Payer)
* **Liquidity B** - Receive payment from Participant - A (Payee)
* **Participant A Clearing B** - Existing or on demand account to record the clearing from A to B

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

### Account Balances - Transfer 1
| Account             | Debits | Credits | Balance |
|---------------------|--------|---------|---------|
| `Opening Balances`  |        |         |         |
| **A Liquidity**     | `0`    | `0`     | `100`   |------> Opening Balance <-------
| **B Liquidity**     | `0`    | `0`     | `100`   |------> Opening Balance <-------
| **A Clearing B**    | `0`    | `0`     | `0`     |------> Opening Balance <-------
| `Transfer 1`        |        |         |         |------> Transer 1 <-------
| A Liquidity         | `70`   |         | `30`    |--> Deduct from A
| A Clearing B        | `70`   | `70`    | `0`     |--> Cleaaring (Recording)
| B Liquidity         |        | `70`    | `170`   |--> Increase at B

> Now let's add Participant C into the mix.
```
DR B Liquidity                                  170
    CR B Clearing (C)                                     170
DR B Clearing (C)                               170
    CR C Liquidity                                        170
```

> A decides to pay C.
```
DR C Liquidity                                  50
    CR C Clearing (A)                                     50
DR C Clearing (A)                               50
    CR A Liquidity                                        50
```

### Account Balances - Transfer 2 & 3
| Account            | Debits | Credits | Balance |
|--------------------|--------|---------|---------|
| `Opening Balances` |        |         |         |
| **A Liquidity**    | `0`    | `0`     | `30`    |------> Opening Balance <-------
| **B Liquidity**    | `0`    | `0`     | `170`   |------> Opening Balance <-------
| **C Liquidity**    | `0`    | `0`     | `100`   |------> Opening Balance <-------
| **A Clearing B**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **B Clearing C**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **C Clearing A**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| `Transfer 2`       |        |         |         |------> Transer 2 <-------
| B Liquidity        | `170`  |         | `0`     |--> Deduct from B
| B Clearing C       | `170`  | `170`   | `0`     |--> Cleaaring (Recording)
| C Liquidity        |        | `170`   | `270`   |--> Increase at C
| `Transfer 3`       |        |         |         |------> Transer 3 <-------
| C Liquidity        | `50`   |         | `220`   |--> Deduct from C
| C Clearing A       | `50`   | `50`    | `0`     |--> Cleaaring (Recording)
| A Liquidity        |        | `50`    | `80`    |--> Increase at A


### Summary
* A's Liquidity has a CR balance of;  `100 - 70 + 50 = 80`
  * Opening balance of `100` units 
  * `70` units transferred to B
  * `50` units received from C
* B's Liquidity has a CR balance of;  `100 + 70 - 170 = 0`
  * Opening balance of `100` units
  * `70` units received from A
  * `170` units sent to C
* C's Liquidity has a CR balance of;  `100 + 170 - 50 = 220`
  * Opening balance of `100` units
  * `70` units received from A
  * `170` units sent to C
* Total liquidity backed by collateral remains constant = `300` units 
* Participant B can no longer clear payments because B's liquidity is exhausted.
* The goal of settlement is to get liquidity back to what it was before clearing.

### Invariants
* A transfer will typically be performed as a 2phase commit
  * The liquidity will be reserved until a commit on the transfer is performed
  * If no commit is received within the provided timeout, the units will be reversed (B back to scheme and scheme back to A)
* In the event of a rollback (timeout/force), the transfer will be rolled back

## SETTLEMENT
An existing transfer between Payer and Payee has been completed successfully. The purpose of the Settlement
reservation is the instruction to reserve the transfer settlement for the Payer and Payee as collateral (_On the previously successful transfer_).
The reservation restricts the Payer from any Funds-Out (_Withdraw of funds from Payer account_) operations.

The settlement reservation may be performed on a per-transaction basis, or as a batch operation (immediate or deferred).

### Accounts
* **A Settlement C** - Settlement from Payer (A) to Payee (C) debited 
* **B Settlement C** - Settlement from Payer (B) to Payee (C) debited 
* **A Liquidity** - Participant A liquidity is restored
* **B Liquidity** - Participant B liquidity is restored
      
### Activity
> An external event or Hub trigger will instruct the Settlement Reservation event.
```
DR A Settlement (C)                             20
    CR A Liquidity                                        20

DR B Settlement (C)                             100
    CR B Liquidity                                        100
```

### Account Balances - Settlement 1, 2 & 3 (single settlement transfer)
| Account            | Debits | Credits | Balance |
|--------------------|--------|---------|---------|
| `Opening Balances` |        |         |         |
| **A Settlement C** | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **B Settlement C** | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Liquidity**    | `0`    | `0`     | `80`    |------> Opening Balance <-------
| **B Liquidity**    | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **C Liquidity**    | `0`    | `0`     | `220`   |------> Opening Balance <-------
| `Settlement 2 & 3` |        |         |         |------> Settlement 2 & 3 <-------
| A Settlement C     | `20`   |         | `-20`   |--> A Settlement to B
| A Liquidity        |        | `20`    | `100`   |--> A Liquidity is restored
| B Settlement C     | `100`  |         | `-100`  |--> B Settlement to C
| B Liquidity        |        | `100`   | `100`   |--> B Liquidity is restored


### BILATERAL Net Settlement:
```
A owes B less what B owes A = 70
B owes C less what C owes B = 170
C owes A less what A owes C = 50

A liquidity plus settlement = 80 + 70 - 50 = 80 + 20 = 100
B liquidity plus settlement = 0 + 170 - 70 = 0 + 100 = 100
C liquidity plus settlement = 220 + 50 - 170 = 220 - 120 = 100
```

### MULTILATERAL Net Settlement: 
```
A owes someone who owes someone else, so A should go direct to C:
(there are multiple ways to arrange who pays what, all are valid)

A owes C (up to what A owes B) less what C owes A = 70 - 50 = 20
B owes C less what A owes C on B's behalf = 170 - 70 = 100

After settlement, A and B's liquidity is now both back up to 100, while C remains at 220.

This is the most efficient way of doing multilateral net because it doesn't require a "pot" at all and minimizes the number of settlement transactions.

Another simpler way to do this is for anyone who owes something to pay into a "pot" and then for anyone who is owed something to be
paid out of the "pot". This requires a pot and a few more transactions.
```

## PARTICIPANT WITHDRAW COLLATERAL FROM HUB
An existing participant A would like to withdraw `units` from the Hub.

### Entities
* Bank (External)
* Hub Collateral - Hub operator collateral
* Participant Liquidity A - The participants liquidity on the Hub

#### 1. Transfer the Participant Liquidity to the Hub Deposit account:
```
DR A Liquidity                                  100
    CR A Collateral                                       100
DR A Collateral                                 100
    CR A Deposit                                          100
```

### Account Balances - Withdraw
| Account            | Debits | Credits | Balance |
|--------------------|--------|---------|---------|
| `Opening Balances` |        |         |         |
| **A Liquidity**    | `0`    | `0`     | `100`   |------> Opening Balance <-------
| **A Collateral**   | `0`    | `0`     | `0`     |------> Opening Balance <-------
| **A Deposit**      | `0`    | `0`     | `-110`  |------> Opening Balance <-------
| `Settlement 2 & 3` |        |         |         |------> Settlement 2 & 3 <-------
| A Liquidity        | `100`  |         | `0`     |--> A Settlement to B
| A Collateral       | `100`  | `100`   | `0`     |--> Recording Collateral withdraw
| A Deposit          |        | `100`   | `-10`   |--> A deposit

### Summary
* A's Liquidity has a net CR balance of;  `100 - 100 = 0`
    * `100` units debited to the bank where A will receive 'cash'
* S's Collateral has a net CR balance of;  `0 + 100 - 100 = 0`
    * `100` units from A, then `100` units to bank
* Bank settles with Hub outside of the Hub
* The Deposit has a negative CR balance due to the `10` unit fee charge

## Participant Close Account
An account may only be closed when the DR/CR Liquidity balance for a participant is `0` units.

### Entities
>>>> TODO

## NOTES

* How do we link the accounts in TB;
    * Create accounts with the same `user_data` per Participant, then mark them as linked?
    * For each account type, we use `ledger`
    * For each currency, we use `code`
