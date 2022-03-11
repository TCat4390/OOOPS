# On-Chain Oracle Only Perpetual Swap

### Introduction
What is liquidity? Liquidity is a market participant's ability to trade with a counterparty when needed. Liquidity is usually provided by so-called "market makers," who use their mathematical models to place orders at various price points in the market and profit from the spread between the orders. In this article, I try to propose a framework to use a decentralized structure that not only allows everyone to have the opportunity to market-mak but also significantly improves the capital efficiency, and at the same time, through such an AMM mechanism, can create a perpetual contract market with low slippage, fast transactions, and no funding rate.

I believe the market maker should only bring together buyers and sellers at different times and price, but it is not biased towards the market itself.

### What is the goal of the protocol?
- To get rid of the funding fee
- Enable the opportunity of market making to everyone
- Users can make market with stablecoins
- Try to approach on-chain market making in a different way

## Mechanism
### Definition
The core concept of this model is as follows.
- Market-making risk is elevated in times of severe market volatility, i.e., volatility should be relevant to the model.
- Market-making funds should be delta-neutral, holding no positions whenever possible.
- Directly use oracle prices so that the contract market is never decoupled from the spot market.

And so, the following model is proposed:

$P_e = P_0*e^{\pm d*V \over T}*e^{\pm A*U}$


$P_0$ is the oracle price
$P_e$ is the execution price
$d$ is the impact of the trade on protocol margin usage, and is calculated using $order\space size\over protocol\space balance$
$T$ is the tolerance factor
$A$ is the amplifier for the inventory risk
$U$ is the current margin usage
$V$ is the volatility of the market

### How it works
When an user executes a trade, the execution price is calculated as follows:

### When the protocol is none or holding long position:

- If the user wants to sell (increase the inventory risk,) the price will be:
$P_e = P_0*e^{-d*V \over T}*e^{-A*U}$
- If the user wants to buy (decrease the inventory risk,) the price will be:
$P_e = P_0*e^{d*V \over T}*e^{-A*U \over 2}$

### When the protocol is holding short position:

- If the user wants to sell (decrease the inventory risk,) the price will be:
$P_e = P_0*e^{-d*V \over T}*e^{A*U \over 2}$
- If the user wants to buy (increase the inventory risk,) the price will be:
$P_e = P_0*e^{d*V \over T}*e^{A*U}$

### Why
$d$ : This "d" factor comes into place to create a quote with a reasonable spread. It's designed to be small enough to ignore for small orders, and for large orders, this will ensure that the executed price is reasonable for the upcoming inventory risk.

$T$ : This factor gives the protocol the power to control how much of the risk will take. The bid-ask spread will be significant with a small tolerance, but it might not be lucrative for an average user to use the protocol. And for large tolerance, the bid-ask will be small, so in theory, users will agree this protocol is an excellent place to trade, but the downside is that the risk of carrying too much inventory will increase.

$A$ : When it comes to inventory risk, keeping it at zero is the priority of the protocol, so a user can be encouraged or punished, given whether the trade decreases or increases the inventory risk. And by keeping the effect of the "A" factor non-symmetrical on both side, the protocol can still ensure the spread is positive while encouraging arbitragers to migrate the risk.

## Examples

With $V = 0.75\space A = 0.1\space T = 10\space PoolBalance = 100M\space USD$:
Assuming the oracle price of ETH is $3000 USD.

