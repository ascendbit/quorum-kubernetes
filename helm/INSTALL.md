# Installation Steps

## Install Genesis

Install genesis helm chart first.

```bash
helm install genesis ../charts/besu-genesis --namespace besu --create-namespace --values genesis-besu.yml
```

Helm chart uses `quorum-genesis-tool` cli to generate genesis file. Above installation generates a secret `besu-genesis` under `besu` namespace, which looks like the following.

```json
{
  "nonce": "0x0",
  "timestamp": "0x58ee40ba",
  "extraData": "0xf8a4a00000000000000000000000000000000000000000000000000000000000000000f87e949fdf0be08163c2a42ae8a2da2589cfde2b23375194fffc4330c031a0185dce6fe1aea4d96c38b3b318943d6ba9de31aa747947d05224f384b7cdf9b360a09496f38a5e35787a6bd046ce0884af159169ca1da294f0e810229eec528d05a9dd010c3224856c3b167f94f984e43345c8375672ad451aaa3c67ee31d1b89ec080c0",
  "gasLimit": "0x2fefd800",
  "gasUsed": "0x0",
  "number": "0x0",
  "difficulty": "0x1",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "mixHash": "0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "config": {
    "chainId": 9988,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip150Hash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "muirglacierblock": 0,
    "berlinBlock": 0,
    "londonBlock": 0,
    "zeroBaseFee": true,
    "qbft": {
      "blockperiodseconds": 3,
      "epochlength": 30000,
      "requesttimeoutseconds": 6
    }
  },
  "alloc": {
    "0x22fcd067cff1f146e51b39ca15e7ab672e21bd50": {
      "balance": "1000000000000000000000000000"
    },
    "0xdb6a19472aa8a5e56ccb2166bda30442bbd6d75a": {
      "balance": "1000000000000000000000000000"
    },
    "0xd9a253bfdf4abd31e14084e4a72678bb8a79fba0": {
      "balance": "1000000000000000000000000000"
    },
    "0x4aeee971243391d2d9ce238c1137d31049e400ad": {
      "balance": "1000000000000000000000000000"
    },
    "0x2edc49380adf16c6ee91817f252fe01e648e1108": {
      "balance": "1000000000000000000000000000"
    },
    "0x486526a845ed7b5b909e9333d2c56244b34a1403": {
      "balance": "1000000000000000000000000000"
    }
  }
}
```

> [!NOTE]\
> After doing helm install for genesis chart, we will manually need to remove `"londonBlock": 0` and `"zeroBaseFee": true` from the genesis configuration because we are not using london hardfork. It will look like this.
>
> ```bash
> "config": {
>   "chainId": 9988,
>   ...,
>   "berlinBlock": 0,
>   "qbft": {
>     ...,
>   }
> }
> ```

### Detailed Genesis configuration explanation

<details>
<summary>General Information</summary>

* nonce: An arbitrary number used for mining the first block (set to 0 here).
* timestamp: The timestamp of the blockchain creation (around September 2016 in this case).
* extraData: Additional data included in the genesis block.
* gasLimit: The maximum gas allowed per block (defines computational complexity).
* gasUsed: Since this is the first block, no gas has been used yet.
* number: The block number, which is always 0 for the genesis block.
* difficulty: The mining difficulty, set to a low value (1) here.
* coinbase: The address that receives mining rewards (set to an empty address here).
* mixHash: A hash used in the mining process (likely empty for the genesis block).
* parentHash: The hash of the parent block, which is always empty for the genesis block.

</details>

<details>
<summary>Configuration</summary>

* chainId: A unique identifier for this blockchain (set to 9988).
* homesteadBlock: Block number where Ethereum's "Homestead" update activates (set to 0 for compatibility).
* eip150Block to muirglacierblock: Blocks where various Ethereum Improvement Proposals (EIPs) activate (all set to 0 for compatibility).
* berlinBlock and londonBlock: Blocks where more recent EIPs activate (set to 0 here).
* zeroBaseFee: Enables a fee burning mechanism for Ethereum transactions (set to true here).
* qbft: Configuration for the Byzantine Fault Tolerance (BFT) consensus mechanism, likely a custom implementation (specifies block period, epoch length, and request timeout).

</details>

<details>
<summary>Allocations</summary>

* This section defines initial balances for specific accounts. Six addresses are listed, each receiving 100,000,000,000,000,000,000 Wei (the smallest denomination of the currency).

</details>

## Install Bootnodes

After that we can install bootnodes.

```bash
helm install bootnode-1 ../charts/besu-node --namespace besu --values bootnode-1.yml
helm install bootnode-2 ../charts/besu-node --namespace besu --values bootnode-2.yml
helm install bootnode-3 ../charts/besu-node --namespace besu --values bootnode-3.yml
```

## Install Validators

Initially, we need to install validator nodes' sync mode set as `FULL` in order to start producing blocks. After finish installing validators and start validating the blocks, we can switch back to `FAST` sync mode.

Default helm chart value for sync mode is set to `FAST`. So update the value files starting from validator-1 till validator-6. Example code snippet of `values/validator-1.yml` file would be like

