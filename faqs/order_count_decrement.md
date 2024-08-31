# Updates to the Spot Order Count Limit Rules

To reward efficient traders on the Spot market, we have upgraded the order count algorithm to decrement the used 10-second and 24-hour order counts when an order is first partially filled or fully filled.

When any order is filled for the first time (partially or fully), the order count will be decremented by 1 for the current 10-second and 24-hour order count windows. The net effect is that any (partially or fully) filled orders will not count towards the 10-second or 24-hour order limits.

## How it works in detail:
Order placement requests (e.g. `POST /api/v3/order`) return the `X-MBX-ORDER-COUNT-24H` header which indicates how many orders have been placed in the current 24-hour UTC time window.
Below is an example of orders being placed and the `X-MBX-ORDER-COUNT-24H` header value:


|Time| Action                       | X-MBX-ORDER-COUNT-24H|
|--- | ---                          | ----                 |
|T1  | Place LIMIT Order A          | 1 (LIMIT Order A counted) |
|T2  | Place LIMIT Order B          | 2 (LIMIT Order B counted) |
|T3  | Order A partially filled     | 1 (LIMIT Order A decremented) |
|T4  | Order A partially filled     | 1 (No decrement; already decremented) |
|T5  | Place LIMIT Order C          | 2 (LIMIT Order C counted) |
|T5  | Place LIMIT Order D          | 3 (LIMIT Order D counted) |
|T6  | Order C filled               | 2 (LIMIT Order C decremented) |
|T7  | Order B filled               | 1 (LIMIT Order B decremented) |
|T8  | Place MARKET Order E         | 2 (MARKET Order E counted*) (See below) |
|T9  | Place MARKET Order E (FILLS) | 1 (MARKET Order E decremented*) (See below) |
|T10  | Place MARKET Order F (No Fills) | 2 (MARKET Order F counted)  |

Note that this will also decrement the 10-second order count window as well; the example is using the 24-hour window for reference.

*  The decrement is an asynchronous process and there will be a short delay between an order being filled and the order count being decremented. Because of this, orders that immediately fill (such as a MARKET order that takes immediately) will return the order count **at the time of order placement** and the returned count **will not include the decrement**. Order decrements should occur within 1000ms.

## Which timezone is this 24-hour window working on?

UTC

## Which window will be decremented if an order is placed in one window and filled in another window?

