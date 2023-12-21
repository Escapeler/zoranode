# How to Run Zoranode


### Software requirements

- [Docker](https://docs.docker.com/desktop/)
- [Python 3](https://www.python.org/downloads/)

### Hardware requirements

We recommend you have this configuration to run a node:

- at least 16 GB RAM
- an SSD drive with at least 200 GB free

### Install Node

1. Select the network you want to run and set `CONDUIT_NETWORK` env variable. You will need to know the `slug` of the network. You can find this in the Conduit console. For public networks you can use the table above. Example:

```
# for Zora Mainnet
export CONDUIT_NETWORK=zora-mainnet-0
```

Note: The external nodes feature must be enabled on the network for this to work. For the public networks above this is already set.

2. Download the required network configuration with:

```
./download-config.py $CONDUIT_NETWORK
```

3. Ensure you have an Ethereum L1 full node RPC available (not Conduit), and copy `.env.example` to `.env` setting `OP_NODE_L1_ETH_RPC`. If running your own L1 node, it needs to be synced before the specific Conduit network will be able to fully sync. Example:

```
# .env file
# [recommended] replace with your preferred L1 (Ethereum, not Conduit) node RPC URL:
OP_NODE_L1_ETH_RPC=https://mainnet.gateway.tenderly.co/<your-tenderly-api-key>
```

4. Start the node!

```
docker compose up --build
```

5. You should now be able to `curl` your Conduit node:

```
curl -d '{"id":0,"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false]}' \
  -H "Content-Type: application/json" http://localhost:8545
```

Note: Some L1 nodes (e.g. Erigon) do not support fetching storage proofs. You can work around this by specifying `--l1.trustrpc` when starting op-node (add it in `op-node-entrypoint` and rebuild the docker image with `docker compose build`.) Do not do this unless you fully trust the L1 node provider.

You can map a local data directory for `op-geth` by adding a volume mapping to the `docker-compose.yaml`:

```yaml
services:
  geth: # this is Optimism's geth client
    ...
    volumes:
      - ./geth-data:/data
```
### Syncing Node

Sync speed depends on your L1 node, as the majority of the chain is derived from data submitted to the L1. You can check your syncing status using the `optimism_syncStatus` RPC on the `op-node` container. Example:

```
command -v jq  &> /dev/null || { echo "jq is not installed" 1>&2 ; }
echo Latest synced block behind by: \
$((($( date +%s )-\
$( curl -s -d '{"id":0,"jsonrpc":"2.0","method":"optimism_syncStatus"}' -H "Content-Type: application/json" http://localhost:7545 |
   jq -r .result.unsafe_l2.timestamp))/60)) minutes
```

### Network Stats

You can see how many nodes you are connected with the following command:

```
curl -d '{"id":0,"jsonrpc":"2.0","method":"opp2p_peerStats","params":[]}' \
  -H "Content-Type: application/json" http://localhost:7545
```
