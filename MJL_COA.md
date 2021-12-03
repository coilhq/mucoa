# Mojaloop - Chart of Accounts

## Ledger Account Type

| Id               | Name              | Description |
|------------------|-------------------|-------------|
| `1`               | POSITION        | Typical accounts from which a DFSP provisions transfers |
| `2`               | SETTLEMENT        | Reflects the individual DFSP Settlement Accounts as held at the Settlement Bank |
| `3`               | HUB_RECONCILIATION        | A single account for each currency with which the hub operates. The account is "held" by the Participant representing the hub in the switch |
| `4`               | HUB_MULTILATERAL_SETTLEMENT        | A single account for each currency with which the hub operates. The account is "held" by the Participant representing the hub in the switch |
| `5`               | INTERCHANGE_FEE        | Fee |
| `6`               | INTERCHANGE_FEE_SETTLEMENT        | Fee Settlement |

## Ledger Entry Type

| Id               | Name              | Description |
|------------------|-------------------|-------------|
| `1`               | PRINCIPLE_VALUE        | The principle amount to be settled between parties, derived on quotes between DFSPs |
| `2`               | INTERCHANGE_FEE        | Fees to be paid between DFSP |
| `3`               | HUB_FEE        | Fees to be paid from the DFSPs to the Hub Operator |
| `4`               | POSITION_DEPOSIT        | Used when increasing Net Debit Cap |
| `5`               | POSITION_WITHDRAWAL        | Used when decreasing Net Debit Cap |
| `6`               | SETTLEMENT_NET_RECIPIENT        | Participant is settlement net recipient |
| `7`               | SETTLEMENT_NET_SENDER        | Participant is settlement net sender |
| `8`               | SETTLEMENT_NET_ZERO        | Participant is settlement net sender |
| `9`               | RECORD_FUNDS_IN        | Settlement account funds in |
| `10`               | RECORD_FUNDS_OUT        | Settlement account funds out |
