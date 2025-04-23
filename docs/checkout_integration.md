---
id: checkout_integration
title: CHECKOUT INTEGRATION
stoplight-id: s7r91z3clvnfv
---

# Checkout Integration
Version: 6.6.2<br/>
Released: 2025/02/26<br/>

## Introduction
---

This document describes the Checkout integration procedures between Checkout service and the website for e-commerce merchants.<br/>
To make your integration easier, please download collection and try it out:<br/>
[<font color=''><ins>Postman Collection</ins></font>](https://identity.getpostman.com/accounts?continue=https%3A%2F%2Fgo.postman.co%2Fnetwork%2Fimport%3Fcollection%3D7445020-069ec96c-fc25-4f5f-b255-26dcecfe6d8e-TVKFyFni%26referrer%3Dhttps%253A%252F%252Fdocumenter.getpostman.com%252Fview%252F7445020%252FTVKFyFni%2523intro%26versionTag%3Dlatest%26traceId%3Dundefined)<br/>

## General Information
---

Checkout service is a fast and easy way to create a secure payment page. It allows collecting and submitting payments and sending them for processing.

To use the Checkout service on the site you have to perform integration. Checkout integration provides a set of APIs that allow customizing payment processing for the business. These protocols implement acquiring payments (purchases) using specific API interaction with the merchant websites.

The API requires request data as json string data and responds also with json string data.

⚠️ **Pay attention** <br/>
> Checkout protocol requires requests to be sent using JSON formatting with the following content type:<br/>
Header:
Content-Type: application/json

### Checkout process

---

Checkout payment flow is shown below.
![](/img/checkout_payment_flow.png)
When a Customer wants to make a purchase on your site the following happens:

1. Customer places an order and initiates payment on the site.
1. Site confirms the order and sends the payment processing request to the Checkout system with information about the order, payment and hash.
1. Checkout system validates the request and sends to the site the response with the redirect link.
1. The site redirects the Customer on the Checkout page by redirect link.
1. Customer selects the payment method, enters the payment data and confirms the payment. The payment method will be specifying automatically If only one method is available.
1. The payment processes at Payment Gateway.
1. Payment Gateway sends a callback to the site with the payment result.
1. The payment result is shown to the Customer.

The payment could be declined in case of invalid data detection.

### Protocol Mapping

It is necessary to check the existence of the protocol mapping before using the Checkout integration.
Merchants can’t make payments if the CHECKOUT protocol is not mapped.

### Customer return after payment

After completing the payment, the payer will be returned to the URL specified in the Authentication request in the "success_url" and "cancel_url" parameters.
If you need to get additional parameters in the return URLs, you should configure it in the Protocol Mappings modules by enabling the "Return parameters" attribute.
In that case, "success_url" and "cancel_url" will contain the following data:

| **Parameter** |**Description**|
| -------- | ---- |
| ```payment_id``` | Payment ID <br /> **Example:** <font color='Grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font> |
| ```trans_id``` | Transaction ID <br /> **Example:** <font color='Grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font> |
| ```order_id``` | Order ID <br /> **Example:** <font color='Grey'>order-1234</font> |
| ```hash``` | Special signature, used to validate data. Addition in Signature section.<br /> Must be SHA1 of MD5 encoded string (uppercased): payment_public_id + order.number + order.amount + order.currency + order.description + merchant.pass |

⚠️ **Pay attention** <br/>

> _that for "cancel_url", the payer's return to the merchant with parameters is possible only after the decline has happened and the payer has closed the payment form (not the browser tab). If the payer closes the payment form without initiating the payment (pressing the PAY button), the redirection to "cancel_url" occurs without passing additional parameters._


### DMS mode
If you need to use Checkout integration to work with payments in DMS-mode (two-step payments), you should set the corresponding MID configuration in the admin panel. Specify DMS-mode for the MID attribute.
In this case, the payment request will be processed as an authorization stage for DMS-mode. The capture requests will be generated and gathered in the queue. You could monitor and manage the capture request queue via the admin panel as well.
For detailed information, please contact your administrator.

### Checkout page description

---

Checkout page is shown to the Customer after a payment initiation. There are the fields to enter the payment data.

### Fields and Validation

The list of the fields on the Checkout page depends on the request parameters and the specified payment method. 

Your customers will not see the fields if the acquirer does not need the information that is transmitted in them. For example, if an alternative payment method is specified, the card data is not displayed on the Checkout page. 

As well, pay attention to the conditional fields such as e-mail or billing address. If the e-mail and billing address (data object) parameters are specified in the request, the Checkout page will not contain them.

Additional fields can also be displayed if a payment method is selected that requests additional data from the Customer.

The Checkout page has the fields validation. In case of the invalid data the error message will be shown and the field will be highlighted.

The list of the general fields and possible errors on the Checkout page is below:

| **Fields** |**Type**| **Limitations**| **Error**|
| -------- | :----: | -------------- |---|
| ```Card number```    |                                 _Integral_                                 | Lun algorithm, length 14-19 numbers                        | Invalid card number                                              |
| ```Expirу Date```    |                                   _Date_                                   | 2-2 numbers (in the format mm-yy),<br />after today's date | The expiration date of card is expired<br /> and not valid.      |
| ```Security code```  |                                 _Integral_                                 | Up to 4 characters                                         | Invalid security code                                            |
| ```Name on card```   |                                  _String_                                  | Up to 32 characters | The name on card field <br />Must contain at least 2 words: <font color='Grey'>first name and last name.</font> Allowed special characters: hyphens, apostrophes, diacritics    |
| ```Country```        |                                   _List_                                   | 2-letters code                                             | Country is required.<br />Please enter a valid Country           |
| ```State/Region``` | _String_<br /> _List_ - for USA, <br />Canada, Australia,<br />Japan, India |                                                            |                                                                  |
| ```City```           |                                  _String_                                  | Up to 32 characters                                        | City is required.<br />Please enter a valid City                 |
| ```Address line```   |                                  _String_                                  | Up to 32 characters                                        | Address line is required.<br />Please enter a valid Address line |
| ```Zip code```       |                                  _String_                                  | Up to 32 characters                                        | Zip code is required.<br />Please enter a valid Zip code         |
| ```Phone number```   |                                  _String_                                  | Up to 32 characters                                        | Phone number is required.<br />Please enter a valid Phone number |

### Pre-routing

To make payment method choice easier for the Customer, pre-routing is provided on the Checkout.

The functionality allows you to set up matches in the admin panel via the Custom routing module, which will determine the list of payment methods suitable for the current payment.

This way, you can restrict your Customers from randomly choosing a payment method that is not available in their region, for example. This will help you to increase the number of successful payments and reduce the risk of declined transactions.

Note that if the Authentication request contains a list of the payment methods in the ```methods``` array, then the pre-routing configurations will be ignored.

### Card data tokenization

For regular customers, we have made the payment page even more convenient and simple.

You can save the customer's card data so that they can reuse it for future payments.

To do this, you need to send the ```req_token = true``` parameter in the Authentication request. And then, in the callback, you will receive a card token. 

Use the token when sending the next Authentication request and your client will see anonymized card details on the payment form, which will greatly simplify the payment process.

### Web information

Checkout service gathers information about browser, which the Customer uses.

When the Customer is on the Checkout page, the service gets the data about the Customer's OS, browser, and browser language. That information is sent to the acquirer in some cases.

### iFrame option

You can use iFrame option to show Checkout page on your domain. All you need is just to past in the code redirect url received in the response to the Authentication request.

Please follow the example:

```
<iframe src=redirect_url height="600" width="300"></iframe>
```

Note that screen size can be adjusted according to your requirements.

*Important:* cross-domain requests are prohibited by security policy of our service.

⚠️ **Pay attention** <br/>

> _By adding our Checkout Payment Page to the iframe on your site, ensure you ALLOW ACCESS TO COOKIE STORAGE._<br/>
_This is required to avoid issues when redirecting the payer back to your site, such as after 3DS authentication. If you prohibit the storage of third-party data in your cookies, you will prevent the identification of the payer after returning from 3DS. This can cause interruptions in the payment session._<br/>

### Page Customization

The administration service provides the ability to stylize the Checkout page in accordance with the preferences of the merchant. The action is available for the authorized users only. To customize the payment page you need:

1. Click the “Configuration → Branding” item on the menu bar. The settings are displayed in the work area.
1. Select a page template to make changes.
1. Change the settings:
   1. login icon selection;
   1. color selection for the following items:
      1. heading;
      1. background;
      1. buttons;
1. Click the “Submit” button to apply the settings. The selected settings will be displayed to the Customers on the Checkout page.

## Authentication request parameters
---

The merchant submits an authorization request and as a result of successful response receives the `redirect_url` - link on the Checkout page.<br />
```
/api/v1/session
```
The authentication performs when the payment is initiated. 

You need to send an authentication POST request in JSON format on Checkout form URL (CHECKOUT_URL) to start checkout process for the Customers. As a result of the authentication request you will receive the following:

- An authentication session is created with a unique identifier (Session ID). The session expires after one hour.
- A link is generated to redirect to the Checkout page: one link corresponds to one payment. The link becomes invalid after a successful payment. The authentication request parameters are below.<br/>

If DMS-mode (two-stages payment) is available, after successful authentication it is necessary to capture. The capture would be performed automatically according to the settings or you can do it manually in the admin portal.


### Request Parameters

|    **Parameter**    | **Type** |**Mandatory, Limitations** | **Description** |
| ------------------- | :------: | ----------------------- | --------------- |
|```merchant_key```|_String_|Required|Key for Merchant identification<br />**Example:** <font color='grey'>xxxxx-xxxxx-xxxxx</font>|
|```operation```|_String_|Required|Defines a payment transaction<br />**Possible values:** <font color='grey'>purchase, debit, transfer</font> <br/> Send the “purchase” value so that the payment page for sale initiation will be shown to the customer.<br/>Send the “debit” value so that the payment page for debit initiation will be shown to the customer.<br/>Send the “transfer” value so that the payment page for transfer initiation will be shown to the customer.<br/>**Example:** <font color='grey'>purchase</font>|
|```methods```|_Array_|Optional|An array of payment methods. Limits the available methods on the Checkout page (the list of the possible values in the Payment methods section). In the case of parameter absence, the pre-routing rules are applied. If pre-routing rules are not configured, all available payment methods are displayed.<br />**Condition:** <font color='grey'>for purchase operation only</font><br />For purchase and debit operations.<br/>**Example:** <font color='grey'>card, paypal, googlepay</font> |
| ```channel_id``` | _String_ | Optional<br/>max: 16 |This parameter is used to direct payments to a specific sub-account (channel).<br /> If the channel is configured for Merchant Mapping, the system matches the value with the corresponding ```channel_id``` value in the request to route the payment. <br /><br />Note: The channel must correspond to one of the payment ```methods``` (brands) listed in the ```methods``` array. If the methods array is empty, only the ```channel_id``` will affect the selection of the payment method (Merchant Mapping).|
|```session_expiry```| _Integer_ | Optional<br />Values from 1 to 720 min<br />|Session expiration time in minutes.<br />Default value = 60.<br />Could not be zero.<br />**Example:** <font color='grey'>60</font> |
|```success_url```|_String_|Required<br/>Valid URL<br/>max: 1024|URL to redirect the Customer in case of the successful payment<br />**Example:** <font color='Blue'>https://example.domain.com/success</font><br />The return of additional parameters can be configured for success_url. See the “Customer return after payment” section for details.|
|```cancel_url```|_String_|Optional <br/>Valid URL<br/>min: 0 max: 1024 | URL to return Customer in case of a payment cancellation (“Close” button on the Checkout page). The logic of redirection on “cancel_url” could be configured in the admin panel (Configuration -> Protocol Mappings section): use the “Maximum count declines” field to set the payment failed attempts quantity before redirection. For example, if the field value is set to "1", then after the first declined attempt, a payer will be redirected to "cancel_url." <br />**Example:** <font color='grey'>https://example.domain.com/cancel</font><br />The return of additional parameters can be configured for cancel_url. See the “Customer return after payment” section for details.|
|```expiry_url```|_String_|Optional<br/>Valid URL<br/>min: 0 max: 1024|URL where the payer will be redirected in case of session expiration<br/>**Example: **<font color='grey'>https://example.domain.com/expiry</font>|
|```error_url```|_String_|Optional<br/>Valid URL<br/>min: 0 max: 1024|URL to return Customer in case of undefined transaction status.<br/>If the URL is not specified, the cancel_url is used for redirection.<br/>**Example: **<font color='grey'>https://example.domain.com/errorurl</font>|
|```url_target```|_String_|Optional <br/>Possible values: ```_self```, ```_parent```, ```_top```.<br></br>Find the result of applying the values in the HTML standard description ([Browsing context names](https://html.spec.whatwg.org/#valid-browsing-context-name)) | Name of, or keyword for a browsing context where Customer should be returned according to HTML specification.<br />**Example:** <font color='grey'>_parent</font>|
| ```req_token``` | _Boolean_|Optional<br/>default - false |Special attribute pointing for further tokenization<br/>If the ```card_token``` is specified, ```req_token``` will be ignored.<br/>For purchase and debit operations.<br/>**Example:** <font color='grey'>false</font>|
| ```card_token``` |_Array of Strings_|Optional<br/>String 64 characters |Credit card token value<br/>For purchase and debit operations.<br/>**Example:** <font color='grey'>f5d6a0ab6fcfb6487a39e2256e50fff3c95aaa97</font>|
|```recurring_init```|_Boolean_|Optional<br/>default - false|Initialization of the transaction with possible following recurring<br />Only for purchase operation <br/>**Example:** <font color='grey'>true</font>|
|```schedule_id```|_String_|Optional<br/>It s available when recurring_init = true | Schedule ID for recurring payments<br/>Only for purchase operation <br/>**Example:** 57fddecf-17b9-4d38-9320-a670f0c29ec0|
|```vat_calc```|_Boolean_|Conditional<br/>true or false<br/>default - false|Indicates the need of calculation for the VAT amount <br/> • 'true' - if VAT calculation needed<br/> • 'false' - if VAT should not be calculated for current payment.<br/>Only for purchase operation <br/>**Example:** <font color='grey'>false</font>|
|```hash```|_String_|Required|Special signature to validate your request to Payment Platform Addition in Signature section.<br />**Example:** <ins>Must be SHA1 of MD5 encoded string (uppercased):</ins> <font color='grey'>order_number + order_amount + order_currency + order_description + password</font>|
|**order**|_Object_|Required|Information about an order|
|```number```|_String_|Required<br/>max: 255<br/>[a-z A-Z 0-9 -!"#$%&'()*+,./:;&@]|  Order ID<br />**Example:** <font color='grey'>order-1234</font>|
|```amount```|_String_|Required<br/>Greater then 0<br/>[0-9]<br/>max: 255|Format depends on currency.<br />Send Integer type value for currencies with zero-exponent. **Example:** <font color='grey'>1000</font><br />Send Float type value for currencies with exponents 2, 3, 4.<br />Format for 2-exponent currencies: <font color='grey'>XX.XX</font> **Example:** <font color='grey'>100.99</font><br />**Pay attention** that currencies 'UGX', 'JPY', 'KRW', 'CLP' must be send in the format <font color='grey'>XX.XX</font>, with the zeros after comma. **Example:** <font color='grey'>100.00</font><br />Format for 3-exponent currencies: <font color='grey'>XXX.XXX</font> **Example:** <font color='grey'>100.999.</font><br />Format for 4-exponent currencies: <font color='grey'>XXX.XXXX</font> **Example:** <font color='grey'>100.9999</font><br />⚠️ **Note!** For crypto currencies use the exponent as appropriate for the specific currency. <br /> **For purchase operation with crypto:** <br /> Do not send parameter at all if you want to create a transaction with an unknown amount - for cases when the amount is set by the payer outside the merchant site (for example, in an app). As well, you have to omit "amount" parameter for hash calculation for such cases.<br /> **For transfer operation:**<br/> Send amount if you want to show default value in the amount field on the Checkout. The customers can change the value manually on the payment form.Send "-1" value if you want to show empty amount field on the Checkout so that the Customers could enter the value manually.|
|```currency```|_String_|Required<br/>3 characters for fiat currencies and from 3 to 6 characters for crypto currencies<br/>[A-Z]<br/><font color='grey'>[ISO 4217](https://www.xe.com/iso4217.php)</font> |  Currency<br />**Example (fiat):** <font color='grey'>USD</font><br />**Example (crypto):** <font color='grey'>USDT</font>|
|```description```|_String_|Required<br/>min: 2 max: 1024<br/>[a-z A-Z 0-9 !"#$%&'()*+,./:;&@]|Product name<br />**Example:** <font color='grey'>Very important gift - # 9</font>|
|**customer**|_Object_|Conditional|Customer's information. Send an object if a payment method needs|
|```name```|_String_|Conditional<br/>min: 2 max: 32<br/>Latin basic<br/>[a-z A-Z]|Customer's name <br/>Condition: If the parameter is NOT specified in the request, then it will be displayed on the Checkout page (if a payment method needs) - the "Cardholder" field<br />**Example:** <font color='grey'>John Doe</font>|
|```email```|_String_|Conditional<br/>min: 2 max: 255<br/>email format| Customer's email address <br/>Condition: If the parameter is NOT specified in the request, then it will be displayed on the Checkout page (if a payment method needs) - the "E-mail" field<br />**Example:** <font color='grey'>[example@mail.com](url)</font>                   |
|```birth_date```|_String_|Conditional<br/>format: yyyy-mm-dd<br/><font color='blue'>[ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)</font>|Payer's birth date<br/>**Example:** <font color='grey'>1970-02-17</font>|
| **billing_address** |_Object_|Conditional|Billing address information.<br/>**Condition:** If the object or some object's parameters are NOT specified in the request, then it will be displayed on the Checkout page (if a payment method needs)|
|```country```|_String_|  Conditional 2 characters<br/>(alpha-2 code)<br/><font color='blue'>[ISO 3166-1](https://www.iban.com/country-codes)</font>|  Billing country <br />**Example:** <font color='grey'>US</font>|
|```state```|_String_|Optional<br/>min: 2 max: 32<br/>[a-z A-Z]<br/>It is 2-letters code for USA, Canada, Australia, Japan, India |  Billing state address <br />**Example:** <font color='grey'>CA</font>|
|```city```|_String_|Conditional<br/>min: 2 max: 40<br/>[a-z A-Z]|Billing city<br/>**Example:** <font color='grey'>Los Angeles</font>|
|```district```|_String_|Optional<br/>min: 2 max: 32<br/>[a-z A-Z 0-9 - space]|City district <br />**Example:** <font color='grey'>Beverlywood</font>|
|```address```|_String_|Conditional<br/>min: 2 max: 32<br/>[a-z A-Z 0-9]|Billing address <br />**Example:** <font color='grey'>Moor Building</font>|
|```house_number```|_String_|Optional<br/>min: 1 max: 9<br/>[a-z A-Z 0-9/ - space]|House number <br />**Example:** <font color='grey'>17/2</font>|
|```zip```|_String_|Conditional<br/>min: 2 max: 10<br/>[a-z A-Z 0-9]|  Billing zip code <br />**Example:** <font color='grey'>123456, MK77</font>|
|```phone```|_String_|Conditional<br/>min: 1 max: 32<br/>[0-9 + () -]|Customer phone number <br/>**Example:** <font color='grey'>347771112233</font>|
|**payee**| _Object_ |Conditional| Payee's information. <br/>Specify additional information about Payee for transfer operation if it is required by payment provider.|
|```name```|_String_|Required<br/>min: 2 max: 32<br/>Latin basic<br/>[a-z A-Z]|Customer's name.<br />**Example:** <font color='grey'>John Doe</font>|
|```email```|_String_|Conditional<br/>email format|Customer's email address.<br />**Example:** <font color='grey'>example@mail.com</font>|
| **payee_billing_address** |_Object_|Conditional|Billing address information for Payee.|
|```country```|_String_|  Conditional 2 characters<br/>(alpha-2 code)<br/><font color='blue'>[ISO 3166-1](https://www.iban.com/country-codes)</font>|  Billing country <br />**Example:** <font color='grey'>US</font>|
|```state```|_String_|Optional<br/>min: 2 max: 32<br/>[a-z A-Z]<br/>It is 2-letters code for USA, Canada, Australia, Japan, India |  Billing state address <br />**Example:** <font color='grey'>CA</font>|
|```city```|_String_|Conditional<br/>min: 2 max: 40<br/>[a-z A-Z]|Billing city<br/>**Example:** <font color='grey'>Los Angeles</font>|
|```district```|_String_|Optional<br/>min: 2 max: 32<br/>[a-z A-Z 0-9 - space]|City district <br />**Example:** <font color='grey'>Beverlywood</font>|
|```address```|_String_|Conditional<br/>min: 2 max: 32<br/>[a-z A-Z 0-9]|Billing address <br />**Example:** <font color='grey'>Moor Building</font>|
|```house_number```|_String_|Optional<br/>min: 1 max: 9<br/>[a-z A-Z 0-9/ - space]|House number <br />**Example:** <font color='grey'>17/2</font>|
|```zip```|_String_|Conditional<br/>min: 2 max: 10<br/>[a-z A-Z 0-9]|  Billing zip code <br />**Example:** <font color='grey'>123456, MK77</font>|
|```phone```|_String_|Conditional<br/>min: 1 max: 32<br/>[0-9 + () -]|Customer phone number <br/>**Example:** <font color='grey'>347771112233</font>|
|**crypto**|_Object_|Optional|Additional information regarding crypto transactions|
|```network```|_String_|Optional<br/>max: 50|You can use an arbitrary value or select one from the following.<br /> **Example:**<br /> <font color='grey'>•	ERC20 <br /> •	TRC20 <br /> •	BEP20 <br /> • BEP2 <br /> •	OMNI <br /> •	solana <br /> •	polygon</font>|
| **parameters** |_Object_|Optional|Extra-parameters required for specific payment method<br />**Example:**<br/><font color='grey'>```"parameters": { "payment_method": { "param1":"val1", "param2":"val2" } }```</font>|
| **custom_data** |_Object_|Optional|Custom data<br/>This block can contain arbitrary data, which will be returned in the callback.<br />**Example:**<br/><font color='grey'>```“custom_data”: {“param1”:”value1”, “param2”:”value2”, “param3”:”value3”}```</font>|


  <details>
 	<summary markdown="span">Authentication (OK)</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "purchase",
  "methods": [
    "card",
    "method1"
  ],
  "parameters": {
    "card": {
      "param1": "val-1",
      "param2": "val-2"
    },
    "method1": {
      "param1": "val-3",
      "param2": "val-4"
    }
  },
  "session_expiry": 60,
  "order": {
    "number": "order-1234",
    "amount": "0.19",
    "currency": "USD",
    "description": "Important gift"
  },
  "cancel_url": "https://example.domain.com/cancel",
  "success_url": "https://example.domain.com/success",
  "expiry_url": "https://example.domain.com/expiry",
  "url_target": "_blank",
  "customer": {
    "name": "John Doe",
    "email": "test@gmail.com"
  },
  "billing_address": {
    "country": "US",
    "state": "CA",
    "city": "Los Angeles",
    "district": "Beverlywood",
    "address": "Moor Building 35274",
    "house_number": "17/2",
    "zip": "123456",
    "phone": "347771112233"
  },
  "card_token": [
    "f5d6a0ab6fcfb6487a39e2256e50fff3c95aaa97075ee5e539bb662fceff4dc1"
  ],
  "req_token": true,
  "recurring_init": true,
  "schedule_id": "9d0f5cc4-f07b-11ec-abf4-0242ac120006",
  "hash": "{{session_hash}}"
}
'
```
 </details>

<details>
 	<summary markdown="span">Debit Operation (OK)</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "debit",
  "methods": [
    "card"
  ],
  "parameters": {
    "card": {
      "param1": "val-1",
      "param2": "val-2"
    },
    "session_expiry": 60,
    "order": {
      "number": "order-1234",
      "amount": "0.19",
      "currency": "USD",
      "description": "Important gift"
    },
    "cancel_url": "https://example.domain.com/cancel",
    "success_url": "https://example.domain.com/success",
    "expiry_url": "https://example.domain.com/expiry",
    "url_target": "_blank",
    "customer": {
      "name": "John Doe",
      "email": "test@gmail.com"
    },
    "billing_address": {
      "country": "US",
      "state": "CA",
      "city": "Los Angeles",
      "disctrict": "Beverlywood",
      "address": "Moor Building 35274",
      "house_number": "17/2",
      "zip": "123456",
      "phone": "347771112233"
    },
    "card_token": [
      "f5d6a0ab6fcfb6487a39e2256e50fff3c95aaa97075ee5e539bb662fceff4dc1"
    ],
    "req_token": true,
    "hash": "{{session_hash}}"
  }
}
'
```
 </details>

<details>
 	<summary markdown="span">Transfer Operation (OK)</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{  
   "merchant_key":"xxxxx-xxxxx-xxxxx",
   "operation":"transfer",
   "order":{
      "number":"order-1234",
      "amount": "100.55",
      "currency":"USD",
      "description":"Important gift"
   },
   "cancel_url":"https://example.com/cancel",
   "success_url":"https://example.com/success",
   "customer":{
        "name":"John Doe",
        "email":"success@gmail.com"
    },
    "payee": {
        "name":"John Doe",
        "email":"success@gmail.com"
        },
    "billing_address":{
        "country":"US",
        "state":"California",
        "city":"Los Angeles",
        "address":"Moor Building 35274 State ST Fremont. U.S.A",
        "zip":"94538",
        "phone":"0987654321",
        "district":"Brentwood",
        "house_number":"123"
    },
    "payee_billing_address":{
        "country":"US",
        "state":"New York",
        "city":"New York",
        "address":"Moor Building 35274 State ST Fremont. U.S.A",
        "zip":"94538",
        "phone":"0987654321",
        "district":"Brentwood",
        "house_number":"123"
    },
   "hash":"{{session_hash}}"
}
'
```
 </details>


<details>
  <summary markdown="span">Example Response (OK)</summary>

```
{
  "redirect_url": "{{CHECKOUT_HOST}/auth/ZXlKMGVYQWlPaUpLVjFRaUxDSmhiR2NpT2lKU1V6STFOaUo5LmV5SnBZWFFpT2pFMU9UWXhPRFl6T0Rnc0ltcDBhU0k2SWpGbE5qTTNObVZoTFdRek1HUXRNVEZsWVMxaE16QXlMVEF5TkRKak1HRTRNekF4TWlJc0ltVjRjQ0k2TVRVNU5qRTRPVGs0T0gwLm9CMmVhdlRtTU5DMXFTajlDVFlqQ0dOMDlHdUs1NXRkQTVpWFR3d2F2cWR0cEpEU2NRWWFaT3Z5dmJSVjJUSFNUVlFlS0NUX3pRdFNycDlKS1M4X0pqUzRMclM5MnUyNXRfSHNGa1FUQ0VOdGtadHQtaGxONERYdVhkLTU5cEhKLUN1RXBqSmZ4UDZEQXhFaVAxWEpRZDlyQldNa1RQVDdGZm1ac0g4LTM5YnV6LTI3MWxKMndkekdvSGJYa0NKVnNTNFJldGxrbno2U3dGd3ZFMW5KNDhwYTBGMDNLWjBpNnhpRFVPR3p2U0ZKdGZfMndDTTdzTTdsemc1TlBmSDl0Q0RKQmZEaG1hUmJCRmR6RlZMZlJncG5tMzB3VWpTMGMxbmt6SkkxOGJTd2Z6Z0hfZFpnc1cyUFhCM2ZLdG9pWDJXeFRsQzlxR204QTRYVm9EQy1mOWxvRHlMd0F5eV9xY3JrWmNuQTJVSjk5Zl91c0cwODZKUlBTT0I4VHVRZndSTzUxSEN2bEU2TXdFYzVYRmtnYjBleEZRcXdpNGE4S2RlWV9HX3ZQam42bnpZODdtVzFINlpQMjJ0dzVzazYtUENMeHdvNXctUmFBWC1mYVVhcEVHTzFLZkVHbndaQWZBZVNyc3U4MV9XQUFJMlN5RUxGWi1IU1lXMUZLWFgybzNNeF93Ty1DS3FLTWZsUTV1cGc2eDAybzhsbFhoeGJlVmVIOWlkMHgzYldRWE9vWk5hWm1MeVpJMmJsT2dtVDV0cHR4NHNQNDNqT0NtYW1sdkxyUkZvQmxCNTJ4V0RUQTBZQnhBLW5meUxCRHRJN0dPaVRWQjJ5cWd1Z1lBdGRfbWFQN2x2YTJpbVJWaHhxT0R5SlRiZThxcDdhWkw4bkJvTHZocnZDOHlv"
}
```

</details>

## Authentication request possible errors

---

### Parameter Values Validation

Checkout service validates the request parameters and sends the error responses in case of an invalid data detection.

<details>
	<summary markdown="span">Example Request - Validation (bad)</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{  
   "merchant_key":"xxxxx-xxxxx-xxxxx",
   "operation":"credit",
   "methods":[ "" ],
   "order":{
      "number":"",
      "amount": "1.2",
      "currency":"Dollar",
      "description":""
   },
   "cancel_url":"1.com",
   "success_url":"",
   "customer":{
      "name":"John Doe",
      "email":"example@mail.com"
   },
   "recurring_init": "true",
   "hash":"728d13b95cf2b6b3ee04b20dc2fc9889ffff1cf4"
}
'
```
</details>

<details>
	<summary markdown="span">Example Response - 400 Bad Request</summary>


```
{
  "error_code": 0,
  "error_message": "Request data is invalid.",
  "errors": [
    {
      "error_code": 100000,
      "error_message": "operation: The value you selected is not a valid choice."
    },
    {
      "error_code": 100000,
      "error_message": "methods: This value should not be blank."
    },
    {
      "error_code": 100000,
      "error_message": "order.number: This value should not be blank."
    },
    {
      "error_code": 100000,
      "error_message": "order.amount: This value should be greater than 0."
    },
    {
      "error_code": 100000,
      "error_message": "order.currency: This value is not valid."
    },
    {
      "error_code": 100000,
      "error_message": "order.description: This value should not be blank."
    },
    {
      "error_code": 100000,
      "error_message": "cancel_url: This value is not a valid URL."
    },
    {
      "error_code": 100000,
      "error_message": "success_url: This value should not be blank."
    }
  ]
}
```
</details>

### Hash Validation


The Checkout service always performs `hash` validation.
The request will be rejected if the `hash` value is invalid.

<details>
	<summary markdown="span">Example Request - Hash Validation</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{  
   "merchant_key":"xxxxx-xxxxx-xxxxx",
   "operation":"purchase",
   "methods":[
      "card"
   ],
   "order":{
      "number":"order-1234",
      "amount": "0.19",
      "currency":"USD",
      "description":"Important gift"
   },
   "cancel_url":"https://example.com/cancel",
   "success_url":"https://example.com/success",
   "customer":{
      "name":"John Doe"
   },
   "hash":"wrong hash"
}
'
```

</details>
<details>
	<summary markdown="span">Example Response - Hash error</summary>

```
{
  "error_code": 0,
  "error_message": "Request data is invalid.",
  "errors": [
    {
      "error_code": 100000,
      "error_message": "hash: Hash is not valid."
    }
  ]
}
```
</details>

## Callback Notification

---

Checkout service sends the callback on the merchant ```notification_URL``` as a result of an operation.


You can receive the callback for the next operation types:

- SALE
- 3DS
- REDIRECT
- REFUND
- VOID
- RECURRING
- CHARGEBACK
- DEBIT
- TRANSFER

⚠️ **Pay attention** <br/>

> _Note that the notification URL may be temporarily blocked due to consistently receiving timeouts in response to the callback._
_If five timeouts accumulate within five minutes for a merchant’s notification URL, it will be blocked for 15 minutes. During this block,_ _all merchants associated with the URL will not receive notifications._
_The block automatically lifts after 15 minutes._
_Additionally, it is possible to manually unblock the URL through the admin panel by navigating to Configuration → Merchants → Edit_ _Merchant. In this case, the block will be removed immediately._
_The timeout counter resets if a callback response is successfully processed. For instance, if there are four timeouts within five minutes_
_but a successful response on the sixth minute, the counter resets._

⚠️ **Pay attention** <br/>

> _In the case of cascading, the logic for sending callbacks differs.<br/> If cascading is triggered for the order, in general case you will receive only callback for the last payment attempt, where the final status of the order (settled or declined) is determined.
In the particular cases, you will receive callback for the first payment attempt with the data for customer’s redirection if it is required by payment provider. Callbacks for intermediate attempts (between the first decline and the last payment attempt) are not sent._<br/>

### List of Possible Transaction Statuses

The possible statuses are listed below:

| **Transaction Status** | **Operation type**                      | **Description**                                         |
| ---------- | -------------------------------------   | ------------------------------------------------------------------- |
|  SUCCESS   | sale, 3ds, redirect, capture, refund, void, recurring, chargeback, debit, transfer | Transaction is successfully completed in Payment Platform|
|    FAIL    | sale, capture, refund, void, recurring, debit, transfer | Transaction has the errors and is not validated by Payment Platform |
|  WAITING   | sale, capture, refund, void, recurring, debit, transfer | Transaction is being processed by Payment Platform                  |
|  UNDEFINED | sale, 3ds, redirect, capture, refund, void, recurring, debit, transfer | Uncertain payment status due to problems interacting with the payment provider |

⚠️ **Pay attention**

> **_that successful transaction does not mean successful final status for the payment._<br />**
>>
>> **_Example for SMS mode (1-step paments) only:_**<br />
>> _Payment is successfully completed if transaction has ```status``` = success and ```type``` = sale. Payment is not completed if transaction has ```status``` = success and ```type``` = redirect._

### Callback parameters

Callback is HTTP POST request whcih includes the following data:

|**Parameter**|**Type**|**Mandatory, Limitations**|**Description**|
| - | :-: | :- | - |
|`id`|String|Required| Transaction ID <br  />**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font>|
|`order_number`|_String_|Required Up to 255 characters| Order ID <br  />**Example:** <font color='grey'>order-1234</font>|
|`order_amount`|_Float_|Required Format: XX.XX, without leading zeroes| Product price <br  />**Example:** <font color='grey'>0.19</font>|
|`order_currency`|_String_|Required Up to 6 characters| Currency code.<br/> 3-characters for fiat currencies and from 3 to 6 characters for crypto currencies <br  />**Example (fiat):** <font color='grey'>USD</font><br/>**Example (crypto):** <font color='grey'>USDT</font>|
|`order_description`|_String_|Required Up to 1024 characters| Product description <br  />**Example:** <font color='grey'>Important gift</font>|
|`order_status`|_String_|Required Up to 20 characters| Payment status: prepare, settled, pending, 3ds, redirect, decline, refund, reversal, chargeback  <br  />**Example:** <font color='grey'>3ds</font>|
|`type`|_String_|Required Up to 36 characters| Operation type: sale, 3ds, redirect, capture, refund, void, chargeback, debit, transfer <br  />**Example:** <font color='grey'>sale</font>|
|`status`|_String_|Required Up to 20 characters| Transaction status: success, fail, waiting, undefined <br  />**Example:** <font color='grey'>success</font>|
|`reason`|_String_|Optional Up to 1024 characters| Decline or error reason. It displays only if the transaction has FAIL status.<br/>**Example:** <font color='grey'>The operation was rejected. Please contact the site support</font>|
| `payee_name` | _String_ | Optional | Payee's first and last name <br  />Returns only for transfer operation<br/>**Example:** <font color='grey'>John Rickher</font>|
| `payee_email` | _String_ | Optional | Payee's e-mail <br  />Returns only for transfer operation<br/>**Example:** <font color='grey'>example@mail.com</font>|
| `payee_country` | _String_ | Optional <br/>Up to 3 characters| Payee's country <br  />Returns only for transfer operation<br/>**Example:** <font color='grey'>US</font> |
| `payee_state` | _String_ | Optional <br/>Up to 32 characters| Payee's state <br  />Returns only for transfer operation<br/>**Example:** <font color='grey'>California</font> |
| `payee_city` | _String_ | Optional <br/>Up to 32 characters| Payee's city <br  />Returns only for transfer operation<br/>**Example:** <font color='grey'>Los Angeles</font> |
| `payee_address` | _String_ | Optional <br/>Up to 32 characters| Payee's city <br  />Returns only for transfer operation<br/>**Example:** <font color='grey'>123 Sample Street</font> |
|`connector_name` ** \* ** |_String_|Optional| Connector's name (Payment Gateway).<br/>**Example:** <font color='grey'>MyPaymentConnector</font>|
|`rrn` ** \* ** |_String_|Optional| Retrieval Reference Number value from the acquirer system.<br/>**Example:** <font color='grey'>123456789012</font>|
|`approval_code` ** \* ** |_String_|Optional| Approval code value from the acquirer system.<br/>**Example:** <font color='grey'>123456</font> |
| `gateway_id` ** \* ** | _String_ | Optional | Gateway ID – transaction identifier provided by payment gateway.<br/>**Example:** <font color='grey'>123-456-789</font> |
| `extra_gateway_id` ** \* ** | _String_ | Optional | Extra Gateway ID – additional transaction identifier provided by payment gateway.<br/>**Example:** <font color='grey'>abc123</font> |
| `merchant_name`** \* ** | _String_ | Optional | Merchant Name<br/>**Example:** <font color='grey'>TESTMerchantName</font>|
| `mid_name` ** \* ** | _String_ | Optional | MID Name<br/>**Example:** <font color='grey'>TESTMIDName</font>|
|`issuer_country` ** \* ** |_String_|Optional| Issuer Country<br/>**Example:**<font color='grey'> DE</font>|
|`issuer_bank` ** \* ** |_String_|Optional| Issuer Bank<br/>**Example:**<font color='grey'> Deutsche Bank</font>|
|`card`|_String_|Optional<br/>Format: ХХХХХХ\*\*\*\*\*\*ХХХХ|Card number mask <br  />**Example:** <font color='grey'>411111\*\*\*\*\*\*1111</font>|
|`card_expiration_date`|_String_|Optional|Card expiration date<br />**Example:** <font color='grey'>12/2022</font>|
|`payee_card`|_String_|Optional|Payee card number mask <br/> Returns only for transfer operation<br />**Example:** <font color='grey'>411111\*\*\*\*\*\*1111</font>|
|`card_token`|_String_|Optional|Card token. It is available if the parameter `req_token` was enabled<br />It is available only for purchase operation<br/>**Example:** <font color='grey'>VjFRaUxDSmhiR2NpT2lKU1V6STFO</font>|
|`crypto_network`|_String_|Optional| Blockchain network where the transaction is taking place <br  />**Example:** <font color='grey'>ERC20</font>|
|`crypto_address`|_String_|Optional| Public address of a cryptocurrency wallet <br  />**Example:** <font color='grey'>bc1QWXYZ8901ghijkLMNOPqrstv2345wxyzABCDE</font>|
|`customer_name`|_String_|Optional| Customer's first and last name <br  />**Example:** <font color='grey'>John Rickher</font>|
|`customer_email`|_String_|Optional<br/>Format: [example@mail.com](url)| Customer's email address <br  />**Example:** <font color='grey'>[fdfd@dfsds.ds](url)</font>|
|`customer_country`|_String_|Optional<br/>Up to 3 characters| Customer's country  <br  />**Example:** <font color='grey'>US</font>|
|`customer_state`|_String_|Optional<br/>Up to 32 characters| Customer's state <br  />**Example:** <font color='grey'>California</font>|
|`customer_city`|_String_|Optional| Customer's city <br  />**Example:** <font color='grey'>Los Angeles</font> |
|`customer_address`|_String_|Optional<br/>Up to 32 characters| Customer's address <br  />**Example:** <font color='grey'>123 Sample Street</font> |
|`customer_ip`|_String_|Required| Customer's IP <br  />**Example:** <font color='grey'>255.41.45.57</font> |
|`date`|_Date_|Optional<br/>Format: YYYY-MM-DD hh:mm:ss| Transaction date <br  />**Example:** <font color='grey'>2020-08-05 07:41:10</font> |
|`recurring_init_trans_id`|_String_|Optional|Reference to the first transaction that initializes the recurring (provided if recurring was initialized)<br/>It is available only for purchase operation<br/>**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87099</font>|
|`recurring_token`|_String_|Optional|Recurring token (provided if recurring was initialized)<br/>**Example:** <font color='grey'>e5f60b35485e</font>|
|`schedule_id`|_String_|Optional|It is available if schedule is used for recurring sale<br/>It is available only for purchase operation|
| ```exchange_rate``` | _Decimal_ | Optional | Rate used to make exchange. <br /> It returns if the currency exchange has been applied for the payment. |
| ```exchange_rate_base``` | _Decimal_ | Optional | The rate used in the double conversion to convert the original currency to the base currency. <br /> It returns if the currency exchange has been applied for the payment. |
| ```exchange_currency``` | _Dictionary_ | Optional | Original currency.<br />It returns if the currency exchange has been applied for the payment. |
| `exchange_amount` | _Integer_ | Optional | Original amount.<br />It returns if the currency exchange has been applied for the payment. |
| `vat_amount` | _Float_ | Optional<br/>Format: XX.XX, without leading zeroes  | Product VAT <br/> Returns if VAT has been calculated for the payment.<br/>It is available only for purchase operation<br/>**Example:** <font color='grey'>0.09</font>|
| `digital_wallet` | _String_ | Optional | Wallet provider: googlepay, applepay. <br/> It is returned if digital wallets were used for card payment creation.<br/> **Example:** <font color='grey'>googlepay</font> |
| `pan_type` | _String_ | Optional | It refers to digital payments, such as Apple Pay and Google Pay, and the card numbers returned as a result of payment token decryption: DPAN (Digital Primary Account Number) and FPAN (Funding Primary Account Number). <br/> Possible values: dpan, fpan. <br/> It is returned if digital wallets were used in card payment creation. <br/>**Example:** <font color='grey'>dpan</font> |
| **custom_data** |_Object_|Optional|Custom data<br/>This block duplicates the arbitrary parameters that were passed in the payment request<br />**Example:**<br/><font color='grey'>```“custom_data”: {“param1”:”value1”, “param2”:”value2”, “param3”:”value3”}```</font>|
|`hash`|_String_|Required| Special signature, used to validate callback  Addition in Signature section. <br /> **Example:** <ins>Must be SHA1 of MD5 encoded string (uppercased):</ins> <font color='grey'>payment\_public\_id + order.number + order.amount + order.currency + order.description + merchant.pass</font> |

** \* ** The parameters are included if the appropriate setup is configured in the admin panel (see “Add Extended Data to Callback” block in the Configurations -> Protocol Mappings section).

### Examples

<details>
  <summary markdown="span">Callback examples</summary>

<details>
  <summary markdown="span">Merchant successful sale callback</summary>

```
id=f0a51dfa-fc43-11ec-8128-0242ac120004&
order_number=order-1234&
order_amount=3.01&
order_currency=USD&
order_description=bloodline&
order_status=settled&
type=sale&
status=success&
card=411111****1111&
card_expiration_date=12/2022&
connector_name=MyPaymentConnector&
rrn=789012345678& 
approval_code=123456&
schedule_id=4e46866c-f84b-11ec-8b4c-0242ac120007&
recurring_init_trans_id=f0a51dfa-fc43-11ec-8128-0242ac120004&
recurring_token=f0e24964-fc43-11ec-a7e0-0242ac120004&
date=2022-07-05 09:22:09&
hash=6d8d440e25bdfc5288616ce567496948d2562852&
gateway_id=123-456-789&
extra_gateway_id=abc123&
merchant_name=TESTMerchantName&
mid_name=TESTMIDName&
issuer_country=DE&
issuer_bank=Deutsche Bank&
customer_name=D D&
customer_email=success@gmail.com&
customer_country=US&
customer_state=California&
customer_city=Los Angeles&
customer_address=Moor Building 35274 State ST Fremont. U.S.A&
customer_ip=10.10.10.2&
exchange_rate_base=26.19&
exchange_rate=6.47&
exchange_currency=UAH&
exchange_amount=100.00
```
</details>

<details>
  <summary markdown="span">Merchant successful debit callback</summary>

```
id=f0a51dfa-fc43-11ec-8128-0242ac120004&
order_number=order-1234&
order_amount=3.01&
order_currency=USD&
order_description=bloodline&
order_status=settled&
type=debit&
status=success&
card=411111****1111&
card_expiration_date=12/2022&
date=2022-07-05+09:22:09&
hash=6d8d440e25bdfc5288616ce567496948d2562852&
customer_name=D D&
customer_email=success@gmail.com&
customer_country=US&
customer_state=California&
customer_city=Los Angeles&
customer_address=Moor Building 35274 State ST Fremont. U.S.A&
customer_ip=10.10.10.2
```
</details>

<details>
  <summary markdown="span">Merchant successful refund callback</summary>

```
 id=f0a51dfa-fc43-11ec-8128-0242ac120004&
 order_number=order-1234&
 order_amount=3.01&
 order_currency=USD&
 order_description=bloodline&
 order_status=settled&
 type=refund&
 status=success&
 card=411111****1111&
 card_expiration_date=12/2022&
 schedule_id=4e46866c-f84b-11ec-8b4c-0242ac120007&
 date=2022-07-05+09:28:01&
 hash=6d8d440e25bdfc5288616ce567496948d2562852&
 customer_name=D D&
 customer_email=success@gmail.com&
 customer_country=US&
 customer_state=California&
 customer_city=Los Angeles&
 customer_address=Moor Building 35274 State ST Fremont. U.S.A&
 customer_ip=10.10.10.2
```
</details>
<details>
  <summary markdown="span">Merchant successful transfer callback</summary>

```
id=f0a51dfa-fc43-11ec-8128-0242ac120004&
order_number=order-1234&
order_amount=3.01&
order_currency=USD&
order_description=bloodline&
order_status=settled&
type=transfer&
status=success&card=411111****1111&
card_expiration_date=12/2022&
payee_card=422222****2222&
date=2022-07-05+09:22:09&
hash=6d8d440e25bdfc5288616ce567496948d2562852&
customer_name=D D&
customer_email=success@gmail.com&
customer_country=US&
customer_state=California&
customer_city=Los Angeles&
customer_address=Moor Building 35275&
customer_ip=10.10.10.2&
payee_name=D D&
payee_email=success@gmail.com&
payee_country=US&
payee_state=California&
payee_city=Los Angeles&payee_address=Moor Building 35274 State ST Fremont. U.S.A
```
</details>
<details>
  <summary markdown="span">Merchant unsuccessful sale callback</summary>

```
id=1f34f446-fc45-11ec-a50f-0242ac120004&
order_number=order-1234&
order_amount=3.01&
order_currency=USD&
order_description=bloodline&
order_status=decline&
type=sale&
status=fail&
card=411111****1111&
card_expiration_date=12/2022&
reason=Declined by processing.&
date=2022-07-05+09:30:35&
hash=7f15d178e9b2c8507dea57f8ed1efddb9573fa6b&
customer_name=D D&
customer_email=success@gmail.com&
customer_country=US&
customer_state=California&
customer_city=Los Angeles&
customer_address=Moor Building 35274 State ST Fremont. U.S.A&
customer_ip=10.10.10.2
```
</details>

<details>
  <summary markdown="span">Merchant unsuccessful debit callback</summary>

```
id=1f34f446-fc45-11ec-a50f-0242ac120004&
order_number=order-1234&
order_amount=3.01&
order_currency=USD&
order_description=bloodline&
order_status=decline&
type=debit&status=fail&
card=411111****1111&
card_expiration_date=12/2022&
reason=Declined by processing.&
date=2022-07-05+09:30:35&
hash=7f15d178e9b2c8507dea57f8ed1efddb9573fa6b&
customer_name=D D&
customer_email=success@gmail.com&
customer_country=US&
customer_state=California&
customer_city=Los Angeles&
customer_address=Moor Building 35274 State ST Fremont. U.S.A&
customer_ip=10.10.10.2
```
</details>

<details>
  <summary markdown="span">Merchant unsuccessful refund callback</summary>

```
id=ba290c62-fc45-11ec-9e91-0242ac120004&
order_number=order-1234&
order_amount=3.01&
order_currency=USD&
order_description=bloodline&
order_status=decline&
type=refund&
status=fail&
card=411111****1111&
card_expiration_date=12/2022&
reason=Declined by processing.&
schedule_id=4e46866c-f84b-11ec-8b4c-0242ac120007&recurring_init_trans_id=ba290c62-fc45-11ec-9e91-0242ac120004&recurring_token=ba51844e-fc45-11ec-932c-0242ac120004&
date=2022-07-05+09:38:00&
hash=bcd78ff8b8e6b75aa1743910641217be6edc3a43&
customer_name=D D&
customer_email=success@gmail.com&customer_country=US&
customer_state=California&
customer_city=Los Angeles&
customer_address=Moor Building 35274 State ST Fremont. U.S.A&
customer_ip=10.10.10.2
```
</details>
<details>

  <summary markdown="span">Merchant unsuccessful transfer callback</summary>

```
id=1f34f446-fc45-11ec-a50f-0242ac120004&
order_number=order-1234&
order_amount=3.01&
order_currency=USD&
order_description=bloodline&
order_status=decline&
type=transfer&
status=fail&
card=411111****1111&
card_expiration_date=12/2022&
payee_card=422222****2222&
date=2022-07-05+09:22:09&
hash=6d8d440e25bdfc5288616ce567496948d2562852&
customer_name=D D&
customer_email=success@gmail.com&
customer_country=US&
customer_state=California&
customer_city=Los Angeles&
customer_address=Moor Building 35275&
customer_ip=10.10.10.2&
payee_name=D D&
payee_email=success@gmail.com&
payee_country=US&
payee_state=California&
payee_city=Los Angeles&
payee_address=Moor Building 35274 State ST Fremont. U.S.A&
vat_amount=0.3
```
</details>
<details>

  <summary markdown="span">Merchant successful crypto callback</summary>

```
id=ac60a7e6-9abf-11ef-a549-0242ac120002&
order_number=order-1234&
order_amount=0.00000001&
order_currency=BTC&
order_description=Important+gift&order_status=pending&
type=init&status=success&
date=2024-11-04+15%3A15%3A49&
hash=871a243619e4fd04db377cfbabe9fe07505febf5&
crypto_network=ERC20&
crypto_address=bc1ABIJKL1234abfjkmoPQRTVXY5678prsx&
customer_name=John+Doe&
customer_email=success%40gmail.com&
customer_country=US&
customer_state=California&
customer_city=Los+Angeles&
customer_address=Moor+Building+35274+State+ST+Fremont.+U.S.A&
customer_ip=10.10.10.2
```
</details>
</details>

## Recurring
---
Recurring payments are commonly used to create new transactions based on already stored cardholder information from previous operations.
This request is sent by POST in JSON format.

**RECURRING request**

```
/api/v1/payment/recurring
```
### Request Parameters

| **Parameter** | **Type** | **Mandatory, Limitations** |**Description**|
| :------------ | :------: | ------------------------------------------| :------------ |
|`merchant_key` |_String_  |Required|Key for Merchant identification<br/>**Example:** <font color='grey'>xxxxx-xxxxx-xxxxx</font>|
|`recurring_init_trans_id`|_String_|Required|Transaction ID of the primary transaction in the Payment Platform<br/>**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font>|
|`recurring_token`|_String_|Required|Recurring token<br/>**Example:** <font color='grey'>9a2f-0242c0a87002</font>|
|`schedule_id`  |  _String_ |                  Optional                  | Schedule ID for recurring payments |
|`hash`|_String_|Required|Special signature to validate your request to Payment Platform Addition in Signature section<br/> <ins> Must be SHA1 of MD5 encoded string (uppercased):</ins> <font color='grey'>recurring_init_trans_id + recurring_token + order.number + order.amount + order.description + merchant_pass</font>|
|**order**|_Object_|||
|`number`|_String_|Required<br/>Not blank<br/>max: 255<br/>[a-zA-Z0-9-]|Order ID<br/>**Example:** <font color='grey'>order-1234</font>|
|`amount`|_Float_|Required<br/>Not blank<br/>Greater then 0<br/>0-9<br/>max: 255|Format depends on currency.<br />Send Integer type value for currencies with zero-exponent. **Example:** <font color='grey'>1000</font><br />Send Float type value for currencies with exponents 2, 3, 4.<br />Format for 2-exponent currencies: <font color='grey'>XX.XX</font> **Example:** <font color='grey'>100.99</font><br />**Pay attention** that currencies 'UGX', 'JPY', 'KRW', 'CLP' must be send in the format <font color='grey'>XX.XX</font>, with the zeros after comma. **Example:** <font color='grey'>100.00</font><br />Format for 3-exponent currencies: <font color='grey'>XXX.XXX</font> **Example:** <font color='grey'>100.999.</font><br />Format for 4-exponent currencies: <font color='grey'>XXX.XXXX</font> **Example:** <font color='grey'>100.9999</font><br />|
|`description`|_String_|Required<br/>min: 2 max: 1024<br/>[a-zA-Z0-9!"#$%&'()*+,./:;&@]|Product name<br/>**Example:** <font color='grey'>Very important gift - # 9</font>|

### Response Parameters

| **Parameter** | **Type** | **Mandatory, Limitations** |**Description**|
| :------------ | :------: | ------------------------------------------| ------------ |
|`status` |_String_  |Required|Payment status<br/>**Example:** <font color='grey'>SETTLED, PENDING, DECLINE</font>|
|`reason`|_String_|Optional|Decline reason translation for unsuccessful payment.<br/>It displays only if the transaction is unsuccessful<br/>**Example:** <font color='grey'>The operation was rejected. Please contact the site support</font>|
|`payment_id`|_String_|Required<br/>Up to 255 characters|Transaction ID (public)<br/>**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font>|
|`date`|_String_|Required<br/>Format: YYYY-MM-DD hh:mm:ss|Transaction date<br/>**Example:** <font color='grey'>2020-08-05 07:41:10</font>|
|`schedule_id`|_String_| Optional | Schedule ID for recurring payments |
|**order**|_Object_|||
|`number`|_String_|Required<br/>max: 255|Order ID<br/>**Example:** <font color='grey'>order-1234</font>|
|`amount`|_String_|Required<br/>Format: XX.XX|Product price (currency will be defined by the first payment)<br/>**Example:** <font color='grey'>0.19</font>|
|`currency`|_String_|Required<br/>3 characters|Currency<br/>**Example:** <font color='grey'>USD</font>|
|`description`|_String_|Required<br/>max: 1024|Product name<br/>**Example:** <font color='grey'>Very important gift - # 9</font>|


<details>
  <summary markdown="span">Example Request</summary>
 
	<details>
	<summary markdown="span">Recurring (settled)</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/payment/recurring' \
--header 'Content-Type: application/json' \
--data-raw '{ 
   "merchant_key":"xxxxx-xxxxx-xxxxx",
   "order":{
      "number":"order-1234",
      "amount": "0.19",
      "description":"very important gift"
},
    "customer": {
        "name": "John Doe",
        "email": "success@gmail.com"
    },     
       "recurring_init_trans_id":"dc66cdd8-d702-11ea-9a2f-0242c0a87002", 
       "recurring_token":"9a2f-0242c0a87002",
       "schedule_id":"57fddecf-17b9-4d38-9320-a670f0c29ec0", 
       "hash":"{{session_hash}}"
}'
```
</details>
<details>
	<summary markdown="span">Recurring (declined)</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/payment/recurring' \
--header 'Content-Type: application/json' \
--data-raw '{ 
    "payment_id": "1f34f446-fc45-11ec-a50f-0242ac120004",
    "date": "2022-07-05 09:30:34",
    "status": "decline",
    "reason": "Declined by processing.",
    "order": {
        "number": "order-1234",
        "amount": "3.01",
        "currency": "USD",
        "description": "bloodline"
    },
    "customer": {
        "name": "John Doe",
        "email": "success@gmail.com"
    }
}'
```
</details>
</details>

<details>
  <summary markdown="span">Example Response</summary>

<details>
  <summary markdown="span">Status Settled</summary>

```
{
  "status": "settled",
  "payment_id": "dc66cdd8-d702-11ea-9a2f-0242c0a87002",
  "date": "2020-08-05 07:41:10",
  "schedule_id":"57fddecf-17b9-4d38-9320-a670f0c29ec0",
  "order": {
    "number": "order-1234",
    "amount": "0.19",
    "currency": "USD",
    "description": "very important gift"
  }
}
```
</details>

<details>
  <summary markdown="span">Status Declined</summary>

```
{
  "status": "declined",
  "reason": "declined by processing",
  "payment_id": "dc66cdd8-d702-11ea-9a2f-0242c0a87002",
  "schedule_id":"57fddecf-17b9-4d38-9320-a670f0c29ec0",
  "date": "2020-08-05 07:41:10",
  "order": {
    "number": "order-1234",
    "amount": "0.19",
    "currency": "USD",
    "description": "very important gift"
  }
}
```
</details>
</details>

## Retry
---

RETRY request is used to retry funds charging for secondary recurring payments in case of soft decline.

This action creates RETRY transaction and can cause payment final status changing.

This request is sent by POST in JSON format.

**RETRY request**

```
/api/v1/payment/retry
```
### Request parameters

| **Parameter** | **Type** | **Mandatory, Limitations**| **Description**|
| ------------- | :------: | ---------------------- | ------------- |
| `merchant_key` |_String_ | Required | Key for Merchant identification<br />**Example:** <font color='grey'>xxxxx-xxxxx-xxxxx</font> |
| `payment_id` | _String_ | Required<br/>Up to 255 characters | Payment ID (public) <br />**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font> |
| `hash` | _String_ | Required | Special signature to validate your request to Payment Platform Addition in Signature section.<br/><ins>Must be SHA1 of MD5 encoded string (uppercased):</ins> <font color='grey'>payment_id + merchant_pass</font> |

<details>
	<summary markdown="span">Request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/payment/retry' \
--header 'Content-Type: application/json' \
--data-raw '{
   "merchant_key":"xxxxx-xxxxx-xxxxx",
   "payment_id":"63c781cc-de3d-11eb-a1f1-0242ac130006",
   "hash":"{{operation_hash}}"
}'
```
</details>

### Response Parameters

| **Parameter** | **Type** | **Mandatory, Limitations**| **Description**|
| ---- | :------: | ----------------------- | ------------- |
| `payment_id` | _String_  | Required<br/>Up to 255 characters | Payment ID (public) <br />**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font> |
| `result` | _String_  | Required | Retry request result <br />**Example:** <font color='grey'>accepted</font> |

<details>
	<summary markdown="span">Response example</summary>

```
{
  "payment_id": "63c781cc-de3d-11eb-a1f1-0242ac130006",
  "result": "accepted"
}
```
</details>

## Get transaction status
---

This request is sent by POST in JSON format.

To get order status you can use one the identifiers:

• ```payment_id``` – transaction ID in the Payment Platform <br/>
• ```order_id``` – merchant’s transaction ID

```
/api/v1/payment/status
```
## GET_TRANS_STATUS by ```payment_id```

> Note: The response logic for a status request depends on the Cascading Context for Get Status setting in Cofiguration --> Protocol Mappings section. If enabled, the system returns the status of the most recently created payment within the cascade (i.e., the payment with the latest creation date), rather than the payment specified in the request. 

### Request Parameters by ```payment_id```

| **Parameter** | **Type** | **Mandatory, Limitations**| **Description**|
| ------------- | :------: | ---------------------- | ------------- |
|`merchant_key`|_String_|Required| Key for Merchant identification<br />**Example:** <font color='grey'>xxxxx-xxxxx-xxxxx</font> |
|`payment_id`|_String_| Required<br/>Up to 255 characters | Payment ID (public)<br />**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font> |
|`hash`|  _String_  | Required|Special signature to validate your request to Payment Platform Addition in Signature section.<br /> <ins>Must be SHA1 of MD5 encoded string (uppercased):</ins> <font color='grey'>payment_id + merchant_pass</font> |

### Response Parameters by ```payment_id```

| **Parameter** | **Type** | **Mandatory, Limitations**| **Description**|
| ------------- | :------: | ----------------------- | ------------- |
|  `payment_id`   |  _String_ | Required <br/> Up to 255 characters|  Payment ID (public) <br />**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font> |
|     `date`      |  _String_  | Required <br/> Format: YYYY-MM-DD hh:mm:ss |  Transaction date <br />**Example:** <font color='grey'>2020-08-05 07:41:10</font> |
|    `status`     |  _String_  | Required |  Payment status: prepare, settled, pending, 3ds, redirect, decline, refund, reversal, void, chargeback<br />**Example:** <font color='grey'>settled</font> |
|    `reason`     |  _String_  | Optional |  Decline reason translation for unsuccessful payment. It displays only if the transaction is unsuccessful<br />**Example:**<font color='grey'> The operation was rejected. Please contact the site support</font> |
|    `recurring_token`     |  _String_  | Optional | Recurring token (provided if recurring was initialized)<br />**Example:**<font color='grey'> e5f60b35485e</font> |
|    `shedule_id`     |  _String_  | Optional | Schedule ID for recurring payments. Only for purchase operation <br />**Example:**<font color='grey'> 57fddecf-17b9-4d38-9320-a670f0c29ec0</font> |
|   **order**   | _Object_ |||
|    `number`     |  _String_  | Required <br/> Up to 255 characters        |  Order ID<br />**Example:** <font color='grey'>order-1234</font>                                                                                                                                                 |
|    `amount`     |  _String_  | Required <br/> Format: XX.XX               |  Product price <br />**Example:** <font color='grey'>0.19</font>                                                                                                  |
|   `currency`    |  _String_  | Required <br/> Up to 3 characters          |  Currency<br />**Example:** <font color='grey'>USD</font>                                                                                                                                                        |
|  `description`  |  _String_  | Required <br/> Up to 1024 characters       |  Product name<br />**Example:** <font color='grey'>Very important gift</font>                                                                                                                                    |
| **customer**  |    _Object_      |||
|     `name`      |  _String_  | Required| Customer's name<br />**Example:** <font color='grey'>John Doe</font>                                                                                                                                            |
|`email`|_String_| Required| Customer's email address<br />**Example:** <font color='grey'>example@mail.com</font>|
|`digital_wallet`|_String_| Optional| Wallet provider: googlepay, applepay. It returns if digital wallets were used for card payment creation. <br />**Example:** <font color='grey'>googlepay</font>|


<details>
	<summary markdown="span">Example Request</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/payment/status' \
--header 'Content-Type: application/json' \
--data-raw '{  
   "merchant_key":"xxxxx-xxxxx-xxxxx",
   "payment_id":"63c781cc-de3d-11eb-a1f1-0242ac130006",
   "hash":"{{operation_hash}}"
}
'
```
</details>

<details>
  <summary markdown="span">Example Response</summary>

<details>
  <summary markdown="span">Status Settled</summary>

```
{
  "payment_id": "24f7401c-fc47-11ec-8d07-0242ac120015",
  "date": "2022-07-05 09:45:03",
  "status": "settled",
  "recurring_token": "e5f60b35485e",
  "shedule_id ": "57fddecf-17b9-4d38-9320-a670f0c29ec0",
  "digital_wallet": "googlepay",
  "order": {
    "number": "f0a51dfa-fc43-11ec-8128-0242ac120004-1",
    "amount": "3.01",
    "currency": "USD",
    "description": "bloodline"
  },
  "customer": {
    "name": "John Doe",
    "email": "success@gmail.com"
  }
}
```
</details>

<details>
  <summary markdown="span">Status Decline</summary>

```
{
  "payment_id": "03e46e96-de42-11eb-aea7-0242ac140002",
  "date": "2021-07-06 10:07:47",
  "status": "decline",
  "reason": "Declined by processing",
  "digital_wallet": "googlepay",
  "order": {
    "number": "order-1234",
    "amount": "0.19",
    "currency": "USD",
    "description": "Important gift"
  },
  "customer": {
    "name": "John Doe",
    "email": "test@mail.com"
  }
}
```
</details>
</details>

## GET_TRANS_STATUS by ```order_id```

Use this request to get the status of the most recent transaction in the order's transaction subsequence.

It is recommended to send an unique order number (the ```order.number``` parameter) in the Authorization request. That way, it will be easier to uniquely identify the payment by ```order_id```. This is especially important if cascading is configured. In this case, several intermediate transactions could be created within one payment.

### Request Parameters by ```order_id```

| **Parameter** | **Type** | **Mandatory, Limitations**| **Description**|
| ------------- | :------: | ---------------------- | ------------- |
|`merchant_key`|_String_|Required| Key for Merchant identification<br />**Example:** <font color='grey'>xxxxx-xxxxx-xxxxx</font>                                                                                |
|`order_id`|_String_| Required<br/>Up to 255 characters | Merchant’s Order Number<br/>**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font>                                                                     |
|`hash`|  _String_  | Required|Special signature to validate your request to Payment Platform Addition in Signature section.<br /> <ins>Must be SHA1 of MD5 encoded string (uppercased):</ins> <font color='grey'>order_id + merchant_pass</font> |

### Response Parameters by ```order_id```

| **Parameter** | **Type** | **Mandatory, Limitations**| **Description**|
| ------------- | :------: | ----------------------- | ------------- |
|`payment_id`|  _String_ | Required <br/> Up to 255 characters|  Transaction ID in Payment Platform <br />**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font>|
|`date`|_String_| Required <br/> Format: YYYY-MM-DD hh:mm:ss |  Transaction date <br />**Example:** <font color='grey'>2020-08-05 07:41:10</font>|
|`status`|_String_| Required<br/>Up to 20 characters|  Payment status: prepare, settled, pending, 3ds, redirect, decline, refund, reversal, void, chargeback<br />**Example:** <font color='grey'>settled</font>|
|`reason`|_String_| Optional<br/>Up to 1024 characters|  Decline reason translation for unsuccessful payment. It displays only if the transaction is unsuccessful<br />**Example:**<font color='grey'> The operation was rejected. Please contact the site support</font> |
|`recurring_token`|_String_| Optional | Recurring token (provided if recurring was initialized)<br />**Example:**<font color='grey'> e5f60b35485e</font> |
|`shedule_id`|_String_| Optional | Schedule ID for recurring payments. Only for purchase operation <br />**Example:**<font color='grey'> 57fddecf-17b9-4d38-9320-a670f0c29ec0</font> |
|**order**|_Object_|||
|`number`|_String_| Required <br/> Up to 255 characters| Merchant’s Order ID <br />**Example:** <font color='grey'>order-1234</font>|
|`amount`|_String_| Required <br/> Format: XX.XX| Product price (currency will be defined by the first payment) <br />**Example:** <font color='grey'>0.19</font>                                                                                                  |
|`currency`|_String_| Required <br/> Up to 3 characters| Currency<br />**Example:** <font color='grey'>USD</font>|
|`description`|_String_| Required <br/> Up to 1024 characters| Product name or other additional information<br/>**Example:** <font color='grey'>Very important gift</font>|
| **customer**  |_Object_|||
|`name`|_String_| Required| Customer's name<br />**Example:** <font color='grey'>John Doe</font>|
|`email`|_String_| Required| Customer's email address<br />**Example:** <font color='grey'>example@mail.com</font>|
|`digital_wallet`|_String_| Optional| Wallet provider: googlepay, applepay. <br/> It is returned if digital wallets were used for card payment<br />**Example:** <font color='grey'>googlepay</font>|

<details>
	<summary markdown="span">Example Request</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/payment/status' \
--header 'Content-Type: application/json' \
--data-raw '{  
   "merchant_key":"xxxxx-xxxxx-xxxxx",
   "order_id":"63c781cc-de3d-11eb-a1f1-0242ac130006",
   "hash":"{{operation_hash}}"
}
'
```
</details>

<details>
  <summary markdown="span">Example Response</summary>

<details>
  <summary markdown="span">Status Settled</summary>

```
{
    "payment_id": "24f7401c-fc47-11ec-8d07-0242ac120015",
    "date": "2022-07-05 09:45:03",
    "status": "settled",
    "recurring_token": "e5f60b35485e",
    "shedule_id ": "57fddecf-17b9-4d38-9320-a670f0c29ec0",
    "digital_wallet": "googlepay",
    "order": {
        "number": "f0a51dfa-fc43-11ec-8128-0242ac120004-1",
        "amount": "3.01",
        "currency": "USD",
        "description": "bloodline"
    },
    "customer": {
        "name": "John Doe",
        "email": "success@gmail.com"
    }
}
```
</details>

<details>
  <summary markdown="span">Status Decline</summary>

```
{
  "payment_id": "03e46e96-de42-11eb-aea7-0242ac140002",
  "date": "2021-07-06 10:07:47",
  "status": "decline",
  "reason": "Declined by processing",
  "digital_wallet": "googlepay",
  "order": {
    "number": "order-1234",
    "amount": "0.19",
    "currency": "USD",
    "description": "Important gift"
  },
  "customer": {
    "name": "John Doe",
    "email": "test@mail.com"
  }
}
```
</details>
</details>

## Refund
---

To make refund you can use Refund request. Use payment public ID from Payment Platform in the request.

This request is sent by POST in JSON format.

**Refund request**

```
/api/v1/payment/refund
```

### Request Parameters

| **Parameter** | **Type** | **Mandatory, Limitations**| **Description**|
| ------------- | :------: | ------------------------| --------------|
| `merchant_key`  |  _String_  | Required| Key for Merchant identification<br />**Example:** <font color='grey'>xxxxx-xxxxx-xxxxx</font>                                                                                         |
|  `payment_id`   |  _String_  | Required<br />Up to 255 characters| Payment ID (public)<br />**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font>|
|    `amount`     |  _String_  | Required<br />Format: XX.XX, without leading zeroes | Amount to refund. It is required for particular refund.<br />**Example:** <font color='grey'>0.19</font>      |
|     `hash`      |  _String_  | Required| Special signature to validate your request to Payment Platform Addition in Signature section.<br /> <ins>Must be SHA1 of MD5 encoded string (uppercased):</ins> <font color='grey'>payment.id + amount + merchant.pass</font> |

### Response Parameters

| **Parameter** | **Type** | **Mandatory, Limitations**|**Description**|
| ------------- | :------: | ----------------------- | ------------ |
|  `payment_id`   |  _String_  | Required Up to 255 characters | Transaction ID (public)<br />**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font> |
|    `result`     |  _String_  | Required| Refund request result **Example:** <font color='grey'>accepted</font>|

<details>
	<summary markdown="span">Example Request</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/payment/refund' \
--header 'Content-Type: application/json' \
--data-raw '{  
   "merchant_key":"xxxxx-xxxxx-xxxxx",
   "payment_id":"63c781cc-de3d-11eb-a1f1-0242ac130006",
   "amount":"0.10",
   "hash":"{{operation_hash}}"
}
'
```
</details>

<details>
	<summary markdown="span">Example Response: Refund request accepted</summary>

```
{
  "payment_id": "63c781cc-de3d-11eb-a1f1-0242ac130006",
  "result": "accepted"
}
```
</details>

## Void
---

To make a void for an operation which was performed the same financial day you can use Void request.

The Void request is allowed for the payments in SETTLED status only.

Use payment public ID from Payment Platform in the request.

**Void request**

```
/api/v1/payment/void
```

### Request Parameters

| **Parameter** | **Type** | **Mandatory, Limitations**| **Description**|
| ------------- | :------: | ------------------------| --------------|
| `merchant_key`  |  _String_  | Required| Key for Merchant identification<br />**Example:** <font color='grey'>xxxxx-xxxxx-xxxxx</font>                                                                                         |
|  `payment_id`   |  _String_  | Required Up to 255 characters| Payment ID (public)<br />**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font>|
|     `hash`      |  _String_  | Required| Special signature to validate your request to Payment Platform Addition in Signature section.<br /> <ins>Must be SHA1 of MD5 encoded string (uppercased):</ins> <font color='grey'>payment.id + merchant.pass</font> |

### Response Parameters

| **Parameter** | **Type** | **Mandatory, Limitations**|**Description**|
| ------------- | :------: | ----------------------- | ------------ |
|  `status` |  _String_  | Required | Payment status<br />**Example:** <font color='grey'>VOID / SETTLED</font> |
|  `payment_id`   |  _String_  | Required Up to 255 characters | Transaction ID (public)<br />**Example:** <font color='grey'>dc66cdd8-d702-11ea-9a2f-0242c0a87002</font> |
| `date` |  _String_  | Required<br /> Format: YYYY-MM-DD hh:mm:ss| Transaction date<br /> **Example:** <font color='grey'>2020-08-05 07:41:10</font>|
| `reason` |  _String_  | Optional | Decline or error reason (for "sale", "void "and "refund" operation types). It displays only if the transaction has FAIL status <br />**Example:** <font color='grey'>The operation was rejected. Please contact the site support</font> |
|   **order**   |     _Object_     |||
|    `number`     |  _String_  | Required |  Order ID<br />**Example:** <font color='grey'>order-1234</font>                                                                                                                                                 |
|    `amount`     |  _String_  | Required |  Product price <br />**Example:** <font color='grey'>0.19</font>                                                                                                  |
|   `currency`    |  _String_  | Required Up to 3 characters|  Currency<br />**Example:** <font color='grey'>USD</font>                                                                                                                                                        |
|  `description`  |  _String_  | Required Up to 1024 characters |  Product name<br />**Example:** <font color='grey'>Very important gift</font>                                                                                                                                    |

<details>
	<summary markdown="span">Example Request</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/payment/void' \
--header 'Content-Type: application/json' \
--data-raw '{  
   "merchant_key":"xxxxx-xxxxx-xxxxx",
   "payment_id":"63c781cc-de3d-11eb-a1f1-0242ac130006",
   "hash":"{{operation_hash}}"
}
'
```
</details>

<details>
	<summary markdown="span">Example Response: Void successful </summary>

```
{
  "status": "SUCCESS", 
  "payment_status": "void",
  "payment_id": "dc66cdd8-d702-11ea-9a2f-0242c0a87002",
  "date": "2020-08-05 07:41:10",
  "order": {
    "number": "order-1234",
    "amount": "0.19",
    "currency": "USD",
    "description": "very important gift"
  }
}
```
</details>

<details>
	<summary markdown="span">Example Response: Void failed</summary>

```
{
  "status": "FAIL", 
  "payment_status": "settled",
  "payment_id": "dc66cdd8-d702-11ea-9a2f-0242c0a87002",
  "date": "2020-08-05 07:41:10",
  "reason": "Declined by processing",
   "order": {
    "number": "order-1234",
    "amount": "0.19",
    "currency": "USD",
    "description": "very important gift"
  }
}
```
</details>

## Signature

---

Sign is signature rule used either to validate your requests to payment platform or to validate callback from payment platform to your system.

### Authentication Signature

It must be SHA1 of MD5 encoded string and calculated by the formula below:

*sha1(md5(strtoupper(id.order.amount.currency.description.PASSWORD)))

// Use the CryptoJSvar

#### Example for JS

```javascript
var to_md5 = order.number + order.amount + order.currency + order.description + merchant.pass;
```
// Use the CryptoJS

_var hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());_

_var result = CryptoJS.enc.Hex.stringify(hash);_

### Get Transaction Status Signature (by ```payment_id```)

It must be SHA1 of MD5 encoded string and calculated by the formula below:

_hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());_

_*sha1(md5(strtoupper))_

// Use the CryptoJSvar

#### Example for JS

```javascript
var to_md5 = payment.id + merchant.pass;
```

// Use the CryptoJS

_var hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());_

_var result = CryptoJS.enc.Hex.stringify(hash);_

### Get Transaction Status Signature (by ```order_id```)

It must be SHA1 of MD5 encoded string and calculated by the formula below:

_hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());_

_*sha1(md5(strtoupper))_

// Use the CryptoJSvar

#### Example for JS

```javascript
var to_md5 = order.id + merchant.pass;
```

// Use the CryptoJS

_var hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());_

_var result = CryptoJS.enc.Hex.stringify(hash);_


### Refund Signature

It must be SHA1 of MD5 encoded string and calculated by the formula below:

_hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString()); \*sha1(md5(strtoupper))_

// Use the CryptoJSvar

#### Example for JS

```javascript
var to_md5 = payment.id + amount + merchant.pass;
```

// Use the CryptoJS

_var hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());_

