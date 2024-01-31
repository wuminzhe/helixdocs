---
title: Relayer Indexer
layout: default
parent: Relayer
nav_order: 1
---

# Relayer Indexer

Before we can understand and try to run a relay, we should first understand the index of the helix. Because relayer relies heavily on and trusts the indexed data, the index is very important to the proper functioning and security of relayer.

## Structure

Helix indexing consists of a two-tier index structure, the first tier indexes the data directly on the chain, often using mature solutions such as thegraph. the second tier aggregates and correlates the data from the first tier.

```
+------------------------------+                +------------------------------+
|      blockchain(contract)    |                |     blockchain(contract)     |
+----------------+-------------+                +----------------+-------------+
                  |                                               |
                  |event                                          | event
                  |                                               |
+----------------v--------------+                  +-------------v-------------+
|         indexer(level1)       |                  |        indexer(level1)    |
|          thegraph             |                  |          thegraph         |
+----------------+--------------+                  +--------------+------------+
                  |                                                |
                  |                                                |
                +-v------------------------------------------------v--+
                |                   indexer(level2)                   |
                +--^----------------------^-----------------------^---+
                   |                      |                       |
+--------------+   |       +------------+ |         +-----------+ |
|   relayer    +---+       |   relayer  +-+         |  relayer  +-+
+--------------+           +------------+           +-----------+
```

Each indexer on the first layer is responsible for indexing and storing data from a single chain. The second layer's service is unique; it connects to the first layer's services, requests data from them sequentially, and associates data from different chains while performing global sorting. The Relayer only requests data from the second-layer indexer and trusts the data from that indexer.

## Deploy

Relayers have the option to deploy their own indexer services. All Helix-related services are already open source on [GitHub](https://github.com/helix-bridge/indexer). Deploying the indexer service is also a two-step process, first deploying the first-layer indexing service, and then deploying the second layer.

- For the first-layer service, navigate to the "subgraph" directory. Relayers can choose either the "default" or "opposite" subgraph directory based on their specific needs. Taking "arbitrum -> linea" as an example, before deploying, the relayer should first start the "graph-node" service. For detailed instructions, refer to the "graph-node" [documentation](https://thegraph.com/docs/en/deploying/hosted-service/).

```shell
// deploy subgraph to graph-node
>> cd subgraph/ln-default-bridge
>> yarn build-arbitrum
>> npx graph create --node http://graph-node:8020/ lndefault/arbitrum
>> npx graph deploy --node http://graph-node:8020/ --ipfs http://ipfs:5001 lndefault/arbitrum
>> yarn build-linea
>> npx graph create --node http://graph-node:8020/ lndefault/linea
>> npx graph deploy --node http://graph-node:8020/ --ipfs http://ipfs:5001 lndefault/linea
```

- For the second-layer service, go into the "apollo" directory, and modify the .env.test file. Set the URL for "ENDPOINT" to your own deployed subgraph URL. Before starting the Apollo service, you should first launch a PostgreSQL service. You can follow the example below to start it using Docker Compose:

```yaml
version: '3'

services:
  postgres:
    container_name: subql-postgres
    ports:
      - 5432:5432
    image: postgres:12-alpine
    restart: always
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: 'admin'
      POSTGRES_PASSWORD: 'admin'
      POSTGRES_DB: 'apollo_graph'
```

Make sure to customize the environment variables (POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD) to suit your needs, and then use docker-compose up to start the PostgreSQL service before launching the Apollo service.
Once you've completed the setup and configuration, you can proceed to compile and start the Apollo service. The default port for the service is 4002. Here are the general steps:

1. Make sure you are in the "apollo" directory where you have your configuration files, including the modified .env.test file.

2. Compile the Apollo service if required. Typically, this can be done by running the necessary build or compile commands.

3. After compiling (if needed), you can start the Apollo service by running a command like `yarn start:test` from the "apollo" directory. you need to initialize the database for the first-time startup using the command `yarn init:db`.

4. The Apollo service should now be running on port 4002 as the default. You can access it by navigating to http://localhost:4002 in your web browser or by making API requests to that endpoint.

```bash
// deploy indexer
>> cd apollo
// update ENDPOINT url in the file .env.test
// start a local postgres service (user:admin, password: admin, db: apollo_graph, port: 5432)
>> yarn generate:prisma
>> yarn build:test
>> yarn init:db
>> yarn start:test
```

## Helix Indexer Service

Helix provides a public indexing service at the address https://apollo.helix.app. Relayers have the option to directly connect to this service. However, it's important to note that Helix does not guarantee the security of indexed data. Relayers should be aware that they are responsible for any data-related risks when using this service.
