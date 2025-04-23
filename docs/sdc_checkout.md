---
id: sdk_checkout
title: SDK CHECKOUT
stoplight-id: zm83vbiw5tgsz
---
Version: 1.2.1<br/>
Released: 2025/02/26<br/>

# Checkout SDK Documentation

## General

For the checkout page, a wide range of settings is available through the admin panel, for example, placing your own logo, changing the colors of basic elements. But if these opportunities are not enough to ensure corporate style, design, merchants also have the option of uploading their own checkout page on the platform resources. In this case, all the advantages of HPP integration are retained, one of which is the absence of necessity for the merchant to conduct a complex and expensive PCI DSS certification.

<div class="attention-box">
⚠️ Currently, only acquiring payment forms with a <font color='Red'>CARD</font> payment method, <font color='Red'>ApplePay</font> and <font color='Red'>GooglePay</font> are supported (where card data is entered on the form itself). P2P, debit payment forms are not supported.
</div>

## Payment form development

When developing your own payment page or adapting an existing one, the following conditions must be met:

1. Technology stack - HTML / CSS / JavaScript
2. Project folder structure:<br />
- /img<br />
- /style<br />
- /js<br />
- /<br />
3. You need to connect Checkout SDK module in the payment page code:
```
<script src="https://{{CHECKOUT_URL}}/sdk/pay-session.js"></script>
```
4. It is necessary to create a new instance of the ```PaySession``` class by passing to the constructor function the identifier of the HTML element of the form for entering data on the payment page and the parameter if on-the-fly validation of the payment form fields is needed (optional parameter). Next, call the ```begin()``` method:
```
<script>
    const session = new PaySession({formName: "paymentForm", validateOnInput: true});
    session.begin();
</script>
```
5. In the payment page HTML code, you need to link the ```<input>``` tags of the form to Checkout SDK module using special ```id``` identifiers. The module interacts only with those ```<input>``` elements that are inside the ```<form>``` tag and are indicated by the corresponding ```id```:<br />

|**```<input>```** | **Type** |
| --- | --- |
| Payment card number | ```payer_card``` |
| CVV code | ```payer_cvv``` |
| Expiry date | ```payer_expiryDate``` |
| Payer name | ```payer_name``` |
| ZIP code | ```payer_zip``` |
| Payer city | ```payer_city``` |
| Payer phone number | ```payer_phone``` |
| Payer email | ```payer_email``` |
| Payer state | ```payer_state``` |
| Payer address | ```payer_address``` |
| Payer country | ```payer_country``` |

> ⚠️ The form must contain at least inputs with such id: ```payer_card```, ```payer_cvv```, ```payer_expiryDate```, ```payer_name```.

6. The Checkout SDK library implements a set of methods for obtaining some session checkout parameters.

**getBillingAddress**() - getting the billing_address object from the merchant's Authentication request. Object properties may be empty if the corresponding information has not been provided by the merchant. Result example:

```
{address: "Moore Building", city: "Los Angeles", country: "US", phone: "12345678", state: "", zip: "12345"}
```
**getOrderInfo**() - obtaining information on the current order. The function returns an object with data, for example:

```
{number: "111-222", amount: "10.00", currency: "USD", description: "Important gift"}
```
**getSessionExpiry**() - getting the checkout session lifetime date/time in unixtime format. Result example:

```
1711378966
```

**getSessionState**() - getting the current session status. Possible values: complete, redirect, waiting и decline. Example:

```
"decline"
```
**getDeclinesMax**() - receiving the set limit on the number of declines on the payment page. This parameter can be defined by the merchant in Authentication request and after receiving such a number of declines an automatic redirect of the user to cancelUrl will be performed.. Example of function call result:

```
2
```
**getCancelUrl**() - getting cancel_url passed by the merchant in Authentication request. Example:

```
"https://example.domain.com/cancel"
```

**getCardTokens**() - getting a set of available payment card tokens. The result of the call will be an array of objects, example:

```
[
 {
    "token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "last": "1111",
    "month": "01",
    "year": "2025",
    "brand": "visa",
    "cardholder_name":"John Doe"
 }
]
```

**getCustomerInfo**() - obtaining customer data. The method should return the following:

```
{
    name: "John Doe",
    email: "example@mail.com",
    birth_date: "1970-02-17"
}
```

**getCustomData**() - obtaining parameter 'custom_data'. The result of the function call is an object that contains the data passed in the 'custom_data' parameter in the authentication request

Example in the request:

```
"custom_data": {
    "provider": "akurateco",
    "brand": "visa",
    "domain": "en"
}
```

Function return example:

```
{"brand": "visa","domain": "en","provider": "akurateco"}
```

**getAvailableBrands**() - returns available brands

Example:
```
["googlepay", "applepay"]
```


**setCardToken**(token: string) - selecting a specific token for the payment process. Stored card data in the token has priority over card data from form fields.

**getActiveCardToken**() - getting active token, if such token was set using setCardToken function. The method returns either a string with the token or an empty value if no active token is selected. Example:

