---
title: Transfer Token
layout: default
parent: User Guides
nav_order: 2
---

# Transfer Token

## Steps

- Visit Helix Bridge at [mainnet](https://helixbridge.app/) or [testnet](https://helix-stg-test.vercel.app/).
- Select source chain, target chain and token you want to transfer.  
  <img src="https://docs.helixbridge.app/transfer.png" style="width:100%;">
- Switch wallet to the source chain and connect wallet.
- Fill the transfer amount, and then you can find the transfer information include fee, estimated time if there are bridges avaliable.
- Click `Transfer` and you will receive a popup for transfer confirmation. Then click `Confirm` after every detail is checked.  
  <img src="https://docs.helixbridge.app/confirm.png" style="width:60%;">
- Then you will be prompted to confirm the transaction in your wallet. After confirming in your wallet, you have submitted the transaction. You can track the transfer progress by clicking on the `transaction history` in the pop-up window.
- All the transfer histories can be found in `Explorer`, and you can filter the history by account address or transaction hash. Click the record, you can see the detail of the transfer.
  <img src="https://docs.helixbridge.app/history_detail.png" style="width:100%;">

## Attention

There may be multiple bridges for the same transfer path, and Helix selects one by default for the user, but the user is free to choose which bridge to use to complete the cross-chain in order to ensure that the costs and efficiency of the transfer are reasonable.

Once the transfer is submitted, it cannot be canceled, please double check the amount, target chain and other information before initiating the cross-chain transfer from your wallet.

The costs, arrival times, caps, etc. given in the hints are all estimates, and there will be some error between them and the actual values.
