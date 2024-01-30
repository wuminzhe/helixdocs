---
group:
  title: 🔹 User Guide
  order: 4
order: 2
---

## Fee for CBA model

Under this model, Helix relies on generic messaging to deliver cross-chain information, with the cost of the underlying messaging accounting for most of the transfer costs. Currently, Helix does not charge protocol fees, so for this type of token bridge, the fees that the user needs to pay are limited to the cost of executing the message channel, which is usually paid by native tokens.

## Fee for LnBridge

The fee consists of two parts, a base fee to cover the LnProvider's gas fee for executing transactions on the target chain and a liquidity fee to compensate the LnProvider for the loss of liquidity.

- Base Fee
  <br>It is configured by LnProvider, and the fee is a fixed value until the LnPriver is updated. This fee needs to be paid to the LnProvider at the moment the user initiates the transfer.
- Liquidity Fee
  <br>Also configured by LnProvider, and this fee is related to the amount of transfer made by the user, and the higher the amount of transfer, the higher the fee charged.

Assuming the user's transfer quantity is `Amount`, the base fee is `BaseFee`, and the liquidity rate is `LiquidityRate` then

```
TotalFee = BaseFee + Amount * LiquidityRate
```

Fees are captured outside of the transfer, so users should set aside a portion of the Token to pay for cross-chain fees when transferring across the chain. LnProvider is open for registration and its fee is freely configurable, Helix backend will grab all the LnProvider information and will choose the one that makes sense for the user based on the configuration parameters. For example, an LnProvider with a lower execution fee will have a higher probability of being selected.
