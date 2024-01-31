---
title: xToken Protocol
layout: default
parent: Protocol
nav_order: 3
---

# xToken Protocol

xToken plays an essential role in Helix's business by hosting user assets and facilitating cross-chain asset transfers. It operates based on an asset registration and cross-chain protocol rooted from the CBA (Cryptocurrency Backed Asset) model.

## Terms

- **Source Chain And Target Chain**  
  Refers to the source blockchain network and the target blockchain network for cross-chain asset transfers via a bridge, respectively. Alternatively, it can denote the caller and the callee of a remote chain call. Generally, a light client of the source chain needs to be established on the target chain to perform cross-chain verification of messages or events from the source chain.

- **Original Token and Mapping Token**  
  This is a pair of relative terms. The original token typically refers to assets that have not been bridged to the target chain, such as BTC on the Bitcoin network, ETH, or USDT on the Ethereum network. On the other hand, the original token is located on the source chain of the bridge, while the mapping token is situated on the target chain of the bridge. The mapping token is a type of asset created with the backing of the original token.

- **Relayer**  
  Relayers are a group of competing and supervising entities responsible for maintaining and executing the bridge's information relaying tasks. While relayers may not significantly impact the safety of a bridge, they play a direct role in ensuring the liveness and effectiveness of the bridge.

## CBA Model

The CBA Model involves deploying the Backing module on the source chain and the Issuing module on the target chain. The asset registration and issuance process is completed through underlying calls to the generic cross-chain messaging channel.

<img src="https://docs.helixbridge.app/cba01.png" style="width:70%; height:70%; text-align:middle; margin-left:15%; margin-right:15%">

- **Backing**  
  Deployed on the source chain, the Backing module locks user tokens. Backing generates locking proofs, and the tokens are held in Backing as collateral for asset issuance mapping on the target chain. These tokens remain locked until a user initiates a reverse redemption operation, at which point they are unlocked and returned to the user's account.

- **Issuing**  
  Deployed on the target chain, the Issuing module is backed by the original token in the locking model. Issuing issues mapping tokens to the user's account. When a user initiates a redemption operation, Issuing burns the mapping token and generates a proof of destruction for the original token redemption.

## Protocol

<img src="https://docs.helixbridge.app/mapping_token.svg" style="width:70%; height:70%; text-align:middle; margin-left:15%; margin-right:15%">

- **Asset registration**  
  Asset registration is the process of registering the original token with the Backing module on the source chain and mapping it to the corresponding mapping token on the target chain. This involves calling the Backing module to provide meta information about the original token, and the message relayer relays this information to the Issuing module on the target chain to create the corresponding mapping token.

- **Tokens Locking and Issuance**  
  After completing asset registration, users can lock the original tokens through the Backing module. Once the Message Relayer synchronizes the locking message to the target chain, the Issuing module mints the same amount of mapping tokens to the user's specified account.

- **Operations After Issuance**  
  On the target chain, mapping tokens adhere to the same token standard as original tokens, allowing users to execute various types of operations such as swapping and transferring.

- **Tokens Burning and Redemption**  
  Users holding mapping tokens on the target chain can burn them by calling the Issuing module. The Message Relayer delivers the burning message to the source chain, and then the Backing module unlocks the original token, transferring it to the user's specified account.

## Transaction Atomicity

- **Protocol V1**  
  In this version, any cross-chain transfer message will receive a reply from the target chain. If the mapping token is successfully issued on the target chain, the original token will be locked on the source chain. If the issue fails, the source chain will return the original token to the users. Therefore, the transaction integrity in this version relies heavily on the transactions of the cross-chain messaging service. If the cross-chain messaging service lacks a response process, the transaction atomicity of the asset bridge cannot be ensured.

- **Protocol V2**  
  In this version, we have updated the transaction processing flow. If the issuance of mapped tokens on the target chain succeeds, the transaction concludes. However, if the issuance fails, the user can obtain a proof of the failure to redeem the original token and complete the transaction. Consequently, we no longer depend on the response result from the cross-chain messaging service and only require the failure proof in case of message delivery failure. This proof typically comprises two parts: a proof of message delivery and a proof of the absence of message success. Compared to the V1, this version can adapt to more cross-chain messaging services and reduce costs. However, it requires an additional refund operation in case of failure.
