# TradeFinance

A DAML smart contract project that models a simple trade finance workflow for gold-backed transactions.

The project demonstrates how parties can propose a trade, accept it, receive regulator approval, settle the deal, and prevent invalid post-settlement disputes.

## Overview

`TradeFinance` contains three main templates:

- `GoldToken`: represents gold ownership in ounces and supports holder-controlled transfers.
- `DealProposal`: lets a seller propose a trade to a buyer.
- `TradeFinanceDeal`: tracks a buyer/seller trade through proposed, approved, settled, and disputed states.

The workflow is designed around DAML authorization:

- Sellers create deal proposals.
- Buyers accept or reject proposals.
- Regulators observe deals and approve proposed trades.
- Buyers settle approved deals or dispute unsettled deals.

## Flow Diagram

![TradeFinance DAML contract flow](docs/diagrams/trade-finance-flow.png)

## Project Structure

```text
.
├── daml.yaml
├── daml
│   ├── TradeFinance.daml
│   └── Test.daml
├── docs
│   └── diagrams
│       ├── trade-finance-flow.excalidraw
│       ├── trade-finance-flow.png
│       └── trade-finance-flow.svg
├── .dlint.yaml
├── .gitattributes
└── .gitignore
```

## Requirements

- DAML SDK `2.10.4`

Check the installed DAML SDK versions:

```bash
daml version
```

## Build

Compile the DAML project:

```bash
daml build
```

## Test

Run the DAML Script tests:

```bash
daml test
```

The test module covers:

- A full proposal, acceptance, approval, and settlement lifecycle.
- Rejection of a dispute after settlement.
- Rejection of an unauthorized approval attempt by an outsider.

## Deploy to Canton

You can deploy this DAML package to a local Canton Sandbox for development, or to an existing remote Canton participant.

### Local Canton Sandbox

Build the DAR:

```bash
daml build
```

Start a local Canton Sandbox in a separate terminal:

```bash
daml sandbox --port 6865 --admin-api-port 6866
```

Deploy the package to the sandbox ledger API:

```bash
daml deploy --host localhost --port 6865
```

Confirm the ledger is reachable:

```bash
daml ledger list-parties --host localhost --port 6865
```

Use port `6865` for DAML ledger API commands. Port `6866` is the Canton admin API port.

### Remote Canton Participant

For a remote Canton participant, replace the placeholders with your network values:

```bash
daml build
daml deploy --host CANTON_HOST --port CANTON_LEDGER_API_PORT
```

If the participant requires authentication, pass an access token file:

```bash
daml deploy \
  --host CANTON_HOST \
  --port CANTON_LEDGER_API_PORT \
  --access-token-file TOKEN_FILE
```

If the participant requires TLS, include the TLS flags provided by the network operator:

```bash
daml deploy \
  --host CANTON_HOST \
  --port CANTON_LEDGER_API_PORT \
  --tls \
  --cacrt path/to/ca.crt \
  --access-token-file TOKEN_FILE
```

For more controlled deployments, upload the built DAR and allocate parties explicitly:

```bash
daml ledger upload-dar \
  --host CANTON_HOST \
  --port CANTON_LEDGER_API_PORT \
  .daml/dist/TradeFinance-0.0.1.dar

daml ledger allocate-parties \
  --host CANTON_HOST \
  --port CANTON_LEDGER_API_PORT \
  Seller Buyer Regulator
```

For mutual TLS, add the certificate and private key flags required by your participant:

```bash
--pem path/to/client.pem --crt path/to/client.crt
```

### Deployment Troubleshooting

- Use the Canton participant ledger API port for `daml deploy` and `daml ledger` commands.
- Use the Canton admin API port only for Canton administration, not DAML package deployment.
- Run `daml build` first if `.daml/dist/TradeFinance-0.0.1.dar` does not exist.
- If the local sandbox cannot start, check whether another process is using ports `6865` or `6866`.
- If a remote participant rejects the connection, confirm the host, ledger API port, token, and TLS certificate settings with the network operator.

## Main Workflow

1. A seller creates a `DealProposal`.
2. The buyer accepts the proposal, creating a `TradeFinanceDeal`.
3. The regulator approves the proposed deal.
4. The buyer settles the approved deal.
5. A settled deal can no longer be disputed.

## License

No license has been specified.
