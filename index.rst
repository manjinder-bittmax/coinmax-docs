.. Bittmax documentation master file, created by
   sphinx-quickstart on Mon Sep 24 15:24:57 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

###################################
Welcome to Bittmax's documentation!
###################################

.. toctree::
   :maxdepth: 2

*********
WebSocket
*********

You can connect to our WebSocket to get updates on your orders and receive market data events in real time. This reduces the amount of data transfer and latency as you no longer have to poll our servers for updates.

To connect with WebSocket send the following request::

   GET wss://bittmax.live/ws
   Connection: Upgrade
   Upgrade: websocket

=============
Subscriptions
=============

Once you are connected, You will need to subscribe to the events published by WebSocket.

.. note:: All communications with WebSocket are in JSON format.

Subscriptions request format::

   {
      "action": "subscribe",
      "channels": [{
         "name": "ohlc",
         "productIds": ["BTC-GUSD"]
      },
      {
         "name": "trades"
      }],
      "productIds": ["ETH-GUSD"]
   }

.. attention:: In the above structure, ohlc channel will subscribe to BTC-GUSD and ETH-GUSD both, whereas trades will only subscribe to ETH-GUSD

Subscriptions reply format::

   {
      "Type": "subscriptions",
      "subscribe": "ok",
      "subscriptions": [
         {
            "name": "ohlc",
            "productIds": ["BTC-GUSD", "ETH-GUSD"]
         },
         {
            "name": "trades",
            "productIds": ["ETH-GUSD"]
         }
      ]
   }

.. hint:: Every message you will receive from WebSocket will include a **Type** field, You can use it to decide what actions to perform

**Subscribe to events**

   To Subscibe to an event, Send a message to WebSocket with ``action: subscribe`` and the channel and product id you want to subscribe for.
   A complete list of :ref:`market_channels`, :ref:`authorized_channels` and :ref:`supported_product_ids` is listed below.

**Unsubscribe from events**

   Similary, to Unsubscribe from an event, repeat the message you sent during *subscribe* and change from ``action: subscribe`` to ``action: unsubscribe``

.. _market_channels:

--------------------
Market Data Channels
--------------------

   Following channels are available to listen for market data.

   .. list-table:: Market Data Channels
      :widths: 15 15 15
      :header-rows: 1

      * - Data
        - Channel Name
        - Supported Product Ids
      * - OHLC
        - ohlc
        - All supported Ids
      * - 24H Changes
        - ticker
        - \* *Star*
      * - Market trades
        - tradeQuote
        - All supported Ids
      * - Depth Snapshot
        - quoteFullSnapshot
        - All supported Ids
      * - Depth Incremental updates
        - quoteIncremental
        - All supported Ids

.. _authorized_channels:

-------------------------------
Authorized Client Data Channels
-------------------------------

   .. important:: Before subscribing to any authorized channel, You must ``register`` your Authorization token, see :ref:`ws-registration`.

   Following channels are available to listen for real time updates.

   .. list-table:: Authorized Channels
      :widths: 15 15 15
      :header-rows: 1

      * - Data
        - Channel Name
        - Supported Product Ids
      * - Order Update
        - orderUpdate
        - All supported Ids
      * - Asset Deposits
        - deposit
        - \* *Star*
      * - Asset Withdrawals
        - withdrawal
        - \* *Star*

.. _supported_product_ids:

---------------------
Supported Product Ids
---------------------

.. list-table::
   :widths: 30
   :header-rows: 1

   * - Product Id
   * - BTC-GUSD
   * - ETH-GUSD


.. _ws-registration:

============
Registration
============

   Registration message format::

      {
         "action" : "register",
         "token": "<Your Bearer token here>"
      }

   Registration reply format::

     {
         "Type": "registration",
         "status": "Registered|Token Invalid"
     }

===============
Data Structures
===============

-----------
Market data
-----------

^^^^
OHLC
^^^^

 ::

    {
      "Type": "ohlc",
      "Candles":
      [
         {
            "Start": Epoch,
            "Interval": Interval in seconds,
            "O": string,
            "H": string,
            "L": string,
            "C": string,
            "V": string
          }...
       ],
      "Symbol": string
    }

.. important:: Candles array will include candles with various time intervals, for e.g. 300, 900, 3600, 14400, 86400

^^^^^^^^^^^^^^^^^^^^^^^^
24 Hour Changes (Ticker)
^^^^^^^^^^^^^^^^^^^^^^^^

   ::

    {
     "Type": "ticker",
     "Tickers":
     [
      {
         "Symbol": "string",
         "PercentChange": string,
         "High": string,
         "Low": string,
         "Ltp": string,
         "Volume": string,
         "Timestamp": Epoch
       }...
      ]
    }

^^^^^^^^^^^^^
Market Trades
^^^^^^^^^^^^^

 ::

  {
   "Type": "tradeQuote",
   "Symbol": string,
   "Taker": string,
   "Ltq": string,
   "Ltp": string,
   "Timestamp": string
  }

 .. note:: Possbile values for Taker are ``maker`` and ``taker``

