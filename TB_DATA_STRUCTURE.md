# TigerBeetle - Data Structure

## Account
Mutable data set for account related data.

| Field            | Type              | Description |
|------------------|-------------------|-------------|
| id               | `u128`            | Global unique id for an account. |
| user_data        | `u128`            | ??? |
| reserved         | `[48]u8 - array`  | Accounting policy primitives. |
| unit             | `u16`             | ??? |
| code             | `u16`             | ??? |
| flags            | `AccountFlags`    | See account flags. |
| debits_reserved  | `u64`             | ??? |
| debits_accepted  | `u64`             | ??? |
| credits_reserved | `u64`             | ??? |
| credits_accepted | `u64`             | ??? |
| timestamp        | `u64`             | ??? |

### AccountFlags - `[packed struct]`

| Field                            | Type              | Description |
|----------------------------------|-------------------|-------------|
| linked                           | `bool`            | Is the account linked to another account. |
| debits_must_not_exceed_credits   | `bool`            | Total debit transfer may not exceed credits. |
| credits_must_not_exceed_debits   | `bool`            | Total credit transfer may not exceed debits. |
| padding                          | `u29`             | ??? |

## Transfer
Transfers for TB are immutable.

| Field             | Type              | Description |
|-------------------|-------------------|-------------|
| id                | `u128`            | Global unique id for an account. |
| debit_account_id  | `u128`            | The unit transfer from account id. |
| credit_account_id | `u128`            | The unit transfer to account id |
| user_data         | `u128`            | ??? |
| reserved          | `[32]u8 - array`  | . |
| timeout           | `u64`             | Used for a 2-phase transfer. |
| code              | `u32`             | ??? |
| flags             | `TransferFlags`   | See transfer flags. |
| amount            | `u64`             | Transfer amount in units. |
| timestamp         | `u64`             | ??? |

### TransferFlags - `[packed struct]`

| Field                            | Type              | Description |
|----------------------------------|-------------------|-------------|
| linked                           | `bool`            | Is the account linked to another account. |
| two_phase_commit                 | `bool`            | Is the transfer a 2phase commit transfer. |
| condition                        | `bool`            | ??? |
| padding                          | `u29`             | ??? |

## Commit
Commits for TB are immutable. A commit can only be performed on a transfer.

| Field             | Type              | Description |
|-------------------|-------------------|-------------|
| id                | `u128`            | Global unique id for a transfer. |
| reserved          | `[32]u8 - array`  | . |
| code              | `u32`             | ??? |
| flags             | `CommitFlags`   | See commit flags. |
| timestamp         | `u64`             | ??? |

### CommitFlags - `[packed struct]`

| Field                            | Type              | Description |
|----------------------------------|-------------------|-------------|
| linked                           | `bool`            | Is the account linked to another account. |
| reject                           | `bool`            | Has the commit been rejected. |
| preimage                         | `bool`            | ??? |
| padding                          | `u29`             | ??? |