_var result = CryptoJS.enc.Hex.stringify(hash);_

### Void Signature

It must be SHA1 of MD5 encoded string and calculated by the formula below:

_hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString()); \*sha1(md5(strtoupper))_

// Use the CryptoJSvar

#### Example for JS

```javascript
var to_md5 = payment.id + merchant.pass;
```

// Use the CryptoJS

_var hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());_

_var result = CryptoJS.enc.Hex.stringify(hash);_

### Recurring Signature

It must be SHA1 of MD5 encoded string and calculated by the formula below:

*sha1(md5(strtoupper(recurring_init_trans_id.recurring_token.order_id.amount.description.merchant.pass)))

// Use the CryptoJS

#### Example for JS

```javascript
var to_md5 = recurring_init_trans_id + recurring_token + order.number + order.amount + order.description + merchant.pass;
```

// Use the CryptoJS

var hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());

var result = CryptoJS.enc.Hex.stringify(hash);

### Signature for return

It must be SHA1 of MD5 encoded string and calculated by the formula below:

_hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());_
*sha1(md5(strtoupper(payment_public_id.order_id.amount.currency.description.PASSWORD)))

Example for JS

```javascript
var to_md5 = payment_public_id + order.number + order.amount + order.currency + order.description + merchant.pass;
```
// Use the CryptoJS

