---
description: Insuring the protocol against collateral volatility.
---

# 🛡️ Hedging Agents

## 🔎 TL;DR

- Hedging Agents (HAs) get perpetual futures from the protocol: they can get leveraged in one transaction on the evolution of the price of a collateral with a multiplier of their choice.
- They are here to insure the protocol against the volatility of the collateral brought by users. With enough demand for HAs, the protocol could resist collateral price drops of up to 99%.
- HAs can make significant gains in case of price increase but also substantial losses when collateral price decreases.
- They pay small transaction fees (potentially around 0.3%) when they open their position and when they close it.
- Contrary to centralized exchanges, they do not have to pay funding rates for holding their positions.

## 🗺 Principle

Angle by essence is highly dependent on collateral volatility. Let's say one stable seeker brings 1 wETH against 2000 agUSD and the price of wETH then decreases by 50% (from 2000$ to 1000$). The protocol then needs to find 1 wETH to ensure the redeemability of the 2000$ of stablecoins and maintain their stability.

We say that the protocol needs to insure itself against the volatility of the collateral. While surges in collateral prices are beneficial to the protocol, drops, as in the example above, are less desirable.

For this reason, Angle transfers this volatility to other actors looking to get leverage on the collateral: Hedging Agents (HAs). They are the agents insuring the protocol against drops in collateral prices, making sure that the protocol has always enough reserves to reimburse users.

## 🔮 Perpetual Futures

Hedging Agents are taking perpetual futures from the protocol. When they come in the protocol to open a position, they bring a certain amount of collateral (their margin), and choose an amount of the same collateral from the protocol they want to hedge (or cover/back). The protocol then stores the oracle value and timestamp at which they opened a position.

Hedging Agents are independent from one another, meaning that the actions of one Hedging Agent have no impact on the position of another Hedging Agent.

Precisely speaking, if a HA enters with an amount `x` of collateral (`x`is the margin) and decides to take on the volatility of an amount `y` of the same collateral (`y` is the amount committed, or the position size) that was brought by users minting stablecoins, then the protocol stores `x`, `y`, the oracle value and the timestamp at which this HA came in.

At any given point in time this HA is entitled to get from the protocol:

$$
\texttt{cash out amount} = x+y\cdot (1-\frac{\texttt{initial oracle price}}{\texttt{current oracle price}})
$$

This formula means that a HA will get back her input `x`, plus or minus the capital gains or losses of the amount `y` she decided to back.

The **PnL** of the HA on this position is therefore:

$$
\texttt{PnL} = y\cdot (1-\frac{\texttt{initial oracle price}}{\texttt{current oracle price}})
$$

Since HAs bring collateral to the protocol, we define their **leverage** as:

$$
\texttt{leverage} = \frac{x+y}{y} =  \frac{\texttt{margin + amount committed}}{\texttt{amount committed}}
$$

### 📈 Price Increase Scenario

When the collateral price increases (with respect to the asset stablecoins are pegged to), besides their margin (amount brought initially), HAs are entitled to get the capital gains they would have made if they had owned the collateral they hedged.

If the HA brought 1 wETH and decided to back 1 wETH from the protocol, at a wETH price of 2000$, then:

$$
x = 1, \space y=1
$$

$$
\texttt{initial oracle price} = 2000
$$

If the price of wETH increases to 4000$, then according to the formula above, this HA can get from the protocol:

$$
\texttt{cash out amount} = 1.5 \space \texttt{wETH}
$$

The HA made 6000$ from her 2000$ initially. If she had just stayed long without leverage, she would have only 4000$.

### 📉 Price Decrease Scenario

When the collateral price decreases (with respect to the asset stablecoins are pegged to), HAs will incur losses on their margin as if they had owned the collateral they covered.

Back to the previous example, if the price of wETH decreases to 1000$, then the cash out amount of the HA becomes:

$$
\texttt{cash out amount} = 1 + 1 \cdot (1-2) = 0
$$

At this point, the HA is liquidated and her collateral goes to the protocol. She cannot claim anything.

In general, the cash out amount of a HA can go to zero, if the price drops to:

$$
\texttt{current price} = \frac{y}{x+y}\cdot \texttt{initial price}
$$

## 💧 HAs Liquidations

In practice, and like in most centralized perpetual swaps exchanges, there is for each HA a maintenance margin meaning that if the value of the theorical cash out amount gets too small compared with the amount committed by the HA, then this HA can get liquidated. A HA can hence get liquidated even with a non null cash out amount.

Mathematically speaking, we define the margin ratio of a HA as:

