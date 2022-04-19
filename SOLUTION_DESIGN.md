# Solution Design: Mojaloop-TigerBeetle Integration

[Glossary](#glossary)
1. [Purpose](#1-purpose)
2. [Introduction](#2-introduction)
3. [Architecture & Design](#3-architecture-&-design)  
3.1. [Architecture and Design Principles](#31-architecture-and-design-principles)  
3.2. [Current Mojaloop Architecture](#32-current-mojaloop-architecture)  
3.3. [Central Ledger Architecture](#33-central-ledger-architecture)  
3.4. [Central Settlement Architecture](#33-central-settlement-architecture)
4. [Requirements](#4-requirements)  
4.1. [Functional Requirements](#41-functional-requirements)  
4.2. [Non-Functional Requirements](#42-non-functional-requirements)
5. [Assumptions, Dependencies & Considerations](#5-assumptions-dependencies--considerations)
6. [Scope Exclusions](#6-scope-exclusions)
7. [Detailed Design](#7-detailed-design)  
7.1. [Participants](#71-participants)  
7.2. [Transfers](#72-transfers)  
7.3. [Settlement](#73-settlement)
8. [Canonical Model](#8-canonical-model)  
8.1. [TigerBeetle](#81-tigerbeetle)  
8.2. [CentralLedger](#72-centralledger)


## Glossary
| Definition  | Description                                                                                                                                                                                                                                                               |
|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Participant | A provider who is a member of a payment scheme, and subject to that scheme's rules.                                                                                                                                                                                       |
| Hub         | The Mojaloop operator.                                                                                                                                                                                                                                                    |
| Transfer    | A debit/credit from one account to another account.                                                                                                                                                                                                                       |
| DFSP        | Digital Financial Service Provider.                                                                                                                                                                                                                                       |
| TigerBeetle | A financial accounting database designed for mission critical safety and performance to power the future of financial services.                                                                                                                                           |
| Endpoint    | An API is a set of protocols and tools to facilitate interaction between two applications. An endpoint is a place on the API where the exchange happens. Endpoints are URIs (Uniform Resource Indices) on an API that an application can access. All APIs have endpoints. |
| JSON        | JSON (JavaScript Object Notation) is a lightweight data-interchange format.                                                                                                                                                                                               |
| REST        | Representational state transfer (REST) is a software architectural style that was created to guide the design and development of the architecture for the World Wide Web.                                                                                                 |
| VPN Tunnel  | A VPN tunnel is an encrypted link between your computer or mobile device and an outside network. A VPN tunnel — short for virtual private network tunnel — can provide a way to cloak some of your online activity.                                                       |
| TLS/SSL     | Transport Layer Security (TLS) certificates—most commonly known as SSL, or digital certificates—are the foundation of a safe and secure internet. TLS/SSL certificates secure internet connections by encrypting data sent between systems.                               |
| HTTPS       | Stands for "HyperText Transport Protocol Secure." HTTPS is the same thing as HTTP, but uses a secure socket layer (TLS/SSL) for security purposes.                                                                                                                        |
| WAF         | A WAF or web application firewall helps protect web applications by filtering and monitoring HTTP traffic between a web application and the Internet (_or internal network_).                                                                                             |


## 1. Purpose
This document proposes the solution architecture and system design for integrating a TigerBeetle database as part of a Mojaloop payments system.

Different sections of this document can be used by an audience that is focussed on:
* the business drivers and user stories for the solution;
* the architecture and solution design aspects for integrating TigerBeetle into Mojaloop; and
* the components and endpoints that constitute the proposed solution.

## 2. Introduction
The original design of the Mojaloop payments system uses Redis for caching and SQL databases to record participant, transaction, settlement and operational data. In the original design, the application layer implements the business processing and financial accounting logic, interacting with the database to persist and retrieve the data. 

TigerBeetle is a distributed database built for native financial accounting support. It leverages the original Mojaloop Central Ledger logic in order to implement the financial accounting logic natively, within the database. In this proposed solution, the Mojaloop application layer optimizes its database interactions and defers the financial accounting logic to TigerBeetle.

## 3. Architecture & Design
### 3.1 Architecture and Design Principles
> TODO
### 3.2. Current Mojaloop Architecture
The diagram below shows the current architecture of a Mojaloop payments hub and it illustrates interactions between the hub and external entities such as a settlement bank, a global account lookup service and the systems of other financial service providers.
<br><br>
![System Context Diagram](solution_design/arch-mojaloop.svg)

### 3.3. Central Ledger Architecture
#### 3.3.1. As Is - Central Ledger
A closer look at the current Central Services architecture, with the Central Ledger using SQL, PostgreSQL and Redis.<br><br>
![System Context Diagram As](solution_design/central-ledger-diagram-as-is.png)

#### 3.3.2. To Be - Central Ledger
This diagram depicts the proposed Central Services architecture with the Central Ledger running TigerBeetle together with SQL and Redis databases.<br><br>
![System Context Diagram](solution_design/central-ledger-diagram-to-be.png)

### 3.4. Central Settlement Architecture
> @jason Add the central-settlement as part of the design diagrams...
#### 3.4.1. As Is - Central Settlement
> TODO
#### 3.4.2. To Be - Central Settlement
> TODO

## 4. Requirements
### 4.1. Functional Requirements
#### 4.1.1. User Stories & Business Processes
* TigerBeetle NodeJS integrated into Central-Ledger
  * Make use of existing configuration `default.json` configuration file for client
  * TigerBeetle NodeJS client to be integrated into central-ledger
  * TigerBeetle enablement through on/off switch
  * TigerBeetle and central-ledger facade (Translate from CL Acc+Transfer to TB Acc+Transfer)
* jMeter endpoints added for full 
* Timeout function to be reliant on TB instead of timer
* Transfer duplicate check performed as part of TB built in functionality
* jMeter testing suite to test the following functionality:
  * Transfer
  * 2-Phase Transfer
  * Account & Participant Creation
  * Account Lookups
  * Transfer Lookups

#### Impact On User Experience
#### System Behaviour In different scenarios
#### Impact on customer support or operations

### 4.2. Non-functional Requirements
#### Performance In TigerBeetle
Making use of TigerBeetle in CentralLedger would mean a significant increase in performance and throughput.
TigerBeetle provides more performance than a general-purpose relational database such as MySQL or an in-memory database such as Redis:

* TigerBeetle **uses small, simple fixed-size data structures** (accounts and transfers) and a tightly scoped domain.
* TigerBeetle **performs all balance tracking logic in the database**. This is a paradigm shift where we move the code once to the data, not the data back and forth to the code in the critical path. This eliminates the need for complex caching logic outside the database. The “Accounting” business logic is built in to TigerBeetle so that you can **keep your application layer simple, and completely stateless**.
* TigerBeetle **supports batching by design**. You can batch all the transfer prepares or commits that you receive in a fixed 10ms window (or in a dynamic 1ms through 10ms window according to load) and then send them all in a single network request to the database. This enables low-overhead networking, large sequential disk write patterns and amortized fsync and consensus across hundreds and thousands of transfers.
  * Everything is a batch. It's your choice whether a batch contains 100 transfers or 10,000 transfers but our measurements show that **latency is _less_ where batch sizes are larger, thanks to Little's Law** (e.g. 50ms for a batch of a hundred transfers vs 20ms for a batch of ten thousand transfers). 
  * TigerBeetle is able to amortize the cost of I/O to achieve lower latency, even for fairly large batch sizes, by eliminating the cost of queueing delay incurred by small batches.
* If your system is not under load, TigerBeetle also **optimizes the latency of small batches**. After copying from the kernel's TCP receive buffer (TigerBeetle does not do user-space TCP), TigerBeetle **does zero-copy Direct I/O** from network protocol to disk, and then to state machine and back, to reduce memory pressure and L1-L3 cache pollution.
* TigerBeetle **uses io_uring for zero-syscall networking and storage I/O**. The cost of a syscall in terms of context switches adds up quickly for a few thousand transfers.
* TigerBeetle **does zero-deserialization** by using fixed-size data structures, that are optimized for cache line alignment to **minimize L1-L3 cache misses**.
* TigerBeetle **takes advantage of Heidi Howard's Flexible Quorums** to reduce the cost of **synchronous replication to one (or two) remote replicas at most** (in addition to the leader) with **asynchronous replication** between the remaining followers. This improves write availability, without sacrificing strict serializability or durability. This also reduces server deployment cost by as much as 20% because a 4-node cluster with Flexible Quorums can now provide the same `f=2` guarantee for the replication quorum as a 5-node cluster.
> ["The major availability breakdowns and performance anomalies we see in cloud environments tend to be caused by subtle underlying faults, i.e. gray failure (slowly failing hardware) rather than fail-stop failure."](https://www.microsoft.com/en-us/research/wp-content/uploads/2017/06/paper-1.pdf)
* TigerBeetle **routes around transient gray failure latency spikes**. 
  * For example, if a disk write that typically takes 4ms starts taking 4 seconds because the disk is slowly failing, TigerBeetle will use cluster redundancy to mask the gray failure automatically without the user seeing any 4 second latency spike. This is a relatively new performance technique in the literature known as "tail tolerance".

#### Safety in TigerBeetle
TigerBeetle is designed to a higher safety standard than a general-purpose relational database such as MySQL or an in-memory database such as Redis:
Strict consistency, CRCs and crash safety are not enough.

* TigerBeetle **detects and repairs disk corruption** ([3.45% per 32 months, per disk](https://research.cs.wisc.edu/wind/Publications/latent-sigmetrics07.pdf)), **detects and repairs misdirected writes** where the disk firmware writes to the wrong sector ([0.042% per 17 months, per disk](https://research.cs.wisc.edu/wind/Publications/latent-sigmetrics07.pdf)), and **prevents data tampering** with hash-chained cryptographic checksums.
* TigerBeetle **uses Direct I/O by design** to side step cache coherency bugs in the kernel page cache after an EIO fsync error.
* TigerBeetle **exceeds the fsync durability of a single disk** and the hardware of a single server because disk firmware can contain bugs and because single server systems fail.
* TigerBeetle **provides strict serializability**, the gold standard of consistency, as a replicated state machine, and as a cluster of TigerBeetle servers (called replicas), for optimal high availability and distributed fault-tolerance.
* TigerBeetle **performs synchronous replication** to a quorum of TigerBeetle servers using the pioneering [Viewstamped Replication](http://pmg.csail.mit.edu/papers/vr-revisited.pdf) and consensus protocol, for low-latency automated leader election and to eliminate the risk of split brain associated with manual failover.
* TigerBeetle is “fault-aware” and **recovers from local storage failures in the context of the global consensus protocol**, providing [more safety than replicated state machines such as ZooKeeper and LogCabin](https://www.youtube.com/watch?v=fDY6Wi0GcPs). For example, TigerBeetle can disentangle corruption in the middle of the committed journal (caused by bitrot) from torn writes at the end of the journal (caused by power failure) to uphold durability guarantees given for committed data and maximize availability.
* TigerBeetle does not depend on synchronized system clocks, does not use leader leases, and **performs leader-based timestamping** so that your application can deal only with safe relative quantities of time with respect to transfer timeouts. To ensure that the leader's clock is within safe bounds of "true time", TigerBeetle combines all the clocks in the cluster to create a fault-tolerant clock that we call ["cluster time"](https://www.tigerbeetle.com/post/three-clocks-are-better-than-one).

##### Secure Transport
* HTTPS on central-ledger endpoints may be configured in NodeJS or WAF/Load-Balancer
* Communication between CentralLedger and TigerBeetle will make use of a secure VPN tunnel
* Non PCI-DSS sensitive information stored in TigerBeetle

##### Secure Storage at Rest
* TigerBeetle stores all data at rest encrypted under 256-bit `XChaCha20`.
* MySQL database for CentralLedger to be configured for:
  * TLS/SSL communication between CentralLedger and MySQL.
  * MysQL Keyring plugin may be used to encrypt data at rest (https://aimlessengineer.com/2021/04/26/data-at-rest-encryption-in-mysql/).
* Security of sensitive data in the database.

##### Other
* Authentication of the CL API
* Update CentralLedger documentation during the course of the project

#### Testing
Existing unit tests for CentralLedger will be updated to test TigerBeetle and CentralLedger integration. 
jUnit will be used to test performance and safety.

Testing coverage includes:
* Unit testing for TigerBeetle NodeJS client
* Integration testing for TigerBeetle NodeJS client
* Integration testing for CentralLedger and TigerBeetle
  * TB enabled
  * TB disabled (_traditional_)
* Performance, throughput and safety (_via jMeter_)

## 5. Assumptions, Dependencies & Considerations

### 5.1 Assumptions
> TODO @jason

### 5.2 Hardware Dependencies
The following hardware dependencies are known.
#### TigerBeetle
#### CentralLedger

### 5.3 Software Dependencies
The following software dependencies are known.
#### TigerBeetle
TigerBeetle release in a single executable file which is supported in the following operating systems:
* Linux (`x64`)
* MacOS (`x64/arm`)
* Windows (`x64`)

#### Mojaloop
The Mojaloop stack relies on the following software components for data storage:
* MySQL
* MongoDB
* Redis
* Kafka
* NodeJS

### 5.4 Considerations
> TODO @jason

## 6. Scope Exclusions
The following functionality will be excluded from Phase-1:
* Integration into CentralSettlement
* Updated NodeJS that merges TigerBeetle `Transfer`/`Commit`

## 7. Detailed Design
The detail design process primarily involves the conversion of the loft from the preliminary design into something that can be built and ultimately flown.

### 7.1. Participants
Sequence related to participants with relation to CentralLedger and TigerBeetle.

#### 7.1.1 Create Participant
![Participant Sequence](solution_design/sequence-participant-tb-enabled-create.png)

1. Participant JSON Payload for HTTP `POST`.
```json
{
  "id": "123",
  "name": "fspJM61d20f876f3c47828fc9f9a70",
  "currency": "USD",
  "newlyCreated": false
}
```
2. Handler invoked from `/jmeter/participants/create` endpoint.
3. Service layer invoked
4. Domain / Service to Facade layer
5. The Database transaction is initiated to store the participant and account data in the following tables;
   1. `participant`
   2. `participantCurrency`
   3. `participantPosition`
6. The TigerBeetle client is invoked to create the participant account.
7. The account create request is received.
8. The account is created and distributed to all TigerBeetle nodes in the cluster via VSR.
9. Errors for the account creation (which is always batched) is returned. In this case there were no errors.
10. The database transaction is regarded as successful, and the database transaction is committed.
11. Result returned.
12. Result returned.
13. Result returned.
14. `JSON` HTTP `200` response returned to indicate success.

#### 7.1.2 Lookup Participant by Name
![Participant Sequence](solution_design/sequence-participant-tb-enabled-lookup.png)

1. DFSP/Mojaloop Adapter invokes HTTP request
2. Handler invoked from `/jmeter/participants/{name}` `GET` endpoint.
3. Service layer invoked.
4. Domain / Service to Facade layer.
5. Participant data is retrieved from Redis via participant `name`.
   1. If the data is not available in the cache, a lookup in the CentralLedger database is performed, followed by caching the participant data.
6. The TigerBeetle client is invoked in order to obtain the `account` information.
7. TigerBeetle client fetches the necessary account information from one of the TigerBeetle nodes.
8. Account data is returned from the TigerBeetle client.
9. **OPTIONAL** Additional account meta-data is fetched based on `accountId`.
10. Result returned.
11. Result returned.
12. Result returned.
13. HTTP `JSON` response with account and balance related information.

### 7.2. Transfers
Sequence related to a transfer with relation to CentralLedger and TigerBeetle.

#### 7.2.1 Create Transfer (2-Phase)
![Transfer Sequence](solution_design/sequence-transfer-tb-enabled-create.png)

1. DFSP submits a Transfer JSON Payload (Prepare followed by Fulfil)
```json
{
  "payerFsp": "fspJM962250a50c654d1a9f3d32b9a",
  "amount": {
      "amount": 95,
      "currency": "USD"
  },
  "condition": "GRzLaTP7DJ9t4P-a_BA0WA9wzzlsugf00-Tn6kESAfM",
  "payeeFsp": "fspJM9bd046148c074bdca6323ab12",
  "ilpPacket": "AYIBgQAAAAAAAASwNGxldmVsb25lLmRmc3AxLm1lci45T2RTOF81MDdqUUZERmZlakgyOVc4bXFmNEpLMHlGTFGCAUBQU0svMS4wCk5vbmNlOiB1SXlweUYzY3pYSXBFdzVVc05TYWh3CkVuY3J5cHRpb246IG5vbmUKUGF5bWVudC1JZDogMTMyMzZhM2ItOGZhOC00MTYzLTg0NDctNGMzZWQzZGE5OGE3CgpDb250ZW50LUxlbmd0aDogMTM1CkNvbnRlbnQtVHlwZTogYXBwbGljYXRpb24vanNvbgpTZW5kZXItSWRlbnRpZmllcjogOTI4MDYzOTEKCiJ7XCJmZWVcIjowLFwidHJhbnNmZXJDb2RlXCI6XCJpbnZvaWNlXCIsXCJkZWJpdE5hbWVcIjpcImFsaWNlIGNvb3BlclwiLFwiY3JlZGl0TmFtZVwiOlwibWVyIGNoYW50XCIsXCJkZWJpdElkZW50aWZpZXJcIjpcIjkyODA2MzkxXCJ9IgA",
  "expiration": null,
  "transferId": "f75f50d8-f584-4451-889b-fee8bc350db0",
  "fulfil": false
}
```
2. Handler invoked from `/jmeter/transfers/prepare` `POST` endpoint.
3. Service layer invoked
4. Domain / Service to Facade layer.
5. The following validations are performed prior to the transfer:
   1. `validateFspiopSourceMatchesPayer` -> Asserts headers['fspiop-source'] matches payload.payerFsp
   2. `validateParticipantByName` -> Asserts payer participant exists by doing a lookup by name (discards result of lookup work)
   3. `validatePositionAccountByNameAndCurrency` -> Asserts account exists for name-currency tuple (discards result of lookup work)
   4. `validateParticipantByName` -> Asserts payee participant exists by doing a lookup by name (discards result of lookup work)
   5. `validatePositionAccountByNameAndCurrency` -> Asserts account exists for name-currency tuple (discards result of lookup work)
   6. `validateAmount` -> Validates allowed scale of decimal places and allowed precision
   7. `validateConditionAndExpiration` -> Validates condition and expiration
   8. `validateDifferentDfsp` -> Asserts lowercase string comparison of `payload.payerFsp` and `payload.payeeFsp` is different
6. Lookup Payer and Payee Participants via name, account type (_POSITION_) and currency
   1. If participant data is not available in Redis, a database lookup is performed, followed by the participant data being cached in Redis
7. TigerBeetle pending `pending = true` //Transfer// created via TigerBeetle NodeJS client.
```zig
Transfer{
    .id = 1002,
    .debit_account_id = 1,
    .credit_account_id = 2,
    .user_data = 0,
    .reserved = [_]u8{0} ** 32,
    .timeout = std.time.ns_per_hour,
    .code = 0,
    .flags = .{
        .pending = true, // Set this transfer to be two-phase.
        .linked = true, // Link this transfer with the next transfer 1003.
    },
    .amount = 1,
}
```
8. Transfer distributed via the TigerBeetle state machine.
9. The transfer is distributed to all 7 TigerBeetle nodes in the cluster.
10. The TigerBeetle API responds with errors during the transfer, which is empty (_no errors_).
11. Return result to service layer.
12. Return result to the handler layer.
13. Prepare the result in JSON format.
14. Result is returned to the DFSP via the REST API
15. Insert duplicate check for //Transfer// into the following database table:
    1. `transferDuplicateCheck`
16. Insert //Transfer// data into the following database tables:
    1. `transfer`
    2. `transferParticipant` (Payer)
    3. `transferParticipant` (Payee)
    4. `ilpPacket`
    5. `transferStateChange`
    6. `participantPosition`
    7. `participantPositionChange`
17. The //Transfer// database transaction is committed.
18. DFSP submits a //Transfer// JSON Fulfi Payload
```json
{
  "transferId": "f75f50d8-f584-4451-889b-fee8bc350db0",
  "fulfil": true
}
```
19. Handler invoked from `/jmeter/transfers/fulfil` `POST` endpoint.
20. Service layer invoked
21. Domain / Service to Facade layer. 
22. New database transaction is initiated for the //Transfer// fulfil.
23. The current open `settlementWindowId` is obtained for the current **OPEN** settlement window.
    1. The settlement window is based on **OPEN** state and currency.
24. The TigerBeetle client is invoked with a //Transfer// `post_pending_transfer = true` property
```zig
Transfer{
    .id = 1001,
    .debit_account_id = 1,
    .credit_account_id = 2,
    .user_data = 0,
    .reserved = [_]u8{0} ** 32,
    .timeout = 0,
    .code = 0,
    .flags = .{ .post_pending_transfer = true }, // Post the pending two-phase transfer.
    .amount = 0, // Inherit the amount from the pending transfer.
},
```
25. Transfer fulfillment distributed via the TigerBeetle state machine.
26. The transfer fulfilment is distributed to all 7 TigerBeetle nodes in the cluster.
27. The TigerBeetle client API responds with errors during the transfer, which is empty (_no errors_).
28. Insert //Transfer// fulfilment data into the following database tables:
    1. `transferFulfilment`
    2. `transferStateChange`
29. Database transaction is commmitted.
30. Return result to service layer.
31. Return result to the handler layer.
32. Prepare the result in JSON format.
33. Result is returned to the DFSP via the REST API

#### 7.2.2 Lookup Transfer by ID
![Transfer Sequence](solution_design/sequence-transfer-tb-enabled-lookup.png)

1. DFSP/Mojaloop Adapter invokes HTTP request
2. Handler invoked from `/jmeter/participants/{name}/transfers/{id}` `GET` endpoint.
3. Service layer invoked.
4. Domain / Service to Facade layer.
5. The TigerBeetle client is invoked in order to obtain the `account` information.
6. TigerBeetle client fetches the necessary account information from one of the TigerBeetle nodes.
7. Transfer data is returned from the TigerBeetle client.
8. **OPTIONAL** Additional transfer meta-data is fetched based on `transactionId`.
9. Result returned.
10. Result returned.
11. Result returned.
12. HTTP `JSON` response with transfer related information.

### 7.3. Settlement
> TODO @jason

## 8. Canonical Model
The following CentralLedger and TigerBeetle Canonical Data Model presents data entities and relationships in the simplest possible form.

### 8.1 TigerBeetle
TigerBeetle supports only `Account` and `Transfer` data types.

#### 8.1.1 Account
Mutable data set for account related data.

| Field            | Type              | Description                                                                                                                      |
|------------------|-------------------|----------------------------------------------------------------------------------------------------------------------------------|
| id               | `u128`            | Global unique id for an account.                                                                                                 |
| user_data        | `u128`            | Implementation specific data on account. Opaque third-party identifier to link this account (many-to-one) to an external entity. |
| reserved         | `[48]u8 - array`  | Accounting policy primitives. Not available.                                                                                     |
| unit             | `u16`             | A transfer unit describing the currency associated with the account.                                                             |
| code             | `u16`             | A chart of accounts code describing the type of account (e.g. clearing, settlement)                                              |
| flags            | `AccountFlags`    | See account flags.                                                                                                               |
| debits_pending   | `u64`             | Balance for reserved debits.                                                                                                     |
| debits_posted    | `u64`             | Balance for accepted debits.                                                                                                     |
| credits_pending  | `u64`             | Balance for reserved credits.                                                                                                    |
| credits_posted   | `u64`             | Balance for accepted credits.                                                                                                    |
| timestamp        | `u64`             | The current state machine timestamp of the account for state tracking.                                                           |

#### 8.1.2 AccountFlags - `[packed struct]`

| Field                            | Type              | Description                                  |
|----------------------------------|-------------------|----------------------------------------------|
| linked                           | `bool`            | Is the account linked to another account.    |
| debits_must_not_exceed_credits   | `bool`            | Total debit transfer may not exceed credits. |
| credits_must_not_exceed_debits   | `bool`            | Total credit transfer may not exceed debits. |
| padding                          | `u29`             | Data to be used for padding.                 |

#### 8.1.3 Transfer
Transfers for TB are immutable.

| Field             | Type              | Description                                                                                                                       |
|-------------------|-------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| id                | `u128`            | Global unique id for an account.                                                                                                  |
| debit_account_id  | `u128`            | The unit transfer from (Payer) account id.                                                                                        |
| credit_account_id | `u128`            | The unit transfer to (Payee) account id                                                                                           |
| user_data         | `u128`            | Implementation specific data on transfer. Opaque third-party identifier to link this transfer (many-to-one) to an external entity |
| reserved          | `[32]u8 - array`  | Accounting policy primitives. Example; ILP packet.                                                                                |
| timeout           | `u64`             | Used for a 2-phase transfer. The maximum wait timeout in milliseconds for a commit.                                               |
| code              | `u32`             | A chart of accounts code describing the reason for the transfer (e.g. deposit, settlement)                                        |
| flags             | `TransferFlags`   | See transfer flags.                                                                                                               |
| amount            | `u64`             | Transfer amount in units.                                                                                                         |
| timestamp         | `u64`             | The current state machine timestamp of the transfer for state tracking.                                                           |

#### 8.1.4 TransferFlags - `[packed struct]`
Transfer flags are properties associated with a Transfer to enable additional Transfer functionality, such as:
* 2-Phase transfers
* Linked Transfer
* Reverting a previously created transfer
* Reverting a previously committed transfer

| Field       | Type              | Description                                    |
|-------------|-------------------|------------------------------------------------|
| linked      | `bool`            | Is the account linked to another account.      |
| pending     | `bool`            | Is the transfer a 2-phase commit transfer.     |
| condition   | `bool`            | Does the transfer support transfer conditions. |
| padding     | `u29`             | Data to be used for padding.                   |

### 8.2 CentralLedger
CentralLedger hosts a wide range of tables in which to store Participant, Account and Transfer related data.

#### 8.2.1 Data Relationships
The following diagrams are used to illustration the relationships between data in CentralLedger.

##### Participants and Accounts
![SQL Relationships - Participants](solution_design/central-ledger-data-participant.png)

##### Transfer
![SQL Relationships - Transfers](solution_design/central-ledger-data-transfer.png)


#### 8.2.2 Participant (`participant`)
| Field         | Type           | Description                                        |
|---------------|----------------|----------------------------------------------------|
| participantId | `int unsigned` | Unique participant identifier.                     |
| name          | `varchar(256)` | Unique participant name.                           |
| description   | `varchar(256)` | Brief description for a participant.               |
| isActive      | `tinyint`      | Is the participant account active.                 |
| createdDate   | `datetime`     | Timestamp of when the participant was created.     |
| createdBy     | `datetime`     | The DFSP responsible for creating the participant. |

#### 8.2.3 Participant Currency (`participantCurrency`)
| Field                 | Type           | Description                                                |
|-----------------------|----------------|------------------------------------------------------------|
| participantCurrencyId | `int unsigned` | Unique participantCurrency identifier.                     |
| participantId         | `int unsigned` | Foreign key for participant table.                         |
| currencyId            | `int unsigned` | Foreign key for currency table.                            |
| ledgerAccountTypeId   | `int unsigned` | Foreign key for ledgerAccountType table.                   |
| isActive              | `tinyint`      | Is the currency active.                                    |
| createdDate           | `datetime`     | Timestamp of when the participantCurrency was created.     |
| createdBy             | `datetime`     | The DFSP responsible for creating the participantCurrency. |

#### 8.2.4 Participant Contact (`participantContact`)
| Field                 | Type           | Description                                                  |
|-----------------------|----------------|--------------------------------------------------------------|
| participantContactId  | `int unsigned` | Unique participantContact identifier.                        |
| participantId         | `int unsigned` | Foreign key for participant table.                           |
| contactTypeId         | `int unsigned` | Foreign key for contactType table.                           |
| value                 | `varchar(256)` | The details for the contact.                                 |
| priorityPreference    | `int(9)`       | The priority for the contact.                                |
| isActive              | `tinyint`      | Whether the contact is active.                               |
| createdDate           | `datetime`     | Timestamp of when the participantContact was created.        |
| createdBy             | `datetime`     | The DFSP responsible for creating the participantContact.    |


#### 8.2.5 Transfer (`transfer`)
| Field          | Type            | Description                                                                    |
|----------------|-----------------|--------------------------------------------------------------------------------|
| transferId     | `varchar(36)`   | Unique transfer identifier.                                                    |
| amount         | `decimal(18,4)` | The amount of the transfer.                                                    |
| currencyId     | `varchar(3)`    | Foreign key to the transfer currency.                                          |
| ilpCondition   | `varchar(256)`  | The condition from the ILP packet.                                             |
| expirationDate | `datetime`      | The timestamp for when the 2-phase transfer expires in the event of no commit. |
| createdDate    | `datetime`      | The timestamp for when the transfer was created.                               |

#### 8.2.6 Transfer Participant (`transferParticipant`)
| Field                         | Type              | Description                                                 |
|-------------------------------|-------------------|-------------------------------------------------------------|
| transferParticipantId         | `bigint unsigned` | Unique transferParticipant identifier.                      |
| transferId                    | `varchar(36)`     | Foreign key for the transfer.                               |
| participantCurrencyId         | `int unsigned`    | Foreign key for the participantCurrencyId.                  |
| transferParticipantRoleTypeId | `int unsigned`    | Foreign key for the transferParticipantRoleTypeId.          |
| ledgerEntryTypeId             | `int unsigned`    | Foreign key for the ledgerEntryTypeId.                      |
| amount                        | `decimal(18,4)`   | The amount of the transfer.                                 |
| createdDate                   | `datetime`        | The timestamp for when the transferParticipant was created. |

#### 8.2.7 ILP Packet (`ilpPacket`)
| Field        | Type          | Description                                       |
|--------------|---------------|---------------------------------------------------|
| transferId   | `varchar(36)` | Foreign key for the transfer.                     |
| value        | `text`        | Complete ilpPacket.                               |
| createdDate  | `datetime`    | The timestamp for when the ilpPacket was created. |

#### 8.2.8 Transfer State Change (`transferStateChange`)
| Field                 | Type              | Description                                                 |
|-----------------------|-------------------|-------------------------------------------------------------|
| transferStateChangeId | `bigint`          | Unique transferStateChange identifier.                      |
| transferId            | `varchar(36)`     | Foreign key for the transfer.                               |
| transferStateId       | `varchar(50)`     | Foreign key for the transferStateId.                        |
| reason                | `varchar(512)`    | Reason for state change.                                    |
| createdDate           | `datetime`        | The timestamp for when the transferStateChange was created. |

#### 8.2.9 Participant Position (`participantPosition`)
| Field                 | Type              | Description                                                      |
|-----------------------|-------------------|------------------------------------------------------------------|
| participantPositionId | `bigint unsigned` | Unique participantPosition identifier.                           |
| participantCurrencyId | `int unsigned`    | Foreign key for the participantCurrency.                         |
| value                 | `decimal(18,4)`   | Current participant position.                                    |
| reservedValue         | `decimal(18,4)`   | Current participant reserved position.                           |
| changedDate           | `datetime`        | The timestamp for when the participantPosition was last updated. |

#### 8.2.10 Participant Position Change (`participantPositionChange`)
| Field                       | Type                 | Description                                                    |
|-----------------------------|----------------------|----------------------------------------------------------------|
| participantPositionChangeId | `bigint unsigned`    | Unique participantPositionChangeId identifier.                 |
| participantPositionId       | `bigint unsigned`    | Foreign key for the participantPosition.                       |
| transferStateChangeId       | `bigint unsigned`    | Foreign key for the transferStateChange.                       |
| value                       | `decimal(18,4)`      | The participant position at time of state change.              |
| reservedValue               | `decimal(18,4)`      | The participant reserved position at time of state change.     |
| createdDate                 | `datetime`           | The timestamp for when the participantPositionChange occurred. |

#### 8.2.11 Participant Limit (`participantLimit`)
| Field              | Type   | Description                    |
|--------------------|--------|--------------------------------|
| participantLimitId | `TODO` | Unique participant identifier. |
| value              | `TODO` | Unique participant identifier. |

#### 8.2.12 Transfer Duplicate Check (`transferDuplicateCheck`)
| Field        | Type           | Description                                                  |
|--------------|----------------|--------------------------------------------------------------|
| transferId   | `varchar(32)`  | Unique transfer identifier (UUID).                           |
| hash         | `varchar(256)` | Unique hash for the transfer JSON request.                   |
| createdDate  | `datetime`     | The timestamp for when the transferDuplicateCheck occurred.  |