At UTC 00:00:00 the 24-hour window will reset. If an order placed on the previous UTC day (e.g. UTC 23:59:00) had a fill on the current day (e.g. UTC 00:01:00), the order count will be decremented in the current window. The same logic applies to the 10-second windows.
# MARKET orders 
  
 **Disclaimer:** 
  
 * The commissions and prices used here are fictional and do not reflect the actual setup on the live exchange. 
 * The explanation pertains to the behavior on the Spot Exchange. 
  
 `MARKET` orders allow users to buy or sell an asset at the best available prices and liquidity until the order is fully filled or the order book's liquidity is exhausted. 
  
 If you do not have sufficient balance for the order to be completely filled, the order will be rejected. 
  
 If the exchange does not have enough liquidity to fill the order, your `MARKET` order will be partially filled to the extent of available liquidity, and the remaining quantity will expire. 
  
 If the order expires due to insufficient liquidity, the `status` field of the API response will be `EXPIRED`. 
  
 Additionally, if you monitor the User Data Stream, in the payload `ExecutionReport`, you will see that both the Execution Type (`x`) and Order Status (`X`) are marked as `EXPIRED`.  
  
 Please refer to the sample payload below. 
  
 **Sample API Response:** 
  
 ```json 
 { 
     "symbol": "LTCBNB", 
     "orderId": 32, 
     "orderListId": -1, 
     "clientOrderId": "8LGC97RRgNPVQk979VIhjt", 
     "transactTime": 1719467634105, 
     "price": "0.00000000", 
     "origQty": "6.00000000", 
     "executedQty": "4.00000000", 
     "cummulativeQuoteQty": "4.00000000", 
     "status": "EXPIRED", 
     "timeInForce": "GTC", 
     "type": "MARKET", 
     "side": "BUY", 
     "workingTime": 1719467634105, 
     "fills": [ 
         { 
             "price": "1.00000000", 
             "qty": "2.00000000", 
             "commission": "0.00100000", 
             "commissionAsset": "BNB", 
             "tradeId": 7 
         }, 
         { 
             "price": "1.00000000", 
             "qty": "2.00000000", 
             "commission": "0.00100000", 
             "commissionAsset": "BNB", 
             "tradeId": 8 
         } 
     ], 
     "selfTradePreventionMode": "NONE" 
 } 
 ``` 
  
 **Sample User Data Stream Payload:** 
  
 ```json 
 { 
   "e": "executionReport", 
   "E": 1719467634107, 
   "s": "LTCBNB", 
   "c": "8LGC97RRgNPVQk979VIhjt", 
   "S": "BUY", 
   "o": "MARKET", 
   "f": "GTC", 
   "q": "6.00000000", 
   "p": "0.00000000", 
   "P": "0.00000000", 
   "F": "0.00000000", 
   "g": -1, 
   "C": "", 
   "x": "NEW", 
   "X": "NEW", 
   "r": "NONE", 
   "i": 32, 
   "l": "0.00000000", 
   "z": "0.00000000", 
   "L": "0.00000000", 
   "n": "0", 
   "N": null, 
   "T": 1719467634105, 
   "t": -1, 
   "I": 62, 
   "w": true, 
   "m": false, 
   "M": false, 
   "O": 1719467634105, 
   "Z": "0.00000000", 
   "Y": "0.00000000", 
   "Q": "0.00000000", 
   "W": 1719467634105, 
   "V": "NONE" 
 } 
 { 
   "e": "executionReport", 
   "E": 1719467634107, 
   "s": "LTCBNB", 
   "c": "8LGC97RRgNPVQk979VIhjt", 
   "S": "BUY", 
   "o": "MARKET", 
   "f": "GTC", 
   "q": "6.00000000", 
   "p": "0.00000000", 
   "P": "0.00000000", 
   "F": "0.00000000", 
   "g": -1, 
   "C": "", 
   "x": "TRADE", 
   "X": "PARTIALLY_FILLED", 
   "r": "NONE", 
   "i": 32, 
   "l": "2.00000000", 
   "z": "2.00000000", 
   "L": "1.00000000", 
   "n": "0.00100000", 
   "N": "BNB", 
   "T": 1719467634105, 
   "t": 7, 
   "I": 63, 
   "w": false, 
   "m": false, 
   "M": true, 
   "O": 1719467634105, 
   "Z": "2.00000000", 
   "Y": "2.00000000", 
   "Q": "0.00000000", 
   "W": 1719467634105, 
   "V": "NONE" 
 } 
 { 
   "e": "executionReport", 
   "E": 1719467634107, 
   "s": "LTCBNB", 
   "c": "8LGC97RRgNPVQk979VIhjt", 
   "S": "BUY", 
   "o": "MARKET", 
   "f": "GTC", 
   "q": "6.00000000", 
   "p": "0.00000000", 
   "P": "0.00000000", 
   "F": "0.00000000", 
   "g": -1, 
   "C": "", 
   "x": "TRADE", 
   "X": "PARTIALLY_FILLED", 
   "r": "NONE", 
   "i": 32, 
   "l": "2.00000000", 
   "z": "4.00000000", 
   "L": "1.00000000", 
   "n": "0.00100000", 
   "N": "BNB", 
   "T": 1719467634105, 
   "t": 8, 
   "I": 65, 
   "w": false, 
   "m": false, 
   "M": true, 
   "O": 1719467634105, 
   "Z": "4.00000000", 
   "Y": "2.00000000", 
   "Q": "0.00000000", 
   "W": 1719467634105, 
   "V": "NONE" 
 } 
 { 
   "e": "executionReport", 
   "E": 1719467634107, 
   "s": "LTCBNB", 
   "c": "8LGC97RRgNPVQk979VIhjt", 
   "S": "BUY", 
   "o": "MARKET", 
   "f": "GTC", 
   "q": "6.00000000", 
   "p": "0.00000000", 
   "P": "0.00000000", 
   "F": "0.00000000", 
   "g": -1, 
   "C": "", 
   "x": "EXPIRED", 
   "X": "EXPIRED", 
   "r": "NONE", 
   "i": 32, 
   "l": "0.00000000", 
   "z": "4.00000000", 
   "L": "0.00000000", 
   "n": "0", 
   "N": null, 
   "T": 1719467634105, 
   "t": -1, 
   "I": 67, 
   "w": false, 
   "m": false, 
   "M": false, 
   "O": 1719467634105, 
   "Z": "4.00000000", 
   "Y": "0.00000000", 
   "Q": "0.00000000", 
   "W": 1719467634105, 
   "V": "NONE" 
 } 
 { 
   "e": "outboundAccountPosition", 
   "E": 1719467634107, 
   "u": 1719467634105, 
   "B": 
   [ 
     { 
       "a": "LTC", 
       "f": "3011.11745670", 
       "l": "0.00000000" 
     }, 
     { 
       "a": "BNB", 
       "f": "996.11865670", 
       "l": "0.00000000" 
     } 
   ] 
 } 
 ```