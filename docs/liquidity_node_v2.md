---
title: Liquidity Node Protocol V2
layout: default
nav_order: 1
---


# Liquidity Node Protocol V2


{: .note }
**This protocol can only be used when tokens are natively issued on both the source chain and the target chains, and their exchange values are equal.** 

## LnProvider

This protocol establishes a liquidity provider role called the LnProvider (Liquidity Node Provider). When a user transfers tokens between two chains, the LnProvider receives tokens on one chain and transfers an equivalent amount of tokens to the user on the other chain.   

During the cross-chain transfer process, the asset token goes through the following flow 👇: <img width="80%" alt="" src="https://github.com/helix-bridge/helix-docs/assets/1608576/85b70d43-70e0-4b43-83c2-2183c1551ae6">

1. **Source Chain: User → LnProvider**   
   On the source chain, the user sends tokens to the LnProvider’s source chain account.     

2. **Target Chain: LnProvider → User**        
   On the target chain, the LnProvider sends an equivalent amount of tokens to the user.

The user pays a fee for using the cross-chain transfer service, and the LnProvider earns a fee for providing liquidity. So, a LnProvider is a market-maker of Helix Token Protocol.

**When LnProvider acts according to the protocol, there is no need for the underlying cross-chain messaging channel, which simplifies the process of asset cross-chain transfer and reduce the cost.**

## LnProvider’s Collateral

According to the protocol, when a LnProvider registers, it must stake collateral to ensure the safety of user assets during the cross-chain process.

1. When a user initiates a cross-chain transfer, the user's assets are transferred to the LnProvider's account. At the same time, an equivalent amount of the LnProvider's collateral is locked. This ensures that the user's assets are protected.
2. When the cross-chain transfer is successful, the user receives the asset on the target chain, and the LnProvider's locked collateral is released on the source chain.

In normal circumstances, these two processes do not need to work with the messaging channel, but bridge messages are still used to track the execution process.

{: .note }
> The locking of collateral occurs on the source chain, while the user’s assets receiving happens on the target chain. Both the two processes are auditable and have proofs on their respective chains.

<img width="70%" alt="" src="https://github.com/helix-bridge/helix-docs/assets/1608576/9b53c6a4-8628-4b53-9cd8-2fc35749c46b">

### Pledged Collateral

Pledged Collateral is the LnProvider's fixed total collateral and does not fluctuate when user do cross-chain transfers.

The only way the pledged collateral changes is:

- when the slasher handles exceptions, or,
- when the LnProvider deposits or withdraws collateral from it. This ensures that the collateral is always sufficient to cover any potential losses or penalties.

### Available Collateral

The available collateral is equal to the pledged collateral minus the collateral locked for pending transactions. The available collateral fluctuates as the status of user cross-chain transactions changes.

When a user initiates a transaction, a certain amount of collateral is locked, reducing the available collateral. Once the transaction is confirmed by LnProvider, the available collateral increases.

## LnProvider Failure

LnProvider may fail, but even if it fails, the user's assets are safe.

If a user initiates a transfer, but the LnProvider fails to transfer tokens to the user on the target chain.

There are two situations:

