# Audit-5: Codehawks First Flights Orderbook

Repo: https://github.com/azriellee/2025-07-orderbook.git

## Audit Experience:

I only spent about 2h going through the code, but the code base was pretty small so I still managed to finish auditing. Unfortunately I was not able to find much, and only came up with 1 finding related to MEV/Front-running that I thought was severe but was ultimately rejected. After going through the finding results, I realised that I was on the right track about the MEV/Front-running issue, just that I did not apply it correctly.

## Findings Summary:

1 High

[H-1] Lack of Slippage Protection in `buyOrder` Results in Sellers Being Able To Manipulate and Drastically Increase Prices Using `amendSellOrder` For Buyers

## Results Summary:

Rank: -
Results: 0

[Invalid] Lack of Slippage Protection in `buyOrder` Results in Sellers Being Able To Manipulate and Drastically Increase Prices Using `amendSellOrder` For Buyers.

Reason: This attack is only possible if Buyers approve the `Orderbook` a lot more tokens than required to proceed with the `buyOrder`. This evaluation makes sense if users always approve just enough tokens instead of more than necessary.

### Severe Findings Missed:

[H] Buy orders can be front-run maliciously - buy orders can be front-run using MEV bots if there is a favourable change in exchange rate between tokens, and sellers are trying to amend their price. This transaction can be monitored in the mempool and attackers can take advantage of this arbirtrage.

[H] Amends or Cancellation of sell orders can be front-run - sell order transactions can also be front-run by malicious sellers if they monitor that someone had purchased their order. The bot could reduce the number of tokens being sold while maintaining the price, causing the buyer to severely overpay.
