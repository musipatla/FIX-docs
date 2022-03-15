---
title: Locates FIX API Specification


toc_footers:
  - [Powered By Slate](https://github.com/slatedocs/slate)

search: true
---

# Introduction
Locates FIX API specification details the tags and values along with description of values for each field for processing locate requests and acceptances from OMS clients. Clear Street currently utilizes FIX 4.2 format.

# Configuring FIX Session
```
// Example Config
[DEFAULT]
ReconnectInterval=5
SenderCompID=LOCATESCLIENT
HeartBtInt=30
SocketAcceptPort=31234
TargetCompID=CLST
HeartBtInt=30
SocketConnectPort=31234
SocketConnectHost=100.0.0.0
BeginString=FIX.4.2

[SESSION]
StartTime=04:00:00
EndTime=22:00:00
```
A FIX session is defined as a unique combination of a `BeginString` (the FIX version number 4.2), a `SenderCompID` (OMS Client ID as defined by Clear Street), and a `TargetCompID`.

A FIX session has an active time period during which all data is transmitted. Clear Street currently accepts connections between 4 AM EST to 10 PM EST on all business days. Clear Street will act as an acceptor and OMS will be the initiator.

A `Logon` message (`Tag 35=A`) must be sent to Clear Street on every business day to indicate the begining of activity and a `Logout` message (`Tag 35=5`)must be sent to Clear Street indicating end of activity for that day. 

Clear Street also recommends exchange of heartbeats every 30 seconds by sending a message with `Tag 35=0`.

Clear Street expects every message sent to have a unique continuous sequence number as part of message with Tag 34=. If there are any gaps in sequence numbers, a sequence reset message will sent to OMS client with tag `34=4`.

All requests must either provide a `Symbol` (Tag 55) or both `IDSource` (Tag 22) and `SecurityID` (Tag 48) to clearly identify a security. `IDSource` (Tag 22) and `SecurityID` are marked as Conditionall Required for this purpose.

# Message Flows

Currently we support two different message flows.

## Preferred Message Flow
1. OMS will send a `Quote Request` message to locate securities
2. Clear Street responsds by sending a `Quote` message with the located securities information including Locate_ID in `Tag 117`.
3. OMS will send a `New Order` meessage by setting `Tag 117` with the Locate ID that Clear Street provided.
4. Clear Street responsds by sending a `Execution Report` message with status update such as accepted/exipred/etc  in `Tag 39`

## Other Message Flow
This flow will be decommissioned in the future once all OMSs migrate to the preferred message flow above.

1. OMS will send a `New Order` message to locate securities
2. Clear Street responsds by sending a `Execution Report` message with the located securities information including Locate_ID in `Tag 37`.
3. OMS will send a `New Order` meessage by setting `Tag 117` with the Locate ID with status code that Clear Street provided. To accept OMS need to send Locate_ID,1 for `Tag 117` and to reject OMS need to send Locate_ID,3 for `Tag 117`.
4. Clear Street responsds by sending a `Execution Report` message with status update such as accepted/exipred/etc in `Tag 39`

# Message List

| Message | Tag 35 | Comments | Direction w.r.t. CLST |
| --- | --- | --- | --- |
| New Order - Single | D | 1. Request to locate a single security <br/>2. Request to Accept/Reject a single locate | Incoming |
| New Order - List | E | 1. Request to locate multiple securities <br/>2. Request to Accept/Reject multiple locates | Incoming |
| Execution Report | 8 | 1. Offer response for a single locate request with locate id <br/>2. Status response for a single locate accept/reject message with locate id <br/>3. Multiple status responses for a list of locate accept/reject messages with locate ids | Outgoing |
| Reject | 3 | 1. Any validation/authentication errors on the Order requests | Outgoing |
| Quote Request | Required | 1. Request to locate a list of securities | Incoming |
| Quote | S | 1. Response for quote request. Multiple responses are send depending on the number of quotes requested. | Outgoing |

# New Order - Single (Request)

```
// Example 1: By Symbol
8=FIX.4.2\u00019=0054\u000135=D\u000149=OMSC\u000156=CLSTLOCT\u000134=1360\u000152=20220208-21:10:01\u000111=12345671\u000155=IBM\u000154=1\u000160=20220208-21:10:00.792\u000140=1\u0001109=ACCOUNTID\u000176=USRNM,PSWD\u000138=1000\u0001440=TRADERID\u00011=ACCOUNTID\u000158=THISISASINGLELOCATEREQUESTBYSYMBOL\u000110=106\u0001
```
```
// Example 2: By CUSIP
8=FIX.4.2\u00019=0054\u000135=D\u000149=OMSC\u000156=CLSTLOCT\u000134=1360\u000152=20220208-21:10:01\u000111=12345672\u000154=1\u000160=20220208-21:10:00.792\u000140=1\u0001109=ACCOUNTID\u000176=USRNM,PSWD\u000138=1000\u000122=1\u000148=459200101\u0001440=TRADERID\u00011=ACCOUNTID\u000158=THISISASINGLELOCATEREQUESTBYCUSIP\u000110=106\u0001
```

`New Order - Single` is a request sent by OMS to locate a single security. 

| Field Name | FIX Tag # | Possible Values | Comments | Example | Format | Length | Required |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MsgType | 35 | D | Message type | D | AlphaNumeric | 1 | Required |
| ClOrdID | 11 |  | Unique ID for the Order |  | AlphaNumeric | 25 | Required |
| Symbol | 55 |  | Security ticker | AAPL | Alpha |  | Required |
| TransactTime | 60 |  | Date+Time in UTC | 20220121-13:27:43.000 | YYYYMMDD-HH:mm:SS:sss | 21 | Required |
| ClientID | 109 |  | Firm ID/MPID | CLST | AlphaNumeric |  | Required |
| ExecBroker | 76 |  | Credential string: username,password | USRNM,PSWD | AlphaNumeric |  | Required |
| OrderQty | 38 |  | Number of share ordered | 1000 | Numeric |  | Required |
| Account | 1 |  | Account ID | ACCTID | AlphaNumeric |  | Required |
| HandInst | 21 | 1-Auto, Private<br/>2-Auto, Public<br/>3-Manual | Instructions for order handling on Broker trading floor | 1 | Numeric | 1 | Optional |
| Side | 54 | 1-Buy<br/>2-Sell<br/>3-Buy Minus<br/>4-Sell Plus<br/>5-Sell Short<br/>6-Sell Short Exempt<br/>7-Undisclosed<br/>8-Cross<br/>9-Cross Short | Side | 1 | Numeric | 1 | Optional |
| OrdType | 40 | 1-Market<br/>2-Limit<br/>3-Stop<br/>4-Stop Limit<br/>5-Market On Close<br/>6-With or Without<br/>7-Limit or Better<br/>8-Limit With or Without<br/>9-On Basis<br/>A-On Close<br/>B-Limit On Close<br/>C-Forex C<br/>D-Previously Quoted<br/>E-Previously Indicated<br/>F-Forext F<br/>G-Forex G<br/>H-Forex H<br/>I-Funari<br/>P-Pegged | Order Type | 1 | AlphaNumeric | 1 | Optional |
| ClearingAccount | 440 |  | Trader ID | TRDRID | AlphaNumeric |  | Optional |
| Text | 58 |  | Any comments by trader | comment | AlphaNumeric |  | Optional |
| IDSource | 22 | 1-CUSIP<br/>2-SEDOL<br/>4-ISIN<br/>8-Exchange Symbol | Security ID type  | 1 | Numeric | 1 | Conditionally Required |
| SecurityID | 48 |  | Security ID as per IDSource tag 22 | 037833100 | AlphaNumeric | 12 | Conditionally Required |


# New Order - List (Request)
```
8=FIX.4.2\u00019=0054\u000135=E\u000149=OMSC\u000156=CLSTLOCT\u000134=1360\u000152=20220208-21:10:01\u000166=10000001\u0001394=3\u000168=2\u000173=2\u000111=12345681\u000167=1\u000155=IBM\u000154=1\u000158=THISISALISTLOCATEREQUESTBYSYMBOL\u000160=20220208-21:10:00.792\u000140=1\u0001109=ACCOUNTID\u000176=USRNM,PSWD\u000138=1000\u0001440=TRADERID\u00011=ACCOUNTID\u000111=12345681\u000167=2\u000155=AAPL\u000154=1\u000158=THISISALISTLOCATEREQUESTBYSYMBOL\u000160=20220208-21:10:00.792\u000140=1\u0001109=ACCOUNTID\u000176=USRNM,PSWD\u000138=2000\u0001440=TRADERID\u00011=ACCOUNTID\u000110=106\u0001
```
`New Order - List` is a request sent by OMS to locate multiple securities. 

| Field Name | FIX Tag # | Possible Values | Comments | Example | Format | Length | Required | Repeatable |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| MsgType | 35 | E | Message type | E | AlphaNumeric | 1 | Required | No |
| ListID | 66 |  | Unique ID for the Order |  | AlphaNumeric |  | Required | No |
| BidType | 394 | 1-Non Disclosed<br/>2-Disclosed<br/>3-No Bidding Process | Code to identify the type of Bid request | 3 | Numeric | 1 | Required | No |
| TotNoOrders | 68 |  | Total number of list order entries across all messages |  | Numeric |  | Required | No |
| NoOrders | 73 |  | Number of orders in this message |  | Numeric |  | Required | No |
| ClOrdID | 11 |  | Unique ID for the Order. Must be the first in the repeatable list. |  | AlphaNumeric | 25 | Required | Yes |
| ListSeqNo | 67 |  | Order number within this list |  | Numeric |  | Required | Yes |
| Symbol | 55 |  | Security ticker | AAPL | Alpha |  | Required | Yes |
| Side | 54 | 1-Buy<br/>2-Sell<br/>3-Buy Minus<br/>4-Sell Plus<br/>5-Sell Short<br/>6-Sell Short Exempt<br/>7-Undisclosed<br/>8-Cross<br/>9-Cross Short | Side | 1 | Numeric | 1 | Optional | Yes |
| Text | 58 |  | Any comments by trader | comment | AlphaNumeric |  | Optional | Yes |
| TransactTime | 60 |  | Date+Time in UTC | 20220121-13:27:43.000 | YYYYMMDD-HH:mm:SS:sss | 21 | Required | Yes |
| OrdType | 40 | 1-Market<br/>2-Limit<br/>3-Stop<br/>4-Stop Limit<br/>5-Market On Close<br/>6-With or Without<br/>7-Limit or Better<br/>8-Limit With or Without<br/>9-On Basis<br/>A-On Close<br/>B-Limit On Close<br/>C-Forex C<br/>D-Previously Quoted<br/>E-Previously Indicated<br/>F-Forext F<br/>G-Forex G<br/>H-Forex H<br/>I-Funari<br/>P-Pegged | Order Type | 1 | AlphaNumeric | 1 | Optional | Yes |
| ClientID | 109 |  | Firm ID/MPID | CLST | AlphaNumeric |  | Required | Yes |
| ExecBroker | 76 |  | Credential string: username,password | USRNM,PSWD | AlphaNumeric |  | Required | Yes |
| OrderQty | 38 |  | Number of share ordered | 1000 | Numeric |  | Required | Yes |
| IDSource | 22 | 1-CUSIP<br/>2-SEDOL<br/>4-ISIN<br/>8-Exchange Symbol | Security ID type | 1 | Numeric | 1 | Conditionally Required | Yes |
| SecurityID | 48 |  | Security ID as per IDSource tag 22 | 037833100 | AlphaNumeric | 12 | Conditionally Required | Yes |
| ClearingAccount | 440 |  | Trader ID | TRDRID | AlphaNumeric |  | Optional | Yes |
| Account | 1 |  | Account ID | ACCTID | AlphaNumeric |  | Required | Yes |


# Execution Report (Request)
```
8=FIX.4.2\u00019=59\u000135=8\u000134=28512\u000149=CLSTLOCT\u000152=20220208-21:10:45.336\u000156=OMSC\u000137=200004\u000111=12345671\u000155=IBM\u000154=1\u000160=20220208-21:10:00.792\u0001109=ACCOUNTID\u000138=1000\u000158=THISISASINGLELOCATERESPONSEBYSYMBOL\u000117=200004\u000120=0\u0001150=B\u000139=B\u00011=ACCOUNTID\u0001151=0\u000114=1000\u00016=23.00\u000144=0.23\u000110=025\u0001
```
`Execution Report` (One Response Message for each item in List of Order Request) is a response sent by Clear Street with a locate. 
There will be one response message for each item if the response is for a list of multiple securities request via `New Order - List`

| Field Name | FIX Tag # | Possible Values | Comments | Example | Format | Length | Required |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MsgType | 35 | 8 | Message type | 8 | AlphaNumeric | 1 | Required |
| OrderID | 37 |  | Unique Locate ID |  | AlphaNumeric |  | Required |
| ClOrdID | 11 |  | ClOrdID from the New Order message |  | AlphaNumeric | 25 | Required |
| Symbol | 55 |  | Security ticker | AAPL | Alpha |  | Required |
| Side | 54 | 1-Buy<br/>2-Sell<br/>3-Buy Minus<br/>4-Sell Plus<br/>5-Sell Short<br/>6-Sell Short Exempt<br/>7-Undisclosed<br/>8-Cross<br/>9-Cross Short | Side | 1 | Numeric | 1 | Required |
| TransactTime | 60 |  | Date+Time in UTC | 20220121-13:27:43.000 | YYYYMMDD-HH:mm:SS:sss | 21 | Required |
| ClientID | 109 |  | Same as Account ID | CLST | AlphaNumeric |  | Required |
| OrderQty | 38 |  | Number of share requested | 1000 | Numeric |  | Required |
| IDSource | 22 | 1-CUSIP<br/>2-SEDOL<br/>4-ISIN<br/>8-Exchange Symbol | Security ID type | 1 | Numeric | 1 | Conditionally Required |
| SecurityID | 48 |  | Security ID as per IDSource tag 22 | 037833100 | AlphaNumeric | 12 | Conditionally Required |
| Text | 58 |  | Any comments by locating broker or indicating partial fills or part of original order | comment | AlphaNumeric |  | Optional |
| ExecID | 17 |  | Unique ID for each message |  | Numeric |  | Required |
| ExecTransType | 20 | 0-New<br/>1-Cancel<br/>2-Correct<br/>3-Status | Transaction type | 0 | Numeric | 1 | Required |
| ExecType | 150 | 0-New<br/>1-Partial Fill<br/>2-Filled<br/>3-Done for Day<br/>8-Rejected<br/>B-Calculated/Offered<br/>C-Expired | Type of Execution report | B | AlphaNumeric | 1 | Required |
| OrdStatus | 39 | 0-New<br/>1-Partial Fill<br/>2-Filled<br/>3-Done for Day<br/>8-Rejected<br/>B-Calculated/Offered<br/>C-Expired | Current status of Order | B | AlphaNumeric | 1 | Required |
| Account | 1 |  | Account ID from Order | ACCTID | AlphaNumeric |  | Required |
| LeavesQty | 151 |  | Amount of shares open for further execution |  | Numeric |  | Required |
| CumQty | 14 |  | Currently executed shares |  | Numeric |  | Required |
| AvgPx | 6 |  | Calculated average price |  | Numeric |  | Required |
| Price | 44 |  | Price |  | Numeric |  | Required |


# New Order-Single (Accept/Reject) 

```
// Example 1: Accept
8=FIX.4.2\u00019=0054\u000135=D\u000149=OMSC\u000156=CLSTLOCT\u000134=1360\u000152=20220208-21:10:01\u000111=12345681\u000160=20220208-21:10:00.792\u0001109=ACCOUNTID\u000176=USRNM,PSWD\u00011=ACCOUNTID\u0001117=100011,1\u000110=106\u0001
```
```
// Example 2: Accept
8=FIX.4.2\u00019=0054\u000135=D\u000149=OMSC\u000156=CLSTLOCT\u000134=1360\u000152=20220208-21:10:01\u000111=12345681\u000160=20220208-21:10:00.792\u0001109=ACCOUNTID\u000176=USRNM,PSWD\u00011=ACCOUNTID\u0001117=100011\u000110=106\u0001
```
```
// Example 3: Reject
8=FIX.4.2\u00019=0054\u000135=D\u000149=OMSC\u000156=CLSTLOCT\u000134=1360\u000152=20220208-21:10:01\u000111=12345681\u000160=20220208-21:10:00.792\u0001109=ACCOUNTID\u000176=USRNM,PSWD\u00011=ACCOUNTID\u0001117=100011,2\u000110=106\u0001
```

`New Order-Single (Accept/Reject)`  is a request sent by OMS to accept/reject a locate that was offered by Clear Street. 

| Field Name | FIX Tag # | Possible Values | Comments | Example | Format | Length | Required |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MsgType | 35 | D | Message type | D | AlphaNumeric | 1 | Required |
| ClOrdID | 11 |  | Unique ID for the Order |  | AlphaNumeric | 25 | Optional |
| HandInst | 21 | 1-Auto, Private<br/>2-Auto, Public<br/>3-Manual | Instructions for order handling on Broker trading floor | 1 | Numeric | 1 | Optional |
| Symbol | 55 |  | Security ticker | AAPL | Alpha |  | Optional |
| Side | 54 | 1-Buy<br/>2-Sell<br/>3-Buy Minus<br/>4-Sell Plus<br/>5-Sell Short<br/>6-Sell Short Exempt<br/>7-Undisclosed<br/>8-Cross<br/>9-Cross Short | Side | 1 | Numeric | 1 | Optional |
| TransactTime | 60 |  | Date+Time in UTC | 20220121-13:27:43.000 | YYYYMMDD-HH:mm:SS:sss | 21 | Required |
| OrdType | 40 | 1-Market<br/>2-Limit<br/>3-Stop<br/>4-Stop Limit<br/>5-Market On Close<br/>6-With or Without<br/>7-Limit or Better<br/>8-Limit With or Without<br/>9-On Basis<br/>A-On Close<br/>B-Limit On Close<br/>C-Forex C<br/>D-Previously Quoted<br/>E-Previously Indicated<br/>F-Forext F<br/>G-Forex G<br/>H-Forex H<br/>I-Funari<br/>P-Pegged | Order Type | 1 | AlphaNumeric | 1 | Optional |
| ClientID | 109 |  | Firm ID/MPID | CLST | AlphaNumeric |  | Required |
| ExecBroker | 76 |  | Credential string: username,password | USRNM,PSWD | AlphaNumeric |  | Required |
| OrderQty | 38 |  | Number of share ordered | 1000 | Numeric |  | Optional |
| IDSource | 22 | 1-CUSIP<br/>2-SEDOL<br/>4-ISIN<br/>8-Exchange Symbol | Security ID type | 1 | Numeric | 1 | Conditionally Required |
| SecurityID | 48 |  | Security ID as per IDSource tag 22 | 037833100 | AlphaNumeric | 12 | Conditionally Required |
| ClearingAccount | 440 |  | Trader ID | TRDRID | AlphaNumeric |  | Optional |
| Account | 1 |  | Account ID | ACCTID | AlphaNumeric |  | Optional |
| QuoteID | 117 | Locate_ID,1-Accept Locate<br/>Locate_ID,2-Reject Locate | Locate ID, Accept/Reject | 123456,1 | AlphaNumeric |  | Required |


# New Order-List (Accept/Reject)
```
8=FIX.4.2\u00019=0054\u000135=E\u000149=OMSC\u000156=CLSTLOCT\u000134=1360\u000152=20220208-21:10:01\u000166=10000002\u0001394=3\u000168=2\u000173=2\u000111=12345681\u000167=1\u000155=IBM\u000154=1\u000158=THISISALISTLOCATEACCEPTBYSYMBOL\u000160=20220208-21:10:00.792\u000140=1\u0001109=ACCOUNTID\u000176=USRNM,PSWD\u000138=1000\u0001440=TRADERID\u00011=ACCOUNTID\u0001117=200005,1\u000111=12345681\u000167=2\u000155=AAPL\u000154=1\u000158=THISISALISTLOCATEACCEPTBYSYMBOL\u000160=20220208-21:10:00.792\u000140=1\u0001109=ACCOUNTID\u000176=USRNM,PSWD\u000138=2000\u0001440=TRADERID\u00011=ACCOUNTID\u0001117=200006,1\u000110=106\u0001
```

`New Order-List (Accept/Reject)` is a request sent by OMS to accept/reject multiple locates that were offered by Clear Street.

| Field Name | FIX Tag # | Possible Values | Comments | Example | Format | Length | Required | Repeatable |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| MsgType | 35 | E | Message type | E | AlphaNumeric | 1 | Required | No |
| ListID | 66 |  | Unique ID for the Order |  | AlphaNumeric |  | Required | No |
| BidType | 394 | 1-Non Disclosed<br/>2-Disclosed<br/>3-No Bidding Process | Code to identify the type of Bid request | 3 | Numeric | 1 | Required | No |
| TotNoOrders | 68 |  | Total number of list order entries across all messages |  | Numeric |  | Required | No |
| NoOrders | 73 |  | Number of orders in this message |  | Numeric |  | Required | No |
| ClOrdID | 11 |  | Unique ID for the Order. Must be the first in the repeatable list. |  | AlphaNumeric | 25 | Required | Yes |
| ListSeqNo | 67 |  | Order number within this list |  | Numeric |  | Required | Yes |
| Symbol | 55 |  | Security ticker | AAPL | Alpha |  | Required | Yes |
| Side | 54 | 1-Buy<br/>2-Sell<br/>3-Buy Minus<br/>4-Sell Plus<br/>5-Sell Short<br/>6-Sell Short Exempt<br/>7-Undisclosed<br/>8-Cross<br/>9-Cross Short | Side | 1 | Numeric | 1 | Optional | Yes |
| Text | 58 |  | Any comments by trader | comment | AlphaNumeric |  | Optional | Yes |
| TransactTime | 60 |  | Date+Time in UTC | 20220121-13:27:43.000 | YYYYMMDD-HH:mm:SS:sss | 21 | Required | Yes |
| OrdType | 40 | 1-Market<br/>2-Limit<br/>3-Stop<br/>4-Stop Limit<br/>5-Market On Close<br/>6-With or Without<br/>7-Limit or Better<br/>8-Limit With or Without<br/>9-On Basis<br/>A-On Close<br/>B-Limit On Close<br/>C-Forex C<br/>D-Previously Quoted<br/>E-Previously Indicated<br/>F-Forext F<br/>G-Forex G<br/>H-Forex H<br/>I-Funari<br/>P-Pegged | Order Type | 1 | AlphaNumeric | 1 | Optional | Yes |
| ClientID | 109 |  | Firm ID/MPID | CLST | AlphaNumeric |  | Required | Yes |
| ExecBroker | 76 |  | Credential string: username,password | USRNM,PSWD | AlphaNumeric |  | Required | Yes |
| OrderQty | 38 |  | Number of share ordered | 1000 | Numeric |  | Required | Yes |
| IDSource | 22 | 1-CUSIP<br/>2-SEDOL<br/>4-ISIN<br/>8-Exchange Symbol | Security ID type | 1 | Numeric | 1 | Conditionally Required | Yes |
| SecurityID | 48 |  | Security ID as per IDSource tag 22 | 037833100 | AlphaNumeric | 12 | Conditionally Required | Yes |
| ClearingAccount | 440 |  | Trader ID | TRDRID | AlphaNumeric |  | Optional | Yes |
| Account | 1 |  | Account ID | ACCTID | AlphaNumeric |  | Required | Yes |
| QuoteID | 117 | Locate_ID,1-Accept Locate<br/>Locate_ID,2-Reject Locate | Locate ID, Accept/Reject | 123456,1 | AlphaNumeric |  | Required | Yes |


# Execution Report (Accept/Reject)
```
8=FIX.4.2\u00019=59\u000135=8\u000134=28512\u000149=CLSTLOCT\u000152=20220208-21:10:45.336\u000156=OMSC\u000137=200005\u000111=12345671\u000155=IBM\u000154=1\u000160=20220208-21:10:00.792\u0001109=ACCOUNTID\u000138=1000\u000158=THISISASINGLELOCATEACCEPTRESPONSEBYSYMBOL\u000117=200005\u000120=0\u0001150=2\u000139=2\u00011=ACCOUNTID\u0001151=0\u000114=1000\u00016=23.00\u000144=0.23\u000110=025\u0001
```
`Execution Report` (One Response Message for each item in List of Order Accept/Reject) is a response sent by Clear Street confirming the accept/reject of a locate. 
There will be one response message for each item if the response is for a list of multiple securities request via `New Order - List`

| Field Name | FIX Tag # | Possible Values | Comments | Example | Format | Length | Required |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MsgType | 35 | 8 | Message type | 8 | AlphaNumeric | 1 | Required |
| OrderID | 37 |  | Unique Locate ID |  | AlphaNumeric |  | Required |
| ClOrdID | 11 |  | ClOrdID from the New Order message |  | AlphaNumeric | 25 | Required |
| Symbol | 55 |  | Security ticker | AAPL | Alpha |  | Required |
| Side | 54 | 1-Buy<br/>2-Sell<br/>3-Buy Minus<br/>4-Sell Plus<br/>5-Sell Short<br/>6-Sell Short Exempt<br/>7-Undisclosed<br/>8-Cross<br/>9-Cross Short | Side | 1 | Numeric | 1 | Required |
| TransactTime | 60 |  | Date+Time in UTC | 20220121-13:27:43.000 | YYYYMMDD-HH:mm:SS:sss | 21 | Required |
| ClientID | 109 |  | Same as Account ID | CLST | AlphaNumeric |  | Required |
| OrderQty | 38 |  | Number of share requested | 1000 | Numeric |  | Required |
| IDSource | 22 | 1-CUSIP<br/>2-SEDOL<br/>4-ISIN<br/>8-Exchange Symbol | Security ID type | 1 | Numeric | 1 | Conditionally Required |
| SecurityID | 48 |  | Security ID as per IDSource tag 22 | 037833100 | AlphaNumeric | 12 | Conditionally Required |
| Text | 58 |  | Any comments by locating broker or indicating partial fills or part of original order | comment | AlphaNumeric |  | Optional |
| ExecID | 17 |  | Unique ID for each message |  | Numeric |  | Required |
| ExecTransType | 20 | 0-New<br/>1-Cancel<br/>2-Correct<br/>3-Status | Transaction type | 0 | Numeric | 1 | Required |
| ExecType | 150 | 0-New<br/>1-Partial Fill<br/>2-Filled<br/>3-Done for Day<br/>8-Rejected<br/>B-Calculated/Offered<br/>C-Expired | Type of Execution report | 2 | AlphaNumeric | 1 | Required |
| OrdStatus | 39 | 0-New<br/>1-Partial Fill<br/>2-Filled<br/>3-Done for Day<br/>8-Rejected<br/>B-Calculated/Offered<br/>C-Expired | Current status of Order | 2 | AlphaNumeric | 1 | Required |
| Account | 1 |  | Account ID from Order | ACCTID | AlphaNumeric |  | Required |
| LeavesQty | 151 |  | Amount of shares open for further execution |  | Numeric |  | Required |
| CumQty | 14 |  | Currently executed shares | 1000 | Numeric |  | Required |
| AvgPx | 6 |  | Calculated average price | 0.23 | Numeric |  | Required |


# Reject
```
8=FIX.4.2\u00019=59\u000135=3\u000134=28512\u000149=CLSTLOCT\u000152=20220208-21:10:45.336\u000156=OMSC\u000145=12345691\u000158=Order rejected. Missing Security ID\u000110=025\u0001
```
`Reject` is sent by Clear Street where there are any issues with the request messages. 

| Field Name | FIX Tag # | Possible Values | Comments | Example | Format | Length | Required |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MsgType | 35 | 3 | Message type | 3 | AlphaNumeric | 1 | Required |
| RefSeqNum | 45 |  | Reference message sequence number | 123456789 | Numeric |  | Required |
| Text | 58 |  | Reason for reject | Order rejected. Missing field ClOrdID | AlphaNumeric |  | Required |


# Quote Request
```
8=FIX.4.2\u00019=0054\u000135=R\u000149=OMSC\u000156=CLSTLOCT\u000134=1360\u000152=20220208-21:10:01\u0001131=123456783\u0001146=2\u0001115=ACCOUNTID\u0001109=ACCOUNTID\u000155=IBM\u000138=1000\u000154=1\u000155=AAPL\u000138=2000\u000154=1\u000110=106\u0001
```
`Quote Request` is a request sent by OMS for locating multiple securities. 

| Field Name | FIX Tag # | Possible Values | Comments | Example | Format | Length | Required | Repeatable |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Message Type | 35 | Required | Message type | Required | AlphaNumeric | 1 | Required | No |
| NoRelatedSym | 146 |  | Number of related symbols in this order | 1 | Numeric |  | Required | No |
| QuoteReqID | 131 |  | Unique ID for the Quote | A12345bc | AlphaNumeric |  | Required | No |
| OnBehalfOfCompID | 115 |  | On behalf of company ID<br/> Same as Firm ID/MPID | CLST | AlphaNumeric |  | Required | No |
| ClientID | 109 |  | Firm ID/MPID | CLST | AlphaNumeric |  | Required | No |
| Symbol | 55 |  | Security ticker | AAPL | Alpha |  | Required | Yes |
| IDSource | 22 | 1-CUSIP<br/>2-SEDOL<br/>4-ISIN<br/>8-Exchange Symbol | Security ID type | 1 | Numeric | 1 | Conditionally Required | Yes |
| SecurityID | 48 |  | Security ID as per IDSource tag 22 | 037833100 | AlphaNumeric | 12 | Conditionally Required | Yes |
| OrderQty | 38 |  | Number of shares for which quote is being requested | 1000 | Numeric |  | Required | Yes |
| Side | 54 | 1-Buy | Side of Quote | 1 | Numeric | 1 | Optional | Yes |


# Quote
```
8=FIX.4.2\u00019=59\u000135=S\u000134=28512\u000149=CLSTLOCT\u000152=20220208-21:10:45.336\u000156=OMSC\u0001131=123456783\u0001115=ACCOUNTID\u0001109=ACCOUNTID\u0001117=100001\u000155=IBM\u0001135=1000\u0001133=0.23\u000110=025\u0001
```
`Quote` is a response sent by Clear Street as a reply to Quote Request message with Locate ID.
There will be one response message for each of the securities requested as part of `Quote Request`. 

| Field Name | FIX Tag # | Possible Values | Comments | Example | Format | Length | Required |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MsgType | 35 | S | Message type | S | AlphaNumeric | 1 | Required |
| QuoteReqID | 131 |  | Unique ID from the Quote Request ReferenceId |  | AlphaNumeric |  | Required |
| OnBehalfOfCompID | 115 |  | On behalf of company ID<br/> Filled with same information from Quote Request message account_id | CLST | AlphaNumeric |  | Optional |
| ClientID | 109 |  | Firm ID/MPID | CLST | AlphaNumeric |  | Required |
| QuoteID | 117 |  | Unique ID for this quote locate_id |  | AlphaNumeric |  | Required |
| Symbol | 55 |  | Security ticker | AAPL | Alpha |  | Required |
| IDSource | 22 | 1-CUSIP<br/>2-SEDOL<br/>4-ISIN<br/>8-Exchange Symbol | Security ID type | 1 | Numeric | 1 | Conditionally Required |
| SecurityID | 48 |  | Security ID as per IDSource tag 22 | 037833100 | AlphaNumeric | 12 | Conditionally Required |
| OfferSize | 135 |  | Number of shares that can be offered<br/>If availability is less the OrderQty then availability<br/>If availability is more than OrderQty then OrderQty<br/>1 if no OrderQty is specified and available<br/>0 if no OrderQty is specified and not available | 1000 | Numeric |  | Required |
| OfferPx | 133 |  | Offer price for the security requested | 0.23 | Numeric |  | Required |
