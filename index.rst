.. Coinmax documentation master file, created by
   sphinx-quickstart on Mon Sep 24 15:24:57 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

###################################
Coinmax API Documentation
###################################

.. toctree::
   :maxdepth: 2

.. _api_credentials:


*******************
Rest API
*******************
You need to generate your API credentials before placing orders via API. Please follow the following steps for the same.

====================
Generate Credentials
====================

1. Go to coinmax.com.au and login. If you don't have an account, please sign up.
2. Once you log in, on the top right corner, click on the the dropdown and click "Settings".
3. On the Settings page, go to "API Credentials".
4. Click “Generate”.
5. Note down the Client Key and Secret somewhere safe, as it will only be displayed once.

.. _place_order:

=============
Placing Order
=============
1. Create the order request with intended parameters. This will be the body of your HTTP request. Below are the details of each supported parameter.

   Order Structure::

      {
          "qty" : "0.0001",
          "price" : "100",
          "side" : "BUY", //Valid options are "BUY" and "SELL"
          "symbol" : "BTC-AUD", Complete list of supported products can be requested from https://coinmax.com.au/api/clientConfig/ under products key
          "type" : "LIMIT", // Supported parameters are "LIMIT" and "STOP LIMIT"
          "triggerPrice" : "78.99" // Only applicable if type is "STOP LIMIT",
          "validity": "GOOD TILL CANCEL", Supported parameters are "GOOD TILL CANCEL", "IMMEDIATE OR CANCEL" and "FILL OR KILL",
          "timestamp": 1538046192974 //Current time in milliseconds
      }

2. Sign your order request using sha256 and your client secret. Set the hex value of your signature in HTTP header "X-API-SIGNATURE".

3. Set your API key in HTTP header "X-API-KEY".

4. Place the request on URL https://coinmax.com.au/api/api-client/order

   Sample NodeJS program::

       const crypto = require('crypto');
       let time = new Date().getTime();
       const order = {
         "symbol": "BTC-AUD",
         "qty": "1",
         "price": "806",
         "side": "BUY",
         "type": "LIMIT",
         "triggerPrice": "",
         "validity": "GOOD TILL CANCEL",
         "timestamp": time
       };
       const CLIENT_KEY = "a385061780f7c9da5f1d1ad53ac644e7";
       const CLIENT_SECRET = "b9a03c3c32de9f2691a6309ba77a5a189b6720ff3a9c2b23e0af0ad7384438ec";

       var sign = crypto.createHmac('sha256', CLIENT_SECRET).update(JSON.stringify(order)).digest(
         'hex')

       var request = require("request");

       var options = {
         url: 'https://coinmax.com.au/api/api-client/order',
         headers: {
           'X-API-KEY': CLIENT_KEY,
           'X-API-SIGNATURE': sign
         },
         method: 'post',
         json: true,
         body: order
       };

       request(options, (error, res, body) => {
         if (error) {
           console.warn(error);
         }
         console.log(body);
       })

============
Cancel Order
============

Follow the same steps as :ref:`place_order`, but alter the body with following parameters:

Cancel Order Structure::

   {
     "orderId": "26001", //Order id to cancel
     "symbol": "ETH-AUD", // Symbol
     "timestamp": time
   }

===========
Client Data
===========

1. You can fetch client data, [funds, orders, trades, deposits, withdrawals] using our API.
2. These APIs use GET HTTP method
3. You can use query params in URL, timestamp is a required parameter to prevent replay attacks.
4. Similar to POST APIs, GET APIs of Coinmax also require signature, but the steps vary.
5. You need to sign the API url instead of request body, By API URL we mean the part ahead of the base url, For e.g, In "https://coinmax.com.au/api/api-client/trades?timestamp=12313443&page=0&symbol=ETH-AUD" "https://coinmax.com.au/api/api-client/" is the base URL and "/trades?timestamp=1540472319692&page=0&symbol=ETH-AUD" is the API URL.
6. List of supported APIs is as below
 * https://coinmax.com.au/api/api-client/orders/
 * https://coinmax.com.au/api/api-client/trades/
 * https://coinmax.com.au/api/api-client/funds
 * https://coinmax.com.au/api/api-client/deposits
 * https://coinmax.com.au/api/api-client/withdrawals

Please note that orders and trades API also support filteration on "symbol" and also supports pagination too (page=0 and so on)

