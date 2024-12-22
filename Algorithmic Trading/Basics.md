# Supply, Demand and Price Discovery
Modern markets are still driven by basic economic principles of supply and demand. 
- When demand outweighs supply, prices of a commodity or service are likely to rise higher to reflect the relative shortage of the commodity or service in relation to the demand for it. 
- Conversely, if the market is flooded with a lot of sellers for a particular product, prices are likely to drop. 
- The market is always trying to reflect the equilibrium price between available supply and demand for a particular product. 
This is the **fundamental driver of price discovery** in today's markets.

The speed at which participants receive information, the speed at which they react, the granularity of the data that they can receive and handle, and the sophistication with which each participant draws trading insights from the data they receive, is where the competition lies in modern trading.

# Market sectors
Market sectors are the different kinds of underlying products that can be traded. The most popular market sectors are commodities (metals, agricultural produce), energy (oil, gas), equities (stocks of different companies), interest rate bonds (coupons you get in exchange for debt, which accrues interest, hence the name), and foreign exchange (cash exchange rates between currencies for different countries)

# Asset Class
Asset classes are the different kinds of actual vehicles that are available for trading at different exchanges. For example, **cash interest rate bonds**, **cash foreign exchange**, and **cash stock** shares are what we described in the previous section, but we can have financial instruments that are derivatives of these underlying products. Derivatives are instruments that are built on top of other instruments and have some additional constraints. The two most popular derivatives are futures and options, and are heavily traded across all derivatives electronic exchanges.

A **call option**, is the right to buy, but not an obligation to buy at expiration
- If the price of the underlying product increases prior to expiration because now, such a party can exercise their option at expiration and buy the underlying product at a price lower than the current market price, and make profit.
- Conversely, if the price of the underlying product goes down prior to expiration, such a party now has the option of backing out of exercising their option and thus, only losing the premium that they paid for.

A **put option**, is the right to sell, but not an obligation to sell at expiration
- If the price of the underlying product decreases prior to expiration, the holder of the put option can exercise their option at expiration and sell the underlying product at a price higher than the current market price, making a profit.
- Conversely, if the price of the underlying product increases prior to expiration, the holder of the put option has the option to back out of exercising it, thereby only losing the premium they paid for the option.

## Market Trade Cycle
The trading exchange maintains a book of client buy orders (bids) and client ask orders (asks), and publishes market data using market data protocols to provide the state of the book to all market participants. 
**Bids**: client buy orders 
**Asks**: client ask orders

Market data feed handlers on the client side decode the incoming market data feed and build a limit order book on their side to reflect the state of the order book as the exchange sees it. This is then propagated through the client's trading algorithm and then goes through the order entry gateway to generate an outgoing order flow. The outgoing order flow is communicated to the exchange via order entry protocols. This, in turn, will generate further market data flow, and so the trading information cycle continues.

## Exchange 
The exchange order book maintains all incoming buy and sell orders placed by clients. It tracks all attributes for incoming orders—prices, number of contracts/shares, order types, and participant identification.
Market participants are allowed to place new orders, cancel existing orders, or modify order attributes such as price and the number of shares/contracts, and the exchange generates public market data in response to each order sent by participants

## Matching Algorithm
- Bids are sorted from the highest price (best price) to the lowest price (worst price). Bids with higher prices have a higher priority as far as matching is concerned.
- Asks are sorted from the lowest price (best price) to the highest price (worst price). Asks with lower prices have a higher priority.
- Bids (or Asks) at the same price are prioritized depending on the matching algorithm. The simplest FIFO (First In First Out) algorithm uses the intuitive rule of prioritizing orders at the same price in the order in which they came in. 
- As regards asks at the same price, the matching prioritization method depends on the matching algorithm adopted by the exchange for the specific product.

When incoming bids are **at or above the best (lowest price) ask orders**, then a match occurs.
Conversely, when incoming asks are **at or below the best (highest price) bid orders**, then a match occurs.

> This will be important later on when we discuss how sophisticated trading algorithms use speed and intelligence to get higher priorities for their orders and how this impacts profitability.