# DAOstack Subgraph

DAOstack subgraph for [TheGraph](https://thegraph.com/) project. A feature article is available [here](https://thegraph.com/blog/daostack-alchemy). 

![image](https://github.com/pat-daostack/subgraph/blob/master/images/arcgraph.jpg)

Our latest gratest [master branch subgraph](https://thegraph.com/explorer/subgraph/daostack/master).

## Getting started

1. `git clone https://github.com/daostack/subgraph.git && cd subgraph`
2. `npm install`

# snglsDAO patch

In order to ensure normal indexing of custom schemes after installing the modules, you need to add several abi files and correct several addresses.

Abi files can be taken in the snglsDAO/dao-contracts/build folder, they need to be placed:

- node_modules/@daostack/arc/build/contracts/LockingSGT4Reputation.json
- node_modules/@daostack/infra/build/contracts/LockingSGT4Reputation.json
- node_modules/@daostack/migration/contracts/0.0.1-rc.32/ContributionReward.json

#### Set current contract addresses here:
##### node_modules/@daostack/migration/migration.json
##### (mainnet/rinkeby)/base/0.0.1-rc.32 in json structure

For LockingSGT4Reputation and ContributionReward contracts
```json
      "0.0.1-rc.32": {
        "GEN": "0x543Ff227F64Aa17eA132Bf9886cAb5DB55DCAddf",
        "DAORegistry": "0x41C24232452057b9c812A94f84b8643Ee0253C44",
        "DAOTracker": "0x32CA3feB2F10bAb5855fCb94293Cb14C3Dfa483E",
        "ControllerCreator": "0xE40BFcE7E9AEe0b8bA7Db8cD9817983e4b243321",
        "DaoCreator": "0x2b52902BFDf0F74a70024c9Ea41c3188f693263B",
        "UController": "0x82e11DC4b5085DFF39AC690aAA4d4F7cDE80002D",
        "GenesisProtocol": "0x917a2C4421fdAD00632d89b3E550230A3a0B0A31",
        "SchemeRegistrar": "0xeEBD13d7dd6496FFE5C6909984e4152fbfcf22DD",
        "UpgradeScheme": "0x4B93FCbC854033afa6F2b8a4763df522298db3B7",
        "GlobalConstraintRegistrar": "0x996b7662aDB2B019e4C25166BE6f3EC68053f7A4",
        "ContributionReward": "0xC0d6d120648B940a14B995690eEf4225A4C290FD",
        "AbsoluteVote": "0x8558a45977E3EEF3b5d6F0143eA83d8287eBd729",
        "QuorumVote": "0xa899aa3Ae426FA960049466853E7ec498013BA4c",
        "TokenCapGC": "0xF85022de61824Da950aDBBAbdDcbd9048dd6e1F2",
        "VoteInOrganizationScheme": "0xe8a4c5702d5289D7eD8A3fD16cD237900cAb2660",
        "OrganizationRegister": "0x9a3B71641c9d22fbA8B92aAC1e80A41f9AA73F98",
        "Redeemer": "0x3a1b52bEe6baB7b5D788Aa0eCcB3941a9D780806",
        "UGenericScheme": "0x17825F606B206B97099F25EED705EDa3550cdeb3",
        "LockingSGT4Reputation": "0x84E5A718f13960822A4422C64b91FbA6Cf29D07f"
      },
```

Also check contract addresses here:
- daos/rinkeby/sngls.json
- daos/mainnet/snglsDao.jsons



## Testing

Run the tests in the host container:

```sh
npm run docker:run
npm run test
npm run docker:stop
```

The tests are run with jest, which takes a number of options that may be useful when developing:

```sh
npm run test -- --watch # re-run the tests after each change
npm run test -- test/integration/Avatar.spec.js # run a single test file
```

## Commands

1. `migrate` - migrate contracts to ganache and write result to `migration.json`.
2. `codegen` - (requires `migration.json`) automatically generate abi, subgraph, schema and type definitions for
   required contracts.
3. `test` - run integration test.
4. `deploy` - deploy subgraph.
5. `deploy:watch` - redeploy on file change.

Docker commands (requires installing [`docker`](https://docs.docker.com/v17.12/install/) and
[`docker-compose`](https://docs.docker.com/compose/install/)):

1. `docker <command>` - start a command running inside the docker container. Example: `npm run docker test` (run
   intergation tests).
2. `docker:stop` - stop all running docker services.
3. `docker:rebuild <command>` - rebuild the docker container after changes to `package.json`.
4. `docker:logs <subgraph|graph-node|ganache|ipfs|postgres>` - display logs from a running docker service.
5. `docker:run` - run all services in detached mode (i.e. in the background).

## Exposed endpoints

After running a command with docker-compose, the following endpoints will be exposed on your local machine:

- `http://localhost:8000/subgraphs/name/daostack` - GraphiQL graphical user interface.
- `http://localhost:8000/subgraphs/name/daostack/graphql` - GraphQL api endpoint.
- `http://localhost:8001/subgraphs/name/daostack` - graph-node's websockets endpoint
- `http://localhost:8020` - graph-node's RPC endpoint
- `http://localhost:5001` - ipfs endpoint.
- (if using development) `http://localhost:8545` - ganache RPC endpoint.
- `http://localhost:5432` - postgresql connection endpoint.

## Add a new contract tracker

In order to add support for a new contract follow these steps:

1. Create a new directory `src/mappings/<contract name>/`
2. Create 4 files:

   1. `src/mappings/<contract name>/mapping.ts` - mapping code.
   2. `src/mappings/<contract name>/schema.graphql` - GraphQL schema for that contract.
   3. `src/mappings/<contract name>/datasource.yaml` - a yaml fragment with:
      1. `abis` - optional - list of contract names that are required by the mapping.
      2. [`entities`](https://github.com/graphprotocol/graph-node/blob/master/docs/subgraph-manifest.md#1521-ethereum-events-mapping) -
         list of entities that are written by the the mapping.
      3. [`eventHandlers`](https://github.com/graphprotocol/graph-node/blob/master/docs/subgraph-manifest.md#1522-eventhandler) -
         map of solidity event signatures to event handlers in mapping code.
   4. `test/integration/<contract name>.spec.ts`

3. Add your contract to `ops/mappings.json`. Under the JSON object for the network your contract is located at, under the `"mappings"` JSON array, add the following.

   1. If your contract information is in the `migration.json` file specified (default is the file under `@daostack/migration` folder, as defined in the `ops/settings.js` file):

      ```json
      {
         "name": "<contract name as appears in `abis/arcVersion` folder>",
         "contractName": "<contract name as appears in migration.json file>",
         "dao": "<section label where contract is defined in migration.json file (base/ dao/ test/ organs)>",
         "mapping": "<contract name from step 2>",
         "arcVersion": "<contract arc version>"
      },
      ```

   2. If your contract does not appear in the migration file:

      ```json
      {
         "name": "<contract name as appears in `abis/arcVersion` folder>",
         "dao": "address",
         "mapping": "<contract name from step 2>",
         "arcVersion": "<contract arc version under which the abi is located in the `abis` folder>",
         "address": "<the contract address>"
      },
      ```

4. (Optionally) add a deployment step for your contract in `ops/migrate.js` that will run before testing.

## Add a new dao tracker

To index a DAO please follow the instructions here: [https://github.com/daostack/subgraph/blob/master/documentations/Deployment.md#indexing-a-new-dao](https://github.com/daostack/subgraph/blob/master/documentations/Deployment.md#indexing-a-new-dao)

## Add a new datasource template

Datasource templates allow you to index blockchain data from addresses the subgraph finds out about at runtime. This is used to dynamically index newly deployed DAOs. To add a new contract ABI that can be used as a template within your mappings, modify the `ops/templates.json` file like so:

```json
{
   "templates": [
      ...,
      {
         "name": "<contract name as appears in `abis/arcVersion` folder>",
         "mapping": "<name of the `src/mappings/...` folder to be used with this contract>",
         "start_arcVersion": "<contract arc version under which the abi is located in the `abis` folder>",
         "end_arcVersion": "(optional) <contract arc version under which the abi is located in the `abis` folder> if not given, all future versions of this `name`'s contract ABI will be added as a template for this mapping"
      }
   ]
}
```

## Deploy Subgraph

To deploy the subgraph, please follow the instructions below:

1. If you are deploying to The Graph for the first time, start with installing the Graph CLI:
`npm install -g @graphprotocol/graph-cli`
Then follow this by logging into your Graph Explorer account using:
`graph auth https://api.thegraph.com/deploy/ <ACCESS_TOKEN>`

   It is also recommended to read this guide: [https://thegraph.com/docs/deploy-a-subgraph](https://thegraph.com/docs/deploy-a-subgraph)

2. Create a `.env` file containing the following:

   ```bash
   network="<TARGET_NETWORK>"
   subgraph="<YOUR_SUBGAPH_NAME>"

   # Not necessary for Docker deployment
   graph_node="https://api.thegraph.com/deploy/"
   ipfs_node="https://api.thegraph.com/ipfs/"
   access_token=<YOUR_ACCESS_TOKEN>

   # Not necessary for The Graph server
   postgres_password=<YOUR_PASSWORD>
   ethereum_node="https://<TARGET_NETWORK>.infura.io/<INFURA-KEY>"
   start_block=<START INDEX BLOCK> (default is 0)
   ```

3. Run: ``npm run deploy``

## Release subgraph images on docker hub

The repository provides a `release.sh` script that will:

- (re)start the docker containers and deploy the subgraph
- commit the images for ipfs and postgres and push these to docker hub

The docker images are available as:

`daostack/subgraph-postgres:${network}-${migration-version}-${subgraph-version}`
`daostack/subgraph-ipfs:${network}-${migration-version}-${subgraph-version}`

## Blacklist a malicious DAO
Add the DAO's Avatar address to the `ops/blacklist.json` file in the proper network array. For example, blacklisting `0xF7074b67B4B7830694a6f58Df06375F00365d2c2` on mainnet would look like:
```json
{
  "private": [],
  "kovan": [],
  "rinkeby": [],
  "mainnet": [
     "0xF7074b67B4B7830694a6f58Df06375F00365d2c2"
  ]
}
```