^^^^^^^^^^
Depth Book
^^^^^^^^^^

   Full Snapshot::

      {
         "Type": "quoteFullSnapshot",
         "Symbol": string,
         "NoOfBuyLevels": Number,
         "Buys": {
            "Price": {
                  "AggrQty": string,
                  "OrderCount": Number
               },...
         },
         "NoOfSellLevels": Number,
         "Sells": {
            "Price": {
                  "AggrQty": string,
                  "OrderCount": Number
               },...
         },
         "SeqNo": Number
      }

   .. note:: Buys and Sells are maps with Price as Key and {AggrQty and OrderCount} Object as value

   Incremental Update::

      {
         "Type": "quoteIncremental",
         "Symbol": string,
         "SeqNo": Number,
         "Side": string,
         "Price": string,
         "AggrQty": string,
         "OrderCount": Number,
         "Action": Number
      }


   .. list-table:: Actions map
      :widths: 15 15
      :header-rows: 1

      * - Action
        - Description
      * - 0
        - Insert
      * - 1
        - Update
      * - 2
        - Delete






   Depth book updates as well as snapshot both provide a ``SeqNo`` which you can use to make sure you do not miss any update. If you miss an update you can request a full snapshot by sending the below request to WebSocket.

   ::

     {
       "action": "dataFetch",
        "channels": [
          {
            "name": "quoteFullSnapshot"
           }
          ],
        "productIds": ["<Your product Id>"]
     }

----------------------
Authorized Client Data
----------------------

.. _CashComponent:

^^^^^^^^^^^^^^
Cash Component
^^^^^^^^^^^^^^

 ::

   {
      "Asset": string,
      "AvailableUnits": string,
      "PendingUnits": string
   }

^^^^^^^^^^^^
Order Update
^^^^^^^^^^^^

 .. parsed-literal::
     {
         "Type": "orderUpdate",
         "Symbol": string,
         "OrderStatus": string,
         "OrderType": string,
         "OrderValidity": string,
         "Side": string,
         "ExecutionId": string,
         "ExchangeOrderId": string,
         "Qty": string,
         "Price": string,
         "PassiveUnits": string,
         "FilledQty": string,
         "LastFillQty": string,
         "LastFillPrice": string,
         "Timestamp": string,
         "Commission": string,
         "Credited": :ref:`CashComponent`,
         "Debited": :ref:`CashComponent`,
         "Error": string
     }

 .. list-table:: Possbile values for OrderStatus::
   :widths:15 15
   :header-rows: 1

   * - Status
   - Description
   * - CANCELLED
   - Your order was cancelled, Check ``Error`` for more details
   * - REJECTED
   - Order was rejected, Check ``Error`` for more details
   * - ACCEPTED
   - Your order is accepted by Exchange
   * - PARTIAL_FILLED
   - Your order was filled partially, ``LastFillQty`` and ``LastFillPrice`` will contain the Quantity Filled and at what Price for this trade
   * - FILLED
   - Your order is completely filled
   * - CANCEL_REJECTED
   - Your request for cancelling your order was rejected, Check ``Error`` for more details

 **Important notes**
    * You may receive ``PARTIAL_FILLED`` events multiple times
    * ``Error`` field will not be available for ``PARTIAL_FILLED`` and ``FILLED`` orders
    * Credited and Debited :ref:`CashComponent` may be nil in some events
    * ``ExecutionId`` will be only available for ``PARTIAL_FILLED`` and ``FILLED`` orders

 .. hint:: ``ExecutionId`` + ``ExchangeOrderId`` = ``Unique Trade Identifier``

^^^^^^^^^^^^^
Asset Deposit
^^^^^^^^^^^^^

.. parsed-literal::

      {
         "Type": "deposit",
         "Asset": string,
         "Amount": string,
         "TxStatus": string,
         "TxId": string,
         "NodeTxId": string,
         "Timestamp": string,
         "Credited": :ref:`CashComponent`,
         "ErrorCode": Number
      }


.. hint:: You can use ``NodeTxId`` to view the transaction on block expolorer.



^^^^^^^^^^^^^^^^
Asset Withdrawal
^^^^^^^^^^^^^^^^^

.. parsed-literal::

      {
        "Type": "withdrawal",
      	"Asset": string,
      	"Amount": string,
      	"ApprovedAmount": string,
      	"Fee": string,
      	"TxStatus": string,
      	"TxId": string,
      	"ReferenceId": string,
      	"NodeTxId": string,
      	"Timestamp": string,
      	"Recipient": string,
      	"Debited": :ref:`CashComponent`,
      	"ErrorCode": Number
      }


.. hint:: You can use ``NodeTxId`` to view the transaction on block expolorer.

.. note:: Use the ``ReferenceId`` to uniquely identify withdrawal request.
