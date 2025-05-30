---
title: Celo Asset Management Guide
description: Access and account management for holding, exchanging, or sending Celo Dollars (cUSD) and Mento stablecoins.
---

# Asset Management

Access and account management for holding, exchanging, or sending Celo Dollars (cUSD) and Mento stablecoins.

:::warning
As of block height 31,056,500 (March 26, 2025, 3:00 AM UTC), Celo is no longer a standalone Layer 1 blockchain—it is now an Ethereum Layer 2!
Some documentation may be outdated as updates are in progress. If you encounter issues, please [file a bug report](https://github.com/celo-org/docs/issues/new/choose).

For the most up-to-date information, refer to our [Celo L2 documentation](https://docs.celo.org/cel2).
:::

---

## Prerequisites

This guide assumes:

- You have read [Key Management](/what-is-celo/about-celo-l1/validator/key-management/summary) on Celo
- You have installed the [Celo Command Line Interface](/cli/) (Celo CLI)

## Choose a Node

In order to execute the tasks listed below, you will need to point the Celo CLI to a node that is synchronized with the [Mainnet](/network/).

## Create an Account

There are two ways to create an account:

- (Recommended) use [accounts generated by Ledger](/wallet/ledger/setup), if you possess a [Ledger hardware wallet](https://shop.ledger.com/products/ledger-nano-s)
- Use CLI to [generate an account](/network/node/run-mainnet#create-an-account-and-get-its-address) -- this approach is less secure and hence not recommended

After creating an account, record its address in environment variables:

```
export CELO_ACCOUNT_ADDRESS=<YOUR-CELO-ACCOUNT-ADDRESS>
```

## Exchange CELO for Mento Stablecoins

Once you have deposited CELO to your account, you can check your balance:

```
celocli account:balance $CELO_ACCOUNT_ADDRESS
```

As an example of a common stablecoin swap, you can exchange CELO for cUSD using the following command. This exchanges CELO for stable tokens (cUSD by default) via the stability mechanism. Note that the unit of value is CELO Wei (1 CELO = 10^18 CELO Wei).

```
celocli exchange:celo --value <VALUE-TO-EXCHANGE> --from $CELO_ACCOUNT_ADDRESS
```

## Transfer Mento Stablecoins

When you have sufficient balance, you can send Mento stablecoins such as cUSD to other accounts. Note that the unit of value is cUSD Wei (1 cUSD = 10^18 cUSD Wei).

```
celocli transfer:dollars --from $CELO_ACCOUNT_ADDRESS --to <RECIPIENT-ADDRESS> --value <VALUE-TO-TRANSFER>
```
