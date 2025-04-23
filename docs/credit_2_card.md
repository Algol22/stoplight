---
id: credit_2_card
title: CREDIT2CARD
stoplight-id: x15pdyu7q0ug8
---

Version 1.2<br/>
Released: 2022/09/12


# Introduction
---
This document describes integration procedures and POST-CREDIT2CARD protocol usage for e-commerce merchants.

POST-CREDIT2CARD protocol implements money transfers transactions between merchant's account and credit card (Card Credit or Account-to-Card payment) with using specific API.

## Integration Process
---

### Client Registration

Before you get an account to access Payment Platform, you must provide the following data to the Payment Platform administrator.

| **Data** | **Description** |
| --- | --- |
| IP List | List of your IP addresses, from which requests to Payment Platform will be sent.|
| CallbackURL | URL which will be receiving the notifications of the processing results of your request to Payment Platform.<br/> The length shouldn't be more than 255 symbols. |
| Contact e-mail | Client's contact email |

With all Payment Platform POST requests at Callback URL the Client must return the string "OK" if he/she successfully received data or return "ERROR".

You should get the following information from administrator to begin working with the Payment Platform.

| **Parameter** | **Description** |
| --- | --- |
| CLIENT_KEY | Unique key to identify the account in Payment Platform (used as request parameter) |
| PASSWORD | Password for Client authentication in Payment Platform (used for calculating hash parameter) |
| PAYMENT_URL | URL to request the Payment Platform |

### PaymentPlatformInteraction