$$
\texttt{margin ratio} = \frac{\texttt{margin + PnL}}{\texttt{amount committed}}
$$

Or to use the above notations:

$$
\texttt{margin ratio} = \frac{x}{y} + (1-\frac{\texttt{initial oracle price}}{\texttt{current oracle price}})
$$

A HA can get liquidated if:

$$
\texttt{margin ratio} \leq \texttt{maintenance margin}
$$

## 🛏️ HAs Hedged Amounts

When HAs enter the protocol, they specify a position size denominated in collateral, representing an amount of the protocol collateral reserves they are hedging. Yet from a protocol perspective, when an HA comes in the protocol, this HA insures a fixed amount of stablecoins.

This quantity remains constant and only depends on variables fixed upon HAs entry. So while HAs only see that they back an amount of collateral from users, from a protocol perspective, each HA insures the protocol for a fixed amount of stablecoins. This is what the accounting of the protocol keeps track of when determining when to let HAs come in or not.

The total amount hedged by HAs for a given collateral/stablecoin pair is hence the sum of the product between the amount committed by HAs and their entry price: it is a measure of how much stablecoins issued are backed and insured.

This quantity is compared to the amount of collateral `in stablecoin value` needed by the protocol to pay back users in case they all want to burn their stablecoins. For example, if a user brings 1 wETH to mint 2000 agUSD, and another one burns 1000 agUSD, the amount to hedge is 1000 USD of wETH. HAs are able to hedge a fraction of this quantity (close to 100%): this is called the target hedge amount .

The hedge ratio of the protocol **for a given stablecoin/collateral pair** is hence defined as:

$$
\texttt{Hedge Ratio} = \frac{\texttt{Total amount hedged by HAs in stablecoin}}{\texttt{Total value of stablecoins issued}}
$$

## 🏢 Insurance of the Protocol Against Collateral Volatility

Here we explain in a more imaged way how the protocol can always have enough collateral to pay back users burning stablecoins in case of price changes of the collateral.

If HAs have a 6x leverage and back all the collateral in the protocol that was used to issue stablecoins:

![](../../.gitbook/assets/h13.jpg)

![](../../.gitbook/assets/ha2.jpg)

![](../../.gitbook/assets/ha1.jpg)

## 🪙 Transaction Fees

In Angle, Hedging Agents have to pay small transaction fees when they open and close positions from the protocol. These transaction fees are computed on the amount that is committed by the HA (the position size). Entry and exit fees for HAs depend on hedging curves, which define transaction fees for HAs based on the hedging ratio of the protocol.

{% hint style="success" %}
Note that on Angle, there is no funding rate to be paid by perpetual futures holders as opposed to most perps exchanges. This allow traders to hold their positions longer at a much lower cost.
{% endhint %}

{% hint style="info" %}
The exact values of the transaction fees for HAs depend on the hedge ratio (sometimes referred to as coverage ratio) of the specific agToken/collateral pair. You can see the current fees situation in the [analytics](https://analytics.angle.money) page related to the collateral/stablecoin pool in question.
{% endhint %}

### Entry Transaction Fees

The entry transaction fees for HAs is an upfront cost paid when a HA opens a position.

The higher the hedging ratio, the more expensive it gets to be an HA. Conversely, HAs should be incentivized to enter positions to help hedge the protocol when the hedging ratio is low: transaction fees would be lower in this case.

![](../../.gitbook/assets/haentry.jpg)

Let' say a HA comes to the protocol with 1 wETH and opens a 2 wETH poisition (hedging the protocol against the changes in price of these 2 wETH). If the transaction fees are 0.3%, then the protocol considers that the HA has a margin of (1 - (0.003 x 2)) = 0,994 wETH for a position of 2 wETH.

### Exit Transaction Fees

Exit fees are paid by HAs when they close their perpetuals. The more collateral is hedged by HAs, the less expensive it is to exit the protocol. When the hedging ratio is low, HAs should be discouraged to exit with higher transaction fees.

![](../../.gitbook/assets/haexit.jpg)

If a HA had an initial margin of 1 wETH and a position size of 2 wETH, then with 0.3% transaction fees, the HA will get in wETH the current value of her perpetual according to the cash out formula above minus 0.3% of 2 wETH (the amount hedged at the opening).

### Fees To Add or Remove Margin

When HAs open a perpetual, they have the opportunity to add or remove to their margin thus decreasing or increasing their leverage. As entry and exit fees depend only on the position size (or committed amount) of HAs, and these add/remove operations do not modify it, no fees are paid for such operations.

![](<../../.gitbook/assets/emoji-ha (1) (6) (10).png>)