```
"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

**clearCardToken**() - to deselect the active token. This method should be used if you want to revert to the option of directly entering card data in the form fields instead of using data from the token.

**getCountries**() - reference function to get a complete list of countries with regions. Returns an array of objects:

```
[
 {
    regions: null,
    value: "AF",
    text: "Afghanistan"
  },
  {
    regions: [ {value: "ACT", text: "Australian Capital Territory"}, ... ],
    value: "AU",
    text: "Australia",
  }
]
```

7. Also the Checkout SDK module implements a number of event listeners, the result of which is available for processing on the form page. Event handling is optional.

**onDecline** - is called when the response ```declined``` is received at the time of payment processing or when checking the session state. Returns an object with the error message: ```{message: string}```.

<details>
  <summary markdown="span">Event handler code example</summary>

```
session.onDecline = function (event) {
   document.querySelector(".decline-message").innerText = event.message;
}
```
</details>

**onUndefined** - event to receive unsupported case when making payment or when checking status <br/>

<details>
  <summary markdown="span">Event handler code example</summary>

```
session.onUndefined = function (event) {
    document.querySelector(".decline-message").innerText = "Unsupported case occurred. To check the status of your payment, please contact support"
};
```
</details>

**onSuccess** - is called when the response ```completed``` is received at the time of payment processing or when checking the session status. Returns an object with successUrl: ```{successUrl: string}```.

<details>
  <summary markdown="span">Event handler code example</summary>

```
session.onSuccess = function (event) {
   location.href = event.successUrl;
}
```
</details>


**onFormReady** - is called after getting the current session. Doesn't return any values.

<details>
  <summary markdown="span">Event handler code example</summary>

```
session.onFormReady = function () {
    document.querySelector(".form-wrapper").classList.add("show");
}
```
</details>

**handleError** - is called after an error occurs when sending a payment processing request. Returns an error object.

**onInputValidation** - is called when an on-the-fly validation error occurs for one of the inputs with ```id``` listed earlier (if the parameter ```validateOnInput: true``` was passed to the constructor) or as a general validation when submitting the form. Returns a validation error object: ```{result: string, inputId: string}```.

<details>
  <summary markdown="span">Event handler code example</summary>

```
session.onInputValidation = function (event) {
      if (event.result === "error") {
           document.getElementById(event.inputId)?.classList.add("text-field--error");
           document.getElementById(event.inputId + "Error")?.classList.add("display-error");
      } else {
           document.getElementById(event.inputId)?.classList.remove("text-field--error");
           document.getElementById(event.inputId + "Error")?.classList.remove("display-error");
      }
 };
```
</details>

8. To add ApplePay and GooglePay payment methods to the payment form you need to:

**ApplePay**

1) The container should be assigned ```id = applepay-container```, in the body of which you need to create a div block for the button and define styles for it.

<details>
  <summary markdown="span">Example code</summary>

```
<div id="applepay-container">
    <div class="applepay-button"></div>
</div>
 
@supports (-webkit-appearance: -apple-pay-button) {
    .applepay-button {
    display: block;
    width: 100%;
    height: 40px;
    -webkit-appearance: -apple-pay-button;
    -apple-pay-button-type: plain;
    -apple-pay-button-style: black;
    overflow: hidden;
    height: 45px;
    }
}
```
</details>


2) An event handler should be added to the button, code example:

```
<div id="applepay-container">
    <div class="applepay-button" onclick="handleApplePayClick()"></div>
</div>
```

**GooglePay**

1) The container should be assigned id = "googlepay-container". Example code:

```
<div id="googlepay-container"></div>
```

2) setGooglePayButtonConfiguration() - used for customizing the appearance of the Google Pay button. Accepts a button configuration object. <br/>
Example:

```
const googlePayButtonConfig = {
    buttonColor: "white",
    buttonType: "short",
    buttonRadius: 8,
    buttonLocale: "uk",
    buttonSizeMode: "static",
}
```
```
session.setGooglePayButtonConfiguration(googlePayButtonConfig);
```

An object can contain only those fields that the user requires. Other fields, if missing, will be replaced with default parameters.

The configuration parameters used are those provided by Google - https://developers.google.com/pay/api/web/reference/request-objects#ButtonOptions

If no button configuration parameters were passed, the default parameters are used:

```
GOOGLE_PAY_BUTTON_CONFIGURATION_DEFAULT = {
    buttonColor: "default",
    buttonType: "buy",
    buttonRadius: 4,
    buttonLocale: "en",
    buttonSizeMode: "fill",
}
```

## Further steps

After developing the payment form, it is necessary to send the project archive, including the source code and all used files, to the manager for review, security testing and placement on the platform resources.

The payment form will be assigned a unique identifier that should be used in Checkout Authentication Request to display this payment form to customers:

```
{"form_id": "xxxxx-xxxxx-xxxxx"}
```
## Example

An example of the payment form code can be downloaded from the [<font color=''><ins>link</ins></font>](sdk-form-example.zip)<br/>