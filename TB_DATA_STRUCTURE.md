# TigerBeetle - Data Structure

## Account
Mutable data set for account related data.

| Field            | Type              | Description |
|------------------|-------------------|-------------|
| id               | `u128`            | Global unique id for an account. |
| user_data        | `u128`            | Implementation specific data on account. Opaque third-party identifier to link this account (many-to-one) to an external entity. |
| reserved         | `[48]u8 - array`  | Accounting policy primitives. Not available. |
| unit             | `u16`             | A transfer unit describing the currency associated with the account. |
| code             | `u16`             | A chart of accounts code describing the type of account (e.g. clearing, settlement) |
| flags            | `AccountFlags`    | See account flags. |
| debits_reserved  | `u64`             | Balance for reserved debits. |
| debits_accepted  | `u64`             | Balance for accepted debits. |
| credits_reserved | `u64`             | Balance for reserved credits. |
| credits_accepted | `u64`             | Balance for accepted credits. |
| timestamp        | `u64`             | The current state machine timestamp of the account for state tracking. |

### AccountFlags - `[packed struct]`

| Field                            | Type              | Description |
|----------------------------------|-------------------|-------------|
| linked                           | `bool`            | Is the account linked to another account. |
| debits_must_not_exceed_credits   | `bool`            | Total debit transfer may not exceed credits. |
| credits_must_not_exceed_debits   | `bool`            | Total credit transfer may not exceed debits. |
| padding                          | `u29`             | Data to be used for padding. |

## Transfer
Transfers for TB are immutable.

| Field             | Type              | Description |
|-------------------|-------------------|-------------|
| id                | `u128`            | Global unique id for an account. |
| debit_account_id  | `u128`            | The unit transfer from (Payer) account id. |
| credit_account_id | `u128`            | The unit transfer to (Payee) account id |
| user_data         | `u128`            | Implementation specific data on transfer. Opaque third-party identifier to link this transfer (many-to-one) to an external entity |
| reserved          | `[32]u8 - array`  | Accounting policy primitives. Not available. |
| timeout           | `u64`             | Used for a 2-phase transfer. The maximum wait timeout in milliseconds for a commit. |
| code              | `u32`             | A chart of accounts code describing the reason for the transfer (e.g. deposit, settlement) |
| flags             | `TransferFlags`   | See transfer flags. |
| amount            | `u64`             | Transfer amount in units. |
| timestamp         | `u64`             | The current state machine timestamp of the transfer for state tracking. |

### TransferFlags - `[packed struct]`

| Field                            | Type              | Description |
|----------------------------------|-------------------|-------------|
| linked                           | `bool`            | Is the account linked to another account. |
| two_phase_commit                 | `bool`            | Is the transfer a 2phase commit transfer. |
| condition                        | `bool`            | Does the transfer support transfer conditions |
| padding                          | `u29`             | Data to be used for padding. |

## Commit
Commits for TB are immutable. A commit can only be performed on a transfer.

| Field             | Type              | Description |
|-------------------|-------------------|-------------|
| id                | `u128`            | Global unique id for a transfer. |
| reserved          | `[32]u8 - array`  | Accounting policy primitives. Not available. |
| code              | `u32`             | ??? |
| flags             | `CommitFlags`     | See commit flags. |
| timestamp         | `u64`             | The current state machine timestamp of the commit for state tracking. |

### CommitFlags - `[packed struct]`

| Field                            | Type              | Description |
|----------------------------------|-------------------|-------------|
| linked                           | `bool`            | Is the account linked to another account. |
| reject                           | `bool`            | Has the commit been rejected. |
| preimage                         | `bool`            | ??? |
| padding                          | `u29`             | Data to be used for padding. |