For the transaction you must send the **server to server HTTPS POST** request to the Payment Platform URL (PAYMENT_URL). In response Payment Platform will return the JSON ([http://json.org/)](http://json.org/))encoded string.

### List of possible actions in Payment Platform

When you make request to Payment Platform, you need to specify action that needs to be done. Possible actions are:

| **Action** | **Description** |
| --- | --- |
| CREDIT2CARD | Creates CREDIT2CARD transaction |
| GET_TRANS_STATUS | Gets status of transaction in Payment Platform |
| GET_TRANS_DETAILS | Gets status of transaction with using Order ID in the Client's system |

### List of possible transaction results and statuses

Result – value that system returns on request. Possible results are:

| **Result** | **Description** |
| --- | --- |
| SUCCESS | Action was successfully completed in Payment Platform |
| DECLINED | Result of unsuccessful action in Payment Platform |
| ERROR | Request has errors and was not validated by Payment Platform |

Status – actual status of transaction in Payment Platform. Possible statuses are:

| **Status** | **Description** |
| --- | --- |
| SETTLED | Successful transaction |
| DECLINED | Not successful transaction |

## Transactions requests

### CREDIT2CARD request

If you want to send a payment for the specific sub-account (channel), you need to use ```channel_id``` that specified in your Payment Platform account settings.

This request is sent by POST in the background (eg, through PHP CURL).

### Request Parameters

| **Parameter** | **Description** | **Values** | **Required** |
| --- | --- | --- | :---: |
| ```action``` | Action type | CREDIT2CARD | + |
| ```client_key``` | Unique client key (CLIENT\_KEY) || + |
| ```channel_id``` | Payment channel (Sub-account) | String up to 16 characters | - |
| ```order_id``` | Transaction ID in the Clients system | String up to 255 characters | + |
| ``` order_amount``` | The amount of the transaction | Numbers in the formXXXX.XX (without leading zeros) | + |
| ```order_currency``` | Currency | 3-letter code | + |
| ```order_description``` | Description of the transaction (product name) | String up to 1024 characters | + |
| ```card_number``` | Credit Card Number | | + |
| ```hash``` | Special signature to validate your request to Payment Platform | see Appendix A, Formula 1 | + |

<details>
	<summary markdown="span">Example Request</summary>

```
curl –d "action=CREDIT2CARD&client_key=c2b8fb04-110f-11ea-bcd3-0242c0a85004&
 channel_id=test&order_id=123456789&order_amount=1.03&order_currency=USD&
 order_description=wine&card_number=4917111111111111&
 hash=a1a6de416405ada72bb47a49176471dc"[https://test.apiurl.com](https://test.apiurl.com/) -k
```
</details>

### Response Parameters

You will get JSON encoded string with transaction result.

**Successful response**

| **Parameter** | **Description** |
| --- | --- |
| ```action``` | CREDIT2CARD |
| ```result``` | SUCCESS |
| ```status``` | SETTLED |
| ```order_id``` | Transaction ID in the Client's system |
| ```trans_id``` | Transaction ID in the Payment Platform |
| ```trans_date``` | Transaction date in the Payment Platform |
| ```descriptor``` | This is a string which the owner of the credit card will see in the statement from the bank. In most cases, this is the Customers support web-site. |

<details>
	<summary markdown="span">Response Example (Successful result)</summary>

```
{
 "action": "CREDIT2CARD",
 "result": "SUCCESS",
 "status": "SETTLED",
 "order_id": "1613117050",
 "trans_id": "e5098d62-6d08-11eb-9da3-0242ac120013",
 "trans_date": "2021-02-12 08:04:15"
 }
```
</details>

**Unsuccessful response**

| **Parameter** | **Description** |
| --- | --- |
| ```action``` | CREDIT2CARD |
| ```result``` | DECLINED |
| ```status``` | DECLINED |
| ```order_id``` | Transaction ID in the Client's system |
| ```trans_id``` | Transaction ID in the Payment Platform |
| ```trans_date``` | Transaction date in the Payment Platform |
| ```decline_reason``` | The reason why the transaction was declined |

<details>
	<summary markdown="span">Response Example (Unsuccessful result)</summary>

```
{
 "action": "CREDIT2CARD",
 "result": "DECLINED",
 "status": "DECLINED",
 "order_id": "1613117050",
 "trans_id": "e5098d62-6d08-11eb-9da3-0242ac120013",
 "trans_date": "2021-02-12 08:04:15",
 "decline_reason": "Declined by processing"
 }
```
</details>

### Callback parameters

**Successful response**

| **Parameter** | **Description** |
| --- | --- |
| ```action``` | CREDIT2CARD |
| ```result``` | SUCCESS |
| ```status``` | SETTLED |
| ```order_id``` | Transaction ID in the Client's system |
| ```trans_id``` | Transaction ID in the Payment Platform |
| ```trans_date``` | Date of CREDIT2CARD action |
| ```hash``` | Special signature to validatecallback. See Appendix A, Formula 2 |

<details>
	<summary markdown="span">Callback Example (Successful result)</summary>

```
"action=CREDIT2CARD&result=SUCCESS&status=SETTLED&order_id=123456789&
trans_id=1d152122-6c86-11eb-8a49-0242ac120013&
hash=84dc0713fa38f18edb85da7aa94eca2e&trans_date=2021-02-11+16%3A28%3A04"
```
</details>

**Unsuccessful response**

| **Parameter** | **Description** |
| --- | --- |
| ```action``` | SALE |
| ```result``` | DECLINED |
| ```status``` | DECLINED |
| ```order_id``` | Transaction ID in the Client's system |
| ```trans_id``` | Transaction ID in the Payment Platform |
| ```trans_date``` | Date of CREDIT2CARD action |
| ```decline_reason``` | Reason of transaction decline.It shows for the transactions with the "DECLINED" status |
| ```hash``` | Special signature to validatecallback. See Appendix A, Formula 2|

<details>
	<summary markdown="span">Callback Example (Unsuccessful result)</summary>

```
"action=CREDIT2CARD&result=SUCCESS&status=SETTLED&order_id=123456789&
trans_id=1d152122-6c86-11eb-8a49-0242ac120013&hash=84dc0713fa38f18edb85da7aa94eca2e
&trans_date=2021-02-11+16%3A28%3A04"
```
</details>

## GET_TRANS_STATUS request

Gets order status from Payment Platform. This request is sent by POST in the background (e.g., through PHP CURL).

### Request parameters

| **Parameter** | **Description** | **Values** | **Required** |
| --- | --- | --- | --- |
| ```action``` | GET_TRANS_STATUS | GET_TRANS_STATUS | + |
| ```client_key``` | Unique client key (CLIENT_KEY) | | + |
| ```trans_id``` | Transaction ID in the Payment Platform | String up to 255 characters | + |
| ```hash``` | Special signature to validate your request to Payment Platform | \*\*(Appendix A) | + |

<details>
	<summary markdown="span">Request Example</summary>

```
curl –d "action= GET_TRANS_STATUS&
client_key=c2b8fb04-110f-11ea-bcd3-0242c0a85004&
trans_id=1d152122-6c86-11eb-8a49-0242ac120013&hash=a1a6de416405ada72bb47a49176471dc"
[https://test.apiurl.com](https://test.apiurl.com/) -k
```
</details>

### Response parameters

| **Parameter** | **Description** |
| --- | --- |
| ```action``` | GET_TRANS_STATUS |
| ```result``` | SUCCESS |
| ```status``` | DECLINED/SETTLED |
| ```order_id``` | Transaction ID in the Client's system |
| ```trans_id``` | Transaction ID in the Payment Platform |
| ```decline_reason``` | Reason of transaction decline.It shows for the transactions with the "DECLINED" status |

<details>
	<summary markdown="span">Response Example (Successful result)</summary>

```
{
 "action": "GET_TRANS_STATUS",
 "result": "SUCCESS",
 "status": "SETTLED",
 "order_id": "1613117050",
 "trans_id": "e5098d62-6d08-11eb-9da3-0242ac120013",
 "decline_reason": "Declined by processing"
 }

```
</details>

## GET_TRANS_DETAILS request

Gets all history of transactions by the order. This request is sent by POST in the background (e.g., through PHP CURL).

### Request parameters

| **Parameter** | **Description** | **Values** | **Required** |
| --- | --- | --- | --- |
| ```action``` | GET_TRANS_DETAILS | GET_TRANS_ DETAILS | + |
| ```client_key``` | Unique client key (CLIENT_KEY) | | + |
| ```trans_id``` | Transaction ID in the Payment Platform | String up to 255 characters | + |
| ```hash``` | Special signature to validate your request to Payment Platform | see Appendix A, Formula 2 | + |

<details>
	<summary markdown="span">Request Example</summary>

```
curl –d "action=GET_TRANS_DETAILS&
client_key=c2b8fb04-110f-11ea-bcd3-0242c0a85004&
trans_id=1d152122-6c86-11eb-8a49-0242ac120013&hash=a1a6de416405ada72bb47a49176471dc"
[https://test.apiurl.com](https://test.apiurl.com/) -k
```
</details>

### Response parameters

| **Parameter** | **Description** |
| --- | --- |
| ```action``` | GET\_TRANS\_ DETAILS |
| ```result``` | SUCCESS |
| ```status``` | DECLINED/SETTLED |
| ```order_id``` | Transaction ID in the Client's system |
| ```trans_id``` | Transaction ID in the Payment Platform |
| ```name``` | Cardholder name |
| ```amount``` | Order amount |
| ```currency``` | Currency |
| ```card``` | Card in the format XXXXXX\*\*\*\*XXXX |
| ```transactions```| Array of transactions with the parameters:<br/> - date<br/>- type (SALE / 3DS / AUTH / CAPTURE / CREDIT / CHARGEBACK / REVERSAL / REFUND)<br/>- status (SUCCESS / FAIL)<br/> - amount |
| ```decline_reason``` | Reason of transaction decline.It shows for the transactions with the "DECLINED" status |

<details>
	<summary markdown="span">Response Example (Unsuccessful result)</summary>

```
{
 "action": "GET_TRANS_STATUS",
 "result": "DECLINED",
 "status": "SETTLED",
 "order_id": "1613117050",
 "trans_id": "e5098d62-6d08-11eb-9da3-0242ac120013",
 "name": "John Doe",
 "amount": "1.03",
 "currency": "USD",
 "card": "491748******7107",
 "transactions": [
        {
        "type": "CREDIT",
        "status": "fail",
        "date": "2021-02-11 16:35:20",
        "amount": "1.03"
        }
                ]
 "decline_reason": "Declined by processing"
 }
```
</details>

<details>
	<summary markdown="span">Transactions Example</summary>

```
{
    "transactions": [
        {
            "date": "2012-01-01 01:10:25",
            "type": "AUTH",
            "status": "success",
            "amount": "1.95"
        },
        {
            "date": "2012-01-01 01:11:30",
            "type": "CAPTURE",
            "status": "success",
            "amount": "1.95"
        },
        {
            "date": "2012-02-06 10:25:06",
            "type": "REFUND",
            "status": "success",
            "amount": "1.95"
        }
    ]
}
```
</details>

## Errors

In case error you get synchronous response from Payment Platform:

| **Parameter** | **Description** |
| --- | --- |
| ```result``` | ERROR |
| ```error_message``` | Error message |

The list of the error codes is shown below.

| **Code** | **Description** |
| --- | --- |
| 204002 | Enabled merchant mappings or MIDs not found. |
| 204005 | Payment action not supported. |
| 204006 | Payment system/brand not supported. |
| 204007 | Day MID limit is not set or exceeded. |
| 204008 | Day Merchant mapping limit is not set or exceeded. |
| 204011 | Payment system/brand not found. |
| 204012 | Payment currency not found. |
| 204013 | Payment action not found. |
| 204014 | Month MID limit is exceeded. |
| 204015 | Week Merchant mapping limit is exceeded. |
| 208001 | Payment not found. |
| 400 | Duplicate request. |
| 400 | Previous payment not completed. |

## Appendix A

Hash - is signature rule used either to validate your requests to payment platform or to validate callback from payment platform to your system. It must be md5 encoded string calculated by rules below:

**Formula 1:**

```hash``` is calculated by the formula:

**md5(strtoupper(PASSWORD.strrev(substr(card_number,0,6).substr(card_number,-4))))***

if ```card_token``` is specified hash is calculated by the formula:

**md5(strtoupper(PASSWORD. strrev(card\_token)))**

**Formula 2:**

```hash``` is calculated by the formula:

**md5(strtoupper(PASSWORD.trans_id.strrev(substr(card_number,0,6).substr(card_number,-4))))**