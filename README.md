<h1 align="center">
  <br>
  <a href="https://kb.ccbill.com/CCBill+API"><img src="https://user-images.githubusercontent.com/81383705/118320543-b10b5400-b4fc-11eb-8290-98b0d7351ca8.png" alt="ccbill-restful-api" width="300"></a>
  <br>
  CCBill RESTful API
  <br>
</h1>

<p align="center">
This document outlines the API resources and endpoints of the CCBill Transaction RESTful API service. Merchants can use these resources to charge consumers with a payment token. This instructional document is provided as a technical resource to CCBill Merchants. It is intended to be read by programmers, technicians, and others with advanced coding skills.
</p>

<p align="center">
  <a href="https://kb.ccbill.com/CCBill+API">API</a> •
  <a href="https://ccbill.com/kb/">Knowledge Base</a> •
  <a href="https://kb.ccbill.com/FlexForms">FlexForms</a> •
  <a href="https://kb.ccbill.com/CCBill+Pay+Merchant+FAQs">CCBill Pay</a> •
  <a href="https://ccbill.com/contact">Support</a>
</p>

## Requirements

* The user has already a [CCBill Account](https://admin.ccbill.com/loginMM.cgi).
* The user has been given [API credentials](https://ccbill.com/contact).
* The user has had their origin request whitelisted. 
* The user possesses intermediate to advanced programming skills.
* User has experience with HTTP/1.1.
* User has experience with RESTful Web Services.
* User has experience with JSON formats.
* The RESTful Transaction API supports TLS 1.2 only.

## Terminology

* **Merchant Account**. Each CCBill merchant receives an account number for tracking purposes. The standard format is 9xxxxx-xxxx, where 9xxxxx is the main account. The main account is a six (6) digit number. For example: "999999".

* **Merchant Sub-account**. CCBill Merchants may open one or more sub-accounts. The sub-account is a four (4) digit number. The standard format is: xxxx. For example: "1234". The sub-account is part of the main account.

* **Payment Token**. A Payment Token identifies a billable entity within the system.

* **Subscription ID:**  Transaction subscription identification number.

* **Merchant Application ID:**  This is the username that was setup for authentication of this API.

* **Merchant Secret:**  This is the password that was setup for authentication of this API.

* **Strong Customer Authentication (SCA):** European laws, such as[PSD2](https://ccbill.com/blog/what-is-psd2), require the use of SCA, such as the [3DS](https://ccbill.com/kb/3d-secure-2) protocol, for online payment processing. When an EU-based cardholder makes a payment online, SCA is initiated. Merchants can use CCBill's Advanced Widget and its functions to facilitate strong customer authentication.

## The Payment Flow

When it comes to using the CCBIll RESTful API, there are 4 request in order for CCBill to properly capture a consumer’s card information and charge their credit card. Below is the sequence of request that need to take place in order to charge a consumer’s credit card.

1. Generate Frontend Token Bearer

2. Create Payment Token ID

3. Generate Backend Token Bearer

4. Charge Payment Token ID

## Authentication and Authorization

The CCBill RESTful Transaction API uses bearer token-based authentication and authorization. Prior to accessing the API, you need to register your application with CCBill. Upon registration, your app will be assigned a merchant application ID and secret key. Use these credentials to generate a token by providing them to the authorization server. Once you have generated an access token (not to be confused with a payment token), provide it in the Authorization header of each API request. 

You will have access until the access token expires or is revoked. The acquired token is a random string of data that does not hold any important piece of information or has value on its own. It works only as an authentication and authorization tool and grants access to an application.

If the authorization token isn't valid, you will receive a 401 or 403 response code.

### Endpoint URL

* [https://api.ccbill.com/ccbill-auth/oauth/token](https://api.ccbill.com/ccbill-auth/oauth/token)

### Headers

* **Content-Type: application/x-www-form-urlencoded**

 * **Authorization: Basic Merchant_ApplicationID:Merchant_Secret**

### Example Request
```
curl - POST 'https://api.ccbill.com/ccbill-auth/oauth/token' \

--header 'Content-Type: application/x-www-form-urlencoded' \

--header 'Authorization: Basic Merchant_ApplicationID:Merchant_Secret ' \

--data-urlencode 'grant_type=client_credentials'
``` 
## How to Integrate with the CCBill Advanced Widget

The CCBill Advanced Widget enables merchants to automate payment token requests. Merchants can design their own interface to call the widget and generate payment tokens.

The JavaScript widget is hosted in a location accessible to merchants allowing them to reference and import the widget into their websites.

The following instructions describe how to set up and use the Advanced Widget library.


### Step 1. Add Preload Link and HTML Elements

Add a preload link and script html elements to the client HTML page that will connect to the CCBill Advanced Widget:
```
<link rel="preload" href="https://js.ccbill.com/v1.2.0/ccbill-advanced-widget.js" as="script"/>
<script type="text/javascript" src="https://js.ccbill.com/v1.2.0/ccbill-advanced-widget.js"></script>
```
**Note:** The version in this URI example is **v1.2.0**. Pay special attention to the version in the URI path as the version number may be subject to change.

### Step 2. Define Field IDs

The Advanced Widget library knows how to extract the relevant form values by relying on default ID attributes or by utilizing the **data-ccbill** attribute. Using the **data-ccbill** attribute is recommended as it is less intrusive and provides more flexibility. The correct format is:
```
data-ccbill='[corresponding field]'
```

The table shows the values that should be set for the **data-ccbill** attribute or, alternatively, in the default ID attribute fields.

| **data-ccbill**                                                                                 | **Default IDs**                                                                                             |
| ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **cardNumber**                                                                                  | **\_ccbillId\_cardNumber**                                                                                  |
| **expMonth**                                                                                    | **\_ccbillId\_expMonth**                                                                                    |
| **expYear**                                                                                     | **\_ccbillId\_expYear**                                                                                     |
| **firstName**                                                                                   | **\_ccbillId\_firstName**                                                                                   |
| **lastName**                                                                                    | **\_ccbillId\_lastName**                                                                                    |
| **address1**                                                                                    | **\_ccbillId\_address1**                                                                                    |
| **address2 (optional)**                                                                         | **\_ccbillId\_address2** (optional)                                                                         |
| **city**                                                                                        | **\_ccbillId\_city**                                                                                        |
| **country**                                                                                     | **\_ccbillId\_country**                                                                                     |
| **state** (State/Province, must pass "XX" if not specified)                                     | **\_ccbillId\_state** (State/Province, must pass "XX" if not specified)                                     |
| **postalCode**                                                                                  | **\_ccbillId\_postalCode**                                                                                  |
| **phoneNumber** (optional)                                                                      | **\_ccbillId\_phoneNumber** (optional)                                                                      |
| **email**                                                                                       | **\_ccbillId\_email**                                                                                       |
| **ipAddress** (recommended hidden field auto populated by JavaScript)                           | **\_ccbillId\_ipAddress** (recommended hidden field auto populated by JavaScript)                           |
| **browserHttpAccept** (optional, recommended hidden field auto populated by JavaScript)         | **\_ccbillId\_browserHttpAccept** (optional, recommended hidden field auto populated by JavaScript)         |
| **browserHttpAcceptEncoding** (optional, recommended hidden field auto-populated by javascript) | **\_ccbillId\_browserHttpAcceptEncoding** (optional, recommended hidden field auto-populated by javascript) |
| **browserHttpAcceptLanguage** (optional, recommended hidden field auto-populated by javascript) | **\_ccbillId\_browserHttpAcceptLanguage** (optional, recommended hidden field auto-populated by javascript) |


### Step 3. Create JavaScript Method

Create a JavaScript method that will call the CCBill Advanced Widget **createPaymentToken()** function. The following example is provided and can be modified as required:

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
### Step 4. Payment Token Generated

The result will contain the newly created payment token, which can be stringified into JSON. It may also contain validation violations that occurred on the data being passed from the form or indicate any errors that might have happened while generating the Payment Token.

At this time, the Client Page will need to either resolve any errors, display the right message on which fields were invalid, or use the new payment token provided.

## Create Payment Token ID

The primary function of the Advanced Widget is **createPaymentToken**. Using this function either returns a created payment token output or an error message if the payment token could not be created with the provided inputs.

### createPaymentToken Function 

This is the main function that Merchants need to incorporate into their JavaScript calls in order to create Payment Tokens. To generate a token multiple parameters need to be passed using the function:

|      PARAMETER                      |      TYPE      |      DESCRIPTION                                                                                                                                                                                                                                                                                    |
|:-------------------------------------|:----------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|     **authToken** (required)            |     string     |     Required input that uses an Oauth token to perform the creation of the Payment Token. This must be a valid Oauth Token for the client account provided.                                                                                                                                     |
|     **clientAccnum** (required)         |     integer    |     Merchant account number.                                                                                                                                                                                                                                                                        |
|     **clientSubacc** (required)         |     integer    |     Merchant subaccount number.                                                                                                                                                                                                                                                                     |
|     **clearPaymentInfo** (optional)     |     boolean    |     Optional input that will   clear the Payment Information fields when the **createPaymentToken** function is called. This will default to not   clearing the Payment Information fields. <br><br>**Note:** Even though this parameter is   optional, this field should be set to **'null'**   if not used.      |
|     **clearCustomerInfo** (optional)    |     boolean    |     Optional input that will clear the Customer Information fields   when the **createPaymentToken**   function is called. This will default to not clearing the Customer   Information fields.  <br><br>**Note:** Even   though this parameter is optional, this field should be set to **'null'** if not used.    |
|     **timeToLive**  (optional)                    |     integer    |     Time for the token to   exist.                                                                                                                                                                                                                                                                  |
|     **numberOfUse** (optional)                    |     integer    |     Total number of times the Payment Token can be used for   purchases. <br><br>**Note:** Even though this parameter is optional, this field should be set to **'null'** if not used.                                                                                                                                                                                                                         |

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
### Field Data Validation

The **createPaymentToken** function will validate field values. If any of the values do not pass validation, no token will be created. If that is the case, the system will **generate a violations array** indicating which fields are invalid.

|      PARAMETER      |      REQUIREMENT                                                                                                         |
|:---------------------|--------------------------------------------------------------------------------------------------------------------------|
|     clientAccnum    |     A range of 90000-999999 and must be a number.                                                                         |
|     clientSubacc    |     A range of 0-9999 and must   be a number.                                                                             |
|     timeToLive     |     A range of 0-2147483647 and must be a number, if max value is   desired then just leave this out (or pass null).      |
|     numberOfUse     |     A range of 0-2147483647 and   must be a number, if max value is desired then just leave this out (or pass null).    |
|     cardNumber      |     Must be a valid credit card number.                                                                                   |
|     expMonth        |     A range of 1-12, is   required and must be a number.                                                                  |
|     expYear         |     A range of 2018-2100, is required and must be a number.                                                               |
|     firstName       |     Required                                                                                                             |
|     lastName        |     Required                                                                                                             |
|     address1        |     Required                                                                                                             |
|     city            |     Required                                                                                                             |
|     country         |     Required                                                                                                             |
|     state           |     Required ("XX" if not available)                                                                                     |
|     postalCode      |     Must be a valid postal code   for the country provided.                                                               |
|     phoneNumber     |     If provided, must be a valid telephone number.                                                                        |
|     email          |     Must be a valid email.                                                                                                |
|     ipAddress       |     Must be a valid IP.                                                                                                   |

The violations object is an array of the following object: 
```
{
  target: Object,
  message: string,
  propertyName: string
}
```

## Strong Customer Authentication

The CCBill Advanced Widget enables merchants to integrate with CCBill's 3DS vendor and incorporate strong customer authentication in their transactions.

Alternatively, merchants can send SCA (3DS) results obtained from a 3DS vendor of their choice. To initiate charges using a payment token the SCA parameters need to be submitted along with other required parameters.

The Oauth Token must be valid and bound to the provided Merchant Account.

### isScaRequired Function

The isScaRequired function determines whether strong customer authentication is needed. The system checks the provided credit card number, merchant account number, subaccount number, and currency code. A valid input results in a Promise, that will eventually resolve to a response with SCA parameters, or will turn into a rejection, due to an error.

|      PARAMETER                      |      TYPE      |      DESCRIPTION                                                                                                                                                                                                                                                                                    |
|:-------------------------------------|:----------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|     **authToken** (required)            |     string     |     Must be a valid Oauth Token for the provided Merchant Account.                                                                                                                                     |
|     **clientAccnum** (required)         |     integer    |     Merchant account number.                                                                                                                                                                                                                                                                        |
|     **clientSubacc** (required)         |     integer    |     Merchant subaccount number.                                                                                                                                                                                                                                                                     |
|     **currencyCode** (required)         |     integer    |     Three-digit currency code [ISO 4217 standard](https://www.iso.org/obp/ui/#search/code/) for the currency used in the transaction.       |

#### Code Example

```
const result = widget.isScaRequired(authToken, clientAccnum, clientSubacc, currencyCode);
```

#### Field Data Validation

|      PARAMETER      |      REQUIREMENT                                                                                                         |
|:---------------------|--------------------------------------------------------------------------------------------------------------------------|
|     clientAccnum    |     A range of 90000-999999 and must be a number.                                                                         |
|     clientSubacc    |     A range of 0-9999 and must   be a number.                                                                             |
|     currencyCode      |   Has to match regular expression **^\\d{3}$**       |
|     credit card number (form input)       |     Must be a valid credit card number.                                                                                   |


The violations object is an array of the following object:

```
{
  target: Object,
  message: string,
  propertyName: string
}
```

### isScaRequiredForPaymentToken Function

The **isScaRequiredForPaymentToken** function determines whether strong customer authentication is required **based on the provided payment token ID and currency code**. A valid input results in a Promise, that will eventually resolve to a response with SCA parameters, or will turn into a rejection, due to an error.

|      PARAMETER                      |      TYPE      |      DESCRIPTION                                                                                                                                                                                                                                                                                    |
|:-------------------------------------|:----------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|     **authToken** (required)            |     string     |     Must be a valid Oauth Token for the provided Merchant Account.                                                                                                                                     |
|     **paymentTokenId** (required)         |     string    |     The payment token ID.                                                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                                                    |
|     **currencyCode** (required)         |     integer    |     Three-digit currency code [ISO 4217 standard](https://www.iso.org/obp/ui/#search/code/) for the currency used in the transaction.       |

#### Code Example
```
const result = widget.isScaRequiredForPaymentToken(authToken, paymentTokenId, currencyCode);
```

#### Field Data Validation

|      PARAMETER      |      REQUIREMENT                                                                                                         |
|:---------------------|--------------------------------------------------------------------------------------------------------------------------|
|     paymentTokenId    |     Must match regular expression **[a-zA-Z0-9]+$**                                                                          |
|     currencyCode      |   Has to match regular expression **^\\d{3}$**       |


The violations object is an array of the following object:

```
{
  target: Object,
  message: string,
  propertyName: string
}
```

### authenticateCustomer Function

The **authenticateCustomer** function enables merchants to obtain the customer authentication result before initiating a 3DS transaction and calling CCBill's Merchant Connect (RESTful) API endpoint. If the call fails, it is going to return an error. 

Several parameters need to be passed to facilitate a strong customer authentication:

|      PARAMETER                      |      TYPE      |      DESCRIPTION                                                                                                                                                                                                                                                                                    |
|:-------------------------------------|:----------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|     **authToken** (required)            |     string     |     Must be a valid Oauth Token for the provided Merchant Account.                                                                                                                                     |
|     **paymentTokenId** (optional)         |     string    |     Use this optional field instead of the card number, card expiry month, and card expiry year. The card information must be present in the associated HTML form if the token ID is not provided.                                                                                                                                                                                                                                                                       |
|     **form** (optional)         |     string    |     A form reference should either be a valid selector or an HTML Form Element that exists on the merchant's web page.                                                                                                                                                                                                                                                                    |
|     **iframeId** (optional)         |     string    |     The 3DS authentication process presents an iframe on the web page to perform its functionality. The Advanced Widget script generates an iframe and injects it into the merchant's web page if the parameter is undefined. If the provided value is null or an empty string, it is regenerated to fit the minimum technical requirements.       |

#### Code examples

Using only required parameters:
```
const result = widget.authenticateCustomer(authToken);
```
Using form:
```
const result = widget.authenticateCustomer(authToken, form);
```
Using iframeId together with form:
```
const result = widget.authenticateCustomer(authToken, form, iframeId);
```
Using all parameters:
```
const result = widget.authenticateCustomer(authToken, form, iframeId, paymentTokenId);
```

#### Field Data Validation

|      PARAMETER      |      REQUIREMENT                                                                                                         |
|:---------------------|--------------------------------------------------------------------------------------------------------------------------|
|     paymentTokenId    |     Must match regular expression **[a-zA-Z0-9]+$**                                                                          |
|     form     |   Must be a valid selector or HTMLFormElement. If it's not provided, the systems attempt to collect the required SCA inputs from the first HTML form it finds on the page.       |
|     iframeId    |     If it is not provided, the system generates one and injects it into the web page.                                                                         |

### 3DS Authentication Error Codes

| **HTTP Status Code** | **HTTP Status Type**  | **Error Code** | **Error Message**                                                        |
| -------------------- | --------------------- | -------------- | ------------------------------------------------------------------------ |
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

## Charge Payment Token ID

After you have generated a new bearer token using the second set of credentials and have generated a payment token ID, you’ll then use those two new tokens to charge the consumer’s credit card.

### Charge Payment Token (Without 3DS Authentication)

#### Endpoint URL

* **https://api.ccbill.com/transactions/payment-tokens/{Payment_Token_ID}**

#### Headers

* **Accept: application/vnd.mcn.transaction-service.api.v.1+json**

#### Request Parameters

|     PARAMETER                    |     TYPE          |     DESCRIPTION                                                         |
|:----------------------------------|:-------------------|:-------------------------------------------------------------------------|
|     **clientAccnum** (required)    |     integer       |     Merchant account number.                                      |
|     **clientSubacc** (required)    |     integer       |     Merchant subaccount number.                                      |
|     **initialPrice** (required)    |     float        |      Initial transaction price.                                       |
|     **initialPeriod** (required)      |     integer       |     The length (in days) of the initial billing period.                                |
|     **recurringPrice** (optional)              |     float        |      The amount the consumer will be charged for each recurring bill.                                    |
|     **recurringPeriod** (optional)           |     integer       |     The length of time between rebills.                             |
|     **rebills** (optional)                     |     integer       |     The total number of times the subscription will rebill.                  |
|     **currencyCode** (optional)                |     integer       |     Three-digit currency code [ISO 4217 standard](https://www.iso.org/obp/ui/#search/code/) for the currency used in the transaction.    |
|     **lifeTimeSubscription** (optional)        |     boolean       |     The presence of this variable with a value of **1** indicates that the transaction is a lifetime subscription.                                    |
|     **createNewPaymentToken** (optional)        |     boolean       |     Return new payment token for subsequent transactions or not.       |
|     **passThroughInfo** (optional)             |     Array[any]    |     Paired information being passed through to the transaction service.      |

#### Example Request
```
{
	"clientAccnum": 900000,
	"clientSubacc": 0,
	"initialPrice":19.99,
	"initialPeriod":30,
    "recurringPrice":19.99,
    "recurringPeriod":30,
    "rebills":99,
    "currencyCode":840,
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


|     PARAMETER            |     TYPE       |     DESCRIPTION                                           |
|:--------------------------|:----------------|:-----------------------------------------------------------|
|     declineCode          |     integer    |     Error   condition value of the transaction.           |
|     declineText         |     boolean    |     Description of decline.                               |
|     declineId            |     string     |     Randomly generated GUID.                             |
|     approved             |     boolean    |     Approval status of the transaction.               |
|     paymentUniqueId      |     string     |     Unique key connected to payment account.            |
|     sessionId            |     string     |     Unique session ID value for transaction.             |
|     subscriptionId       |     integer    |     Subscription   ID to which the transaction belongs.    |
|     newPaymentTokenId    |     string     |     New   payment token for subsequent transactions.       |

#### Example Response
```
{"declineCode":null,"declineText":null,"denialId":null,"approved":true,"paymentUniqueId":"dG4P1t8dL58pA3rNxE+Phw","sessionId":null,"subscriptionId":121095101000018190,"newPaymentTokenId":null}
```

### Charge Payment Token (With 3DS Authentication)

#### Endpoint URL

* **https://api.ccbill.com/transactions/payment-tokens/threeds/{payment_token_id}**

#### Headers

* **Accept: application/vnd.mcn.transaction-service.api.v.1+json**

#### Request Parameters

| PARAMETER                                 | TYPE | DESCRIPTION                                                                                                                                                                                |
| --------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **createNewPaymentToken**                     | boolean  | Return new payment token for subsequent transactions or not.                                                                                                                                   |
| **initialPrice** (required)                  | number   | Price of the initial transaction.                                                                                                                                                              |
| **clientAccnum** (required)                  | integer  | Merchant account number.                                                                                                                                                                       |
| **clientSubacc** (required)                 | integer  | Merchant subaccount number.                                                                                                                                                                    |
| **initialPeriod** (required)                 | integer  | The length (in days) of the initial billing period.                                                                                                                                            |
| **recurringPrice** (optional)                | number   | The amount the consumer will be charged for each recurring bill.                                                                                                                               |
| **recurringPeriod** (optional)               | Integer  | The length of time between rebills.                                                                                                                                                            |
| **rebills** (optional)                       | integer  | The total number of times the subscription will rebill.                                                                                                                                        |
| **currencyCode** (optional)                  | integer  | Three-digit currency code ([ISO 4217 standard](https://www.iso.org/obp/ui/#search/code/)) for the currency used in the transaction.                                                            |
| **lifeTimeSubscription** (optional)          | Boolean  | The presence of this variable with a value of **1** indicates that the transaction is a lifetime subscription.                                                                                 |
| **passThroughInfo** (optional)               | array    | Paired information being passed through to the transaction service.                                                                                                                            |
| **threedsCardToken** (required)              | integer  | The payment card number (PAN).                                                                                                                                                                 |
| **threedsEci** (required)                    | string   | An Electronic Commerce Indicator (ECI).<br>Values: '**0**', '**1**', '**2**', '**5**', '**6**', or '**7**'.                                                                                    |
| **threedsStatus** (required)                 | string   | The status of the 3DS verification ('**Y**', '**N**', '**A**', etc.)                                                                                                                           |
| **threedsVersion** (required)                | string   | The version of the 3DS protocol to be followed for this specific card and transaction.<br>Available versions are **1.0.2** and **2.1.0**                                                       |
| **threedsXid** (optional/required)           | string   | The transaction identifier (**XID**) is a unique tracking number set by the merchant for 3DS. **It is a required parameter for threedsVersion 1.0.2**                                          |
| **threedsCavv** (optional/required)          | string   | A digital signature that proves that the transaction has been 3DS verified. The signature is obtained through a 3DS verification flow and is a **required parameter for threedsVersion 1.0.2** |
| **threedsCavvAlgorithm** (optional/required) | string   | CAVV Algorithm for threeds request. **The threedsCavvAlgorithm parameter is required for threedsVersion 1.0.2**                                                                                |
| **threedsDsTransId** (optional/required)     | string   | Directory Server Transaction ID. **The threedsDsTransId parameter is required for threedsVersion 2.1.0**                                                                                       |
| **threedsAcsTransId** (optional/required)    | string   | Access Control Server Transaction ID. **The threedsAcsTransId parameter is required for threedsVersion 2.1.0**                                                                                 |
| **threedsSdkTransId** (optional)             | string   | The 3DS vendor's transaction ID.                                                                                                                                                               |
| **threedsAuthenticationType** (optional)     | string   | A digital signature that proves that the transaction has been 3DS verified. The signature is obtained through a 3DS verification flow (**v2.1.0**).                                            |
| **threedsAuthenticationValue** (optional)    | string   | A digital signature that proves that the transaction has been 3DS verified. The signature is obtained through a 3DS verification flow (**v2.1.0**).                                            |
| **threedsClientTransactionId** (required)    | string   | The parameter is automatically generated by the Advanced Widget. Its purpose is to identify the origin of the 3DS authentication transaction.                                                                                                                             |
| **threedsSuccess** (optional)                | boolean  | Verification of 3DS authentication success or failure.                                                                                                                                         |
| **threedsAmount** (optional)                 | integer  | The amount to be charged (same as **initialPrice**).                                                                                                                                           |
| **threedsCurrency** (optional)               | integer  | The 3-digit currency code for the currency to be used in this transaction.<br>Example value: **840**                                                                                           |
| **threedsError** (optional/required)         | string   | Error received from the 3DS vendor during the strong customer authentication process. **The parameter is required if no threedsVersion is provided.**                                          |
| **threedsErrorDetail** (optional)            | string   | Error details related to the _threedsError_.                                                                                                                                                   |
| **threedsErrorCode** (optional)              | string   | Error code.                                                                                                                                                                                    |
| **threedsResponse** (optional)               | string   | The complete response in case of an error.                                                                                                                                                     |

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
$request->setBody('{"clientAccnum":"901505","clientSubacc":"0","initialPrice":"10.00","initialPeriod":"30","threedsEci":"05","threedsError":"","threedsStatus":"Y","threedsSuccess":"true","threedsVersion":"1.0.2","threedsXid":"aWQteHc4ajJnNGIxZW8gICAgICA=","threedsCavv":"cGFzc3dvcmQxMjM0NTZwYXNzd28=","threedsCavvAlgorithm":"SHA-256","threedsAmount":"10","threedsClientTransactionId":"id-wl9r6duc5zj","threedsCurrency":"USD","threedsSdkTransId":"","threedsAcsTransId":"","threedsDsTransId":"","threedsAuthenticationType":"","threedsCardToken":"4111111111111111","threedsErrorDetail":"","threedsErrorCode":"","threedsResponse":""}');
try {
$response = $request->send();
echo $response->getBody();
} catch (HttpException $ex) {
echo $ex;
}
?>
```

#### Response Parameters

| PARAMETER                    | TYPE | DESCRIPTION                                |
| --------------------------------- | -------- | ------------------------------------------------- |
| errorCode         | integer  | Error condition value of the transaction.         |
| approved           | boolean  | Description of decline.                           |
| paymentUniqueId    | string   | Unique key connected to the payment account.      |
| sessionId        | string   | Unique session ID value for the transaction.      |
| subscriptionId    | integer  | Subscription ID to which the transaction belongs. |
| newPaymentTokenId  | string   | New payment token for subsequent transactions.    |

#### Example Response
```
{
  "errorCode": "integer",
  "approved": "boolean",
  "paymentUniqueId": "string",
  "sessionId": "string",
  "subscriptionId": "integer",
  "newPaymentTokenId": "string"
}
```

Once the payment token has been charged, a [webhooks HTTP POST notification](https://kb.ccbill.com/Webhooks+User+Guide) will be triggered so that you may capture the transaction information. This webhooks event will be a [“UpSaleSuccess”](https://kb.ccbill.com/Webhooks+User+Guide#UpSaleSuccess).

## Read Payment Token ID

Use this API endpoint to obtain data on a previously made charge. You will need to identify the charge by the payment token ID.

### Endpoint URL

* [https://api.ccbill.com/payment-tokens/{paymentTokenId}](https://api.ccbill.com/payment-tokens/%7bpaymentTokenId%7d)

### Headers

* **Accept: application/vnd.mcn.transaction-service.api.v.2+json**

### Example Request
```
GET 'https://api.ccbill.com/payment-tokens/{{PAYMENT_TOKEN_ID}}' \

--header 'Authorization: Bearer {{BACKEND_ACCESS_TOKEN}}' \

--header 'Accept: application/vnd.mcn.transaction-service.api.v.2+json'
```
### Response Parameters


|     PARAMETER                 |     TYPE             |     DESCRIPTION                                                             |
|:-------------------------------|:----------------------|:-----------------------------------------------------------------------------|
|     paymentTokenId            |     string           |     Complex   representation of Payment Token Id.                           |
|     programParticipationId    |     integer          |     The   Program connected to the Payment Token.                            |
|     originalPaymentTokenId    |     string           |     Reference   to a previous Token Id.                                      |
|     clientAccnum             |     integer          |     Merchant   account number.                                               |
|     clientSubacc              |     integer          |     Merchant   subaccount number.                                            |
|     createdDatetime          |     datetime-only    |     Date and   time of creation of the Payment Token.                       |
|     timeToLive                |     integer          |     Time for   the token to exist (hours).                                   |
|     validNumberOfUse          |     integer          |     Total   number of times the Payment Token can be used for purchases.     |
|     paymentInfoId             |     string           |     Information   associated with the payment.                               |
|     subscriptionId            |     integer          |     Identification   of the subscription associated with the transaction.    |
|     cvv2Response              |     string           |     The   result of CVV2 verification.                                       |
|     avsResponse               |     string           |     The result   of AVS verification.                                       |

### Example Response:
```
{"paymentTokenId":"01390f2aae864749a6437e007936529b","programParticipationId":null,"originalPaymentTokenId":null,"clientAccnum":999999,"clientSubacc":0,"createdDatetime":"2021-04-02T23:09:02","timeToLive":30,"validNumberOfUse":3,"paymentInfoId":null,"errors":null,"subscriptionId":"121092501000146223","cvv2Response":M,"avsResponse":M}
```

## CCBill RESTful API Error Codes

| **HTTP Status Code** | **HTTP Status Type**  | **Error Code** | **Error Message**                                                                                                                                                                                      |
| -------------------- | --------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
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
| 401                  | Unauthorized          | N/A            | invalid\_token.                                                                                                                                                                                        |
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

Become part of the CCBill community to get updates on new features, help us improve the platform, and engage with other merchants and developers.

* Follow [@CCBillBIZ on Twitter](https://twitter.com/CCBillBIZ).
* Visit [CCBill's Blog](https://ccbill.com/blog) and read about latest developments in payment processing.

### Resources

* [API Documentation](https://kb.ccbill.com/CCBill+RESTful+Transaction+API)
* [Webhooks User Guide](https://kb.ccbill.com/Webhooks+User+Guide)
* [Knowledge Base](https://kb.ccbill.com/Welcome)
* [Blog](https://ccbill.com/blog)

### Contact CCBill

Get in touch with us if you have questions or need help with CCBill RESTful API.

<p align="left">
  <a href="https://twitter.com/CCBillBIZ">Twitter</a> •
  <a href="https://www.facebook.com/ccbillBIZ/">Facebook</a> •
  <a href="https://ae.linkedin.com/company/ccbill">LinkedIn</a> •
  <a href="https://www.instagram.com/ccbillbiz/">Instagram</a> •
  <a href="https://www.youtube.com/c/CCBillBiz/featured">YouTube</a> •
  <a href="https://ccbill.com/contact">Support</a>
</p>