- If the LnProvider fails to transfer tokens to the user, and [the collateral is on the target chain(Default Type)](https://www.notion.so/Helix-Token-Protocol-c12817f371404edc8c381eefd55e7917?pvs=21).  
  The user can submit the transfer proof to the target chain through the messaging channel. The target chain combines the proof with evidence that the LnProvider failed to transfer the tokens, and extracts collateral from the LnProvider to compensate the user.

  {: .note }
  > This should be the situation users prefer. 

- If the LnProvider fails to transfer tokens to the user, and [the collateral is on the source chain](https://www.notion.so/Helix-Token-Protocol-c12817f371404edc8c381eefd55e7917?pvs=21).  
  The user can use the messaging channel to send proof of the failed transfer to the source chain. The source chain can then combine this proof with the original transfer proof to verify the LnProvider's failure. Once the source chain has verified the LnProvider's failure, it can extract collateral from the LnProvider to compensate the user.

Either of the above methods is used to ensure the security of the user's assets during the process. The choice between them depends on the direction of the bridge message. 

# Key Terms Explaination

## LnProvider(Liquidity Node Provider)

A LnProvider has two components: an account and a client program. 

* **LnProvider Account**  
  The account exists on both the source chain and the target chain. On the source chain, the account receives assets from users who initiate cross-chain transfers. On the target chain, the account distributes those assets to users who have successfully completed a cross-chain transfer. The account is registered to the asset bridge and has a collateral.

* **LnProvider Client Program**  
  The client program, which is associated with the account, monitors user transfer events and automatically transfers tokens from the account on the target chain to the user's account, completing the cross-chain process.

## Slasher

This compensation mechanism ensures the security of users' assets during the process, although they add some complexity for regular users. To address this issue, a separate role called a "slasher" is introduced.

Slasher is responsible for monitoring the behavior of LnProviders and, in the event of an LnProvider failure, helping users complete transactions through alternative means. When necessary, slasher utilizes the messaging channel to extract the LnProvider's collateral and impose penalties. This ensures that the user's transaction will eventually be completed, even in exceptional circumstances. The slasher helps users complete the process without having to understand the underlying details. This improves the user experience and makes it easier for regular users to take advantage of cross-chain transfers.

## TransferId

```
TransferId = Hash(params，lastTransferId)
```
  
Where:
* **params**:   
  The params refers to the specific parameters of the cross-chain transfer, including token address, quantity, LnProvider, receiver, etc.

* **lastTransferId**:  
  The lastTransferId of the first transaction is 0. For the k-th transaction, the lastTransferId is the transferId of the (k-1)th transaction.

### Purpose

The purpose of this TransferId mechanism are for:

- **Uniqueness**  
  The system maintains a 'lastTransferId' value, which is updated after each transaction is completed. This current lastTransferId value is used to generate the TransferId for the next transaction, ensuring that no two transactions share the same TransferId. This mechanism prevents data duplication and ensures the integrity of the system.

- **Ordering**  
  In each transaction, the Hash parameter includes the TransferId of the previous transaction, ensuring the sequencing of transactions.



## Type 1(Opposite) - Collateral on Source Chain

This type need a messaging channel **from Target Chain** to **Source Chain**.

For this type we implement the following asset bridge cross-chain protocol.

### Safety Assumptions

1. There should be a secure and reliable general-purpose messaging channel for communication from the source chain to the target chain. This channel must maintain the integrity and confidentiality of messages sent from source chain to target chain.
2. The source chain and the target chain use the same time(the same world clock), allowing for consistent timestamping and handling of time-sensitive events across both chains.

### Interaction Flow - LnProvider Registration

LnProvider registration is the process by which a LnProvider becomes part of the cross-chain asset transfer protocol. This process involves a few key steps, during which the LnProvider provides information about itself and commits to certain obligations. Once registered, the LnProvider is responsible for facilitating cross-chain transfers and adhering to the protocol's rules.

1. Decide the tokens to support  
   The LnProvider must select which tokens it will support for cross-chain transfers. These tokens must exist on both the source and target chains, so that transfers can be facilitated.

2. Decide the pledging collateral  
   In order to participate in the cross-chain asset transfer protocol, the LnProvider must pledge a certain amount of tokens as collateral on source chain.

3. Decide on the fees to charge  
   The LnProvider must determine the fee charged for cross-chain transfer. The fee are intended to cover the cost of the LnProvider's activities on the target chain, including **1. gas fee**, as well as **2. LnProvider’s profit**.

4. Completing registration by sending a transaction **on source chain**  
   The LnProvider completes the registration process by calling an interface on the source chain, submitting collateral, and setting the fees.

   Once registration is complete, the LnProvider will be recognized by the protocol and can begin offering cross-chain transfer services.

5. Running the client program  
   After registration, the LnProvider needs to run a client program that constantly monitors the network for cross-chain transfer requests. When a request is received, the program automatically executes a transaction to complete the transfer.

The registration process described above allows the LnProvider to become a liquidity provider and participate in the cross-chain asset transfer protocol.

### Interaction Flow - Transfer

1. To initiate a cross-chain transfer, a user first selects the token they wish to transfer, specify the amount, and choose an LnProvider to facilitate the transfer.

   {: .note }
   > An intermediary party can assist the user in selecting an appropriate LnProvider, if necessary.

2. The user retrieves the latest state from the blockchain and takes a snapshot of the state. Then, the user check the available collateral amount of the LnProvider to ensure it is sufficient. If not, the cross-chain transfer cannot proceed.

   {: .note }
   > In actual products, this step will be completed by the front-end code of users.

3. If the collateral is sufficient, the user then calls the on-chain api with the snapshot, initiating the transfer.

4. The blockchain verifies that the snapshot is valid. If the on-chain state has changed since the snapshot was taken, the transfer fails. Otherwise, the user's assets are transferred to the LnProvider’s account on source chain, and a transfer proof is generated.

5. The LnProvider monitors the event confirming the transfer **on the source chain**. Upon confirmation, LnProvider transfer the tokens to the user's account **on the target chain** and generate a proof of this transaction.

   In this step, **a timeout period is set**, and the LnProvider is required to complete the transfer within this time limit. If the LnProvider fails to do so, the slasher can do slashing operation.

### Interaction Flow - Slashing

1. On the target chain, the `Slasher` will transfer tokens from its own account to the user's account to cover the transaction (this is known as "slash and cover"), and generate a transfer proof.
2. The `Slasher` will then send this transfer proof to the source chain through the underlying bridge messaging system. Upon receiving the message, the source chain will verify the transaction. If the verification is successful, the source chain will extract the `LnProvider's` collateral to compensate the slasher for covering the assets and to impose a penalty on the `LnProvider`.
3. The determination of whether the transfer is within the timeout period is based on comparing the time when the user sends the transaction (the timestamp of the block on the source chain where the transaction is included) with the current time on the target chain. The synchronization of clocks between the two chains is ensured by the safety assumptions, allowing the timeout time to be effective within a certain error range.
4. This mechanism ensures that the `LnProvider` fulfills its responsibility to complete the transfer within the specified time, and in the event of a failure, the slasher steps in to cover the assets and initiate penalties, thus maintaining the security and reliability of the cross-chain asset transfer protocol.

### Ordering Mechanism

- Every cross-chain transaction on the target chain must be executed or be subject to a 'slash' operation.
- 'Slash' operations cannot be executed before the timeout period has expired.
- For every cross-chain transaction on the target chain, it must follow a preceding transaction that was executed.
- When validating a "slash" transaction on the source chain, it must be ensured that all preceding transactions have not been subjected to a "slash" operation, or if any "slash" operations were applied, they have all been executed and completed.

The above content describes how to ensure the execution order in the cross-chain asset transfer protocol, ensuring that before any transaction is completed, all preceding transactions have been completed.This sequencing mechanism effectively prevents malicious withdrawals of collateral, ensuring that users only need to consider the available collateral when sending a cross-chain transfer.

With this mechanism, users can be confident that their transactions will be processed reliably.

{: .note }
> The specific implementation of maintaining the sequence is as follows:
> 1. Set SId(1) to a certain initial value, indicating that there were no slash transactions before the first transaction.
> 2. For each transaction (the k-th transaction) on the target chain, check if the previous transaction (the (k-1)-th transaction) has been sent and further determine whether it has been slashed. Based on the status of the previous transaction, determine the closest slash transaction Id (SId(k)) before the k-th transaction.
> 3. If the previous transaction is a slash transaction, meaning that the (k-1)-th transaction has been subject to a slash operation, then SId(k) = Id(k-1), which means that SId(k) for the k-th transaction is set to the Id(k-1) of the (k-1)-th transaction.
> 4. If the previous transaction is not a slash transaction, meaning that the (k-1)-th transaction was normally executed by LnProvider, then SId(k) = SId(k-1), which means that SId(k) for the k-th transaction is set to the SId(k-1) of the (k-1)-th transaction. 

By following this approach, when sending the slash transaction Id(k) from the target chain to the source chain, the information of SId(k) is included. The source chain, when validating the slash transaction, verifies whether SId(k) has been completed, ensuring that the execution process of the k-th transaction is correctly ordered. This sequencing mechanism effectively prevents transactions from being maliciously tampered with or prematurely executed, ensuring the security and reliability of the transactions.



## Type 2(Default) - Collateral on Target Chain

**This type need a messaging channel from Source Chain** to **Target Chain.**

For this type we implement the following asset bridge cross-chain protocol.

### Safety Assumptions

1. A secure and reliable message channel connects the source chain to the target chain. This channel ensures that messages can be reliably sent from the source chain to the target chain, and the integrity and confidentiality of the messages are maintained.
2. The source chain and the target chain use the same time(the same world clock), allowing for consistent timestamping and handling of time-sensitive events across both chains.

### Differences from Type1

The basic interaction process is similar to the message channel from the target chain to the source chain, with the following differences:

- **LnProvider's collateral is pledged on the target chain.**

- **Slashing**  
  When the slasher executes a slash operation, it sends a transaction to the target chain that does not require to cover the user any tokens.

  If the provider delivers after the timeout period, a portion of its collateral will be deducted as a penalty. This penalty compensates the slasher for the fees incurred during the slash operation.

  When the slashing request arrives at the target chain, it checks whether the transaction needs to be slashed in the following scenarios:

  1. If the cross-chain transaction is not past the timeout period, the slashing request is considered invalid and will not be executed.
  2. If the cross-chain transaction is already past the timeout period and the LnProvider has been slashed, then the slashing request is considered invalid and will not be executed.
  3. If the cross-chain transaction is past the timeout period and has not yet been delivered, the slashing request will be executed. The user will receive the transferred assets, and the slasher will receives fees and the penalty assets.
  4. If the cross-chain transaction is past the timeout period, but the provider delivered the transaction within the allowed time, then the slash operation is not permitted and will not be executed.
  5. If the cross-chain transaction is past the timeout period and the LnProvider delivered after the timeout, the slash operation is only partially valid. The slasher can receive some of the penalty assets, but not the full amount, to compensate for the gas and bridge fees incurred by the user.

- **Ordering**  
  The ordering requirement states that any transaction on the target chain must have its preceding transaction completed before the current transaction can be finalized. This can be accomplished in two ways: by the LnProvider completing the transaction as normal, or by the slasher executing a slash operation.

  That is, before a transaction is slashed, it's important to make sure that any collateral associated with that transaction cannot be extracted by its subsequent transactions.

  Additionally, we demand that LnProvider includes the state of the extraction into the snapshot when withdrawing collateral. At the time of extraction, all transactions prior to that state must have been completed, ensuring that the available collateral does not decrease during the period from the initiation of the cross-chain transaction to its execution on the target chain.