_var hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());_

_var result = CryptoJS.enc.Hex.stringify(hash);_

### Callback Signature

It must be SHA1 of MD5 encoded string and calculated by the formula below:

_hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString()); *sha1(md5(strtoupper(payment_public_id.order_id.amount.currency.description.PASSWORD)))_

#### Example for JS

```javascript
var to_md5 = payment_public_id + order.number + order.amount + order.currency + order.description + merchant.pass;
```

// Use the CryptoJS

_var hash = CryptoJS.SHA1(CryptoJS.MD5(to_md5.toUpperCase()).toString());_

_var result = CryptoJS.enc.Hex.stringify(hash);_

## Testing
---

You can test your integration. To do the test you need to perform the actions with the API using (in test mode). For instance, send the request and receive the response with a link on the Checkout page.<br/>To mark your transactions as test in the system, you should use Merchant Test Key as value of the ```merchant_key``` parameter. You can contact your administrator to make sure that you have the necessary settings to work with test transactions.


Use the following test data to emulate the different scenarios.

**Scenario: SUCCESS payment**

**Card data:**

`Card number` 4111 1111 1111 1111  
`Expiry Date` 01/38  
`CVV2` any 3 digits

⚠️ Use that card data to create recurring payments and get a recurring token for the initial recurring. Testing recurring payments does not work with other test cards.

