<h1 align="center">
  <br>
  <a href="https://kb.ccbill.com/CCBill+API"><img src="https://user-images.githubusercontent.com/81383705/118320543-b10b5400-b4fc-11eb-8290-98b0d7351ca8.png" alt="ccbill-restful-api" width="300"></a>
  <br>
  CCBill RESTful API
  <br>
</h1>

<p align="center">
This document describes the API resources and endpoints of the CCBill Transaction RESTful API service, as well as the CCBill Advanced Widget, a JavaScript library that makes the use of the API easier to use from a web page. Merchants can use these resources to charge consumers with a payment token. This instructional document is provided as a technical resource to CCBill Merchants. It is intended to be read by programmers, technicians, and other persons with coding skills.
</p>

<p align="center">
  <a href="https://ccbill.com/doc/ccbill-restful-transaction-api">API</a> •
  <a href="https://ccbill.com/kb/">Knowledge Base</a> •
  <a href="https://ccbill.com/contact">Support</a>
</p>

## Requirements

* The user already has a [CCBill Account](https://admin.ccbill.com/loginMM.cgi).
* The user has been provided with the [API credentials](https://ccbill.com/contact).
* The user has had their domain whitelisted with help of [CCBill Support](https://ccbill.com/contact).
* The user has experience with RESTful Web Services.
* The user has experience with JSON format.
* The RESTful Transaction API supports TLS 1.2 only.

**Note:** To maintain PCI compliance at all times, use CCBill’s Advanced Widget and ensure that payment details are sent directly to CCBill without them being sent through your server. Always load the CCBill’s JavaScript libraries via ```https://js.ccbill.com``` to remain compliant. Don’t bundle or host the scripts yourself.

## Terminology

* **Merchant Account**. Each CCBill merchant receives an account number for tracking purposes. The standard format is 9xxxxx-xxxx, where 9xxxxx is the main account. The main account is a six (6) digit number. For example: "999999".

* **Merchant Sub-account**. CCBill Merchants may open one or more sub-accounts. The sub-account is a four (4) digit number. The standard format is: xxxx. For example: "1234". The sub-account is part of the main account.

* **[Payment Token](https://ccbill.com/kb/credit-card-tokenization)**. A Payment Token identifies a billable entity within the system.

* **Subscription ID:**  Transaction subscription identification number.

* **Merchant Application ID:**  This is the client ID that the merchant has received upon signing up to use the CCBill RESTful API.

* **Merchant Secret:**  This is the password that was set up for the authentication with the CCBill RESTful API.

* **[Strong Customer Authentication (SCA)](https://ccbill.com/kb/strong-customer-authentication):** European laws, such as [PSD2](https://ccbill.com/blog/what-is-psd2), require the use of SCA, such as the [3DS](https://ccbill.com/kb/3d-secure-2) protocol, for online payment processing. When an EU-based cardholder makes a payment online, SCA is initiated. Merchants can use CCBill's Advanced Widget and its functions to facilitate strong customer authentication.

## The Payment Flow

Below are the 3 essential steps for charging the consumer using a payment token:

1. Generate the CCBill OAuth Bearer Token
   * **the request to generate the `CCBill OAuth` token must be sent from your back-end services directly to the CCBill API, and it can not be requested from within the browser**

2. Create the [Payment Token](https://ccbill.com/kb/credit-card-tokenization)

3. Charge the Payment Token

Click the image or open in a new tab to review the sequence diagram in full size.

<a href="https://user-images.githubusercontent.com/81383705/237035677-5ae3004a-8ed6-41d2-a5b8-c103462dbde8.png" target="_blank" rel="noopener"><img src="https://user-images.githubusercontent.com/81383705/237035677-5ae3004a-8ed6-41d2-a5b8-c103462dbde8.png" alt="ccbill-restful-api" width="300"></a>

While all of the above steps can be completed by making requests from your backend to our API endpoints, you can also use the `CCBill Advanced Widget` (JS library) to:
* create the payment tokens
* check whether the 3DS verification is required or not, and
* perform the strong customer authentication (SCA) from within the browser

Click the image or open in a new tab to review the sequence flow for creating and charging of payment tokens with 3DS verification.

<a href="https://user-images.githubusercontent.com/81383705/237035691-46280338-e7e3-451a-94aa-d7c6b6338874.png" target="_blank" rel="noopener"><img src="https://user-images.githubusercontent.com/81383705/237035691-46280338-e7e3-451a-94aa-d7c6b6338874.png" alt="ccbill-restful-api" width="300"></a>

Creation of the `CCBill OAuth` token and charging payment tokens are not supported by the `CCBill Advanced Widget` and must be performed by making API calls from your back-end services.

## 1. Generate the CCBill OAuth Bearer Token

The [CCBill RESTful Transaction API](https://ccbill.com/doc/ccbill-restful-transaction-api) uses OAuth based authentication and authorization. Prior to accessing the API, you need to [register your application with CCBill]((https://ccbill.com/contact)). 

Upon registration, your application will be assigned a `merchant application ID` and a `secret key`. Use these credentials to generate the `CCBill Oauth Bearer Token` (aka the `access token`) by providing them to the authorization server. **Please note that this step cannot be done from within the browser, and you must make the call from your backend**.

Once you have generated the CCBill OAuth token (which is not to be confused with the `payment token`), you need to place it in the `Authorization header` of each API request. You will then be able to access the CCBill Transaction API until the access token expires or is revoked.

The acquired CCBill OAuth token is a random string of data that does not hold any important piece of information, and it has no value on its own. It works only as an authentication and authorization tool and grants access to an application.

### Endpoint URL

* [https://api.ccbill.com/ccbill-auth/oauth/token?grant_type=client_credentials](https://api.ccbill.com/ccbill-auth/oauth/token?grant_type=client_credentials)

### Headers

* `Content-Type: application/x-www-form-urlencoded`

* `Authorization: Basic MerchantApplicationID:SecretKey`

### Example Request Using CURL
```
curl - POST 'https://api.ccbill.com/ccbill-auth/oauth/token' \

--header 'Content-Type: application/x-www-form-urlencoded' \

--header 'Authorization: Basic Merchant_ApplicationID:Merchant_Secret ' \

--data-urlencode 'grant_type=client_credentials'
``` 
## 2. Create Payment Token ID (CCBill Advanced Widget)

The `CCBill Advanced Widget` makes the creation of payment tokens easier by encapsulating the API calls that need to be made into a JavaScript function that can be used from within your web page. 

The library is hosted on CCBill's content distribution network (CDN) which makes import into the merchants' websites quick and easy from any location in the world.

The following instructions describe how to set up and use the `CCBill Advanced Widget` library:

### Step 1. Add Preload Link and HTML Elements

Add the following preload link and script HTML elements to the HTML page that will host your payment form:
```
<link rel="preload" href="https://js.ccbill.com/v1.3.0/ccbill-advanced-widget.js" as="script"/>
<script type="text/javascript" src="https://js.ccbill.com/v1.3.0/ccbill-advanced-widget.js"></script>
```
**Note:** The version in this URI example is **v1.3.0**. Pay special attention to the version in the URI path as the version number may be subject to change.

### Step 2. Define the Field IDs

The `CCBill Advanced Widget` library is able to extract the values from the relevant form fields by relying on the default ID attributes or by utilizing the `data-ccbill` custom HTML element attribute. Using the `data-ccbill` attribute is recommended as it is less intrusive and allows the merchant looser coupling. 

When using the custom attribute the correct format is:
```
<input type="text" data-ccbill="[corresponding field name]" />
```

When using the default IDs the correct format is:
```
<input type="text" id="_ccbillId_[corresponding field name]" />
```

The table below shows the values that should be set for the `data-ccbill` attribute or, alternatively, in the default ID attribute fields.

| **data-ccbill**                                                                                            | **Default IDs**                                                                                                      |
|------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| **nameOnCard**                                                                                             | **\_ccbillId\_nameOnCard**                                                                                           |
| **cardNumber**                                                                                             | **\_ccbillId\_cardNumber**                                                                                           |
| **expMonth**                                                                                               | **\_ccbillId\_expMonth**                                                                                             |
| **expYear**                                                                                                | **\_ccbillId\_expYear**                                                                                              |
| **firstName**                                                                                              | **\_ccbillId\_firstName**                                                                                            |
| **lastName**                                                                                               | **\_ccbillId\_lastName**                                                                                             |
| **address1** (optional)                                                                                    | **\_ccbillId\_address1** (optional)                                                                                  |
| **address2** (optional)                                                                                    | **\_ccbillId\_address2** (optional)                                                                                  |
| **city** (optional)                                                                                        | **\_ccbillId\_city** (optional)                                                                                      |
| **country**                                                                                                | **\_ccbillId\_country**                                                                                              |
| **state** (optional)                                                                                       | **\_ccbillId\_state** (optional)                                                                                     |
| **postalCode**                                                                                             | **\_ccbillId\_postalCode**                                                                                           |
| **phoneNumber** (optional)                                                                                 | **\_ccbillId\_phoneNumber** (optional)                                                                               |
| **email**                                                                                                  | **\_ccbillId\_email**                                                                                                |
| **currencyCode** (A three-digit currency code for the currency used in the transaction. Required for SCA.) | **_ccbillId_currencyCode** (A three-digit currency code for the currency used in the transaction. Required for SCA.) |
| **ipAddress** (optional, recommended hidden field auto populated by JavaScript)                            | **\_ccbillId\_ipAddress** (optional, recommended hidden field auto populated by JavaScript)                          |
| **browserHttpAccept** (optional, recommended hidden field auto populated by JavaScript)                    | **\_ccbillId\_browserHttpAccept** (optional, recommended hidden field auto populated by JavaScript)                  |
| **browserHttpAcceptEncoding** (optional, recommended hidden field auto-populated by javascript)            | **\_ccbillId\_browserHttpAcceptEncoding** (optional, recommended hidden field auto-populated by javascript)          |
| **browserHttpAcceptLanguage** (optional, recommended hidden field auto-populated by javascript)            | **\_ccbillId\_browserHttpAcceptLanguage** (optional, recommended hidden field auto-populated by javascript)          |


### Step 3. Create JavaScript Method

Create a JavaScript function that will call the `CCBill Advanced Widget's` `createPaymentToken()` function. This is the main function merchants need to incorporate into their JavaScript in order to create payment tokens.

To generate a token, multiple parameters need to be passed using the function:

| PARAMETER                        | TYPE    | DESCRIPTION                                                                                                                                                                                                                                                                                                                                                                    |
|----------------------------------|---------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **authToken** (required)         | string  | Required input that uses an OAuth token to perform the creation of the Payment Token. This must be a valid OAuth token generated using the merchant application ID and the secret key dedicated to the client account involved in the transaction.                                                                                                                             |
| **clientAccnum** (required)      | integer | Merchant account number.                                                                                                                                                                                                                                                                                                                                                       |
| **clientSubacc** (required)      | integer | Merchant subaccount number.                                                                                                                                                                                                                                                                                                                                                    |
| **clearPaymentInfo** (optional)  | boolean | An optional flag that will, if set to true, result in clearance of the payment information fields when the `createPaymentToken` function is called. If `null` is provided, this will default to false, and the payment information fields will not be cleared. <br /><br />**Note:** Even though this parameter is optional, this field should be set to `null` if not used.   |
| **clearCustomerInfo** (optional) | boolean | An optional flag that will, if set to true, result in clearance of the customer information fields when the `createPaymentToken` function is called. If `null` is provided, this will default to false, and the customer information fields will not be cleared. <br /><br />**Note:** Even though this parameter is optional, this field should be set to `null` if not used. |
| **timeToLive**  (optional)       | integer | The time interval that defines how long the token should be valid (hours).                                                                                                                                                                                                                                                                                                     |
| **numberOfUse** (optional)       | integer | The total number of times a specific payment token can be used for purchases. <br /><br />**Note:** Even though this parameter is optional, this field should be set to `null` if not used.                                                                                                                                                                                    |

The following example is provided and can be modified as required:

```
const widget = new ccbill.CCBillAdvancedWidget(applicationId);
try {
       const result = widget.createPaymentToken(oauthToken, clientAccnum, clientSubacc, clearPaymentInfo, clearCustomerInfo, timeToLive, numberOfUse);
       result.then(
           (data) => {
               console.log("SUCCESS");
               return data.json();
           },
           (error) => {
               console.log("ERROR");
               return error.json();
           }).then(json => {                
               console.log("RESULT :[" + JSON.stringify(json) + "]");
           }).catch((error) => {
           console.error("ERROR2 [" + error + "]");
       });
       console.log(`FINISHED`);
   } catch (error) {
       const errors = [];
       error.forEach(function(item) {
         const msg = item.message.split(".");
         errors.push(msg[1]);
       });
       console.error(`ERROR ` + JSON.stringify(errors));
       alert("ERROR: Unable to generate Payment Token: " + JSON.stringify(errors));
   }
```
### Code Examples

#### Using only required parameters

```
const result = widget.createPaymentToken(oauthToken, clientAccnum, clientSubacc);
```

#### Using field clearing flags

```
const result = widget.createPaymentToken(oauthToken, clientAccnum, clientSubacc, clearPaymentInfo, clearCustomerInfo);
```

#### Using time to live and number of use but no flags

```
const result = widget.createPaymentToken(oauthToken, clientAccnum, clientSubacc, null, null, timeToLive, numberOfUse);
```

#### Using all parameters

```
const result = widget.createPaymentToken(oauthToken, clientAccnum, clientSubacc, clearPaymentInfo, clearCustomerInfo, timeToLive, numberOfUse);
```

### Step 4. Payment Token Generated

The response returned from `createPaymentToken` function will contain the newly created payment token, which can be converted into JSON format.

In case of any invalid input the response will include validation errors that were found during input validation.

Finally, it may also include any errors that might have happened during the process of payment token generation.

#### Field Data Validation

The `createPaymentToken` function will validate the input field values. If any of the values do not pass validation, the library will generate a **violations array** which will hold the information on which exact inputs were invalid.

| PARAMETER    | REQUIREMENT                                                                                                                                          |
|--------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| clientAccnum | A range between 900000 and 999999 and must be a number.                                                                                              |
| clientSubacc | A range of 0-9999 and must be a number.                                                                                                              |
| timeToLive   | A range of 0-2147483647 and must be a number, if max value is desired then just leave this out (or pass null).                                       |
| numberOfUse  | A range of 0-2147483647 and must be a number, if max value is desired then just leave this out (or pass null).                                       |
| cardNumber   | Must be a valid credit card number.                                                                                                                  |
| expMonth     | A range of 1-12, it is required and must be a number.                                                                                                |
| expYear      | A range of 2018-2100, is required and must be a number.                                                                                              |
| firstName    | Required                                                                                                                                             |
| lastName     | Required                                                                                                                                             |
| address1     | Optional                                                                                                                                             |
| city         | Optional                                                                                                                                             |
| country      | Required and must be represented as a two-letter country code as defined in [ISO 3166-1](https://www.iso.org/obp/ui/#iso:std:iso:3166:-1:ed-4:v1:en) |
| state        | Optional, but if provided must be a two-letter state code as defined in [ISO 3166-2](https://www.iso.org/obp/ui/#iso:std:iso:3166:-2:ed-4:v1:en)     |
| postalCode   | Must be a valid postal code for the country provided.                                                                                                |
| phoneNumber  | If provided, must be a valid telephone number.                                                                                                       |
| email        | Must be a valid email address.                                                                                                                       |
| ipAddress    | Optional (If present, valid IP addresses must be provided as a request parameter or through the **X-Origin-IP** header).                             |

The violations object is an array of the following objects:

```
{
  target: Object,
  message: string,
  propertyName: string
}
```

## Strong Customer Authentication

The `CCBill Advanced Widget` enables merchants to integrate with CCBill's 3DS vendor and thus create an additional layer of security for their transactions using `strong customer authentication (SCA)`.

Alternatively, merchants can send SCA (3DS) parameters obtained from a CCBill's 3DS vendor. To initiate charges using a payment token with this additional layer of security, the SCA parameters must be provided along with the other required parameters.

The OAuth token must be valid and bound to the provided merchant account.

### isScaRequired Function

The `isScaRequired` function determines whether the strong customer authentication is required by law. The system uses the provided credit card number, the merchant account number, the subaccount number, and the currency code to determine if SCA is required. If the input is valid the function will return a Promise object, which will eventually resolve to a response which includes the SCA parameters, or it will reject the call, due to any errors.

| PARAMETER                   | TYPE    | DESCRIPTION                                                    |
|-----------------------------|---------|----------------------------------------------------------------|
| **authToken** (required)    | string  | Must be a valid OAuth token for the provided merchant account. |
| **clientAccnum** (required) | integer | Merchant account number.                                       |
| **clientSubacc** (required) | integer | Merchant subaccount number.                                    |

The merchant payment form needs to contain a (hidden if necessary) text input field or a (hidden if necessary) select element which will contain the information on the `currencyCode` to be used. The value must be represented by a three-digit currency code as defined in [ISO 4217 standard](https://www.iso.org/obp/ui#iso:std:iso:4217:ed-8:v1:en).

The Advanced Widget will automatically collect the `currencyCode` value if the form is created according to the outlined rules. Merchants can:

* Utilize the `data-ccbill` attribute to specify the currency code field.

      <input data-ccbill="currencyCode" type="text" />

      or

      <select data-ccbill="currencyCode" />
      
        <option>…</option>
        <option>…</option>
        ...
      
      </select>

* Use the default `_ccbillId_currencyCode` attribute.

      <input id="_ccbillId_currencyCode" type="text" />

      or

      <select id="_ccbillId_currencyCode" />

        <option>…</option>
        <option>…</option>
        ...

      </select>

#### Code Example

```
const result = widget.isScaRequired(authToken, clientAccnum, clientSubacc);
```

#### Field Data Validation

| PARAMETER                       | REQUIREMENT                                    |
|---------------------------------|------------------------------------------------|
| clientAccnum                    | A range of 900000-999999 and must be a number.  |
| clientSubacc                    | A range of 0-9999 and must be a number.        |
| currencyCode                    | Has to match the regular expression `^\\d{3}$` |
| credit card number (form input) | Must be a valid credit card number.            |


The violations object is an array of the following object:

```
{
  target: Object,
  message: string,
  propertyName: string
}
```

### isScaRequiredForPaymentToken Function

The `isScaRequiredForPaymentToken` function determines whether strong customer authentication is required **based on the provided payment token ID and the supplied currency code**. If the input is valid the function will return a Promise object, which will eventually resolve to a response with SCA parameters, or a rejection response, due to any errors.

| PARAMETER                     | TYPE   | DESCRIPTION                                                                                |
|-------------------------------|--------|--------------------------------------------------------------------------------------------|
| **authToken** (required)      | string | Must be a valid OAuth token for the provided merchant account.                             |
| **paymentTokenId** (required) | string | Unique string identifying the payment token, must match regular expression `[a-zA-Z0-9]+$` |                                                                                                                                                                                                                                                                    |

The merchant payment form needs to contain a (hidden if necessary) text input field or a (hidden if necessary) select element which will contain the information on the `currencyCode` to be used. The value must be represented by a three-digit currency code as defined in [ISO 4217 standard](https://www.iso.org/obp/ui#iso:std:iso:4217:ed-8:v1:en).

The Advanced Widget will automatically collect the `currencyCode` value if the form is created according to the outlined rules. Merchants can:

* Utilize the `data-ccbill` attribute to specify the currency code field.

      <input data-ccbill="currencyCode" type="text" />

      or

      <select data-ccbill="currencyCode" />
      
        <option>…</option>
        <option>…</option>
        ...
      
      </select>

* Use the default `_ccbillId_currencyCode` attribute.

      <input id="_ccbillId_currencyCode" type="text" />

      or

      <select id="_ccbillId_currencyCode" />

        <option>…</option>
        <option>…</option>
        ...

      </select>

#### Code Example

```
const result = widget.isScaRequiredForPaymentToken(authToken, paymentTokenId);
```

#### Field Data Validation

| PARAMETER      | REQUIREMENT                                                                                |
|----------------|--------------------------------------------------------------------------------------------|
| paymentTokenId | Unique string identifying the payment token, must match regular expression `[a-zA-Z0-9]+$` |
| currencyCode   | Has to match regular expression `^\\d{3}$`                                                 |


The violations object is an array of the following object:

```
{
  target: Object,
  message: string,
  propertyName: string
}
```

### authenticateCustomer Function

The `authenticateCustomer` function enables merchants to obtain the strong customer authentication parameters before initiating a 3DS transaction and calling CCBill's Merchant Connect (RESTful) API `charge 3DS transaction` endpoint. If the call fails, it is going to return an error.

Several parameters must be passed to facilitate the strong customer authentication:

| PARAMETER                     | TYPE   | DESCRIPTION                                                                                                                                                                                                                                                                                                                              |
|-------------------------------|--------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **authToken** (required)      | string | Must be a valid OAuth token for the provided merchant account.                                                                                                                                                                                                                                                                           |
| **clientAccnum** (required)   | number | The clientAccnum value must correspond to the one used in generating the OAuth token and must be a number within the 900000-999999 range.                                                                                                                                                                                                |
| **clientSubacc** (required)   | number | The clientSubacc value must correspond to the one used in generating the OAuth token and should be a number within the 0-9999 range.                                                                                                                                                                                                     |                                                                                                                                                                                      
| **form** (optional)           | string | The form reference should either be a valid selector or an HTML Form Element that exists on the merchant's web page. Please note that if the formId is not provided, the Widget will find the first form HTML element on the page and assume that that is the payment form.                                                              |
| **iframeId** (optional)       | string | The 3DS authentication process presents an iframe on the web page to perform its functionality. The Advanced Widget script generates an iframe and injects it into the merchant's web page if the parameter is undefined. If the provided value is null or an empty string, it is regenerated to fit the minimum technical requirements. |
| **paymentTokenId** (optional) | string | Use this optional field instead of the card number, card expiry month, and card expiry year. The card information must be present in the associated HTML form if the token ID is not provided.                                                                                                                                           |

#### Code examples

Using only required parameters:

```
const result = widget.authenticateCustomer(authToken, clientAccnum, clientSubacc);
```

Using the form:

```
const result = widget.authenticateCustomer(authToken, clientAccnum, clientSubacc, form);
```

Using iframeId in addition to the form:

```
const result = widget.authenticateCustomer(authToken, clientAccnum, clientSubacc, form, iframeId);
```

Using all parameters:

```
const result = widget.authenticateCustomer(authToken, form, clientAccnum, clientSubacc, iframeId, paymentTokenId);
```

#### Field Data Validation

| PARAMETER      | REQUIREMENT                                                                                                                                                                     |
|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| paymentTokenId | Unique string identifying the payment token, must match regular expression `[a-zA-Z0-9]+$`                                                                                      |
| clientAccnum   | A number within the 900000-999999 range. It must match the value used to create the OAuth token.                                                                                |
| clientSubacc   | A number within the 0-9999 range. It must match the value used to create the OAuth token.                                                                                       |
| form           | Must be a valid selector or an HTMLFormElement. If it's not provided, the system will attempt to collect the required SCA inputs from the first HTML form it finds on the page. |
| iframeId       | If it is not provided, the system generates one and injects it into the web page.                                                                                               |

### 3DS Authentication Error Codes

| **HTTP Status Code** | **HTTP Status Type**  | **Error Code** | **Error Message**                                                        |
|----------------------|-----------------------|----------------|--------------------------------------------------------------------------|
| 500                  | Internal Server Error | 200000         | Authorization Failed due to Unknown Issue.                               |
| 500                  | Internal Server Error | 200001         | Authorization Failed due to Unexpected Response from Third Party SCA.    |
| 500                  | Internal Server Error | 200400         | Authorization Failed due to Bad Request to Third Party SCA.              |
| 500                  | Internal Server Error | 200401         | Authorization Failed due to Unauthorized Request to Third Party SCA.     |
| 500                  | Internal Server Error | 200500         | Failed to Authenticate Transaction.                                      |
| 500                  | Internal Server Error | 200501         | API failed to parse JSON.                                                |
| 400                  | Bad Request           | 200502         | Payment token does not exist.                                            |
| 400                  | Bad Request           | 200503         | No card associated with payment token.                                   |
| 500                  | Internal Server Error | 200600         | Failed to Retrieve Status.                                               |
| 500                  | Internal Server Error | 200601         | API failed to Parse JSON.                                                |
| 404                  | Not Found             | 200602         | Transaction ID Not Found.                                                |
| 400                  | Bad Request           | 200603         | Validation Error Transaction ID not valid UUID.                          |
| 400                  | Bad Request           | 200604         | Validation Error Correlation Id not valid UUID.                          |
| 400                  | Bad Request           | 200605         | Validation Error Transaction Updated Format not YYYY-MM-DD HH:mm:ss zzz. |

## 3. Charge Payment Token ID

After you have generated a new bearer token, and after you have generated the payment token, you will then be able to use those two new tokens to charge the consumer’s credit card.

### Charge Payment Token (Without 3DS Authentication)

#### Endpoint URL

* **https://api.ccbill.com/transactions/payment-tokens/{paymentTokenId}**

#### Headers

* `Accept: application/vnd.mcn.transaction-service.api.v.1+json`

#### Request Parameters

| PARAMETER                            | TYPE       | DESCRIPTION                                                                                                                                                          |
|:-------------------------------------|:-----------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **clientAccnum** (required)          | integer    | Merchant account number.                                                                                                                                             |
| **clientSubacc** (required)          | integer    | Merchant subaccount number.                                                                                                                                          |
| **initialPrice** (required)          | float      | Initial transaction price.                                                                                                                                           |
| **initialPeriod** (required)         | integer    | The length (in days) of the initial billing period.                                                                                                                  |
| **currencyCode** (optional)          | string    | The three-digit currency code, as per the [ISO 4217 standard](https://www.iso.org/obp/ui#iso:std:iso:4217:ed-8:v1:en), to be used in the transaction.           |
| **recurringPrice** (optional)        | float      | The amount the consumer will be charged for each recurring bill.                                                                                                     |
| **recurringPeriod** (optional)       | integer    | The length of time between rebills.                                                                                                                                  |
| **rebills** (optional)               | integer    | The total number of times the subscription will rebill.                                                                                                              |
| **lifeTimeSubscription** (optional)  | boolean    | The presence of this variable with a value of **1** indicates that the transaction is a lifetime subscription.                                                       |
| **createNewPaymentToken** (optional) | boolean    | Whether to create and return a new payment token to be used for subsequent transactions or not.                                                                      |
| **passThroughInfo** (optional)       | Array[any] | Key/value pairs that can be passed through to the transaction service and received in the webhook response.                                                          |

#### Example Request

```
{
  "clientAccnum": 900000,
  "clientSubacc": 0,
  "initialPrice":19.99,
  "initialPeriod":30,
  "currencyCode":840,
  "recurringPrice":19.99,
  "recurringPeriod":30,
  "rebills":99,
  "lifeTimeSubscription":false,
  "createNewPaymentToken":false,
  "passThroughInfo":[
       {
          "name":"custom1",
          "value":"10000000942"
       },
       {
          "name":"custom2",
          " value":"10058"
       }
  ]
}
```

#### Response Parameters

| PARAMETER         | TYPE    | DESCRIPTION                                                                     |
|-------------------|---------|---------------------------------------------------------------------------------|
| declineCode       | integer | The error code pertaining to the error that has caused the transaction failure. |
| declineText       | boolean | Description of the reason why the transaction was declined.                     |
| declineId         | string  | Randomly generated GUID unique to this specific error occurrence.               |
| approved          | boolean | Approval status of the transaction.                                             |
| paymentUniqueId   | string  | Unique key connected to the payment account.                                    |
| sessionId         | string  | Unique session ID value pertaining to the transaction.                          |
| subscriptionId    | integer | Subscription ID which uniquely identifies the transaction.                      |
| newPaymentTokenId | string  | The new payment token ID to be used for subsequent transactions (if created).   |

#### Example Response

```
{
  "declineCode": null,
  "declineText": null,
  "denialId": null,
  "approved": true,
  "paymentUniqueId": "dG4P1t8dL58pA3rNxE+Phw",
  "sessionId": null,
  "subscriptionId": 121095101000018190,
  "newPaymentTokenId":null
}
```

### Charge Payment Token (With 3DS Authentication)

#### Endpoint URL

* **https://api.ccbill.com/transactions/payment-tokens/threeds/{payment_token_id}**

#### Headers

* `Accept: application/vnd.mcn.transaction-service.api.v.1+json`

#### Request Parameters

| PARAMETER                                    | TYPE    | DESCRIPTION                                                                                                                                                                                      |
|----------------------------------------------|---------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **createNewPaymentToken**                    | boolean | Whether to create and return a new payment token to be used for subsequent transactions or not.                                                                                                  |
| **initialPrice** (required)                  | number  | Price of the initial transaction.                                                                                                                                                                |
| **clientAccnum** (required)                  | integer | Merchant account number.                                                                                                                                                                         |
| **clientSubacc** (required)                  | integer | Merchant subaccount number.                                                                                                                                                                      |
| **initialPeriod** (required)                 | integer | The length (in days) of the initial billing period.                                                                                                                                              |
| **currencyCode** (optional)                  | string | The three-digit currency code, as per the [ISO 4217 standard](https://www.iso.org/obp/ui#iso:std:iso:4217:ed-8:v1:en), to be used in the transaction.                                            |
| **recurringPrice** (optional)                | number  | The amount the consumer will be charged for each recurring bill.                                                                                                                                 |
| **recurringPeriod** (optional)               | Integer | The length of time between rebills.                                                                                                                                                              |
| **rebills** (optional)                       | integer | The total number of times the subscription will rebill.                                                                                                                                          |
| **lifeTimeSubscription** (optional)          | Boolean | The presence of this variable with a value of **1** indicates that the transaction is a lifetime subscription.                                                                                   |
| **passThroughInfo** (optional)               | array   | Key/value pairs that can be passed through to the transaction service and received in the webhook response.                                                                                      |
| **threedsCardToken** (required)              | string | The encrypted cardToken you receive through the 3DS verification process. As we require only the first 16 characters, trim the string to that length before sending it to the CCBill API. Sending a string longer than 16 characters results in an error. <br/>Example value: **gjeoB5NdJ1r6p0dG**                                                                                                                           |
| **threedsEci** (required)                    | string  | An Electronic Commerce Indicator (ECI).<br />Possible values are: '**0**', '**1**', '**2**', '**5**', '**6**', or '**7**'.                                                                       |
| **threedsStatus** (required)                 | string  | The status of the 3DS verification ('**Y**', '**N**', '**A**', etc.)                                                                                                                             |
| **threedsVersion** (required)                | string  | The version of the 3DS protocol to be followed for this specific card and transaction.<br />The supported versions are **1.0.2** and **2.1.0**                                                   |
| **threedsXid** (optional/required)           | string  | The transaction identifier (**XID**) is a unique tracking number set by the merchant for 3DS. **It is a required parameter for threedsVersion 1.0.2**                                            |
| **threedsCavv** (optional/required)          | string  | A digital signature that proves that the transaction has been 3DS verified. The signature is obtained through the 3DS verification flow and is a **required parameter for threedsVersion 1.0.2** |
| **threedsCavvAlgorithm** (optional/required) | string  | CAVV algorithm to be used in the 3DS request. **The threedsCavvAlgorithm parameter is required for 3DS version 1.0.2**                                                                           |
| **threedsDsTransId** (optional/required)     | string  | The Directory Server Transaction ID. **The threedsDsTransId parameter is required for 3DS version 2.1.0**                                                                                        |
| **threedsAcsTransId** (optional/required)    | string  | Access Control Server Transaction ID. **The threedsAcsTransId parameter is required for 3DS version 2.1.0**                                                                                      |
| **threedsSdkTransId** (optional)             | string  | The 3DS vendor's transaction ID.                                                                                                                                                                 |
| **threedsAuthenticationType** (optional)     | string  | A digital signature that proves that the transaction has been 3DS verified. The signature is obtained through the 3DS verification flow (**v2.1.0**).                                            |
| **threedsAuthenticationValue** (optional)    | string  | A digital signature that proves that the transaction has been 3DS verified. The signature is obtained through the 3DS verification flow (**v2.1.0**).                                            |
| **threedsClientTransactionId** (required)    | string  | The parameter is automatically generated by the `CCBill Advanced Widget`. Its purpose is to identify the origin of the 3DS authentication transaction.                                           |
| **threedsSuccess** (required)                | boolean | The result of the 3DS verification process.                                                                                                                                                      |
| **threedsAmount** (optional)                 | integer | The amount to be charged (must be the same value as the **initialPrice**).                                                                                                                       |
| **threedsCurrency** (optional)               | integer | The 3-digit currency code for the currency to be used in this transaction.<br />Example value: **840**                                                                                           |
| **threedsError** (optional/required)         | string  | The error received from the 3DS vendor during the `strong customer authentication` process. **This parameter is required if the SCA parameters are not provided.**                               |
| **threedsErrorDetail** (optional)            | string  | Error details related to the `threedsError` parameter.                                                                                                                                           |
| **threedsErrorCode** (optional)              | string  | The error code related to the `threedsError` parameter.                                                                                                                                          |
| **threedsResponse** (optional)               | string  | The complete response received in case of an error encountered during the 3DS verification process.                                                                                              |

#### Example Request

```
<?php
  $request = new HttpRequest();
  $request->setUrl('https://api.ccbill.com/transactions/payment-tokens/threeds/01047ed6f3b440c7a2ccc6abc1ad0a84');
  $request->setMethod(HTTP_METH_POST);
  $request->setHeaders(array(
    'Cache-Control' => 'no-cache',
    'Authorization' => 'Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsibWNuLXRyYW5zYWN0aW9uLXNlcnZpY2UiLCJtY24tYWRtaW4tc2VydmljZSJdLCJzY29wZSI6WyJjcmVhdGVfdG9rZW4iLCJyZWFkX3Rva2VuIiwiY2hhcmdlX3Rva2VuIiwiY3JlYXRlX3Byb2dyYW0iLCJyZWFkX3Byb2dyYW0iLCJjcmVhdGVfcHJvZ3JhbV9wYXJ0aWNpcGF0aW9uIiwicmVhZF9wcm9ncmFtX3BhcnRpY2lwYXRpb24iLCJtb2RpZnlfcHJvZ3JhbV9wYXJ0aWNpcGF0aW9uIl0sImV4cCI6MTUzNzM4MDczNiwiYXV0aG9yaXRpZXMiOlsiTUNOX0FQSV9UT0tFTl9DSEFSR0VSIiwiTUNOX0FQSV9UT0tFTl9DUkVBVE9SIiwiTUNOX0FQSV9BRE1JTiJdLCJqdGkiOiI4YzI2Njg1MC00NjMzLTQzZDMtYjZjOC1lNzIyY2ExNjQ1YTUiLCJjbGllbnRfaWQiOiI1MjE3NjhhYTc1OGQxMWU4YWE2YjAwNTA1NjlkMTU4NSJ9.HRYXZFATkIcI2_LJ1W_xo67IfBnbN9atyYNzyHqseLxYUxzgwBsAV5rNqCixKemOrDIeQLBN4jrwRsBIHDpEvshwBC8XmTodDJzpGmMaU9s1r20RV68X0_d1yTgSDke_Of7VCrVmJRbSuDl7AgsfTqQ1J7nWyu9vcIaER93ms-vadser_Ot9Z68_HAmCJL3DCLpdIFq3PYtBMKKKqXbvhfhSZQZD3b6-aewAnBo0VzpvK6tREqw1rv9_73oAvYcW2aHAj79ILr8viWMM40LyDKMMYOYkneg3hJUQsUVeh9WzztYUJKzERYNXje9fYIGN-eofoLvX7OZJ3eXmIfkrfQ',
    'Content-Type' => 'application/json'
  ));
  $request->setBody('{"clientAccnum":"901505","clientSubacc":"0","initialPrice":"10.00","currencyCode":"840","initialPeriod":"30","threedsEci":"05","threedsError":"","threedsStatus":"Y","threedsSuccess":"true","threedsVersion":"1.0.2","threedsXid":"aWQteHc4ajJnNGIxZW8gICAgICA=","threedsCavv":"cGFzc3dvcmQxMjM0NTZwYXNzd28=","threedsCavvAlgorithm":"SHA-256","threedsAmount":"10","threedsClientTransactionId":"id-wl9r6duc5zj","threedsCurrency":"USD","threedsSdkTransId":"","threedsAcsTransId":"","threedsDsTransId":"","threedsAuthenticationType":"","threedsCardToken":"gjeoB5NdJ1r6p0dG","threedsErrorDetail":"","threedsErrorCode":"","threedsResponse":""}');

  try {
    $response = $request->send();
    echo $response->getBody();
  } catch (HttpException $ex) {
    echo $ex;
  }
?>
```

#### Response Parameters

| PARAMETER         | TYPE    | DESCRIPTION                                                                           |
|-------------------|---------|---------------------------------------------------------------------------------------|
| errorCode         | integer | The error code pertaining to the error encountered during the transaction processing. |
| approved          | boolean | Approval status of the transaction.                                                   |
| paymentUniqueId   | string  | Unique key connected to the payment account.                                          |
| sessionId         | string  | Unique session ID value pertaining to the transaction.                                |
| subscriptionId    | integer | Subscription ID which uniquely identifies the transaction.                            |
| newPaymentTokenId | string  | The new payment token ID to be used for subsequent transactions (if created).         |

#### Example Response
```
{
  "errorCode": "100100",
  "approved": "false",
  "paymentUniqueId": "dG4P1t8dL58pA3rNxE+Phw",
  "sessionId": null,
  "subscriptionId": "922243301000000141",
  "newPaymentTokenId": null
}
```

Once the payment token has been charged, a [webhooks HTTP POST notification](https://ccbill.com/doc/webhooks-user-guide) will be triggered so that you may capture the transaction information. This webhooks event will be of [“UpSaleSuccess”](https://ccbill.com/doc/webhooks-user-guide#ftoc-heading-19) type.

## Read Payment Token ID

Use this API endpoint to obtain the data on a previously made charge. You will need to identify the charge by supplying the relevant payment token ID.

### Endpoint URL

* [https://api.ccbill.com/payment-tokens/{paymentTokenId}](https://api.ccbill.com/payment-tokens/%7bpaymentTokenId%7d)

### Headers

* `Accept: application/vnd.mcn.transaction-service.api.v.2+json`

### Example Request

```
GET 'https://api.ccbill.com/payment-tokens/{{PAYMENT_TOKEN_ID}}' \

--header 'Authorization: Bearer {{BACKEND_ACCESS_TOKEN}}' \

--header 'Accept: application/vnd.mcn.transaction-service.api.v.2+json'
```

### Response Parameters

| PARAMETER              | TYPE          | DESCRIPTION                                                                                |
|------------------------|---------------|--------------------------------------------------------------------------------------------|
| paymentTokenId         | string        | Unique string identifying the payment token, must match regular expression `[a-zA-Z0-9]+$` |
| programParticipationId | integer       | The Program ID relevant to the Payment Token.                                              |
| originalPaymentTokenId | string        | Reference to the previous token ID.                                                        |
| clientAccnum           | integer       | Merchant account number.                                                                   |
| clientSubacc           | integer       | Merchant subaccount number.                                                                |
| createdDatetime        | datetime-only | Date and time of the creation of the payment token.                                        |
| timeToLive             | integer       | The time interval that defines how long the token will be valid (hours).                   |
| validNumberOfUse       | integer       | The total number of times the Payment Token can be used for purchases.                     |
| paymentInfoId          | string        | Information associated with the payment.                                                   |
| subscriptionId         | integer       | Identification of the subscription associated with the transaction.                        |

### Example Response:
```
{  
  "paymentTokenId": "01390f2aae864749a6437e007936529b",
  "programParticipationId": null,
  "originalPaymentTokenId": null,
  "clientAccnum": 999999,
  "clientSubacc": 0,
  "createdDatetime": "2021-04-02T23:09:02",
  "timeToLive": 30,
  "validNumberOfUse": 3,
  "paymentInfoId": null,
  "errors": null,
  "subscriptionId": "121092501000146223",
}
```

## CCBill RESTful API Error Codes

| **HTTP Status Code** | **HTTP Status Type**  | **Error Code** | **Error Message**                                                                                                                                                                                      |
|----------------------|-----------------------|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 400                  | Bad Request           | 100100         | Invalid client account.                                                                                                                                                                                |
| 400                  | Bad Request           | 100102         | Expired payment token.                                                                                                                                                                                 |
| 400                  | Bad Request           | 100105         | Invalid payout type.                                                                                                                                                                                   |
| 400                  | Bad Request           | 100106         | Invalid privacy type.                                                                                                                                                                                  |
| 400                  | Bad Request           | 100107         | Invalid program category.                                                                                                                                                                              |
| 400                  | Bad Request           | 100108         | Invalid payment type.                                                                                                                                                                                  |
| 400                  | Bad Request           | 100109         | Invalid program participation.                                                                                                                                                                         |
| 400                  | Bad Request           | 200000         | Problem with attribute size.                                                                                                                                                                           |
| 400                  | Bad Request           | 200000         | Attribute required.                                                                                                                                                                                    |
| 400                  | Bad Request           | 200000         | Problem with the attribute values.                                                                                                                                                                     |
| 400                  | Bad Request           | 200000         | Invalid credit card number.                                                                                                                                                                            |
| 400                  | Bad Request           | 200000         | Invalid email address.                                                                                                                                                                                 |
| 400                  | Bad Request           | 200000         | Invalid Client Accnum.                                                                                                                                                                                 |
| 400                  | Bad Request           | 200000         | Invalid Client Subacc.                                                                                                                                                                                 |
| 400                  | Bad Request           | 200000         | Invalid Subscription ID.                                                                                                                                                                               |
| 400                  | Bad Request           | 200000         | Invalid Target Client Accnum.                                                                                                                                                                          |
| 400                  | Bad Request           | 200000         | Invalid Target Client Subacc.                                                                                                                                                                          |
| 400                  | Bad Request           | 200000         | Invalid Program Participation ID.                                                                                                                                                                      |
| 400                  | Bad Request           | 200000         | Too many programs found between merchants.                                                                                                                                                             |
| 400                  | Bad Request           | 200000         | Program does not exist between merchants.                                                                                                                                                              |
| 400                  | Bad Request           | 200000         | Program is Inactive.                                                                                                                                                                                   |
| 400                  | Bad Request           | 200000         | Invalid Token.                                                                                                                                                                                         |
| 400                  | Bad Request           | 200000         | Customer and Payment Information required for zero Subscription ID.                                                                                                                                    |
| 400                  | Bad Request           | 200000         | Invalid 3DS Cavv.                                                                                                                                                                                      |
| 400                  | Bad Request           | 200000         | Invalid 3DS Cavv Algorithm.                                                                                                                                                                            |
| 400                  | Bad Request           | 200000         | Invalid 3DS Xid.                                                                                                                                                                                       |
| 400                  | Bad Request           | 200000         | Invalid 3DS Ds Trans Id.                                                                                                                                                                               |
| 400                  | Bad Request           | 200000         | Invalid 3DS Acs Trans Id.                                                                                                                                                                              |
| 400                  | Bad Request           | 200000         | Threeds request parameters need to contain either 3DS verification parameters or error parameters to be valid.                                                                                         |
| 400                  | Bad Request           | 200000         | Problem with attribute digit format.                                                                                                                                                                   |
| 400                  | Bad Request           | 200000         | Problem with data type.                                                                                                                                                                                |
| 400                  | Bad Request           | 200000         | Invalid Program ID.                                                                                                                                                                                    |
| 400                  | Bad Request           | 200000         | Invalid date range.                                                                                                                                                                                    |
| 401                  | Unauthorized          | N/A            | Invalid token.                                                                                                                                                                                         |
| 403                  | Forbidden             | 100020         | Forbidden                                                                                                                                                                                              |
| 404                  | Not Found             | 100101         | Invalid payment token.                                                                                                                                                                                 |
| 404                  | Not Found             | 100104         | Lookup Object Not Found.                                                                                                                                                                               |
| 500                  | Internal Server Error | 100103         | Transaction error.                                                                                                                                                                                     |
| 500                  | Internal Server Error | 100105         | Unable to complete the transaction required to create a payment token.                                                                                                                                 |
| 500                  | Internal Server Error | 100106         | There was an internal error, or a database error and the requested action could not be completed, or Data Link is inactive for user.                                                                   |
| 500                  | Internal Server Error | 100107         | The IP address the client was attempting to authenticate on was not in the valid range.                                                                                                                |
| 500                  | Internal Server Error | 100108         | The client's account has been deactivated for use on the Datalink system or the client is not permitted to perform the requested action.                                                               |
| 500                  | Internal Server Error | 100109         | The client has unsuccessfully logged into the system 3 or more times in the last hour. The client should wait an hour before attempting to login again and is advised to review the login information. |
| 500                  | Internal Server Error | 100110         | Throttled, number of transactions has reached limit.                                                                                                                                                   |
| 500                  | Internal Server Error | 100111         | Throttled, total money amount has reached the limit.                                                                                                                                                   |
| 500                  | Internal Server Error | 100112         | Unable to determine if 3DS is required.                                                                                                                                                                |
| 500                  | Internal Server Error | 100113         | Unable to retrieve authorized amount.                                                                                                                                                                  |

## CCBill Community

Become part of the CCBill community to get updates on the new features, help us improve the platform, and engage with other merchants and developers.

* Follow [@CCBillBIZ on Twitter](https://twitter.com/CCBillBIZ).
* Visit [CCBill's Blog](https://ccbill.com/blog) and read about latest developments in payment processing.

### Resources

* [API Documentation](https://ccbill.com/doc/ccbill-restful-transaction-api)
* [Webhooks User Guide](https://ccbill.com/doc/webhooks-user-guide)
* [Knowledge Base](https://ccbill.com/kb/)
* [Blog](https://ccbill.com/blog)

### Contact CCBill

Get in touch with us if you have questions or need help with the CCBill RESTful API.

<p align="left">
  <a href="https://twitter.com/CCBillBIZ">Twitter</a> •
  <a href="https://www.facebook.com/ccbillBIZ/">Facebook</a> •
  <a href="https://ae.linkedin.com/company/ccbill">LinkedIn</a> •
  <a href="https://www.instagram.com/ccbillbiz/">Instagram</a> •
  <a href="https://www.youtube.com/c/CCBillBiz/featured">YouTube</a> •
  <a href="https://ccbill.com/contact">Support</a>
</p>
