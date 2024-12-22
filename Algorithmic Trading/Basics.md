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

Matching Process with example:

1. **Key Terms:**
    
    - **Aggressive Orders:** These are orders that are willing to trade immediately at the best available price in the market. They "take" liquidity from the order book. For example, a market order is an aggressive order because it matches with existing orders rather than waiting.
    - **Passive Orders:** These are resting orders already in the order book. They "provide" liquidity and are waiting to be matched with incoming orders.
    - **Limit Price:** This is the maximum price the buyer is willing to pay or the minimum price the seller is willing to accept. It serves as a boundary beyond which a match cannot occur.
2. **Matching Process:**
    
    - When an aggressive order arrives, it starts to match with the best available passive orders on the opposite side of the book (buy orders match with sell orders and vice versa).
    - Matching happens at prices that are favorable to the aggressive order. For instance, if it’s a buy order, it matches with the lowest-priced sell orders first (because those are the best deals for the buyer).
    - This process continues as long as the prices of the remaining passive orders are within the limit price of the aggressive order.
3. **Two Possible Outcomes:**
    
    - **Case 1: The Aggressive Order Is Fully Matched.**
        - If the quantity of the aggressive order is smaller than or equal to the available quantity of matching passive orders, the entire aggressive order is matched. For example:
            - A buyer aggressively submits an order to buy 100 shares at a limit price of $50.
            - The order book has sell orders at $48 (50 shares) and $49 (50 shares).
            - The aggressive buy order is fully matched, buying 50 shares at $48 and 50 shares at $49.
    - **Case 2: The Remaining Passive Orders Have Prices Worse Than the Limit Price of the Aggressive Order.**
        - If the remaining passive orders in the book are priced outside the limit price of the aggressive order, no further match can occur.
        - For instance:
            - A buyer aggressively submits an order to buy 100 shares at a limit price of $50.
            - The order book has sell orders at $48 (50 shares), $49 (30 shares), and $51 (20 shares).
            - The buy order matches with the 50 shares at $48 and 30 shares at $49.
            - However, the remaining 20 shares priced at $51 are higher than the buyer's limit of $50, so no match occurs for those shares. The unmatched part of the buy order (20 shares) remains unexecuted or gets cancelled if it was a market order.
4. **The Fundamental Rule:**
    
    - An order cannot be matched at a price worse than the limit price it was entered at. This ensures that traders are only matched at prices they are comfortable with.
    - For buyers, this means they will not pay more than their limit price.
    - For sellers, this means they will not sell for less than their limit price.

This process guarantees fairness and price priority in the order book, ensuring that orders are executed within the boundaries set by the traders who placed them.

## **Price-Time Priority** (FIFO)
It is a fundamental rule in most order-driven financial markets that governs how orders are matched in the order book. It ensures fairness by prioritizing orders based on two factors: **price** and, when prices are the same, **time of entry**. Here’s a detailed explanation:

---

### **1. Price Priority** (Primary Priority)

- **Definition**: Orders are prioritized based on their price, with better prices taking precedence.
    
    - **For Buy Orders**: Higher prices are better. A buy order with a higher price will be matched before one with a lower price.
    - **For Sell Orders**: Lower prices are better. A sell order with a lower price will be matched before one with a higher price.
- **Example**: Suppose the order book looks like this:
    
    - **Buy Side**:
        
        - $100 (50 shares)
        - $99 (30 shares)
    - **Sell Side**:
        
        - $101 (20 shares)
        - $102 (40 shares)
    - If a sell order arrives to sell 20 shares at $100, it will first match with the **buy order at $100**, because $100 is the best price for the seller (highest buy price available).
        

---

### **2. Time Priority** (Secondary Priority)

- **Definition**: If two or more orders are at the same price, the order that was entered first is given priority.
    
    - This ensures fairness by rewarding traders who submitted their orders earlier.
    - Orders at the same price are executed in the order they were entered into the book.
- **Example**: Suppose the order book contains the following buy orders:
    
    - **Buy Orders**:
        - $100 (50 shares, entered at 10:00 AM)
        - $100 (30 shares, entered at 10:01 AM)
    - If a sell order arrives to sell 60 shares at $100, the system will match:
        - First 50 shares with the order entered at **10:00 AM**.
        - The remaining 10 shares with the order entered at **10:01 AM**.

---

### **Why Price-Time Priority Matters**

1. **Fairness and Transparency**:
    
    - The system ensures that the best price is always executed first.
    - Traders who act earlier at the same price level are rewarded, encouraging quick decision-making.
2. **Efficient Markets**:
    
    - Encourages liquidity by incentivizing traders to post competitive prices to get their orders executed sooner.
    - Ensures that orders flow through the book systematically without arbitrary prioritization.
3. **Practical Implications**:
    
    - Traders submitting orders at competitive prices are more likely to get matched.
    - Traders with less competitive prices may need to wait or adjust their price to secure a match.