**Result:** customer is redirected to the "success_URL"


**Scenario: FAILED payment (decline)**

**Card data:**

`Card number` 4111 1111 1111 1111  
`Expiry Date` 02/38  
`CVV2` any 3 digits

**Result:** customer gets an error message: Declined by processing.

**Scenario: SUCCESS 3DS payment**

**Card data:**

`Card number` 4111 1111 1111 1111  
`Expiry Date` 05/38  
`CVV2` any 3 digits

**Result:** customer is redirected to the "success_URL" after 3DS verification

**Scenario: FAILED 3DS payment**

**Card data:**

`Card number` 4111 1111 1111 1111  
`Expiry Date` 06/38  
`CVV2` any 3 digits

**Result:** customer gets unsuccessful sale after 3DS verification


**Scenario: FAILED payment (Luhn algorithm)**

**Card data:**

`Card number` 1111 2222 3333 4444  
`Expiry Date` 01/38  
`CVV2` any 3 digits

**Result:** customer gets an error message: Bad Request. Brand of card does not support.

## Payment methods
---
You can choose  the payment methods that you want to display on the payment form. 
Use ```methods``` array in the Authentication request and specify the values that correspond to the payment methods.
As well, you can use the value of the payment method to add specific parameters to the Authentification request for the ```paramaters``` object. These parameters will be sent to the acquirer to pass the necessary data for successful payment.