The bid and ask price of executing an order of $5M USD will be:
```
Pool holds long. Margin used: 10.0%
Bid: 2959.03 Ask: 2996.25 Spread: 124.07 bps
Pool holds long. Margin used: 9.0%
Bid: 2961.99 Ask: 2997.75 Spread: 119.19 bps
Pool holds long. Margin used: 8.0%
Bid: 2964.96 Ask: 2999.25 Spread: 114.31 bps
Pool holds long. Margin used: 7.0%
Bid: 2967.92 Ask: 3000.75 Spread: 109.42 bps
Pool holds long. Margin used: 6.0%
Bid: 2970.89 Ask: 3002.25 Spread: 104.53 bps
Pool holds long. Margin used: 5.0%
Bid: 2973.86 Ask: 3003.75 Spread: 99.63 bps
Pool holds long. Margin used: 4.0%
Bid: 2976.84 Ask: 3005.25 Spread: 94.72 bps
Pool holds long. Margin used: 3.0%
Bid: 2979.82 Ask: 3006.76 Spread: 89.80 bps
Pool holds long. Margin used: 2.0%
Bid: 2982.80 Ask: 3008.26 Spread: 84.87 bps
Pool holds long. Margin used: 1.0%
Bid: 2985.78 Ask: 3009.77 Spread: 79.94 bps
Pool holds long. Margin used: 0.0%
Bid: 2988.77 Ask: 3011.27 Spread: 75.00 bps
```
```
Pool holds short. Margin used: -10.0%
Bid: 3003.75 Ask: 3041.53 Spread: 125.94 bps
Pool holds short. Margin used: -9.0%
Bid: 3002.25 Ask: 3038.49 Spread: 120.81 bps
Pool holds short. Margin used: -8.0%
Bid: 3000.75 Ask: 3035.46 Spread: 115.69 bps
Pool holds short. Margin used: -7.0%
Bid: 2999.25 Ask: 3032.42 Spread: 110.58 bps
Pool holds short. Margin used: -6.0%
Bid: 2997.75 Ask: 3029.39 Spread: 105.47 bps
Pool holds short. Margin used: -5.0%
Bid: 2996.25 Ask: 3026.37 Spread: 100.38 bps
Pool holds short. Margin used: -4.0%
Bid: 2994.75 Ask: 3023.34 Spread: 95.29 bps
Pool holds short. Margin used: -3.0%
Bid: 2993.26 Ask: 3020.32 Spread: 90.20 bps
Pool holds short. Margin used: -2.0%
Bid: 2991.76 Ask: 3017.30 Spread: 85.13 bps
Pool holds short. Margin used: -1.0%
Bid: 2990.27 Ask: 3014.28 Spread: 80.06 bps
Pool holds short. Margin used: 0.0%
Bid: 2988.77 Ask: 3011.27 Spread: 75.00 bps
```

As you can see, the bid-ask price will be non-symmetrical when the usage of the protocol balance is not zero.

The bid-ask price may go above or below the oracle price at the same time to encourage the arbitragers to trade and reduce the inventory risk when the protocol carries too much.

## Simulation
### Environment
- the data used is close prices of Binance BTC-USDT pair from 2017-08-17 12:00 to 2022-02-16 23:00, using hourly candle
- all pnl is calculated using the close price as described above
- the starting balance for the protocol is $100M USD
- there will be 100 users randomly placing trades at each candle, 45% of the decision is either long or short and will not do anything if there's already an open position, 5% of trade is to close the position
- the order size is uniformly distributed between $100 to $100K USD
- $V = 0.75\space A = 0.1\space T = 100\space PoolBalance = 100M\space USD$

### Simulation Purpose
In this simulation, I want to know how the pool's balance changes over time, what the margin usage looks like, and the spread of opening a 10M position at a given moment.

And is this idea profitable or not if the protocol has enough participants.

### Result
#### The pool's balance
![](https://i.imgur.com/xZcIBhf.png)

The y-axis is the balance ratio, and the x-axis is the data index, so 3.00 means the balance 3x in the end.

#### Pool's margin usage
![](https://i.imgur.com/I0NksXy.png)

The y-axis is the margin usage, and the x-axis is the data index. The result shows at an extreme scenario, the usage never goes above or under $\pm 15\%$, although this is not mathematically guaranteed and is highly related to the parameters of the protocol.

#### The spread of executing a $10M order at any given time
![](https://i.imgur.com/t71H70E.png)
What this graph show is even in the beginning when a $10M order will use 10% of the pool's margin, the spread is still acceptable, and with the increasing balance of the protocol, the spread comes as little as 53 bps.

## Notes 
1. The protocol relies heavily on arbitragers to ensure the usage is close to zero, so incentives matter.
2. This protocol needs a fast oracle source, thus, a blockchain with a short block time is required.

## TODO
- Mathematically prove the relationship between protocol parameters and the spread gave the margin usage and order size.