Sample NodeJS program::

   const crypto = require('crypto');
   let time = new Date().getTime();

   const CLIENT_KEY = "a64cdc31716649d4c8fd79b89ab965d8";
   const CLIENT_SECRET = "d37e9c4f137601f2c59b796a033a99a35e20a6757e754f30d00cff9c438b0cac";

   let baseURL = "https://coinmax.com.au/api/api-client"
   let apiURL = `/trades?timestamp=${time}&page=0&symbol=ETH-AUD`;
   let reqURl = `${baseURL}${apiURL}`;

   var sign = crypto.createHmac('sha256', CLIENT_SECRET).update(apiURL).digest(
     'hex')

   console.log(sign);

   var request = require("request");

   var options = {
     url: reqURl,
     headers: {
       'X-API-KEY': CLIENT_KEY,
       'X-API-SIGNATURE': sign
     },
     method: 'get',
     json: true
   };

   request(options, (error, res, body) => {
     if (error) {
       console.warn(error);
     }
     console.log(body);
   })


*********
WebSocket
*********

You can connect to our WebSocket to get updates on your orders and receive market data events in real time. This reduces the amount of data transfer and latency as you no longer have to poll our servers for updates.

To connect with WebSocket send the following request::

   GET wss://coinmax.com.au/ws?X-API-KEY={API_KEY}&timestamp=1544020774432&X-API-SIGNATURE={SIGNATURE}
   Connection: Upgrade
   Upgrade: websocket

.. note:: API_KEY can be obtained by :ref:`api_credentials` and SIGNATURE is the hex value of sha256 of ``API key + timestamp`` with Client Secret

Sample NodeJS program::

    var ws = require('ws');
    var crypto = require('crypto');
    let time = new Date().getTime();

    const CLIENT_KEY = "dec5b6dad847f157a51735d34fd09e79";
    const CLIENT_SECRET = "1f9bcfaa4542676463f953224fdbf779e92fbc9898aa56146797a152097182fd";

    var sign = crypto.createHmac('sha256', CLIENT_SECRET).update(CLIENT_KEY + time).digest(
          'hex')

    const coinmaxWebsocket = new ws(`wss://coinmax.com.au/ws?X-API-KEY=${CLIENT_KEY}&timestamp=${time}&X-API-SIGNATURE=${sign}`);
    coinmaxWebsocket.on('open', function open() {
          coinmaxWebsocket.send(JSON.stringify({
                  "action": "subscribe",
                  "channels": [{
                          "name": "ohlc",
                          "productIds": ["ETH-AUD"]
                  },
                  {
                          "name": "trades"
                  }],
                  "productIds": ["ZEC-AUD"]
          }));
    });

    coinmaxWebsocket.on('message', function incoming(data) {
          console.log(data);
    });

=============
Subscriptions
=============

Once you are connected, you will need to subscribe to the events published by WebSocket.

.. note:: All communications with WebSocket are in JSON format.

Subscriptions request format::

   {
      "action": "subscribe",
      "channels": [{
         "name": "ohlc",
         "productIds": ["ETH-AUD"]
      },
      {
         "name": "trades"
      }],
      "productIds": ["ZEC-AUD"]
   }

.. attention:: In the above structure, ``ohlc`` channel will subscribe to ETH-AUD and ZEC-AUD both, whereas trades will only subscribe to ZEC-AUD

Subscriptions reply format::

   {
      "Type": "subscriptions",
      "subscribe": "ok",
      "subscriptions": [
         {
            "name": "ohlc",
            "productIds": ["ETH-AUD", "ZEC-AUD"]
         },
         {
            "name": "trades",
            "productIds": ["ZEC-AUD"]
         }
      ]
   }

.. hint:: Every message you will receive from WebSocket will include a **Type** field, You can use it to decide what actions to perform

**Subscribe to events**

   To Subscibe to an event, send a message to WebSocket with ``action: subscribe`` and the channel and product id you want to subscribe for.
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
   * - ETH-AUD
   * - ZEC-AUD


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


**********
Fees
**********
We calculate fees as a fraction of the notional value of each trade (i.e., price × amount). Any fees will be applied at the time an order is placed. For partially filled orders, only the executed portion is subject to trading fees.
Fees we charge for Maker trades is 0.10% while those for Taker trades is 0.15%. If any other users have joined by using your referral link, you gain 0.10% of the fees that we charge them on their trades. 

**********
Disclaimer
**********
Please note that this is a beta version of the Coinmax API. The
API is provided on an
“as is” and “as available” basis. Coinmax does not give any warranties,
whether express or implied, as to the suitability or usability of the
API, its software or any of its content. Coinmax will not be liable for any loss, whether such loss is direct,
indirect, special or consequential, suffered by any party as a result
of their use of the Coinmax API. Any use of the API is done at the
user’s own risk and the user will be solely responsible for any
damage to any computer system or loss of data that results from such
activities. Should you encounter any bugs, glitches, lack of functionality or
other problems on the website, please let us know immediately so we
can rectify these accordingly. Your help in this regard is greatly
appreciated.