### card
If the payer chooses the ```card``` payment method, fields with card data will be displayed on the Checkout page.

For some connector services, it is necessary to send additional parameters in the Authenticaion request for the ```paramaters``` object (for more information, contact your manager).

#### Additional parameters - Set 1 (BNG)
| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```bnrg_installm_def``` | _Numeric justified to 2 digits_ | Required |Indicates the number of months that will elapse from the purchase until the total or partial charge is made to the cardholder's account (initial deferral).<br></br>Possible values:<br></br> ```01``` - one month <br></br>```00``` - no delay initial|
| ```bnrg_installm_months``` | _Numeric justified to 2 digits_ | Required |Indicates the number of monthly payments in which the total amount of the transaction will be divided.<br></br>Example: ```03``` - 3 months|
| ```bnrg_installm_plan```| _Numeric justified to 2 digits_ | Required |Indicates if the promotion It will be ́ with interest or without interest. <br></br>Possible values:<br></br>```03``` - no interest<br></br>```05``` - with interest<br></br>```07``` - defer only initial.|

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "card"
    ],
        "parameters": {
                "card": {
            "bnrg_installm_def": "03",
            "bnrg_installm_months": "03",
            "bnrg_installm_plan": "03"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "MXN",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}",
}
'
```
</details>

#### Additional parameters - Set 2 (FCP)
| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```document_type``` |_String_| Required |Type of the document number specified above. Possibe values: cpf|
| ```document_number``` |_String_| Required |Client’s document number|
| ```social_name``` |_String_| Required | Client's social name|
| ```fiscal_country``` |_String_| Required | An alpha-2 country code of the customer’s tax country|

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "card"
    ],
        "parameters": {
                "card": {
            "document_type": "cpf",
            "document_number": "231.002.999-00",
            "social_name": "Harry Potter",
            "fiscal_country": "BR",
            "toBankAccountId": "a1a1134a-32c6-442c-90c9-66b587d5be00"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "BRL",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}",
}'
```
</details>

#### Additional parameters - Set 3 (LBP)
| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```descriptor``` |_String_ (20 characters)| Optional |Transaction descriptor |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "card"
    ],
        "parameters": {
                "card": {
            "descriptor": "Testy International LTD"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "BRL",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}",
}
'
```
</details>

#### Additional parameters - set 4 TWD

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```customer_phone``` | _String_ | Optional | Phone number of the customer.|
| ```customer_telnocc``` | _String_ | Optional | Country phone code of the customer. |
| ```shipping_street1``` | _String_ | Optional | Building name, and/or street name of the customer's shipping address. |
| ```shipping_city``` | _String_ | Optional | City of the customer's shipping address. |
| ```shipping_state``` | _String_ | Optional | State or region of the customer's shipping address. |
| ```shipping_postcode``` | _String_ | Optional | Postal code/ Zip code of the customer's shipping address.<br />Max – 9 digits. |
| ```shipping_country``` | _String_ | Optional | Country of the shipping address.<br />3-digits code |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "purchase",
  "methods": [
    "card"
  ],
  "parameters": {
    "card": {
      "customer_phone": "1234567890",
      "customer_telnocc ": "123",
      "shipping_street1": " Moor Building 35274",
      "shipping_ city": "Los Angeles",
      "shipping_state": "CA",
      "shipping_postcode": "809654321",
      "shipping_ country": "US"
    }
  },
  "order": {
    "number": "order-1234",
    "amount": "1000.19",
    "currency": "USD",
    "description": "Important gift"
  },
  "cancel_url": "https://example.com/cancel",
  "success_url": "https://example.com/success",
  "customer": {
    "name": "John Doe",
    "email": "test@gmail.com"
  },
  "billing_address": {
    "country": "US",
    "state": "CA",
    "city": "Los Angeles",
    "address": "Moor Building 35274",
    "zip": "123456",
    "phone": "347771112233"
  },
  "hash": "{{session_hash}}"
}
'
```
</details>

#### Additional parameters - Set 5 (ERS)
This set should be used for debit operation.

