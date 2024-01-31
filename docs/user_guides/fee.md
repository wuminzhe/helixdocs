---
title: Fee
layout: default
parent: User Guides
nav_order: 3
---

# Fee

## Fee for CBA Model

Under the CBA model, Helix relies on generic cross-chain messaging channel to transmit cross-chain information, with the cost of the underlying messaging being the primary factor contributing to the transfer expenses. Currently, Helix Bridge does not charge protocol fees. Therefore, in this type of token bridge, the fees incurred by users are limited to the cost associated with executing the messaging channel, typically paid using native tokens.

## Fee for LnBridge

For LnBridge, the fee structure consists of two components: a base fee to cover the gas fees incurred by the LnProvider when executing transactions on the target chain and a liquidity fee designed to compensate the LnProvider for the loss of liquidity.

1. Base Fee  
   The base fee is determined and set by the LnProvider, remaining a fixed value until the LnProvider updates it. Users are required to pay this fee to the LnProvider at the time they initiate the transfer.

2. Liquidity Fee  
   Similar to the base fee, the liquidity fee is also determined by the LnProvider. However, this fee is directly related to the amount of the transfer initiated by the user. As the transfer amount increases, the corresponding fee also increases.

Assuming the user's transfer quantity is denoted as "Amount," the base fee as "BaseFee," and the liquidity rate as "LiquidityRate," then the total fee for the transfer can be calculated as follows:

```
TotalFee = BaseFee + Amount * LiquidityRate
```

It's important to note that these fees are separate from the actual transfer and need to be accounted for by users. Users should set aside a portion of the token to cover the cross-chain fees. LnProvider registration is permission-less, and their fee is freely configurable. Helix's backend system will gather information from various LnProviders and choose the most suitable one for users based on their preferences. For instance, an LnProvider with a lower fee will have a higher chance of being recommended by the system.
