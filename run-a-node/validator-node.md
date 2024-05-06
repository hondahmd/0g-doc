# Validator Node

This document outlines the steps to deploy your own validator node.

### Hardware Requirement

```
- Memory: 8 GB RAM
- CPU: 4 cores
- Disk: 500 GB NVME SSD
- Bandwidth: 100 MBps for Download / Upload
```

### Server Timezone Configuration

Make sure your server timezone configuration is UTC. Check your current timezone by running `timedatectl`

Note: Having a different timezone configuration may cause a `LastResultHash` mismatch error and take down your node!

### Install 0gchaind via CLI

```bash
git clone -b v0.1.0 <https://github.com/0glabs/0g-chain.git>
./0g-chain/networks/testnet/install.sh
source .profile
```

#### Set Chain ID

<pre class="language-bash"><code class="lang-bash"><strong>0gchaind config chain-id zgtendermint_16600-1
</strong></code></pre>

### Initialize Node

We need to initialize the node to create all the necessary validator and node configuration files:

```bash
0gchaind init <your_validator_name> --chain-id zgtendermint_16600-1
```

Note: The validator name can only contain ASCII characters.

By default, the `init` command creates config and data folder under `~/.0gchaind`(i.e `$HOME`). In the config directory, the most important files for configuration are `app.toml` and `config.toml`.

> Note, you could specify `--home` to overwrite the default work directory.

### Genesis & Seeds

#### Copy the Genesis File

Check the `genesis.json` file from [this link](https://github.com/0glabs/0g-chain/releases/download/v0.1.0/genesis.json) and copy it over to the config directory: `$HOME/.0gchaind/config/genesis.json`. This is a genesis file with the chain-id and genesis accounts balances.

```bash
sudo apt install -y unzip wget
wget -P ~/.0gchaind/config https://github.com/0glabs/0g-chain/releases/download/v0.1.0/genesis.json
```

Then verify the correctness of the genesis configuration file:

```bash
0gchaind validate-genesis
```

#### Add Seed Nodes

Your node needs to know how to find [peers](https://docs.tendermint.com/v0.34/tendermint-core/using-tendermint.html#peers). You’ll need to add healthy [seed nodes](https://docs.tendermint.com/v0.34/tendermint-core/using-tendermint.html#seed) to `$HOME/.0gchaind/config/config.toml`.

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
80afd077ba051de6a83c57cb85d78e8f3aa28e3a@54.241.167.190:26656,fd72f8832cfc78fa12e26f538c0f45c47505bf0a@54.176.175.48:26656,6bda685fe5c4c1b32b9fa73f622aaa7da71bbb81@54.193.250.204:26656,1c8d8f92870ab7c6b1ee31c44e762a5939e6637d@54.215.187.94:26656
```

#### Add Persistent Peers

You can set the `persistent_peers` field in `$HOME/.0gchaind/config/config.toml` to specify peers that your node will maintain persistent connections with.

## Start Testnet

Start the node and sync up to the latest block height. Note that the first time you start the sync up, it may take longer time to run.

```bash
0gchaind start
```

Make sure you've synced your node to the latest block height before running the following steps.

### Create Validator

You could either create a new account or import from an existing key. To create a new account:

```bash
0gchaind keys add <key_name>
```

Here if you want to get the public address which starts with `0x`, you could first run the following command to get your key’s private key.

```bash
0gchaind keys unsafe-export-eth-key <key_name>
```

Then import the returned private key to a wallet (Metamask for example) to get the public address.

As a next step, you must acquire some testnet tokens either by wallet transfer or requesting on the [faucet](https://faucet.0g.ai/) before submitting your validator account address.

```bash
0gchaind tx staking create-validator \\
  --amount=10000000000ua0gi \\
  --pubkey=$(0gchaind tendermint show-validator) \\
  --moniker="<your_validator_name>" \\
  --chain-id=zgtendermint_16600-1 \\
  --commission-rate="0.10" \\
  --commission-max-rate="0.20" \\
  --commission-max-change-rate="0.01" \\
  --min-self-delegation="1000000" \\
  --gas="5000000" \\
  --gas-prices="50000000000neuron" \\
  --from=<key_name>
```

Check that it is in the validator set:

```bash
0gchaind q staking validators -o json --limit=1000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '.tokens + " - " + .description.moniker' | sort -gr | nl
```

Note that only top 120 staked validators will be selected as active validators.

By any chance your validator is put in jail, use this command to unjail it

```bash
0gchaind tx slashing unjail --from <key_name> --gas=500000 --gas-prices=99999neuron -y
```

### Upgrading Your Node

Note these instructions are for full nodes that have ran on previous versions of and would like to upgrade to the latest testnet version.

#### Reset Data

Note: If the version you are upgrading to is not breaking from the previous one, you **should not** reset the data. If this is the case you can skip to Restart step.

First, remove the outdated files and reset the data.

```bash
rm $HOME/.0gchaind/config/addrbook.json $HOME/.0gchaind/config/genesis.json
0gchaind tendermint unsafe-reset-all --home $HOME/.0gchaind
```

Your node is now in a pristine state while keeping the original `priv_validator.json` and `config.toml`. If you had any sentry nodes or full nodes setup before, your node will still try to connect to them, but may fail if they haven’t also been upgraded.

#### Restart

```bash
0gchaind start
```

