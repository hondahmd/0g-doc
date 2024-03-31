# Deployment and Integration

0G System is composed of multiple components, each with its own functionalities. Detailed steps are provided as a guideline to deploy the whole and complete system. The guideline assumes the deployment environment is Linux/Ubuntu.

### Prerequisite

0G Storage and DA services interact with on-chain contracts for blob root confirmation and PoRA mining.

For official deployed contract addresses, visit [this page](docs/contract-addresses.md).

### 1. Storage Node

First step is to deploy the storage node. As a distributed storage system, the system can have multiple instances.

1. Install dependencies

* For Linux

<pre><code><strong>$ sudo apt-get update
</strong>$ sudo apt-get install clang cmake build-essential
</code></pre>

* For Mac

```bash
$ brew install llvm cmake
```

2. Install rustup

```bash
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

3. Install Go

* For Linux

<pre class="language-bash"><code class="lang-bash"><strong># Download the Go installer
</strong>$ wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz

# Extract the archive
$ sudo rm -rf /usr/local/go &#x26;&#x26; sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

# Add /usr/local/go/bin to the PATH environment variable by adding the following line to your ~/.profile.
$ export PATH=$PATH:/usr/local/go/bin
</code></pre>

* For Mac

Download the Go installer from [https://go.dev/dl/go1.22.0.darwin-amd64.pkg](https://go.dev/dl/go1.19.3.darwin-amd64.pkg).\
Open the package file you downloaded and follow the prompts to install Go.

4. Then download the source code

```bash
$ git clone git@github.com:0glabs/0g-storage-node.git
```

5. Build the source code

```bash
$ cd 0g-storage-node
$ git submodule update --init

# Build in release mode
$ cargo build --release
```

6. Update the `run/config.toml`

```toml
# p2p port
network_libp2p_port

# rpc endpoint
rpc_listen_address

# peer nodes, modify to your own ips
network_boot_nodes

# flow contract address
log_contract_address

# mine contract address
mine_contract_address

# layer one blockchain rpc endpoint
blockchain_rpc_endpoint

# block number to start the sync
log_sync_start_block_number

# location for db, network logs
db_dir
network_dir
```

7. Run the storage service

```shell
$ cd run

# consider using tmux in order to run in background
$ ../target/release/zgs_node --config config.toml
```

### 2. Storage KV

Second step is to launch the kv service.

1. Follow the same steps to install dependencies and rust in [Stage ](deployment-and-integration.md#id-2.-storage-node)1
2. Download the source code

```bash
$ git clone git@github.com:0glabs/0g-storage-kv.git
```

3. Build the source code

<pre class="language-bash"><code class="lang-bash">$ cd 0g-storage-kv
<strong>$ git submodule update --init
</strong>
# Build in release mode
$ cargo build --release
</code></pre>

4. Copy the `config_example.toml` to `config.toml` and update the parameters

```toml
# rpc endpoint
rpc_listen_address
# ips of storage service, separated by ","
zgs_node_urls = "http://ip1:port1,http://ip2:port2,..."

# layer one blockchain rpc endpoint
blockchain_rpc_endpoint

# flow contract address
log_contract_address

# block number to start the sync, better to align with the config in storage service
log_sync_start_block_number
```

5. Run the kv service

```bash
$ cd run

# consider using tmux in order to run in background
$ ../target/release/zgs_kv --config config.toml
```

### 3. Data Availability Service

Next step is to start the 0GDA service which is the primary service to send requests to.

1. Follow the same steps to install dependencies, go and rust in [Stage 1](deployment-and-integration.md#id-1.-storage-node)
2. Download the source code

```bash
$ git clone git@github.com:0glabs/0g-data-avail.git
```

#### Disperse Service

3. Update the `Makefile` under the `0g-data-avail/disperser` folder

* For encoder

```makefile
# grpc port
--disperser-encoder.grpc-port 34000

# metric port
--disperser-encoder.metrics-http-port 9109

# number of workers, can be determined by the number of cores
--kzg.num-workers

# max concurrent request
--disperser-encoder.max-concurrent-requests

# size of request pool, can be larger than the number of cores
--disperser-encoder.request-pool-size
```

* For batcher

```makefile
# layer one blockchain rpc endpoint
--chain.rpc

# private key of wallet account, can also set as environment variable
--chain.private-key

# modify the gas limit for different chains
--chain.gas-limit

# batch size limit, can be a relative large number like 1000
--batcher.batch-size-limit

# number of segments to upload in single rpc request
--batcher.storage.upload-task-size

# interval for disperse finality
--batcher.finalizer-interval

# aws configs, can be set into environment variables as well
--batcher.aws.region
--batcher.aws.access-key-id
--batcher.aws.secret-access-key
--batcher.s3-bucket-name
--batcher.dynamodb-table-name

# endpoints of storage services, for multiple endpoints, separate them one by one
--batcher.storage.node-url
--batcher.storage.node-url

# endpoint of kv service
--batcher.storage.kv-url

# flow contract address
--batcher.storage.flow-contract

# timeout for encoding, set based on the instance capacitgy
--encoding-timeout 10s
```

* For disperser

```makefile
# port to listen on the requests
--disperser-server.grpc-port

# aws configs, can be set into environment variables as well
# note the keys are different from which in batcher
--disperser-server.aws.region
--disperser-server.aws.access-key-id
--disperser-server.aws.secret-access-key
--disperser-server.s3-bucket-name
--disperser-server.dynamodb-table-name
```

4. Build the source code

```bash
$ cd 0g-data-avail/disperser
$ make build
```

5. Run encoder, batcher and disperser

```bash
# encoder
$ make run_encoder

# batcher
$ make run_batcher

# disperser
$ make run_server
```

#### Retrieve Service

6. Update the `Makefile` under `0g-data-avail/retriever` folder

```makefile
# grpc port to listen on requests
--retriever.grpc-port

# endpoints of storage services
--retriever.storage.node-url
--retriever.storage.node-url

# endpoint of kv service
--retriever.storage.kv-url

# flow contract addres
--retriever.storage.flow-contract
```

7. Build the source code

```bash
$ cd 0g-data-avail/retriever
$ make build
```

8. Run retriever

```bash
$ make run
```

### 4. Benchmark

To conduct integration test

1. Install extra dependency

```bash
$ sudo apt-get install protobuf-compiler
```

2. Download the source code

```bash
$ git clone git@github.com:0glabs/0g-da-example-rust.git
```

3. Build the source code

```bash
$ cargo build
```

4. Run the test

```bash
$ cargo run -- zgda-disperse --rps <int> --max-out-standing <int> --url <endpoint> --block-size <int> --chunk-size <int> --target-chunk-num <int>
```

Note,

* `rps` and `max-out-standing` are set to control the speed of the requests
* `url` is the endpoint of the disperse service in [Stage 3](deployment-and-integration.md#disperse-service)
* `block-size` is the size of the total data in bytes
* `chunk-size` is the same as the blob size in bytes of each request sent to the disperse service
* `target-chunk-num` is the number of the chunks to define in 0GDA service. It's used to divide the blob into corresponding number of pieces. It's hard bounded by the blob size.

### You are all set !