| Parameter               | Type     | Mandatory | Description                                                  |
|-------------------------|----------|-----------|:--------------------------------------------------------------:|
| ```payer_identity_type```     | _String_ | Optional  | The type of Senders identification document                 |
| ```payer_identity_id```       | _String_ | Optional  | The number of Senders identification document               |
| ```payer_identity_country```  | _String_ | Optional  | The country of issuance of the Senders identification document |
| ```payer_identity_exp_date``` | _String_ | Optional  | The expiration date of the Senders identification document  |
| ```payer_nationality```       | _String_ | Optional  | Senders nationality                                         |
| ```payer_country_of_birth```  | _String_ | Optional  | Senders country of birth                                    |
| ```payee_first_name```        | _String_ | Optional  | Receivers first name                                        |
| ```payer_last_name```         | _String_ | Optional  | Receivers last name                                         |
| ```payee_middle_name```       | _String_ | Optional  | Receivers middle name                                       |
| ```payee_address```           | _String_ | Optional  | Receivers street                                            |
| ```payee_city```              | _String_ | Optional  | Receivers city                                              |
| ```payee_state```             | _String_ | Optional  | Receivers state                                             |
| ```payee_country```           | _String_ | Optional  | Receivers country                                           |
| ```payee_zip```               | _String_ | Optional  | Receivers postal code                                       |
| ```payee_phone```             | _String_ | Optional  | Receivers phone number                                      |
| ```payee_birth_date```        | _String_ | Optional  | Receivers date of birth                                     |
| ```payee_identity_type```     | _String_ | Optional  | The type of Receivers identification document               |
| ```payee_identity_id```       | _String_ | Optional  | The number of Receivers identification document             |
| ```payee_identity_country```  | _String_ | Optional  | The country of issuance of Receivers identification document |
| ```payee_identity_exp_date``` | _String_ | Optional  | The expiration date of Receivers identification document  |
| ```payee_nationality```       | _String_ | Optional  | Receivers nationality                                       |
| ```payee_country_of_birth```  | _String_ | Optional  | Receivers country of birth                                  |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "purchase",
  "methods": [
    "card"
  ],
  "parameters": {
    "card": {
      "payer_identity_type": "Passport",
      "payer_identity_id": "ABC123456",
      "payer_identity_country": "KZ",
      "payer_identity_exp_date": "2026-05-15",
      "payer_nationality": "Kazakhstani",
      "payer_country_of_birth": "Kazakhstan",
      "payee_first_name": "John",
      "payer_last_name": "Doe",
      "payee_middle_name": "Smith",
      "payee_address": "123 Main Street",
      "payee_city": "Almaty",
      "payee_state": "Almaty Province",
      "payee_country": "Kazakhstan",
      "payee_zip": "050000",
      "payee_phone": "77123456789",
      "payee_birth_date": "1990-01-01",
      "payee_identity_type": "National ID",
      "payee_identity_id": "XYZ987654",
      "payee_identity_country": "Kazakhstan",
      "payee_identity_exp_date": "2030-12-31",
      "payee_nationality": "Kazakhstani",
      "payee_country_of_birth": "Kazakhstan"
    }
  },
  "order": {
    "number": "order-1234",
    "amount": "1000.19",
    "currency": "USD",
    "description": "Important gift"
  },
  "cancel_url": "https://example.com/cancel",
  "success_url": "https://example.com/success",
  "customer": {
    "name": "John Doe",
    "email": "test@gmail.com"
  },
  "billing_address": {
    "country": "US",
    "state": "CA",
    "city": "Los Angeles",
    "address": "Moor Building 35274",
    "zip": "123456",
    "phone": "347771112233"
  },
  "hash": "{{session_hash}}"
}
'
```
</details>


### afsbenefit
payment_method = ```afsbenefit```

### airtel
payment_method = ```airtel```

### paydefi

If the payer chooses the ```allpay``` payment method, the customer’s confirmation via app may be necessary to finish the payment.

### Apple Pay

To use the ```applepay``` payment method, configure the Wallets setting in the admin dashboard. Ensure that the payment provider has confirmed the Apple Pay certificates and verified your domain in the Apple Pay account.

To enable payers to make Apple Pay payments, you can either connect to the Checkout page or work directly via the S2S protocol:

**Checkout Page**: No additional development is required on your side.

**S2S Protocol**: You must be able to obtain the Apple Pay payment token and include it in the SALE request.

Regardless of the protocol you have to make settings in the admin panel and set the following configurations:

  * Merchant Identifier, Country and Shop Name – according to the merchant’s data from [<font color=''><ins>Apple Pay Developer account</ins></font>](https://idmsa.apple.com/IDMSWebAuth/signin?appIdKey=891bd3417a7776362562d2197f89480a8547b108fd934911bcbea0110d07f757&path=%2Faccount%2Fresources%2Fidentifiers%2Flist%2Fmerchant&rv=1)<br/>
  * Certificate and Private Key – generated Apple Pay certificate and private key<br/>
  * Merchant Capabilities and Supported Networks – configurations for Apple Pay payments<br/>
  * Processing Private Key – private key that uses for payment token decryption (requires if payment provider supports).<br/>

We also recommend checking out the demo provided by Apple for Apple Pay: https://applepaydemo.apple.com/

---

**Set up Apple Pay**

If you are using the Checkout protocol for Apple Pay payments, you will need to configure your Apple Developer account. Follow these steps to set up Apple Pay in your Apple Developer account:

**1.** **Create a Merchant ID** in the "Certificates, Identifiers & Profiles" section.

**2.** **Register and verify the domains** involved in the interaction with Apple Pay (e.g., the checkout page URL where the Apple Pay button is placed) in the "Merchant Domains" section. Download the verification file and provide it to support to complete the domain verification.

**3.** **Create a Merchant Identity Certificate** in the "Merchant Identity Certificate" section:
   * Generate a pair of certificates (\*.csr and \*.key) and upload the *.csr file in the "Merchant Identity Certificate" section.
   * Create a Merchant Identity Certificate based on the generated CSR file.
   * Download the *.pem file from the portal.

⚠️ **Note**<br/>
> The \*.csr file can be obtained from the payment provider that processes Apple Pay, and the \*.pem file should be uploaded to your payment provider's dashboard, following their instructions.
>

**4.** **Create a Processing Private Key** in the "Apple Pay Payment Processing Certificate" section:
  * Generate a pair of certificates (\*.csr and \*.key) and upload the *.csr file.
  * Create a Payment Processing Certificate based on the generated CSR file.

For more detailed instructions on setting up Apple Pay, refer to the following resource: [<font color=''><ins>Learn more about setting up Apple Pay</ins></font>](https://developer.apple.com/documentation/passkit_apple_pay_and_wallet/apple_pay/setting_up_apple_pay)<br/>

Next, enter the data from your Apple Pay developer account into the platform's admin panel:
Go to the "Merchants" section, initiate editing, open the "Wallets" tab, navigate to the Apple Pay settings, and fill in the following fields:
  * **Merchant Identifier:** Enter the Merchant ID.
  * **Certificate:** Upload the *.pem file downloaded from the "Merchant Identity Certificate" section.
  * **Private Key:** Upload the *.key file from the certificate pair generated for the Merchant Identity Certificate.
  * **Processing Private Key (required for token decryption)**: Paste the text from the Processing Private Key file. Ensure it is a single line of text (without spaces or breaks) placed between "BEGIN" and "END."

---

  **Apple Pay Payment Flow**


By default, all Apple Pay payments on the platform are classified as virtual. As a result: <br/>
* Card details are not stored for these transactions.
*	Functionality is limited for DMS payments and the creation of recurring transactions. <br/>

You can maximize the benefits of Apple Pay payments by leveraging card flow for digital wallets:
*	Access to post-transaction operations, such as capture in DMS mode
*	Support for recurring payments (MIT) – scheduled or on-demand
*	Smart routing for optimized payment processing
*	Payment cascading for improved success rates

>Note! Ensure your provider supports these operations.

To access the card flow for Apple Pay payments, you need to:
*	Set up the Processing Private Key in the admin panel settings ("Merchants" section → "Wallets" tab) to enable token decryption.
*	Verify that your payment provider supports card flow processing (ask your support if available).<br/>

How It Works:
*	If both requirements are met, the system will always decrypt the Apple Pay token during payment and store the decrypted card data for future transactions.
*	You can view decrypted card details in the Transaction Details section of the admin panel.
*	If any requirement is not met, the payment will be processed using the virtual flow instead.
*	If your provider supports card flow but does not accept decrypted data, platform can still store the card details for processing. However, some features (e.g., smart routing and cascading) will not be available.


### araka
payment_method = ```araka```

### astropay

If the payer chooses the ```astropay``` payment method, the redirection to another page will happen to finish the payment.

### awcc
You can add to the Authentication request a specific list of parameters which are possible for the ```awcc``` payment method.

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```network_type``` |_String_| Optional |Network Type of cryptoType, if applicable. <br/>Currently, only the symbol USDT have different network types. <br/>The following network types are available for USDT: ```eth``` (default) or ```tron```|
| ```bech32``` |_Boolean_| Optional |	Defaults to false. <br/>If set to true, it will return bech32 segwit address for BTC address, or BCH cash address for BCH. |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "awcc"
    ],
        "parameters": {
                "awcc": {
            "network_type": "eth",
            "bech32": "false"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "BRL",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}",
}
'
```
</details>


### axxi-cash
If the payer chooses the ```axxi-cash``` payment method, the _ID document_ field will be additionally displayed on the Checkout page.
After the _Pay_ button is clicked, the payer will be redirected to the page with the PDF payment ticket. 

### axxi-transfer
If the payer chooses the ```axxi-transfer``` payment method, the additional fields will be displayed on the Checkout page. The set of the fields depends on the payer's billing country.

There is a list of possible additional fields on the Checkout page:

  - _Choose your bank_ ;
  - _Person Type_ ;
  - _Document Type_ ;
  - _ID document_ .

A list of the fields depends on payer's country.

After the fields are filled, the payer will be redirected to the bank's site to complete the transfer.

However, for some billing countries (for example, Mexico), redirection does not happen. Instead, the payment details will be displayed on the Checkout page. The payer needs to use them in order to complete the transfer directly at the bank.

### axxi-pin
If the payer chooses the ```axxi-pin``` payment method, the PIN voucher field will be displayed on the Checkout page. The payer should fill in the field to redeem the voucher and see the message with the result of the operation.

### a2a_transfer
payment_method = ```a2a_transfer```

### beeline

If the payer chooses the ```beeline``` payment method, it needs to be specified the phone numbers of payer and (optionally) of receiver. The OTP code will be sent to the payer to confirm the payment. 

### billplz
If the payer chooses the ```billplz``` payment method, the customer’s confirmation via app may be necessary to finish the payment.
You can add to the Authentication request a specific list of parameters which are possible for the ```billplz``` payment method.

| Parameter               | Type          | Mandatory | Description                                 |
|-------------------------|---------------|-----------|:---------------------------------------------:|
| ```bank_code```              | _String_      | Required  | SWIFT Bank Code that represents bank.       |
| ```bank_account_number```    | _String_      | Required  | Bank account number.                        |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "purchase",
  "methods": [
    "billplz"
  ],
  "parameters": {
    "billplz": {
      "bank_code": "DUMMYBANKVERIFIED",
      "bank_account_number ": "232673199763"
    }
  },
  "order": {
    "number": "order-1234",
    "amount": "1000.19",
    "currency": "MYR",
    "description": "Important gift"
  },
  "cancel_url": "https://example.com/cancel",
  "success_url": "https://example.com/success",
  "customer": {
    "name": "John Doe",
    "email": "test@gmail.com"
  },
  "billing_address": {
    "country": "US",
    "state": "CA",
    "city": "Los Angeles",
    "address": "Moor Building 35274",
    "zip": "123456",
    "phone": "347771112233"
  },
  "hash": "{{session_hash}}"
}
'
```
</details>

### bitolo-inr
payment_method = ```bitolo-inr```

### bitolo-brl
payment_method = ```bitolo-brl```

### bpwallet
If the payer chooses the ```bpwallet``` payment method, the redirection to another page will happen to finish the payment.

### cardpaymentz

If the payer chooses the ```cardpaymentz``` payment method, the redirection to another page will happen to finish the payment.

### citizen
payment_method = ```citizen```

### cnfmo
If the payer chooses the ```cnfmo``` payment method, the redirection to another page will happen to finish the payment.

### coinspaid
payment_method = ```coinspaid```

### cup-on
payment_method = ```cup-on```

### crypto-btg
Use the ```crypto-btg``` payment method to perform cryptocurrency payments without specifying a fixed amount. If the payer selects the ```crypto-btg``` payment method, a crypto wallet address will be generated and displayed at checkout.

### cybersource
payment_method = ```cybersource```

### dcp
If the payer chooses the ```dcp``` payment method, the redirection to another page will happen to finish the payment.

### dl
If the payer chooses the ```dl``` payment method, the redirection to another page will happen to finish the payment.
The following parameter is required:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```document_number``` | _String_ | Conditional| Customer's ID.<br/> **Condition:** If the parameter is NOT specified in the request, then it will be collected on the Checkout page |
<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "purchase",
  "methods": [
    "dl"
  ],
  "parameters": {
      "dl": {
        "document_number": "document_number"
      }
   },
  "order": {
    "number": "order-1234",
    "amount": "100",
    "currency": "USD",
    "description": "Important gift"
  },
  "cancel_url": "https://example.domain.com/cancel",
  "success_url": "https://example.domain.com/success",
  "customer": {
    "name": "John Doe",
    "email": "test@gmail.com"
  },
  "billing_address": {
    "country": "BR",
    "city": "city",
    "address": "address",
    "house_number": "17/2",
    "zip": "123456",
    "phone": "347771112233"
  },
  "hash": "{{session_hash}}"
}'
```
</details>

