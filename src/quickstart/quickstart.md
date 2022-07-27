# Quick-start

When running the proof of concept, there are three features that need to be run simultaneously: the blockchain [node](#hardhat-node), the [real-world service provider](#server), and the [consumer](#client).

## Hardhat node

Clone the `sep-contracts` repository:

```bash
git clone https://github.com/rndlabs/sep-contracts
```

Install the dependencies:

```bash
cd sep-contracts
yarn
```

Compile `typechain` artifacts:

```bash
yarn hardhat compile
```

Generate a `mnemonic` that you can use for test accounts. To do so, you can use [this BIP39 generator](https://iancoleman.io/bip39/). At the Mnemonic Code Converter:

* Select '24' words from the drop-down box.
* Press 'GENERATE'
* Copy the results of the `BIP39 Mnemonic` field to clipboard for the next step.

Configure the environment variables (`.env`), within the cloned repository's root directory to be similar to:

```raw title=".env"
# network specific node uri : `"ETH_NODE_URI_" + networkName.toUpperCase()`
ETH_NODE_URI_MAINNET=https://eth-mainnet.alchemyapi.io/v2/<apiKey>
ETH_NODE_URI_SOKOL=https://sokol.poa.network
ETH_NODE_URI_GNOSIS=https://rpc.xdaichain.com
# generic node uri (if no specific found) :
ETH_NODE_URI=https://{{networkName}}.infura.io/v3/<apiKey>

# network specific mnemonic : `"MNEMONIC_ " + networkName.toUpperCase()`
# MNEMONIC_MAINNET=<mnemonic for mainnet>
# generic mnemonic (if no specific found):
MNEMONIC=<paste your mnemonic here with greater than/less than symbols>

# forking
# HARDHAT_FORK=gnosis

# coinmarketcap api key for gas report
COINMARKETCAP_API_KEY=
REPORT_GAS=true

# Etherscan API key for automatic verification of contracts
ETHERSCAN_API_KEY=
```

Now start the hardhat local blockchain for testing:

```bash
yarn hardhat node
```

When starting the hardhat local node, search through the console output for the address at which the `ServiceProvider` contract was deployed. Example:

```
deploying "ServiceProvider" (tx: 0x717c1eb6649abe7b92a0a2bead9b9f3b505da385980486283e5912ac90d699a9)...: deployed at 0x29b67856f9ca63dF5E688454B17F70Afd5071aa0 with 1758370 gas
```

In the above example, the `ServiceProvider` contract was deployed to `0x29b67856f9ca63dF5E688454B17F70Afd5071aa0`. Copy the address where it deployed on **your** local hardhat node as this will be used in subsequent configuration. For now though, your local blockchain node is running and ready to accept connections!

## Server

Open another terminal window and clone the `sep` repository.

> Be sure not to clone `sep` into the `sep-contracts` directory.

```bash
git clone https://github.com/rndlabs/sep
```

Install the dependencies:

```bash
yarn                  # installs lerna for monorepo
yarn lerna bootstrap  # bootstrap packages
```

Change to the `demo` directory:

```bash
cd packages/demo
```

Generate the protobuf TypeScript:

```bash
yarn protoc --ts_out ./src/proto --proto_path ./src/proto ./src/proto/*
```

Copy over the `typechain` artifacts from the `sep-contracts` repo:

```bash
cp -pR path/to/sep-contracts/typechain .
```

Configure the environment varibles (`.env`) in the `demo` directory to be similar to:

```raw title="packages/demo/.env"
MNEMONIC=<paste your mnemonic from before here with greater than/less than symbols>
SERVER_KEY_INDEX=1
CLIENT_KEY_INDEX=2
RPC_URL=http://127.0.0.1:8545
SEP_DAPP_NAME=sep-movi
SEP_DAPP_VERSION=1
REGISTRY_ADDRESS=<fill in with deployment address from sep-contracts>
SELLER_NAME=Testing Seller
SELLER_DATA_URI=http://testurl/
SELLER_LOCATION=u173z
```

Now, you can start the `server`, and this will simulate being a service provider:

```bash
yarn ts-node ./src/server.ts
```

At this point, you should see the following activity:

1. The `server` registers a test `seller`.
2. The `server` registers 5 `space`s for the `sellerHash` from (1).
3. The `server` connects to `Waku` to monitor applicable `content-topic`s.

This activity will be visible on the [node](#hardhat-node) as well as on the `server`.

## Client

Open another terminal window and change into the `demo` package's directory:

```bash
cd path/to/demo
```

All the configuration for the `client` has already been done from the previous steps, so we can now run the `client` to make a query for a ride:

```bash
yarn ts-node ./src/client.ts
```

At this point, you should see the following activity:

1. The `client` connects to `Waku` and asks (ping) for all `sellerHash` in a certain shard.
2. The `server` responds (pong)with it's `sellerHash` and some additional details.
3. The `client` verifies the pong from the `server` against the `ServiceProvider` contract.
4. The `client`, knowing there is a legitimate `seller`, asks for a stay with specific parameters (check-in, check-out, number of adults, number of children, number of spaces).
5. The `server` receives the query from (4) and dispatches this to an `auctioneer` that takes the `ask` parameters as input. It's then determined if the `seller` wants to make a `bid` for this business.
6. The `server` crafts a *signed* bid to `client`.
7. The `client` verifies the `server`'s bid, and chooses one of the offers.
8. The `client` uses the bid's `signature` to make the deal on-chain.
9. The `server`, through event monitoring, can see that the `client` has made the deal. 🥳


>Sometimes there are connectivity issues to `Waku` from the `client`. Use `Ctrl + C` to terminate the `client` and start it again.