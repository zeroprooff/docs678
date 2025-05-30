---
title: Running a Celo Validator
description: How to get a Validator node running on the Celo Mainnet.
---

# Running a Validator

How to get a Validator node running on the Celo Mainnet.

:::warning
As of block height 31,056,500 (March 26, 2025, 3:00 AM UTC), Celo is no longer a standalone Layer 1 blockchain—it is now an Ethereum Layer 2!
Some documentation may be outdated as updates are in progress. If you encounter issues, please [file a bug report](https://github.com/celo-org/docs/issues/new/choose).

For the most up-to-date information, refer to our [Celo L2 documentation](https://docs.celo.org/cel2).
:::

---

:::info

If you would like to keep up-to-date with all the news happening in the Celo community, including validation, node operation and governance, please sign up to our [Celo Signal mailing list here](https://share.hsforms.com/1Qrhush1vSA2WIamd_yL4ow53n4j).

You can add the [Celo Signal public calendar](https://calendar.google.com/calendar/u/0/embed?src=c_9su6ich1uhmetr4ob3sij6kaqs@group.calendar.google.com) as well which has relevant dates.

:::

## What is a Validator?

Validators help secure the Celo network by participating in Celo’s proof-of-stake protocol. Validators are organized into Validator Groups, analogous to parties in representative democracies. A Validator Group is essentially an ordered list of Validators.

Just as anyone in a democracy can create their own political party, or seek to get selected to represent a party in an election, any Celo user can create a Validator group and add themselves to it, or set up a potential Validator and work to get an existing Validator group to include them.

While other Validator Groups will exist on the Celo Network, the fastest way to get up and running with a Validator will be to register a Validator Group, register a Validator, and affliate that Validator with your Validator Group. The addresses used to register Validator Groups and Validators must be unique, which will require that you create two accounts in the step-by-step guide below.

Because of the importance of Validator security and availability, Validators are expected to run a "proxy" node in front of each Validator node. In this setup, the Proxy node connects with the rest of the network, and the Validator node communicates only with the Proxy, ideally via a private network.

[Read more about Celo's mission and why you may want to become a Validator.](https://medium.com/celoorg/calling-all-chefs-become-a-celo-validator-c75d1c2909aa) - This article still uses the term Celo Gold which is the deprecated name for the Celo native asset, which now is referred to simply as "Celo" or preferably "CELO".

## Prerequisites

### Staking Requirements

Celo uses a [proof-of-stake](/what-is-celo/about-celo-l1/protocol/pos/) consensus mechanism, which requires Validators to have locked CELO to participate in block production. The current requirement is 10,000 CELO to register a Validator, and 10,000 CELO _per member validator_ to register a Validator Group.

If you do not have the required CELO to lock up, you can try out of the process of creating a validator on the Baklava network by following the [Running a Validator in Baklava guide](/what-is-celo/about-celo-l1/validator/run/baklava)

We will not discuss obtaining CELO here, but it is a prerequisite that you obtain the required CELO.

### Hardware requirements

The recommended Celo Validator setup involves continually running two instances:

- 1 **Validator node**: should be deployed to single-tenant hardware in a secure, high availability data center
- 1 **Validator Proxy node**: can be a VM or container in a multi-tenant environment (e.g. a public cloud), but requires high availability

Celo is a proof-of-stake network, which has different hardware requirements than a Proof of Work network. proof-of-stake consensus is less CPU intensive, but is more sensitive to network connectivity and latency. Below is a list of standard requirements for running Validator and Proxy nodes on the Celo Network:

#### Validator node

- CPU: At least 4 cores / 8 threads x86_64 with 3ghz on modern CPU architecture newer than 2018 Intel Cascade Lake or Ryzen 3000 series or newer with a Geekbench 5 Single Threaded score of >1000 and Multi Threaded score of > 4000
- Memory: 32GB
- Disk: 512GB SSD or NVMe (resizable). Current chain size at August 16th is ~190GB, so 512GB is a safe bet for the next 1 year. We recommend using a cloud provider or storage solution that allows you to resize your disk without downtime.
- Network: At least 1 GB input/output Ethernet with a fiber (low latency) Internet connection, ideally redundant connections and HA switches.

Some cloud instances that meet the above requirements are:

- GCP: n2-highmem-4, n2d-highmem-4 or c3-highmem-4
- AWS: r6i.xlarge, r6in.xlarge, or r6a.xlarge
- Azure: Standard_E4_v5, or Standard_E4d_v5 or Standard_E4as_v5

#### Proxy node

- CPU: At least 4 cores / 8 threads x86_64 with 3ghz on modern CPU architecture newer than 2018 Intel Cascade Lake or Ryzen 3000 series or newer with a Geekbench 5 Single Threaded score of >1000 and Multi Threaded score of > 4000
- Memory: 16GB
- Disk: 512GB SSD or NVMe (resizable). Current chain size at August 16th is ~190GB, so 512GB i,s a safe bet for the next 1 year. We recommend using a cloud provider or storage solution that allows you to resize your disk without downtime.
- Network: At least 1 GB input/output Ethernet with a fiber (low latency) Internet connection, ideally redundant connections and HA switches.

Some cloud instances that meet the above requirements are:

- GCP: n2-standard-4, n2d-standard-4 or c3-standard-4
- AWS: M6i.xlarge, M6in.xlarge, or M6a.xlarge
- Azure: Standard_D4_v5, or Standard_D4_v4 or Standard_D4as_v5

In addition, to get things started, it will be useful to run a node on your local machine that you can issue CLI commands against.

### Networking requirements

In order for your Validator to participate in consensus, it is **critically** important to configure your network correctly.

Your Proxy node must have static, external IP addresses, and your Validator node must be able to communicate with the Proxy, either via an internal network or via the Proxy's external IP address.

On the Validator machine, port 30503 should accept TCP connections from the IP address of your Proxy machine. This port is used by the Validator to communicate with the Proxy.

On the Proxy machine, port 30503 should accept TCP connections from the IP address of your Validator machine. This port is used by the Proxy to communicate with the Validator.

On the Proxy machine, port 30303 should accept TCP and UDP connections from all IP addresses. This port is used to communicate with other nodes in the network.

To illustrate this, you may refer to the following table:

| Machine \\ IPs open to | 0\.0\.0\.0/0 \(all\) | your\-validator\-ip | your\-proxy\-ip |
| ---------------------- | -------------------- | ------------------- | --------------- |
| Validator              |                      |                     | tcp:30503       |
| Proxy                  | tcp:30303, udp:30303 | tcp:30503           |                 |

### Software requirements

#### On each machine

- **You have Docker installed.**

  If you don’t have it already, follow the instructions here: [Get Started with Docker](https://www.docker.com/get-started). It will involve creating or signing in with a Docker account, downloading a desktop app, and then launching the app to be able to use the Docker CLI. If you are running on a Linux server, follow the instructions for your distro [here](https://docs.docker.com/install/#server). You may be required to run Docker with `sudo` depending on your installation environment.
  You can check you have Docker installed and running if the command `docker info` works properly.

#### On your local machine

- **You have celocli installed.**

  See [Command Line Interface (CLI)](/cli/)for instructions on how to get set up.

- **You are using the latest Node 10.x LTS**

  Some users have reported issues using the most recent version of node. Use the LTS for greater reliability.

:::info

A note about conventions:
The code snippets you'll see on this page are bash commands and their output.

When you see text in angle brackets &lt;&gt;, replace them and the text inside with your own value of what it refers to. Don't include the &lt;&gt; in the command.

:::

### Key Management

Private keys are the central primitive of any cryptographic system and need to be handled with extreme care. Loss of your private key can lead to irreversible loss of value.

This guide contains a large number of keys, so it is important to understand the purpose of each key. [Read more about key management.](/what-is-celo/about-celo-l1/validator/key-management/summary)

#### Unlocking

Celo nodes store private keys encrypted on disk with a password, and need to be "unlocked" before use. Private keys can be unlocked in two ways:

1. By running the `celocli account:unlock` command. Note that the node must have the "personal" RPC API enabled in order for this command to work.
2. By setting the `--unlock` flag when starting the node.

It is important to note that when a key is unlocked you need to be particularly careful about enabling access to the node's RPC APIs.

### Environment variables

There are a number of environment variables in this guide, and you may use this table as a reference.

| Variable                                    | Explanation                                                                                                                          |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| CELO_IMAGE                                  | The Docker image used for the Validator and proxy containers                                                                         |
| CELO_VALIDATOR_GROUP_ADDRESS                | The account address for the Validator Group                                                                                          |
| CELO_VALIDATOR_ADDRESS                      | The account address for the Validator                                                                                                |
| CELO_VALIDATOR_GROUP_SIGNER_ADDRESS         | The validator (group) signer address authorized by the Validator Group account.                                                      |
| CELO_VALIDATOR_GROUP_SIGNER_SIGNATURE       | The proof-of-possession of the Validator Group signer key                                                                            |
| CELO_VALIDATOR_SIGNER_ADDRESS               | The validator signer address authorized by the Validator Account                                                                     |
| CELO_VALIDATOR_SIGNER_PUBLIC_KEY            | The ECDSA public key associated with the Validator signer address                                                                    |
| CELO_VALIDATOR_SIGNER_SIGNATURE             | The proof-of-possession of the Validator signer key                                                                                  |
| CELO_VALIDATOR_SIGNER_BLS_PUBLIC_KEY        | The BLS public key for the Validator instance                                                                                        |
| CELO_VALIDATOR_SIGNER_BLS_SIGNATURE         | A proof-of-possession of the BLS public key                                                                                          |
| CELO_VALIDATOR_GROUP_VOTE_SIGNER_ADDRESS    | The address of the Validator Group vote signer                                                                                       |
| CELO_VALIDATOR_GROUP_VOTE_SIGNER_PUBLIC_KEY | The ECDSA public key associated with the Validator Group vote signer address                                                         |
| CELO_VALIDATOR_GROUP_VOTE_SIGNER_SIGNATURE  | The proof-of-possession of the Validator Group vote signer key                                                                       |
| CELO_VALIDATOR_VOTE_SIGNER_ADDRESS          | The address of the Validator vote signer                                                                                             |
| CELO_VALIDATOR_VOTE_SIGNER_PUBLIC_KEY       | The ECDSA public key associated with the Validator vote signer address                                                               |
| CELO_VALIDATOR_VOTE_SIGNER_SIGNATURE        | The proof-of-possession of the Validator vote signer key                                                                             |
| PROXY_ENODE                                 | The enode address for the Validator proxy                                                                                            |
| PROXY_INTERNAL_IP                           | (Optional) The internal IP address over which your Validator can communicate with your proxy                                         |
| PROXY_EXTERNAL_IP                           | The external IP address of the proxy. May be used by the Validator to communicate with the proxy if PROXY_INTERNAL_IP is unspecified |
| DATABASE_URL                                | The URL under which your database is accessible, currently supported are `postgres://`, `mysql://` and `sqlite://`                   |

## Validator Node Setup

This section outlines the steps needed to configure your Proxy and Validator nodes so that they are ready to sign blocks once elected.

### Environment Variables

First we are going to set up the main environment variables related to the Mainnet network. Run these on both your **Validator** and **Proxy** machines:

```bash
export CELO_IMAGE=us.gcr.io/celo-org/geth:mainnet
```

### Pull the Celo Docker image

In all the commands we are going to see the `CELO_IMAGE` variable to refer to the Docker image to use. Now we can get the Docker image on your Validator and Proxy machines:

```bash
docker pull $CELO_IMAGE
```

#### Account Creation

:::info

Please complete this section if you are new to validating on Celo.

:::

##### Account and Signer keys

Running a Celo Validator node requires the management of several different keys, each with different privileges. Keys that need to be accessed frequently (e.g. for signing blocks) are at greater risk of being compromised, and thus have more limited permissions, while keys that need to be accessed infrequently (e.g. for locking CELO) are less onerous to store securely, and thus have more expansive permissions. Below is a summary of the various keys that are used in the Celo network, and a description of their permissions.

| Name of the key      | Purpose                                                                                                                                                                                                                                            |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Account key          | This is the key with the highest level of permissions, and is thus the most sensitive. It can be used to lock and unlock CELO, and authorize vote and validator keys. Note that the account key also has all of the permissions of the other keys. |
| Validator signer key | This is the key that has permission to register and manage a Validator or Validator Group, and participate in BFT consensus.                                                                                                                       |
| Vote signer key      | This key can be used to vote in Validator elections and on-chain governance.                                                                                                                                                                       |

Note that Account and all the signer keys must be unique and may not be reused.

##### Generating Validator and Validator Group Keys

First, you'll need to generate account keys for your Validator and Validator Group.

:::warning

These keys will control your locked CELO, and thus should be handled with care.
Store and back these keys up in a secure manner, as there will be no way to recover them if lost or stolen.

:::

```bash
# On your local machine
mkdir celo-accounts-node
cd celo-accounts-node
docker run -v $PWD:/root/.celo --rm -it $CELO_IMAGE account new
docker run -v $PWD:/root/.celo --rm -it $CELO_IMAGE account new
```

This will create a new keystore in the current directory with two new accounts.
Copy the addresses from the terminal and set the following environment variables:

```bash
# On your local machine
export CELO_VALIDATOR_GROUP_ADDRESS=<YOUR-VALIDATOR-GROUP-ADDRESS>
export CELO_VALIDATOR_ADDRESS=<YOUR-VALIDATOR-ADDRESS>
```

### Start your Accounts node

Next, we'll run a node on your local machine so that we can use these accounts to lock CELO and authorize the keys needed to run your validator. To do this, we need to run the following command to run the node.

```bash
# On your local machine
mkdir celo-accounts-node
cd celo-accounts-node
docker run --name celo-accounts -it --restart always --stop-timeout 300 -p 127.0.0.1:8545:8545 -v $PWD:/root/.celo $CELO_IMAGE --verbosity 3 --syncmode full --http --http.addr 0.0.0.0 --http.api eth,net,web3,debug,admin,personal --datadir /root/.celo
```

:::danger

**Security**: The command line above includes the parameter `--http.addr 0.0.0.0` which makes the Celo Blockchain software listen for incoming RPC requests on all network adaptors. Exercise extreme caution in doing this when running outside Docker, as it means that any unlocked accounts and their funds may be accessed from other machines on the Internet. In the context of running a Docker container on your local machine, this together with the `docker -p 127.0.0.1:localport:containerport` flags allows you to make RPC calls from outside the container, i.e from your local host, but not from outside your machine. Read more about [Docker Networking](https://docs.docker.com/network/network-tutorial-standalone/#use-user-defined-bridge-networks) here.

:::

### Deploy a Validator Machine

To actually register as a validator, we'll need to generate a validating signer key. On your Validator machine (which should not be accessible from the public internet), follow very similar steps:

```bash
# On the validator machine
# Note that you have to export $CELO_IMAGE on this machine
export CELO_IMAGE=us.gcr.io/celo-org/geth:mainnet
mkdir celo-validator-node
cd celo-validator-node
docker run -v $PWD:/root/.celo --rm -it $CELO_IMAGE account new
export CELO_VALIDATOR_SIGNER_ADDRESS=<YOUR-VALIDATOR-SIGNER-ADDRESS>
```

#### Proof-of-Possession

:::info

Please complete this step if you are running a validator on Celo for the first time.

:::

In order to authorize our Validator signer, we need to create a proof that we have possession of the Validator signer private key. We do so by signing a message that consists of the Validator account address. To generate the proof-of-possession, run the following command:

```bash
# On the validator machine
# Note that you have to export CELO_VALIDATOR_ADDRESS on this machine
export CELO_VALIDATOR_ADDRESS=<CELO-VALIDATOR-ADDRESS>
docker run -v $PWD:/root/.celo --rm -it $CELO_IMAGE account proof-of-possession $CELO_VALIDATOR_SIGNER_ADDRESS $CELO_VALIDATOR_ADDRESS
```

Save the signer address, public key, and proof-of-possession signature to your local machine:

```bash
# On your local machine
export CELO_VALIDATOR_SIGNER_ADDRESS=<YOUR-VALIDATOR-SIGNER-ADDRESS>
export CELO_VALIDATOR_SIGNER_SIGNATURE=<YOUR-VALIDATOR-SIGNER-SIGNATURE>
export CELO_VALIDATOR_SIGNER_PUBLIC_KEY=<YOUR-VALIDATOR-SIGNER-PUBLIC-KEY>
```

Validators on the Celo network use BLS aggregated signatures to create blocks in addition to the Validator signer (ECDSA) key. While an independent BLS key can be specified, the simplest thing to do is to derive the BLS key from the Validator signer key. When we register our Validator, we'll need to prove possession of the BLS key as well, which can be done by running the following command:

```bash
# On the validator machine
docker run -v $PWD:/root/.celo --rm -it $CELO_IMAGE account proof-of-possession $CELO_VALIDATOR_SIGNER_ADDRESS $CELO_VALIDATOR_ADDRESS --bls
```

Save the resulting signature and public key to your local machine:

```bash
# On your local machine
export CELO_VALIDATOR_SIGNER_BLS_SIGNATURE=<YOUR-VALIDATOR-SIGNER-SIGNATURE>
export CELO_VALIDATOR_SIGNER_BLS_PUBLIC_KEY=<YOUR-VALIDATOR-SIGNER-BLS-PUBLIC-KEY>
```

We'll get back to this machine later, but for now, let's give it a proxy.

### Deploy a proxy

```bash
# On the proxy machine
mkdir celo-proxy-node
cd celo-proxy-node
export CELO_VALIDATOR_SIGNER_ADDRESS=<YOUR-VALIDATOR-SIGNER-ADDRESS>
```

You can then run the proxy with the following command. Be sure to replace `<YOUR-VALIDATOR-NAME>` with the name you'd like to appear on Celostats. The validator name shown in [Celostats](https://stats.celo.org/) will be the name configured in the proxy.

Additionally, you need to unlock the account configured in the `etherbase` option. It is recommended to create a new account and independent account only for this purpose. Be sure to write a new password to `./.password` for this account (different to the Validator Signer password)

```bash
# On the proxy machine
# First, we create a new account for the proxy
docker run --name celo-proxy-password -it --rm  -v $PWD:/root/.celo $CELO_IMAGE account new --password /root/.celo/.password
```

Notice the public address returned by this command, that can be exported and used for running the proxy node:

```bash
# On the proxy machine
export PROXY_ADDRESS=<PROXY-PUBLIC-ADDRESS>
docker run --name celo-proxy -it --restart unless-stopped --stop-timeout 300 -p 30303:30303 -p 30303:30303/udp -p 30503:30503 -p 30503:30503/udp -v $PWD:/root/.celo $CELO_IMAGE --verbosity 3 --syncmode full --proxy.proxy --proxy.proxiedvalidatoraddress $CELO_VALIDATOR_SIGNER_ADDRESS --proxy.internalendpoint :30503 --etherbase $PROXY_ADDRESS --unlock $PROXY_ADDRESS --password /root/.celo/.password --allow-insecure-unlock --light.serve 0 --datadir /root/.celo --celostats=<YOUR-VALIDATOR-NAME>@stats-server.celo.org
```

Hint: If you are running into trouble peering with the full nodes, one of the first things to check is whether your container's ports are properly configured (i.e. specifically, `-p 30303:30303 -p 30303:30303/udp` - which is set in the proxy node's command, but not the account node's command).

:::info

You can detach from the running container by pressing `ctrl+p ctrl+q`, or start it with `-d` instead of `-it` to start detached. Access the logs for a container in the background with the `docker logs` command.

:::

**NOTES**

- For the proxy to be able to send stats to [Celostats](https://stats.celo.org/), both the proxy and the validator should set the `celostats` flag
- If you are deploying multiple proxies for the same validator, the `celostats` flag should be added in only one of them

### Get your Proxy's connection info

Once the Proxy is running, we will need to retrieve its enode and IP address so that the Validator will be able to connect to it.

```bash
# On the proxy machine, retrieve the proxy enode
docker exec celo-proxy geth --exec "admin.nodeInfo['enode'].split('//')[1].split('@')[0]" attach | tr -d '"'
```

Now we need to set the Proxy enode and Proxy IP address in environment variables on the Validator machine.

If you don't have an internal IP address over which the Validator and Proxy can communicate, you can set the internal IP address to the external IP address.

If you don't know your proxy's external IP address, you can get it by running the following command:

```bash
# On the proxy machine
dig +short myip.opendns.com @resolver1.opendns.com
```

Then, export the variables on your Validator machine.

```bash
# On the Validator machine
export PROXY_ENODE=<YOUR-PROXY-ENODE>
export PROXY_EXTERNAL_IP=<PROXY-MACHINE-EXTERNAL-IP-ADDRESS>
export PROXY_INTERNAL_IP=<PROXY-MACHINE-INTERNAL-IP-ADDRESS>
```

You will also need to export `PROXY_EXTERNAL_IP` on your local machine.

```bash
# On your local machine
export PROXY_EXTERNAL_IP=<PROXY-MACHINE-EXTERNAL-IP-ADDRESS>
```

### Connect the Validator to the Proxy

When your Validator starts up it will attempt to create a network connection with the proxy machine. You will need to make sure that your proxy machine has the appropriate firewall settings to allow the Validator to connect to it.

Specifically, on the proxy machine, port 30303 should allow TCP and UDP connections from all IP addresses. And port 30503 should allow TCP connections from the IP address of your Validator machine.

Test that your network is configured correctly by running the following commands:

```bash
# On your local machine, test that your proxy is accepting TCP connections over port 30303.
# Note that it will also need to be accepting UDP connections over this port.
nc -vz $PROXY_EXTERNAL_IP 30303
```

```bash
# On your Validator machine, test that your proxy is accepting TCP connections over port 30503.
nc -vz $PROXY_INTERNAL_IP 30503
```

Once that is completed, go ahead and run the Validator. Be sure to write your Validator signer password to `./.password` for the following command to work, or provide your password another way.

```bash
# On the Validator machine
cd celo-validator-node
docker run --name celo-validator -it --restart unless-stopped --stop-timeout 300 -p 30303:30303 -p 30303:30303/udp -v $PWD:/root/.celo $CELO_IMAGE --verbosity 3 --syncmode full --mine --etherbase $CELO_VALIDATOR_SIGNER_ADDRESS --nodiscover --proxy.proxied --proxy.proxyenodeurlpairs=enode://$PROXY_ENODE@$PROXY_INTERNAL_IP:30503\;enode://$PROXY_ENODE@$PROXY_EXTERNAL_IP:30303 --unlock=$CELO_VALIDATOR_SIGNER_ADDRESS --password /root/.celo/.password --light.serve 0 --celostats=<YOUR-VALIDATOR-NAME>@stats-server.celo.org
```

At this point your Validator and Proxy machines should be configured, and both should be syncing to the network. You should see `Imported new chain segment` in your node logs, about once every 5 seconds once the node is synced to the latest block which you can find on the [Network Stats](https://stats.celo.org) page.

:::info

You can run multiple proxies by deploying additional proxies per the instructions in the [Deploy a proxy](/network/mainnet/run-validator#deploy-a-proxy) section. Then add all of the proxies' enodes as a comma seperated list using the `--proxy.proxyenodeurlpairs` option. E.g. if there are two proxies, that option's usage would look like `--proxy.proxyenodeurlpairs=enode://$PROXY_ENODE_1@$PROXY_INTERNAL_IP_1:30503\;enode://$PROXY_ENODE_1@$PROXY_EXTERNAL_IP_1:30303,enode://$PROXY_ENODE_2@$PROXY_INTERNAL_IP_2:30503\;enode://$PROXY_ENODE_2@$PROXY_EXTERNAL_IP_2:30303`

:::

### Limiting block space for transactions paid in alternative ERC-20 gas currencies

As described in the protocol documentation, Celo allows users to [pay for gas using tokens](/what-is-celo/about-celo-l1/protocol/transaction/erc20-transaction-fees). There is a governable list of accepted tokens. However, the Celo blockchain client starting with version 1.8.1 implements a protective mechanism that allows validators to control the percentage of available block space used by transactions paid with an alternative fee currency (other than CELO) more precisely.

There are two new flags that control this behavior:

1. `celo.feecurrency.limits` with a comma-separated `currency_address_hash=limit` mappings for currencies listed in the `FeeCurrencyWhitelist` contract, where `limit` represents the maximal fraction of the block gas limit as a float point number available for the given fee currency. The addresses are not expected to be checksummed.

For example, `0x765DE816845861e75A25fCA122bb6898B8B1282a=0.1,0xD8763CBa276a3738E6DE85b4b3bF5FDed6D6cA73=0.05,0xEd6961928066D3238134933ee9cDD510Ff157a6e=0`.

2. `celo.feecurrency.default` - an overridable default value (initially set to `0.5`) for currencies not listed in the limits map, meaning that if not specified otherwise, a transaction with a given fee currency can take up to `50%` of the block space. CELO token doesn’t have a limit.

Based on historical data, the following default configuration is proposed:

```bash
--celo.feecurrency.limits.default=0.5
--celo.feecurrency.limits="0x765DE816845861e75A25fCA122bb6898B8B1282a=0.9,0xD8763CBa276a3738E6DE85b4b3bF5FDed6D6cA73=0.5,0xe8537a3d056DA446677B9E9d6c5dB704EaAb4787=0.5"
```

It imposes the following limits:

- cUSD up to 90%
- cEUR up to 50%
- cREAL up to 50%
- any other token except CELO - 50%
- CELO doesn't have a limit

## Registering as a Validator

### Register the Accounts

You've now done all the infrastructure setup to get a validator and proxy running. To run a validator on Mainnet, you must lock CELO to participate in block production. Once you have CELO in you validator and validation group accounts, you can view their balances:

```bash
# On your local machine
celocli account:balance $CELO_VALIDATOR_GROUP_ADDRESS
celocli account:balance $CELO_VALIDATOR_ADDRESS
```

You can also look at an account's current balance and transaction history on [Celo Explorer](https://explorer.celo.org/). Enter the address into the search bar.

Once these accounts have a balance, unlock them so that we can sign transactions. Then, we will register the accounts with the Celo core smart contracts:

```bash
# On your local machine
celocli account:unlock $CELO_VALIDATOR_GROUP_ADDRESS
celocli account:unlock $CELO_VALIDATOR_ADDRESS
celocli account:register --from $CELO_VALIDATOR_GROUP_ADDRESS --name <NAME YOUR VALIDATOR GROUP>
celocli account:register --from $CELO_VALIDATOR_ADDRESS --name <NAME YOUR VALIDATOR>
```

Check that your accounts were registered successfully with the following commands:

```bash
# On your local machine
celocli account:show $CELO_VALIDATOR_GROUP_ADDRESS
celocli account:show $CELO_VALIDATOR_ADDRESS`
```

### Lock up CELO

Lock up CELO for both accounts in order to secure the right to register a Validator and Validator Group. The current requirement is 10,000 CELO to register a validator, and 10,000 CELO _per member validator_ to register a Validator Group. For Validators, these locked-up CELO remain locked for approximately 60 days following deregistration. For groups, these locked-up CELO remains locked for approximately 60 days following the removal of the Nth validator from the group.

```bash
# On your local machine
celocli lockedgold:lock --from $CELO_VALIDATOR_GROUP_ADDRESS --value 10000000000000000000000
celocli lockedgold:lock --from $CELO_VALIDATOR_ADDRESS --value 10000000000000000000000
```

This amount (10,000 CELO) represents the minimum amount needed to be locked in order to register a Validator and Validator group. **Note that you will want to be sure to leave enough CELO unlocked to be able to continue to pay transaction fees for future transactions (such as those issued by running some CLI commands)**.
Check that your CELO was successfully locked with the following commands:

```bash
# On your local machine
celocli lockedgold:show $CELO_VALIDATOR_GROUP_ADDRESS
celocli lockedgold:show $CELO_VALIDATOR_ADDRESS
```

### Run for election

In order to be elected as a Validator, you will first need to register your group and Validator. Note that when registering a Validator Group, you need to specify a [commission](/what-is-celo/about-celo-l1/protocol/pos/validator-groups.md#group-share), which is the fraction of epoch rewards paid to the group by its members.
We don't want to use our account key for validating, so first let's authorize the validator signing key:

```bash
# On your local machine
celocli account:authorize --from $CELO_VALIDATOR_ADDRESS --role validator --signature 0x$CELO_VALIDATOR_SIGNER_SIGNATURE --signer 0x$CELO_VALIDATOR_SIGNER_ADDRESS
```

Confirm by checking the authorized Validator signer for your Validator:

```bash
# On your local machine
celocli account:show $CELO_VALIDATOR_ADDRESS
```

Then, register your Validator Group by running the following command. Note that because we did not authorize a Validator signer for our Validator Group account, we register the Validator Group with the account key.

```bash
# On your local machine
celocli validatorgroup:register --from $CELO_VALIDATOR_GROUP_ADDRESS --commission 0.1
```

You can view information about your Validator Group by running the following command:

```bash
# On your local machine
celocli validatorgroup:show $CELO_VALIDATOR_GROUP_ADDRESS
```

Next, register your Validator by running the following command. Note that because we have authorized a Validator signer, this step could also be performed on the Validator machine. Running it on the local machine allows us to avoid needing to install the [`celocli`](/cli/) on the Validator machine.

```bash
# On your local machine
celocli validator:register --from $CELO_VALIDATOR_ADDRESS --ecdsaKey $CELO_VALIDATOR_SIGNER_PUBLIC_KEY --blsKey $CELO_VALIDATOR_SIGNER_BLS_PUBLIC_KEY --blsSignature $CELO_VALIDATOR_SIGNER_BLS_SIGNATURE
```

Affiliate your Validator with your Validator Group. Note that you will not be a member of this group until the Validator Group accepts you. This command could also be run from the Validator signer, if running on the validator machine.

```bash
# On your local machine
celocli validator:affiliate $CELO_VALIDATOR_GROUP_ADDRESS --from $CELO_VALIDATOR_ADDRESS
```

Accept the affiliation:

```bash
# On your local machine
celocli validatorgroup:member --accept $CELO_VALIDATOR_ADDRESS --from $CELO_VALIDATOR_GROUP_ADDRESS
```

Next, double check that your Validator is now a member of your Validator Group:

```bash
# On your local machine
celocli validator:show $CELO_VALIDATOR_ADDRESS
celocli validatorgroup:show $CELO_VALIDATOR_GROUP_ADDRESS
```

Use both accounts to vote for your Validator Group. Note that because we have not authorized a vote signer for either account, these transactions must be sent from the account keys.

```bash
# On your local machine
celocli election:vote --from $CELO_VALIDATOR_ADDRESS --for $CELO_VALIDATOR_GROUP_ADDRESS --value 10000000000000000000000
celocli election:vote --from $CELO_VALIDATOR_GROUP_ADDRESS --for $CELO_VALIDATOR_GROUP_ADDRESS --value 10000000000000000000000
```

Double check that your votes were cast successfully:

```bash
# On your local machine
celocli election:show $CELO_VALIDATOR_GROUP_ADDRESS --group
celocli election:show $CELO_VALIDATOR_GROUP_ADDRESS --voter
celocli election:show $CELO_VALIDATOR_ADDRESS --voter
```

Users in the Celo protocol receive epoch rewards for voting in Validator Elections only after submitting a special transaction to enable them. This must be done every time new votes are cast, and can only be made after the most recent epoch has ended. For convenience, we can use the following command, which will wait until the epoch has ended before sending a transaction:

```bash
# On your local machine
# Note that this may take some time, as the epoch needs to end before votes can be activated
celocli election:activate --from $CELO_VALIDATOR_ADDRESS --wait && celocli election:activate --from $CELO_VALIDATOR_GROUP_ADDRESS --wait
```

Check that your votes were activated by re-running the following commands:

```bash
# On your local machine
celocli election:show $CELO_VALIDATOR_GROUP_ADDRESS --voter
celocli election:show $CELO_VALIDATOR_ADDRESS --voter
```

If your Validator Group elects validators, you will receive epoch rewards in the form of additional Locked CELO voting for your Validator Group from your account addresses. You can see these rewards accumulate with the commands in the previous set, as well as:

```bash
# On your local machine
celocli lockedgold:show $CELO_VALIDATOR_GROUP_ADDRESS
celocli lockedgold:show $CELO_VALIDATOR_ADDRESS
```

You're all set! Elections are finalized at the end of each epoch, roughly once a day in the Mainnet network. After that hour, if you get elected, your node will start participating BFT consensus and validating blocks. After the first epoch in which your Validator participates in BFT, you should receive your first set of epoch rewards.
You can inspect the current state of the validator elections by running:

```bash
# On your local machine
celocli election:list
```

You can check the status of your validator, including whether it is elected and signing blocks, at [stats.celo.org](https://stats.celo.org/) or by running:

```bash
celocli validator:status --validator $CELO_VALIDATOR_ADDRESS
```

You can see additional information about your validator, including uptime score, by running:

```bash
# On your local machine
celocli validator:show $CELO_VALIDATOR_ADDRESS
```

## Deployment Tips

### Running the Docker containers in the background

There are different options for executing Docker containers in the background. The most typical one is to use in your docker run commands the `-d` option. Also for long running processes, especially when you run in on a remote machine, you can use a tool like [screen](https://ss64.com/osx/screen.html). It allows you to connect and disconnect from running processes providing an easy way to manage long running processes.

It's out of the scope of this documentation to go through the `screen` options, but you can use the following command format with your `docker` commands:

```bash
screen -S <SESSION NAME> -d -m <YOUR COMMAND>
```

For example:

```bash
screen -S celo-validator -d -m docker run --name celo-validator -it --restart unless-stopped --stop-timeout 300 -p 127.0.0.1:8545:8545 .......
```

You can list your existing `screen` sessions:

```bash
screen -ls
```

And re-attach to any of the existing sessions:

```bash
screen -r -S celo-validator
```

### Stopping containers

You can stop the Docker containers at any time without problem. If you stop your containers that means those containers stop providing service.
The data directory of the Validator and the proxy are Docker volumes mounted in the containers from the `celo-*-dir` you created at the very beginning. So if you don't remove that folder, you can stop or restart the containers without losing any data.

It is recommended to use the Docker stop timeout parameter `-t` when stopping the containers. This allows time, in this case 60 seconds, for the Celo nodes to flush recent chain data it keeps in memory into the data directories. Omitting this may cause your blockchain data to corrupt, requiring the node to start syncing from scratch.

You can stop the `celo-validator` and `celo-proxy` containers running:

```bash
docker stop celo-validator celo-proxy -t 60
```

And you can remove the containers (not the data directory) by running:

```bash
docker rm -f celo-validator celo-proxy
```