### dlocal
If the payer chooses the ```dlocal``` payment method, the redirection to another page will happen to finish the payment.
The following parameter is required:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```document_number``` | _String_ | Conditional| Customer's ID.<br/> **Condition:** If the parameter is NOT specified in the request, then it will be collected on the Checkout page |
<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "purchase",
  "methods": [
    "dlocal"
  ],
  "parameters": {
      "dlocal": {
        "document_number": "document_number"
      }
   },
  "order": {
    "number": "order-1234",
    "amount": "100",
    "currency": "USD",
    "description": "Important gift"
  },
  "cancel_url": "https://example.domain.com/cancel",
  "success_url": "https://example.domain.com/success",
  "customer": {
    "name": "John Doe",
    "email": "test@gmail.com"
  },
  "billing_address": {
    "country": "BR",
    "city": "city",
    "address": "address",
    "house_number": "17/2",
    "zip": "123456",
    "phone": "347771112233"
  },
  "hash": "{{session_hash}}"
}'
```
</details>

### doku-hpp
If the payer chooses the ```doku-hpp``` payment method, the redirection to another page will happen to finish the payment.
You can add to the Authentication request a specific list of parameters which may be required for the ```doku-hpp``` payment method.


| Parameter               | Type          | Mandatory | Description                                 |
|-------------------------|---------------|-----------|:---------------------------------------------:|
| ```product_reference_id```              | _String_      | Conditional  | SKU/item ID of the item in this transaction. This parameter is mandatory if you want to use Akulaku, Kredivo and Indodana. Allowed chars: alphabetic, numeric, special chars. Max Length: 64.       |
| ```product_name```    | _String_      | Conditional  | Name of the product item. This parameter is mandatory if you want to use Kredivo, Jenius and Indodana. Allowed chars: alphabetic, numeric, special chars. Max Length: 255                  |
| ```product_quantity```    | _Number_      | Conditional  | Quantity of the product item. This paramater mandatory if you want to use Kredivo, Akulaku, Indodana and Jenius. Allowed chars: numeric. Max Length: 4                     |
| ```product_sku```    | _String_      | Conditional  | SKU of the product item. This paramater mandatory if you want to use Kredivo, Akulaku and Indodana.                        |
| ```product_category```    | _String_      | Conditional | Category of the product item. This paramater mandatory if you want to use Kredivo, Akulaku and Indodana.                         |
| ```product_url```    | _String_      | Conditional  | URL to the product item on merchant site. This paramater mandatory if you want to use Kredivo and Indodana.                         |
| ```product_image_url```    | _String_      | Conditional  | URL (image) of the product item on merchant site. This paramater mandatory if you want to use Indodana.                        |
| ```product_type```    | _String_      | Conditional | Type of the item in this transaction. This paramater mandatory if you want to use Indodana                       |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "purchase",
  "methods": [
    "doku-hpp"
  ],
  "parameters": {
    "doku-hpp`": {
      "product_reference_id": "1",
      "product_name": "Fresh flowers",
      "product_quantity": "1",
      "product_sku": "FF01",
      "product_category": "gift-and-flowers",
      "product_url": "http://example.com/",
      "product_image_url": "http://example.com/",
      "product_type": "ABC"
    }
  },
  "order": {
    "number": "order-1234",
    "amount": "1000",
    "currency": "IDR",
    "description": "Important gift"
  },
  "cancel_url": "https://example.com/cancel",
  "success_url": "https://example.com/success",
  "customer": {
    "name": "John Doe",
    "email": "test@gmail.com"
  },
  "billing_address": {
    "country": "US",
    "state": "CA",
    "city": "Los Angeles",
    "address": "Moor Building 35274",
    "zip": "123456",
    "phone": "347771112233"
  },
  "hash": "{{session_hash}}"
}
'
```
</details>


### dpbanktransfer
If the payer chooses the ```dpbanktransfer``` payment method, the additional information must be provided by the payer on the Checkout page.

The payer should enter the details about the beneficiary and select an account from the list of accounts available for transfer.

The confirmation code can be requested at any stage of the transfer.

### e-xezine
payment_method = ```e-xezine```

### fairpay

If the payer chooses the ```fairpay``` payment method, the set of the fields on the Checkout page depends on the payer's billing country.
Personal Identification Number is required if payer specifies one of the following countries: Brazil, Chile, and Colombia.

### fawry

If the payer chooses the ```fawry``` payment method, the customer’s confirmation via app may be necessary to finish the payment.

### fxmb-india
payment_method = ```fxmb-india```

### fxmb-netbanking
payment_method = ```fxmb-netbanking```

### gigadat
If the payer chooses the ```gigadat``` payment method, the redirection to another page will happen to finish the payment.

### Google Pay

To provide the payers with the possibility of Google Pay™ payment you can connect to the Checkout or work directly via S2S protocol. 

If you are using **Checkout integration** to work with ```googlepay``` payment method, it requires no additional code implementation.<br/>

If you are using **S2S integration** to work with ```googlepay``` payment method, you must be able to obtain the Google Pay payment token and include it in the SALE request.


To work with Google Pay™ payments through gateway, you need to make settings in the admin panel. You can set the following configurations:<br/>
    • choose the environment: ```TEST``` or ```PRODUCTION```<br/>
    • specify "Allowed Auth Method" - ```PAN_ONLY``` or ```CRYPTOGRAM_3DS```<br/>
    • determine "Supported Networks" - MASTERCARD, VISA, AMEX, DISCOVER, JCB<br/>
These configuration will be applied and used when platform interacts with Google Pay. <br/>


⚠️ **Pay attention** <br/>

> _in the case of PAN_ONLY, the responsibility for passing 3ds is transferred to the acquirer, through which the payment is processed_

---

**Set up Google Pay**

To set up Google Pay, you first need to determine who acts as the gateway when processing payments: the platform or the payment provider. This depends on which side decrypts the Google Pay payment token.
   * If the platform decrypts the token and then sends the decrypted payment data to the payment provider, the platform acts as the gateway.
   * If the payment provider decrypts the token (i.e., expects to receive an encrypted payment token in the payment request), the payment provider is the gateway.

The gateway (whether it is the platform or the payment provider processing Google Pay) must provide the following information:

   * **Gateway**: The name of the gateway.
   * **Gateway Merchant ID**: The merchant identifier in the gateway's system.
  
⚠️ **Note** <br/>

> If the payment provider is the gateway, it is important to clarify whether they expect the token to include additional information, such as the payer's address or phone number.

The next step is to enter the data into the admin panel.

Go to the "Merchants" section, initiate editing, open the "Wallets" tab, navigate to the Google Pay settings, and fill in the following fields:
   * **Merchant Identifier**: Enter the identifier from the Google Business Console. If the platform is the gateway, you can duplicate the Gateway Merchant ID here.
   * **Gateway**: The gateway for interaction with Google Pay.
   * **Gateway Merchant ID**: The merchant identifier in the Google gateway.
   * **Private Key (required for decrypted token)**: The key for decrypting the payment token. If the platform is the gateway, the key should be provided by the platform. If the payment provider is the gateway, the key is not required.
   * **Include Additional Parameters to Google Token**: Depending on the payment provider's requirements.

Additionally, to work with Google Pay payments, you must verify the checkout domains from which Google Pay is processed in the Google Business Console. For domain verification, you may need to contact support.

---

**Google Pay Payment Flow**

By default, all Google Pay payments on the platform are classified as virtual. As a result:
*	Card details are not stored for these transactions.
*	Functionality is limited for DMS payments and the creation of recurring transactions.<br/>

You can maximize the benefits of Google Pay payments by leveraging card flow for digital wallets:<br/>
*	Access to post-transaction operations, such as capture in DMS mode
*	Support for recurring payments (MIT) – scheduled or on-demand
*	Smart routing for optimized payment processing
*	Payment cascading for improved success rates

>**Note!** Ensure your provider supports these operations.<br/>

To access the card flow for Google Pay payments, you need to:
*	Set up the Private Key in the admin panel settings ("Merchants" section → "Wallets" tab) to enable token decryption.
*	Verify that your payment provider supports card flow processing (ask your support if available).<br/>

How It Works:
*	If both requirements are met, the system will always decrypt the Google Pay token during payment and store the decrypted card data for future transactions.
*	You can view decrypted card details in the Transaction Details section of the admin panel.
*	If any requirement is not met, the payment will be processed using the virtual flow instead.
*	If your provider supports card flow but does not accept decrypted data, platform can still store the card details for processing. However, some features (e.g., smart routing and cascading) will not be available.


### hayvn

If the payer chooses the ```hayvn``` payment method, it needs to be specified the crypto currency for payment. After creating a payment, the payer will be shown the data necessary to complete the transaction.

### helio
If the payer chooses the ```helio``` payment method, the redirection to widget page will happen to finish the payment.

### help2pay
payment_method = ```help2pay```

### ideal_crdz

If the payer chooses the ideal_crdz payment method, the redirection to another page will happen to finish the payment.

### instant-bills-pay

If the payer chooses the ```instant-bills-pay``` payment method, the redirection to another page will happen to finish the payment.

### ipasspay
payment_method = ```ipasspay```

### jvz

If the payer chooses the ```jvz``` payment method, the confirmation necessary to finish the payment via APP.

### kashahpp
If the payer chooses the ```kashahpp``` payment method, the redirection to another page will happen to finish the payment.


### m2p-debit

You should add to the Authentication request a specific list of parameters in the “parameters” object that are needed for the <br /> ```m2p-debit``` payment method.

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```paymentGatewayName```  | _String_ | Required | Name of payment gateway that will be used for deposit |
| ```paymentCurrency``` | _String_ | Required | CRYPTO currency Symbol of currency that will be deposited |
| ```tradingAccountLogin``` | _String_ | Optional | Depositor’s trading account id |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "m2p-debit"
    ],
        "parameters": {
                "m2p-debit": {
            "paymentGatewayName": "gateway",
            "paymentCurrency": "BTC",
            "tradingAccountLogin": "login"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "BRL",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}",
}
'
```
</details>

### m2p-withdrawal

You should add to the Authentication request a specific list of parameters in the “parameters” object that are needed for the ```m2p-withdrawal``` payment method.

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```withdrawCurrency``` | _String_ | Required | CRYPTO currency Cryptocurrency to which currency is converted |
| ```address``` | _String_ | Required | User cryptocurrency wallet address to withdraw |
| ```tradingAccountLogin``` | _String_ | Optional | Trading account ID of a user requesting a withdrawal |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "m2p-withdrawal"
    ],
        "parameters": {
                "m2p-withdrawal": {
            "withdrawCurrency": "BTC",
            "address": "mkzePZfMeoYsoGcavSJFT7UKSjsU2BEgzt",
            "tradingAccountLogin": "login"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "BRL",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}",
}
'
```
</details>

### mcpayhpp
If the payer chooses the ```mcpayhpp``` payment method, the redirection to another page will happen to finish the payment.

### mcpayment
If the payer chooses the ```mcpayment``` payment method, the QR-code will be swown additionally. The customer needs to scan the QR-code to finish the payment.

### moov-money
This method is used for mobile numbers in Benin. If the payer chooses ```moov-money``` payment method, confirmation is required through a mobile device.

### moov-togo
This method is used for mobile numbers in Togo. If the payer chooses ```moov-togo``` payment method, confirmation is required through a mobile device.

### mpesa
If the payer chooses the ```mpesa``` payment method, the QR-code will be swown additionally. The customer needs to scan the QR-code to finish the payment.

### mtn-mobile-money
This method is used for mobile numbers in Benin. If the payer chooses ```mtn-mobile-money``` payment method, confirmation is required through a mobile device.

### naps
payment_method = ```naps```

### netbanking-upi
payment_method = ```netbanking-upi```

### next-level-finance
If the payer chooses the ```next-level-finance``` payment method, the redirection to another page will happen to finish the payment.

### nimbbl
If the payer chooses the ```nimbbl``` payment method, the confirmation necessary to finish the payment via APP. 

### nv-apm
If the payer chooses the ```nv-apm``` payment method, the redirection to another page will happen to finish the payment.

### om-wallet
If the payer chooses the ```om-wallet``` payment method, the field for entering the phone number in international format will be additionally displayed on the Checkout page.

### one-collection 

If the payer chooses the ```one-collection``` payment method, the additional fields will be required on the Checkout page.
The payer needs to specify Bank name, Bank Code, Branch Name, Branch Code, and Account Number.
After clicking the Pay button, the information required to complete the payment will be displayed.

### pagsmile

If the payer chooses the ```pagsmile``` payment method, the redirection to another page will happen to finish the payment. The following parameters are required, except for transactions in Guatemala, Panama, or Costa Rica:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```document_number``` | _String_ | Conditional| Customer's ID.<br/> **Condition:** If the parameter is NOT specified in the request, then it will be collected on the Checkout page |
| ```document_type``` | _String_ | Conditional| Customer's identification type.<br/> **Condition:** If the parameter is NOT specified in the request, then it will be collected on the Checkout page |
<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "purchase",
  "methods": [
    "pagsmile"
  ],
  "parameters": {
      "pagsmile": {
        "document_number": "document_number",
        "document_type": "document_type"
      }
   },
  "order": {
    "number": "order-1234",
    "amount": "100",
    "currency": "BRL",
    "description": "Important gift"
  },
  "cancel_url": "https://example.domain.com/cancel",
  "success_url": "https://example.domain.com/success",
  "customer": {
    "name": "John Doe",
    "email": "test@gmail.com"
  },
  "billing_address": {
    "country": "BR",
    "city": "city",
    "address": "address",
    "house_number": "17/2",
    "zip": "123456",
    "phone": "347771112233"
  },
  "hash": "{{session_hash}}"
}'
```
</details>

### panapay-netbanking

If the payer chooses the ```panapay-netbanking``` payment method, the payer will be shown the data necessary to complete the payment.


### panapay-upi

If the payer chooses the ```panapay-upi``` payment method, the payer will be shown the data necessary to complete the payment.


### papara

If the payer chooses the ```papara``` payment method, the redirection to another page will happen to finish the payment. You have to specify in your request the next parameter:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```memberId``` | _String_ | Conditional| Customer ID.<br/> **Condition:** If the parameter is NOT specified in the request, then it will be collected on the Checkout page |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "purchase",
  "methods": [
    "papara"
  ],
  "parameters": {
    "papara": {
      "memberId": "member Id"
   },
  "session_expiry": 60,
  "order": {
    "number": "order-1234",
    "amount": "100",
    "currency": "TRY",
    "description": "Important gift"
  },
  "cancel_url": "https://example.domain.com/cancel",
  "success_url": "https://example.domain.com/success",
  "expiry_url": "https://example.domain.com/expiry",
  "url_target": "_blank",
  "customer": {
    "name": "John Doe",
    "email": "test@gmail.com"
  },
  "billing_address": {
    "country": "TR",
    "city": "Ankara",
    "address": "Moor Building 35274",
    "house_number": "17/2",
    "zip": "123456",
    "phone": "347771112233"
  },
  "hash": "{{session_hash}}"
}
```
</details>

### payablhpp

If the payer chooses the ```payablhpp``` payment method, the redirection to another page will happen to finish the payment.

### paydefi

If the payer chooses the ```allpay``` payment method, the customer’s confirmation via app may be necessary to finish the payment.

### payftr-in
If the payer chooses the ```payftr-in``` payment method, the redirection to another page will happen to finish the payment.

### payneteasyhpp 

If the payer chooses the ```payneteasyhpp``` payment method, the redirection to to widget page happen to finish the payment.

### payok-promptpay

If the payer chooses the ```payok-promptpay``` payment method, the QR-code will be displayed to the payer. The payer should use the code to finish the payment.

### payok-upi

If the payer chooses the ```payok-upi``` payment method, the customer’s confirmation necessary to finish the payment.
You can add to the Authentication request a specific list of parameters which are possible for the ```payok-upi``` payment method.

**Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
 ```personal_id``` | _String_ | Conditional | Payer's Identity ID.<br/>**Condition:** If the parameter is NOT specified in the request for India,<br/>then it will be collected on the Checkout page |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "payok-upi"
    ],
        "parameters": {
            "payok-upi": {
            "personal_id": "personal id"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "MYR",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}"
}
'
```
</details>

### paypal
payment_method = ```paypal```

### paythrough-upi

You should add to the Authentication request a specific list of parameters in the “parameters” object that are needed for the ```paythrough-upi``` payment method.

 **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
 ```upi_id``` | _String_ | Required | Upi Id |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "paythrough-upi"
    ],
        "parameters": {
                "paythrough-upi": {
            "upi_id": "joyoberoihrl@okaxis"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "INR",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "IN",
        "state": "Maharashtra",
        "city": "Mumbai",
        "address": " Shop No.12, Rendezvous, Raviraj Oberoi Cmplx",
        "zip": "400053",
        "phone": "9818218660"
    },
    "hash": "{{session_hash}}"
}
'
```
</details>

### paytota
payment_method = ```paytota```

### phone-pe

If the payer chooses the ```phone-pe``` payment method, the customer’s confirmation via app may be necessary to finish the payment.

### pix
If the payer chooses the ```pix``` payment method, the field for entering the document number will be additionally displayed on the Checkout page.
After the _Pay_ button is clicked, the QR-code could be displayed to the payer. The payer should use the code to finish the payment.

### ppro-alipay
payment_method = ```ppro-alipay```

### ppro-bitpay
payment_method = ```ppro-bitpay```

### ppro-dragonpay
payment_method = ```ppro-dragonpay```

### ppro-fpx
payment_method = ```ppro-fpx```

### ppro-ideal
payment_method = ```ppro-ideal```

### ppro-klarna
payment_method = ```ppro-klarna```

### ppro-przelewy24
payment_method = ```ppro-przelewy24```

### ppro-safetypay
payment_method = ```ppro-safetypay```

### ppro-skrill
payment_method = ```ppro-skrill```

### ppro-sofort
payment_method = ```ppro-sofort```

### ppro-trustly
payment_method = ```ppro-trustly```

### ppro-unionpay
payment_method = ```ppro-unionpay```

### ppro-wechat5
payment_method = ```ppro-wechat5```

### ppro-wechatonline
payment_method = ```ppro-wechatonline```

### pr-cash
payment_method = ```pr-cash```

### pr-creditcard
payment_method = ```pr-creditcard```

### pr-cryptocurrency
payment_method = ```pr-cryptocurrency```

### pr-online
payment_method = ```pr-online```

### ptbs
If the payer chooses the ```ptbs``` payment method, the redirection to another page will happen to finish the payment.

### ptn-email

If the payer chooses the ```ptn-email``` payment method, the email will be sent to the Payer with the link to payment page.

You can send additional parameters for this payment method:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```SecurityQuestion``` | _String_ | Optional | The security question. |
| ```SecurityQuestionAnswer``` |  _String_  | Optional | SecurityQuestionAnswer standards:<br /> - No spaces.<br />- Length must be between 3 and 25 characters long. |

### ptn-inapp

If the payer chooses the ```ptn-inapp``` payment method, the redirection to another page will happen to finish the payment.

### ptn-sms

If the payer chooses the ```ptn-sms``` payment method, the sms will be sent to the Payer with the link to payment page.

You can send additional parameters for this payment method:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```SecurityQuestion``` | _String_ | Optional | The security question. |
| ```SecurityQuestionAnswer``` |  _String_  | Optional | SecurityQuestionAnswer standards:<br /> - No spaces.<br />- Length must be between 3 and 25 characters long. |

### pyk-bkmexpress

If you set the value ```pyk-bkmexpress``` for the brand parameter you have to specify in your request the next parameter required for Turkey:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```orgCode``` | _String_ | Conditional | Payer's Institution (Bank) Code.<br/> **Condition:** If the parameter is NOT specified in the request for Turkey, then it will be collected on the Checkout page |
| ```accountName``` | _String_ | Conditional | Payer's name, length 1~64 characters; Example: Colin Ford. Must be exactly the same as the actual payer's name.<br/> **Condition:** If the parameter is NOT specified in the request for Turkey, then it will be collected on the Checkout page |
| ```accountNumber``` | _String_ | Conditional | Payer's Institution (Bank) account number. Must be exactly the same as the account number of the actual payment.<br/> **Condition:** If the parameter is NOT specified in the request for Turkey, then it will be collected on the Checkout page |

### pyk-momo

If you set the value ```pyk-momo``` for the brand parameter you have to specify in your request the next parameter required for Vietnam:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```personal_id``` | _String_ | Conditional | Payer's Identity ID.<br/> **Condition:** If the parameter is NOT specified in the request for Vietnam, then it will be collected on the Checkout page |

### pyk-nequipush

If you set the value ```pyk-nequipush``` for the brand parameter you have to specify in your request the next parameter required for Colombia:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```personal_id``` | _String_ | Conditional | Payer's Identity ID.<br/> **Condition:** If the parameter is NOT specified in the request for Colombia, then it will be collected on the Checkout page |

### pyk-paparawallet

If you set the value ```pyk-paparawallet``` for the brand parameter you have to specify in your request the next parameter required for Turkey:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```orgCode``` | _String_ | Conditional | Payer's Institution (Bank) Code.<br/> **Condition:** If the parameter is NOT specified in the request for Turkey, then it will be collected on the Checkout page |
| ```accountName``` | _String_ | Conditional | Payer's name, length 1~64 characters; Example: Colin Ford. Must be exactly the same as the actual payer's name.<br/> **Condition:** If the parameter is NOT specified in the request for Turkey, then it will be collected on the Checkout page |
| ```accountNumber``` | _String_ | Conditional | Payer's Institution (Bank) account number. Must be exactly the same as the account number of the actual payment.<br/> **Condition:** If the parameter is NOT specified in the request for Turkey, then it will be collected on the Checkout page |

### pyk-promptpay

If the payer chooses the ```pyk-promptpay``` payment method, the QR-code will be displayed to the payer. The payer should use the code to finish the payment.

If you set the value ```pyk-promptpay``` for the brand parameter you have to specify in your request the next parameter required for Thailand:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```orgCode``` | _String_ | Conditional | Payer's Institution (Bank) Code.<br/> **Condition:** If the parameter is NOT specified in the request for Thailand, then it will be collected on the Checkout page |
| ```accountName``` | _String_ | Conditional | Payer's name, length 1~64 characters; Example: Colin Ford. Must be exactly the same as the actual payer's name.<br/> **Condition:** If the parameter is NOT specified in the request for Thailand, then it will be collected on the Checkout page |
| ```accountNumber``` | _String_ | Conditional | Payer's Institution (Bank) account number. Must be exactly the same as the account number of the actual payment.<br/> **Condition:** If the parameter is NOT specified in the request for Thailand, then it will be collected on the Checkout page |

### pyk-truemoney

If you set the value ```pyk-truemoney``` for the brand parameter you have to specify in your request the next parameter required for Thailand:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```orgCode``` | _String_ | Conditional | Payer's Institution (Bank) Code.<br/> **Condition:** If the parameter is NOT specified in the request for Thailand, then it will be collected on the Checkout page |
| ```accountName``` | _String_ | Conditional | Payer's name, length 1~64 characters; Example: Colin Ford. Must be exactly the same as the actual payer's name.<br/> **Condition:** If the parameter is NOT specified in the request for Thailand, then it will be collected on the Checkout page |
| ```accountNumber``` | _String_ | Conditional | Payer's Institution (Bank) account number. Must be exactly the same as the account number of the actual payment.<br/> **Condition:** If the parameter is NOT specified in the request for Thailand, then it will be collected on the Checkout page |

### pyk-upi

If the payer chooses the ```pyk-upi``` payment method, the customer’s confirmation necessary to finish the payment.
You can add to the Authentication request a specific list of parameters which are possible for the ```pyk-upi``` payment method.

**Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
 ```personal_id``` | _String_ | Conditional | Payer's Identity ID.<br/>**Condition:** If the parameter is NOT specified in the request for India,<br/>then it will be collected on the Checkout page |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "pyk-upi"
    ],
        "parameters": {
            "pyk-upi": {
            "personal_id": "personal id"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "MYR",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}"
}
'
```
</details>

