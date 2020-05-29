---
layout: default
---

# Stock Trading System

**This is a mini-trading system simulation written in Java, whose components (market data and order simulators, FIX acceptor and initiator, back-end for market data and filled/rejected orders) interact by queues and topics over ActiveMQ.**

*** 
## Overview

In a real-world trading environment, there are a handful of independent systems that must communicate with each other in order to carry out the day-to-day activities of a stock market. An exchange will have an automated system in place to verify and accept orders via some form of communication, commonly following the FIX protocol, and execute those orders according to a matching engine. They are also responsible for keeping a book of orders for assets at any given time. Any client wishing to make trades will have an Order Management System (OMS) that acts as a platform for managing the flow of orders it routes to/order confirmations it receives from the exchange. A 3rd party stock market data system pools market data from stock exchanges to be viewed by clients such as stockbrokers and stock traders and may also store up-to-date logs of historic market data with published orders.

***
## High-Level Architecture

![Architecture](assets/images/TradingMachineArchitecture.jpg)

***

## Main Implemented Components

This project was built on Ubuntu and Eclipse, using the following technologies: Java, QuickFIX/J (FIX 5.0), Maven, ActiveMQ, MongoDB and MySql. 

The following are the set of implemented applications that run concurrently in order to simulate the interactions that take place in a real stock market:

* **Market Data Feed**
  - Randomly builds ask/ bid prices and ask/ bid sizes for a given symbol and then publishes them onto a topic every X seconds.
* **Orders Feed**
  - Randomly builds market, limit and stop orders and then publishes them onto a queue every X seconds.
* **FIX Acceptor**
  - Listens on the market data and orders queues and provides order execution by a matching engine. It can deal with market, limit and stop orders. Market orders are always filled unless they're FOK, specifically, a market price will always be available from the market data while the quantity might not match the bid/ ask size. Limit and stop orders will be filled only if their limit/ stop price and quantity match the market data. Limit orders are subject to FOK too.
* **FIX Initiator**
  - Acts as an OMS, routing orders to the acceptor. It listens on the orders queue and forwards them to the FIX acceptor. If the acceptor replies with filled orders, then it publishes them on a topic.
* **Orders Back-end Store**
  - Subscribes to the orders topic. It stores executed and rejected orders to the MongoDB and MySql back-ends. The scripts to set up the MySQL database, tables and stored procedure, are provided in TradingServices/src/main/resources.
* **Trade Monitor UI**
  - Subscribes to the orders and market data topics to show live execution/ rejection/ market data pluse the ones stored in the MongoDB repository. Furthermore, in the orders tab, it shows various order statistics.

***

## Trade Monitor UI

UI for filled and rejected orders along with statistics:

![Orders](assets/images/TradeMonitorUI.jpg)

* * * 

UI for on-going market data:

![MarketData](assets/images/MarketDataUI.jpg)

***

## Running the Various Components

- ServicesRunner starts the market data, orders feed and (executed + rejected) orders subscriber. Furthermore, it starts a statistics service which builds various statistics on aggregated data.
- TradingMachineServer starts the FIX acceptor.
- TradingMachineOrderRouter starts the OMS and FIX initiator.
- TradeMonitorUI starts the monitor UI.

