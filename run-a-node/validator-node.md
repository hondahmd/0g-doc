# Validator Node

This document outlines the steps to deploy your own validator node.

### Server Timezone Configuration

Make sure your server timezone configuration is UTC. Check your current timezone by running `timedatectl`

Note: Having a different timezone configuration may cause a `LastResultHash` mismatch error and take down your node!

### Install evmosd via CLI

```bash
git clone -b testnet <https://github.com/0glabs/0g-evmos.git>
./0g-evmos/networks/testnet/install.sh
source .profile
```

#### Set Chain ID

```bash
evmosd config chain-id zgtendermint_9000-1
```

### Initialize Node

We need to initialize the node to create all the necessary validator and node configuration files:

```bash
evmosd init <your_validator_name> --chain-id zgtendermint_9000-1
```

Note: The validator name can only contain ASCII characters.

By default, the `init` command creates config and data folder under `~/.evmosd` (i.e `$HOME`). In the config directory, the most important files for configuration are `app.toml` and `config.toml`.

> Note, you could specify `--home` to overwrite the default work directory.

### Genesis & Seeds

#### Copy the Genesis File

Check the `genesis.json` file from [this link](https://github.com/0glabs/0g-evmos/releases/download/v1.0.0-testnet/genesis.json) and copy it over to the config directory: `$HOME/.evmosd/config/genesis.json`. This is a genesis file with the chain-id and genesis accounts balances.

```bash
sudo apt install -y unzip wget
wget -P ~/.evmosd/config <https://github.com/0glabs/0g-evmos/releases/download/v1.0.0-testnet/genesis.json>
```

Then verify the correctness of the genesis configuration file:

```bash
evmosd validate-genesis
```

#### Add Seed Nodes

Your node needs to know how to find [peers](https://docs.tendermint.com/v0.34/tendermint-core/using-tendermint.html#peers). You’ll need to add healthy [seed nodes](https://docs.tendermint.com/v0.34/tendermint-core/using-tendermint.html#seed) to `$HOME/.evmosd/config/config.toml`.

The format of the `config.toml` file is as follows:

```toml
#######################################################
###           P2P Configuration Options             ###
#######################################################
[p2p]

# ...

# Comma separated list of seed nodes to connect to
seeds = "<node-id>@<ip>:<p2p port>"
```

We provide four seed nodes below.

```toml
8c01665f88896bca44e8902a30e4278bed08033f@54.241.167.190:26656,b288e8b37f4b0dbd9a03e8ce926cd9c801aacf27@54.176.175.48:26656,8e20e8e88d504e67c7a3a58c2ea31d965aa2a890@54.193.250.204:26656,e50ac888b35175bfd4f999697bdeb5b7b52bfc06@54.215.187.94:26656
```

#### Add Persistent Peers

You can set the `persistent_peers` field in `$HOME/.evmosd/config/config.toml` to specify peers that your node will maintain persistent connections with.

## Start Testnet

Start the node and sync up to the latest block height. Note that the first time you start the sync up, it may take longer time to run.

```bash
evmosd start
```

Make sure you've synced your node to the latest block height before running the following steps.

### Create Validator

You could either create a new account or import from an existing key. To create a new account:

```bash
evmosd keys add <key_name>
```

Here if you want to get the public address which starts with `0x`, you could first run the following command to get your key’s private key.

```bash
evmosd keys unsafe-export-eth-key <key_name>
```

Then import the returned private key to a wallet (Metamask for example) to get the public address.

As a next step, you must acquire some testnet tokens either by wallet transfer or requesting on the [faucet](https://faucet.0g.ai/) before submitting your validator account address.

```bash
evmosd tx staking create-validator \\
  --amount=10000evmos \\
  --pubkey=$(evmosd tendermint show-validator) \\
  --moniker="<your_validator_name>" \\
  --chain-id=zgtendermint_9000-1 \\
  --commission-rate="0.10" \\
  --commission-max-rate="0.20" \\
  --commission-max-change-rate="0.01" \\
  --min-self-delegation="1000000" \\
  --gas="5000000" \\
  --gas-prices="50000000000aevmos" \\
  --from=<key_name>
```

Check that it is in the validator set:

```bash
evmosd q staking validators -o json --limit=1000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '.tokens + " - " + .description.moniker' | sort -gr | nl
```

Note that only top 100 staked validators will be selected as active validators.

By any chance your validator is put in jail, use this command to unjail it

```bash
evmosd tx slashing unjail --from <key_name> --gas=500000 --gas-prices=99999aevmos -y
```

### Upgrading Your Node

Note these instructions are for full nodes that have ran on previous versions of and would like to upgrade to the latest testnet version.

#### Reset Data

Note: If the version you are upgrading to is not breaking from the previous one, you **should not** reset the data. If this is the case you can skip to Restart step.

First, remove the outdated files and reset the data.

```bash
rm $HOME/.evmosd/config/addrbook.json $HOME/.evmosd/config/genesis.json
evmosd tendermint unsafe-reset-all --home $HOME/.evmosd
```

Your node is now in a pristine state while keeping the original `priv_validator.json` and `config.toml`. If you had any sentry nodes or full nodes setup before, your node will still try to connect to them, but may fail if they haven’t also been upgraded.

#### Restart

```bash
evmosd start
```