### pyk-viettelpay

If you set the value ```pyk-viettelpay``` for the brand parameter you have to specify in your request the next parameter required for Vietnam:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```personal_id``` | _String_ | Conditional | Payer's Identity ID.<br/> **Condition:** If the parameter is NOT specified in the request for Vietnam, then it will be collected on the Checkout page |


### pyk-zalopay

If you set the value ```pyk-zalopay``` for the brand parameter you have to specify in your request the next parameter required for Vietnam:

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```personal_id``` | _String_ | Conditional | Payer's Identity ID.<br/> **Condition:** If the parameter is NOT specified in the request for Vietnam, then it will be collected on the Checkout page |

### paypal
payment_method = ```paypal```

### paythrough-upi

You should add to the Authentication request a specific list of parameters in the “parameters” object that are needed for the ```paythrough-upi``` payment method.

 **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
 ```upi_id``` | _String_ | Required | Upi Id |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "paythrough-upi"
    ],
        "parameters": {
                "paythrough-upi": {
            "upi_id": "joyoberoihrl@okaxis"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "INR",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "IN",
        "state": "Maharashtra",
        "city": "Mumbai",
        "address": " Shop No.12, Rendezvous, Raviraj Oberoi Cmplx",
        "zip": "400053",
        "phone": "9818218660"
    },
    "hash": "{{session_hash}}"
}
'
```
</details>

### saopayments

payment_method = ```saopayments```

### securepaycard
payment_method = ```securepaycard```

### sepa

If the payer chooses the ```sepa``` payment method, the redirection to another page will happen to finish the payment.

### skrill

If the payer chooses the ```skrill``` payment method, the redirection to another page will happen to finish the payment.

### sofortuber
If the payer chooses the ```sofortuber``` payment method, the payer will be shown the data necessary to complete the payment.

### stcpay

payment_method = ```stcpay```

### stripe-js

If the payer chooses the ```stripe-js``` payment method, the redirection to another page will happen to finish the payment.

### swifipay-deposit
If the payer chooses the ```swifipay-deposit``` payment method, the redirection to another page will happen to finish the payment.


### sz-in-imps

If the payer chooses the ```sz-in-imps``` payment method, the redirection to another page will happen to finish the payment.

### sz-in-paytm

If the payer chooses the ```sz-in-paytm``` payment method, the redirection to another page will happen to finish the payment.

### sz-in-upi

You should add to the Authentication request a specific list of parameters in the “parameters” object that are needed for the ```sz-in-upi``` payment method.

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```upiAddress``` | _String_ | Required | Payer's UPI address |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "sz-in-upi"
    ],
        "parameters": {
                "sz-in-upi": {
            "upiAddress: "address@upi" 
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "INR",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}",
}
'
```
</details>

### sz-jp-p2p

If the payer chooses the ```sz-jp-p2p``` payment method, the redirection to another page will happen to finish the payment.

### sz-kr-p2p

If the payer chooses the ```sz-kr-p2p``` payment method, the redirection to another page will happen to finish the payment.

### sz-my-ob

If the payer chooses the ```sz-my-ob``` payment method, the redirection to another page will happen to finish the payment.

### sz-th-ob

If the payer chooses the ```sz-th-ob``` payment method, the redirection to another page will happen to finish the payment.

### sz-th-qr

If the payer chooses the ```sz-th-qr``` payment method, the redirection to another page will happen to finish the payment.

### sz-vn-ob

If the payer chooses the ```sz-vn-ob``` payment method, the redirection to another page will happen to finish the payment.

### sz-vn-p2p

If the payer chooses the ```sz-vn-p2p``` payment method, the redirection to another page will happen to finish the payment.

### trustgate

If the payer chooses the ```trustgate``` payment method, the redirection to another page will happen to finish the payment.

### tabby

You should add to the Authentication request a specific list of parameters in the “parameters” object that are needed for the ```tabby``` payment method.

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```category``` | _String_ | Required | Required as name of high-level category (Clothes, Electronics,etc.); or a tree of category-subcategory1-subcategory2; or id of the category and table with category-ids data mapped provided. |
| ```buyer_registered_since``` | _String_ | Required | Time the customer got registred with you, in UTC, and displayed in ISO 8601 datetime format. |
| ```buyer_loyalty_level``` | _Number_ | Required | Default: 0<br/>Customer's loyalty level within your store, should be sent as a number of successfully placed orders in the store with any payment methods. |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "tabby"
    ],
        "parameters": {
                "tabby": {
            "category": " Clothes ",
            "buyer_registered_since": " 2019-08-2414:15:22",
            "buyer_loyalty_level": 0
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "SAR",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}"
}
'
```
</details>

### tamara

You should add to the Authentication request a specific list of parameters in the “parameters” object that are needed for the ```tamara``` payment method.

 **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```shipping_amount``` | _Number_ | Required | Shipping amount |
| ```tax_amount``` | _Number_ | Required | Tax amount |
| ```product_reference_id``` | _String_ | Required | The unique id of the item from merchant's side  |
| ```product_type``` | _String_ | Required | Product type |
| ```product_sku``` | _String_ | Required | Product sku |
| ```product_amount``` | _Number_ | Required | Product amount |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "tamara"
    ],
        "parameters": {
                "tamara": {
            " shipping_amount ": 1.01,
            "tax_amount": 1.01,
            "product_reference_id": "item54321",
            "product_type": "Clothes",
            "product_sku": "ABC-12345-S-BL",
            "product_amount": 998.17
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "SAR",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}"
}
'
```
</details>

### togocom
This method is used for mobile numbers in Togo. If the payer chooses ```togocom``` payment method, confirmation is required through a mobile device.

### unipay
payment_method = ```unipay```

### unipayment

If the payer chooses the ```unipaymentpayment``` method, the redirection to another page will happen to finish the payment.

### vcard

payment_method = ```vcard```

### vouchstar

If the payer chooses the ```vouchstar``` method, the redirection to another page will happen to finish the payment.

### vp-wallet

payment_method = ```vp-wallet```

### vpayapp_upi

If the payer chooses the ```vpayapp_upi``` payment method, the UPI address will be requested additionally on the Checkout page. After the Pay button is clicked, the payer will be redirected to another page to finish the payment.

### webpaygate
If the payer chooses the ```webpaygate``` payment method, the payer will be shown the data necessary to complete the payment.


### winnerpay

If the payer chooses the ```winnerpay``` payment method, the payer will be shown the data necessary to complete the payment.

### xprowirelatam-ted

If the payer chooses the ```xprowirelatam-ted``` payment method, the customer’s confirmation via app may be necessary to finish the payment.

### xprowirelatam-cash

If the payer chooses the ```xprowirelatam-cash``` payment method, the customer’s confirmation via app may be necessary to finish the payment.

### xprowirelatam-bank-transfer

If the payer chooses the ```xprowirelatam-bank-transfer``` payment method, the customer’s confirmation via app may be necessary to finish the payment.

### xprowirelatam-bank-slip

If the payer chooses the ```xprowirelatam-bank-slip``` payment method, the customer’s confirmation via app may be necessary to finish the payment.

### xprowirelatam-picpay

If the payer chooses the ```xprowirelatam-picpay``` payment method, the customer’s confirmation via app may be necessary to finish the payment.

### xprowirelatam-pix

If the payer chooses the ```xprowirelatam-pix``` payment method, the customer’s confirmation via app may be necessary to finish the payment.

### xswitfly
If the payer chooses the ```xswitfly``` payment method, the redirection to another page will happen to finish the payment.

### yapily
If the payer chooses the ```yapily``` payment method, the redirection to another page will happen to finish the payment.
As well, the customer could be asked to enter additional data on Checkout.

Payment flow and list of the required parameters depend on specific region and payment providers rules.

| **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | --- |
| ```payee_name``` | _String_ | Required | Provide here the account provider code of the institution holding the account indicated in the Account parameter. |
| ```payee_country``` | _String_ | Required  | This parameter enables you to attach a file to the transaction. This is useful, for example, in the case where you may want to attach a scanned receipt, or a scanned payment authorization, depending on your internally established business rules. This parameter requires you to provide the name of the file you are attaching, as a string, for example “receipt.doc” or “receipt.pdf”. Note that the contents of this parameter are ignored if you have not provided the contents of the file using NarrativeFileBase64 below. |
| **payee_identifications** | Array  | Required  | The account identifications that identify the Payer bank account. Array should contain the objects with the Identifications combinations: ```type``` and ```identification``` parameters.|
| ```type``` | _String_ | Required | Used to describe the format of the account. Possible values: <br/> - SORT_CODE <br/> - ACCOUNT_NUMBER <br/> - IBAN <br/> - BBAN <br/> - BIC <br/> - PAN <br/> - MASKED_PAN <br/> - MSISDN <br/> - BSB <br/> - NCC <br/> - ABA <br/> - ABA_WIRE <br/> - ABA_ACH <br/> - EMAIL <br/> - ROLL_NUMBER <br/> - BLZ <br/> - IFS <br/> - CLABE <br/> - CTN <br/> - BRANCH_CODE <br/> - VIRTUAL_ACCOUNT_ID  |
| ```identification``` | _String_ | Required  | Account Identification. The value associated with the account identification type. |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
  "merchant_key": "xxxxx-xxxxx-xxxxx",
  "operation": "purchase",
  "methods": [
    "yapily"
  ],
  "parameters": {
    "yapily": {
      "payee_name": "Payee Name",
      "payee_country": "GB",
      "payee_identifications": [
        {
          "type": "SORT_CODE",
          "identification": "123456"
        },
        {
          "type": "ACCOUNT_NUMBER",
          "identification": "12345678"
        }
      ]
    }
  },
  "order": {
    "number": "order-1234",
    "amount": "1000.19",
    "currency": "UGX",
    "description": "Important gift"
  },
  "cancel_url": "https://example.com/cancel",
  "success_url": "https://example.com/success",
  "customer": {
    "name": "John Doe",
    "email": "test@gmail.com"
  },
  "billing_address": {
    "country": "US",
    "state": "CA",
    "city": "Los Angeles",
    "address": "Moor Building 35274",
    "zip": "123456",
    "phone": "347771112233"
  },
  "hash": "{{session_hash}}"
}
'
```
</details>

### yo-uganda-limited

You can add to the Authentication request a specific list of parameters in the “parameters” object that are needed for the <br /> ```yo-uganda-limited``` payment method.

 **Parameter** | **Type** | **Mandatory** | **Description** |
| --- | --- | --- | :---: |
| ```AccountProviderCode``` | _String_ | Optional | Provide here the account provider code of the institution holding the account indicated in the Account parameter. |
| ```NarrativeFileName``` | _String_ | Optional | This parameter enables you to attach a file to the transaction. This is useful, for example, in the case where you may want to attach a scanned receipt, or a scanned payment authorization, depending on your internally established business rules. This parameter requires you to provide the name of the file you are attaching, as a string, for example “receipt.doc” or “receipt.pdf”. Note that the contents of this parameter are ignored if you have not provided the contents of the file using NarrativeFileBase64 below. |
| ```NarrativeFileBase64``` | _String_ | Optional | This parameter enables you to attach a file to the transaction. This is useful, for example, in the case where you may want to attached a scanned receipt, or a scanned payment authorization, depending on your business rules. This parameter requires you to provide the contents of the file you are attaching, encoded in base-64 encoding. Note that the contents of this parameter are ignored if you have not provided a file name using NarrativeFileName above. |
| ```ProviderReferenceText``` | _String_ | Optional | In this field, enter text you wish to be present in any confirmation message which the mobile money provider network sends to the subscriber upon successful completion of the transaction. Some mobile money providers automatically send a confirmatory text message to the subscriber upon completion of transactions. This parameter allows you to provide some text which will be appended to any such confirmatory message sent to the subscriber. |

<details>
  <summary markdown="span">Authentication request example</summary>

```
curl --location -g --request POST '{{CHECKOUT_HOST}}/api/v1/session' \
--header 'Content-Type: application/json' \
--data-raw '{
    "merchant_key": "xxxxx-xxxxx-xxxxx",
    "operation": "purchase",
    "methods": [
        "yo-uganda-limited"
    ],
        "parameters": {
                "yo-uganda-limited": {
            "AccountProviderCode": "89128734651234",
            "NarrativeFileName": "file.img",
            "NarrativeFileBase64": "iVBORw0KGgoAAAANSUhEUgAAAAEAAAADCAYAAABS3WWCAAAABHNCSVQICAgIfAhkiAAAABl0RVh0U29mdHdhcmUAZ25vbWUtc2NyZWVuc2hvdO8Dvz4AAAA1aVRYdENyZWF0aW9uIFRpbWUAAAAAANGB0YAsIDEwLdGC0YDQsC0yMDIzIDEyOjA4OjQ3ICswMzAwEHyhlwAAABdJREFUCJlj+P///38mBgYGBiYGBgYGADH7BAGR0RGuAAAAAElFTkSuQmCC",
            "ProviderReferenceText"”: "thank you",
            "AccountProviderCode": "provider1234"
        }
    },   
    "order": {
        "number": "order-1234",
        "amount": "1000.19",
        "currency": "UGX",
        "description": "Important gift"
    },
    "cancel_url": "https://example.com/cancel",
    "success_url": "https://example.com/success",
     "customer": {
        "name": "John Doe",
        "email": "test@gmail.com"
    },
    "billing_address": {
        "country": "US",
        "state": "CA",
        "city": "Los Angeles",
        "address": "Moor Building 35274",
        "zip": "123456",
        "phone": "347771112233"
    },
    "hash": "{{session_hash}}",
}
'
```
</details>

### zeropay
payment_method = ```zeropay```