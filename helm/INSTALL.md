# Installation Steps

## Install Genesis and Bootnode

Install genesis helm chart and bootnode chart first.

```bash
helm install genesis ../charts/besu-genesis --namespace besu --create-namespace --values genesis-besu.yml
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

## Add Validators

Check how many validators are there first
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

Add new validator with helm.

```bash
helm install validator-6 ../charts/besu-node --namespace besu --values validator-6.yml
```

From the validator node, runn the following to vote the new node to be validator

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

Check whether the proposal is acknowledged.

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

### Reference
https://besu.hyperledger.org/private-networks/tutorials/kubernetes/charts