```bash
node:
  besu:
    envBesuOpts: ""
    metrics:
      serviceMonitorEnabled: false
    sync:
      mode: FULL
```

Then we can start install validators node.

```bash
helm install validator-1 ../charts/besu-node --namespace besu --values validator-1.yml
helm install validator-2 ../charts/besu-node --namespace besu --values validator-2.yml
helm install validator-3 ../charts/besu-node --namespace besu --values validator-3.yml
helm install validator-4 ../charts/besu-node --namespace besu --values validator-4.yml
helm install validator-5 ../charts/besu-node --namespace besu --values validator-5.yml
helm install validator-6 ../charts/besu-node --namespace besu --values validator-6.yml
```

After that wait for a while and check for the log to start producing blocks.

```bash
2024-03-26 07:00:28.006+00:00 | BftProcessorExecutor-QBFT-0 | INFO  | QbftBesuControllerBuilder | Imported #1 / 0 tx / 0 pending / 0 (0.0%) gas / (0x9bd
2024-03-26 07:00:31.006+00:00 | BftProcessorExecutor-QBFT-0 | INFO  | QbftBesuControllerBuilder | Imported #2 / 0 tx / 0 pending / 0 (0.0%) gas / (0x285
2024-03-26 07:00:34.007+00:00 | BftProcessorExecutor-QBFT-0 | INFO  | QbftBesuControllerBuilder | Produced #3 / 0 tx / 0 pending / 0 (0.0%) gas / (0xa72
```

If the blocks start to produce, we can change back to `FAST` sync mode. Change the value in the validator value file and run helm upgrade.

> [!NOTE]\
> **Upgrade node after node**

```bash
helm upgrade --install validator-1 ../charts/besu-node --namespace besu --values validator-1.yml
helm upgrade --install validator-2 ../charts/besu-node --namespace besu --values validator-2.yml
helm upgrade --install validator-3 ../charts/besu-node --namespace besu --values validator-3.yml
helm upgrade --install validator-4 ../charts/besu-node --namespace besu --values validator-4.yml
helm upgrade --install validator-5 ../charts/besu-node --namespace besu --values validator-5.yml
helm upgrade --install validator-6 ../charts/besu-node --namespace besu --values validator-6.yml
```

Check the node log to see if it is running as `FAST` sync.

```bash
2024-03-25 09:42:39.838+00:00 | main | INFO  | Besu |
####################################################################################################
#                                                                                                  #
# Besu version 24.3.0                                                                              #
#                                                                                                  #
# Configuration:                                                                                   #
# Network: Custom genesis file                                                                     #
# /etc/genesis/genesis.json                                                                        #
# Network Id: 1337                                                                                 #
# Data storage: Forest                                                                             #
# Sync mode: Fast                                                                                  #
# RPC HTTP APIs: DEBUG,ETH,ADMIN,WEB3,IBFT,NET,TRACE,EEA,PRIV,QBFT,PERM,TXPOOL                     #
# RPC HTTP port: 8545                                                                              #
# Using LAYERED transaction pool implementation                                                    #
# Using STACKED worldstate update mode                                                             #
#                                                                                                  #
```

## Install RPC Node

```bash
helm install rpc-1 ../charts/besu-node --namespace besu --values rpc-1.yml
helm install rpc-2 ../charts/besu-node --namespace besu --values rpc-2.yml
```

## Install Archive Node

```bash
helm install archive-1 ../charts/besu-node --namespace besu --values archive-1.yml
helm install archive-2 ../charts/besu-node --namespace besu --values archive-2.yml
```

## Install Blockscout

```bash
helm install blockscout blockscout/blockscout-stack -n blockscout --create-namespace -f blockscout.yml
```

## Miscellaneous

Any operations that could be done during day 2 will be listed below.

- [Add New Validator](#add-new-validator)

### Add New Validator

1. Check how many validators are there first

```bash
curl --location 'https://besu-rpc.abc-dev.network' \
--header 'Content-Type: application/json' \
--data '{
    "jsonrpc": "2.0",
    "method": "qbft_getValidatorsByBlockNumber",
    "params": [
        "latest"
    ],
    "id": 1
}'
```

2. Add new validator with helm.

```bash
helm install validator-7 ../charts/besu-node --namespace besu --values validator-7.yml
```

3. From the validator node, runn the following to vote the new node to be validator

```bash
curl --location 'http://localhost:8545' \
--header 'Content-Type: application/json' \
--data '{
    "jsonrpc": "2.0",
    "method": "qbft_proposeValidatorVote",
    "params": [
        "0xXXXXXXXXXXXX",
        true
    ],
    "id": 1
}'
```

4. Check whether the proposal is acknowledged.

```bash
curl --location 'http://localhost:8545' \
--header 'Content-Type: application/json' \
--data '{
    "jsonrpc": "2.0",
    "method": "qbft_getPendingVotes",
    "params": [],
    "id": 1
}'
```

With enough nodes proposing the node to be validator,after next block validation, the node will become validator.

## References

<https://besu.hyperledger.org/private-networks/tutorials/kubernetes/charts>
