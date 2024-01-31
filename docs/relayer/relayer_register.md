---
title: Run a Relayer Node(V2)
layout: default
parent: Relayer
nav_order: 2
---

# Run a Relayer Node(V2)

## Overview

Helix is a completely open system where anyone can register as a Relayer without any barriers, contribute liquidity to the system, and earn profits. Before becoming a Relayer, you should have basic knowledge of blockchain, especially in the area of contract interaction. The code for Helix is open-source, and Relayers can either use the default code to run a Node or optimize the client according to their needs, promptly fixing any issues encountered.

Each Relayer entity is associated with two chains (the source chain and the target chain of the bridge) and a Token.

Prerequisites:

1. Prepare two accounts with same address on the two intended chains and deposit a certain amount of Token in each.
2. Register as a Relayer on the Helix UI, which involves staking a certain amount of collateral.
3. Pull the client code to your local environment, configure bridge information, and compile and run the code.

Now, let's run a Relayer Node on the testnet using the example of (arbitrum-sepolia -> sepolia, USDC).

## Registration

Open the Helix UI and navigate to the [Relayer Dashboard](https://testnet.helixbridge.app/relayer/dashboard).

- **(1/3) Select Chain and Token**

  Choose the source chain as `arbitrum-sepolia`, the target chain as `sepolia`, and the Token as `USDC`. Click **Confirm** and the page will provide basic information about the bridge you are about to register. Here, you can find the bridge type is Opposite. Click **Next** to proceed.

  {: .note }
  For LnBridge V2, there are only two types: Default and Opposite. The difference between these two types lies in the location of the Relayer's staked collateral. In Default type, the collateral is staked on the target chain, while in Opposite type, the collateral is staked on the source chain.

- **(2/3) Deposit Collateral & Set Fees**

  - Enter the desired amount of collateral and click **Confirm**. Subsequently, a wallet confirmation prompt will appear (this transaction is executed on the target chain). If your wallet is not currently connected to the target chain, you will be prompted to switch to the target chain before proceeding with the collateral pledge.

    {: .note }
    Helix does not impose any restrictions on the amount of collateral that a Relayer can stake; it is entirely specified by the Relayer. The more collateral pledged, the better the depth available for each transfer. However, it's important to note that higher collateral amounts correspond to increased pledge costs for the Relayer.

  - Entering the baseFee and liquidityFee rate. Click **Confirm**, and a wallet confirmation prompt will appear afterward(this transaction is executed on the source chain).

    {: .note }
    The Relayer's final income per transfer is calculated as the `baseFee + liquidityFeeRate * Amount`, where Amount is the transfer amount.

- **(3/3) Approve**

  Authorize the contract to transfer Token `USDC` on the target chain. This authorization ensures that the `Relayer Client` can perform relay operations successfully on the target chain. Since there is also authorization in the second step, this step can be skipped if the authorized amount is sufficiently large.

After completing the registration, you can open the `Manage` tab to view the registered information. You will notice that the status of the relayer is `Offline` because you have not ye started the relayer client. Please note that there may be some delay in the synchronization of registration information.

<img src="https://docs.helixbridge.app/register.gif" style="width:100%;">

## Run the client

Access the client code on [GitHub](https://github.com/helix-bridge/relayer) and pull it to your local machine.

### Configuration

The configuration information for the Relayer is stored in the file `.maintain/configure.json`.

```js
{
  "indexer": "https://apollo-test.helixbridge.app/graphql",
  "relayGasLimit": 600000,
  "chains": [
      {
          "name": "sepolia",
          "rpc": "https://rpc2.sepolia.org",
          "native": "ETH",
          "chainId": 11155111
      },
      {
          "name": "arbitrum-sepolia",
          "rpc": "https://sepolia-rollup.arbitrum.io/rpc",
          "native": "ETH",
          "chainId": 300
      }
  ],
  "bridges": [
      {
          "fromChain": "arbitrum-sepolia",
          "toChain": "sepolia",
          "sourceBridgeAddress": "0xbA96d83E2A04c4E50F2D6D7eCA03D70bA2426e5f",
          "targetBridgeAddress": "0xbA96d83E2A04c4E50F2D6D7eCA03D70bA2426e5f",
          "encryptedPrivateKey": "<ENCRYPTED_PRIVATEKEY>",
          "feeLimit": 0.01,
          "reorgThreshold": 20,
          "bridgeType": "lnv2-opposite",
          "providers": [
              {
                  "fromAddress": "0x8A87497488073307E1a17e8A12475a94Afcb413f",
                  "toAddress": "0x0ac58Df0cc3542beC4cDa71B16D06C3cCc39f405",
                  "swapRate": 2000
              }
          ]
      }
   ]
}
```

- **indexer**: It's the second-layer index service introduced in this [section](https://docs.helixbridge.app/helixbridge/relayer_indexer) can be accessed through the following link.
  <table style="width:80%">
  <tr>
  <th style="width:20%">Network</th><th>URL</th>
  </tr>
  <tr>
  <td>Testnet</td><td>https://apollo-test.helixbridge.app/graphql</td>
  </tr>
  <tr>
  <td>Mainnet</td><td>https://apollo.helixbridge.app/graphql</td>
  </tr>
  </table>

- **relayGasLimit**: The gas limit for the client to send the relay transaction. If not set, it will automatically estimate a reasonable value.
- **chains**: The list of information about the chains that the relayer needs, including name, chainId, and the URL for accessing the RPC node.
- **bridges**: The list of token bridge paths that can support multiple directions simultaneously. The fields include:
  - **fromChain** and **toChain**: Must match the names defined in fileld **chains**
  - **sourceBridge** and **targetBridge**: Representing the addresses of the Helix LnBridgeV2 contract (can be queried in section [Contract Address](https://docs.helixbridge.app/helixbridge/testnet))
  - **encryptedPrivateKey**: It's the relayer's encrypted private key, corresponding to the account registered during registration

    {: .note }
    Execute the command `yarn crypto`. Follow the prompts to input your password and private key. After pressing Enter, it will print the private key encrypted with the provided password. Then replace the `<ENCRYPTED_PRIVATEKEY>` in the configuration file with the encrypted private key obtained.

  - **feeLimit**: Controls the maximum cost of a relay operation, protecting the relayer from excessive gas fees
  - **reorgThreshold**: It's an assumption about the block confirmation of transactions initiated by users on the source chain – the larger, the safer
  - **bridgeType**: Indicates the type of bridge, currently taking values of `lnv2-default`, `lnv2-opposite`, and `lnv3`, consistent with the type displayed during relayer registration
  - **providers**: List the addresses of token pairs on the source and target chains, as well as the exchange rate for the native token on the target chain.

    {: .note }
    The swapRate is the conversion rate from the native token on the target chain to the transfer token. For example, the native token on Ethereum is ETH, and the token to be transferred is USDC, the conversion rate might be approximately 2500 at 16/01/2024. As prices fluctuate, the Relayer needs to periodically adjust this ratio.

### Install & Run

After completing the configuration, you can execute the following commands one by one to compile and start the client:

```
>> yarn install
>> yarn build
>> yarn start
```

After the execution is complete, it will prompt you to enter a password. This password is the one used to encrypt the private key using `yarn crypto`.

```
>> yarn start
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [NestFactory] Starting Nest application...
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [InstanceLoader] UtilsModule dependencies initialized +49ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [InstanceLoader] DataworkerModule dependencies initialized +0ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [InstanceLoader] DiscoveryModule dependencies initialized +1ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [InstanceLoader] TasksModule dependencies initialized +0ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [InstanceLoader] ConfigHostModule dependencies initialized +1ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [InstanceLoader] AppModule dependencies initialized +0ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [InstanceLoader] ScheduleModule dependencies initialized +0ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [InstanceLoader] ConfigModule dependencies initialized +1ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [InstanceLoader] ConfigureModule dependencies initialized +1ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [InstanceLoader] RelayerModule dependencies initialized +1ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [RoutesResolver] AppController {/}: +6ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [RouterExplorer] Mapped {/, GET} route +3ms
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [dataworker] data worker started
[Nest] 46997  - 01/16/2024, 5:42:11 PM     LOG [relayer] relayer service start
Password:******
```

### Client Run Status

Return to the Relayer Dashboard page on the UI, enter the **Manage** page, and you will observe that the status of your Relayer has changed to `Online`.

## Tips

If the token bridge is the `Default` type, there will be slight differences in the registration process, and in the client's configuration, the bridgeType should be modified to lnv2-default.
