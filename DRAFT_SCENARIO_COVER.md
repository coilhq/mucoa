# Settlement models:
- immediate gross bilateral
- deferred net bilateral (between two participants)
- deferred net multilateral (between a participant and the scheme)
- immediate net multilateral (between a participant and the scheme)

immediate gross:

# Transfer

# Transfer 1 [A to B]
```
DR Participant A Liquidity                  100
    CR Participant A Payable To Scheme                  100
DR Participant A Payable To Scheme          100
    CR Participant B Liquidity                          100
```

# Transfer 2 [B to A]
```
DR Participant B Liquidity                  10
    CR Participant B Payable To Scheme                  10
DR Participant B Payable To Scheme          10
    CR Participant A Liquidity                          10
```


> insight: we don't need journal entries for deferred net bi/multilateral
> we don't need to change our chart of accounts

settlement report (immediate gross bilateral):

    A paid to B = 100
    B paid to A = 10

settlement report (deferred net bilateral):

    A owes B = 100 - 10 = 90

# Settlement

# Reserve
The reservation may be immediate or deferred.

## Reserve 1 - A to B
```
DR Participant A Payable To Scheme              100
    CR Participant B Receivable From Scheme                 100
```
## Reserve 2 - B to A
```
DR Participant B Payable To Scheme              10
    CR Participant A Receivable From Scheme                 10
```

At this point we have deferred gross multilateral settlement for reservation is:

    Participant A receivable from Scheme = 10
    Participant B receivable from Scheme = 100

deferred net multilateral:

    A owes B = 100 (gross)
    B owes A = 10 (gross)
    A owes B = 90 (net)
    A owes Scheme = 100
    B owes Scheme = 10

# Commit
The commit may be immediate or deferred.

## Commit 1.1
```
DR Participant B Receivable From Scheme         100
    CR Participant A Payable To Scheme                      100
```

## Commit 1.2
```
DR Participant A Receivable From Scheme         10
    CR Participant B Payable To Scheme                      10
```

At this point we have deferred gross multilateral settlement for reservation is:

    Scheme Recon A paid to Settlement A = 100
    Scheme Recon B paid to Settlement B = 10
    Scheme Settlement to the Participant A = 100
    Scheme Settlement to the Participant B  = 10

deferred net multilateral:

    A settled B = 90 (100 - 10 = 90)
    A settled Scheme = 100
    B settled Scheme = 10
> Both Scheme payable and receivable Accounts are set to `0` after settlement commit (happy path).
> The Participant Liquidity remains unaffected since transfer. 

# Other Notes;

* MJL
* [Multilateral deferred net settlement] - Settlements are deferred net if a number of transfers are settled together. Net settements (in which a number of transfers are settled together) are by definition deferred (since it takes time to construct a batch.)
    * Settlements are gross if each transfer is settled separately. Gross settlements may be immediate or deferred. They are deferred if approval for settlement from outside the Hub is required, and immediate if the Hub can proceed to settlement of a transfer without requiring any external approval. At present, Mojaloop only supports immediate gross settlements.
* [Bilateral deferred net settlement] - Settlements are bilateral if each pair of participants settles with each other for the net of all transfers between them. Settlements are multilateral if each participant settles with the Hub for the net of all transfers to which it has been a party, no matter who the other party was.
    *
* Immediate gross settlement

> A settlement model specifies a way in which a Mojaloop Hub will settle a set of transfers. In the simple case, there is only one settlement model and it settles all the transfers that are processed by the Hub. However, Mojaloop supports more than one settlement model for a single scheme. This allows, for instance, a scheme to define different settlement models for different currencies, or for different ledger account types.
