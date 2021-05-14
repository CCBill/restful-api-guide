<h1 align="center">
  <br>
  <a href="https://user-images.githubusercontent.com/81383705/118319705-6dfcb100-b4fb-11eb-969c-70b1992863c2.png" alt="ccbill-restful-api" width="300"></a>
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

* **Subscription ID:**  Transaction subscription identification number

* **Merchant Application ID:**  This is the username that was setup for authentication of this API.

* **Merchant Secret:**  This is the password that was setup for authentication of this API.

## The Payment Flow

When it comes to using the CCBIll RESTful API, there are 4 request in order for CCBill to properly capture a consumer’s card information and charge their credit card. Below is the sequence of request that need to take place in order to charge a consumer’s credit card.

1. Generate Frontend Token Bearer

2. Create Payment Token ID

3. Generate Backend Token Bearer

4. Charge Payment Token ID

## Authentication and Authorization:

The CCBill RESTful Transaction API uses bearer token-based authentication and authorization. Prior to accessing the API, you need to register your application with CCBill. Upon registration, your app will be assigned a merchant application ID and secret key. Use these credentials to generate a token by providing them to the authorization server. Once you have generated an access token (not to be confused with a payment token), provide it in the Authorization header of each API request. You will have access until the access token expires or is revoked. The acquired token is a random string of data that does not hold any important piece of information or has value on its own. It works only as an authentication and authorization tool and grants access to an application.

If the authorization token isn't authenticated correctly, you will either receive a 401 or 403 response code.

### End point URL:

* [https://api.ccbill.com/ccbill-auth/oauth/token](https://api.ccbill.com/ccbill-auth/oauth/token)

### Headers:

* **Content-Type: application/x-www-form-urlencoded**

 * **Authorization: Basic Merchant_ApplicationID:Merchant_Secret**

### Example Request:
```
curl - POST 'https://api.ccbill.com/ccbill-auth/oauth/token' \

--header 'Content-Type: application/x-www-form-urlencoded' \

--header 'Authorization: Basic Merchant_ApplicationID:Merchant_Secret ' \

--data-urlencode 'grant_type=client_credentials'
``` 
## Create Payment Token ID

The main function of the Advanced Widget is **createPaymentToken**. Using this function either returns a created payment token output or an error message if the payment token was unable to be created with the inputs provided.  
  
This is the main function that Merchants need to incorporate into their JavaScript calls in order to create **Payment Tokens**. There are multiple parameters that need to be passed into the function in order to generate a token:
|      Parameter                      |      Type      |      Description                                                                                                                                                                                                                                                                                    |
|:-------------------------------------|:----------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|     authToken (required)            |     string     |     Required input that uses an Oauth token to perform the creation   of the Payment Token. This must be a valid Oauth Token for the client account   provided.                                                                                                                                     |
|     clientAccnum (required)         |     integer    |     Merchant account number.                                                                                                                                                                                                                                                                        |
|     clientSubacc (required)         |     integer    |     Merchant subaccount number.                                                                                                                                                                                                                                                                     |
|     clearPaymentInfo (optional)     |     boolean    |     Optional input that will   clear the Payment Information fields when the **createPaymentToken** function is called. This will default to not   clearing the Payment Information fields. <br><br>**Note:** Even though this parameter is   optional, this field should be set to **'null'**   if not used.      |
|     clearCustomerInfo (Optional)    |     boolean    |     Optional input that will clear the Customer Information fields   when the **createPaymentToken**   function is called. This will default to not clearing the Customer   Information fields.  <br><br>**Note:** Even   though this parameter is optional, this field should be set to **'null'** if not used.    |
|     timeToLive                      |     integer    |     Time for the token to   exist.                                                                                                                                                                                                                                                                  |
|     validNumberOfUse                |     integer    |     Total number of times the Payment Token can be used for   purchases.                                                                                                                                                                                                                            |

### Code Examples

#### Using only required parameters
```
var result = widget.createPaymentToken(oauthToken, clientAccnum, clientSubacc);
```
#### Using field clearing flags
```
var result = widget.createPaymentToken(oauthToken, clientAccnum, clientSubacc, clearPaymentInfo, clearCustomerInfo);
```
#### Using time to live and number of use but no flags
```
var result = widget.createPaymentToken(oauthToken, clientAccnum, clientSubacc, null, null, timeToLive, numberOfUse);
```
#### Using all parameters
```
var result = widget.createPaymentToken(oauthToken, clientAccnum, clientSubacc, clearPaymentInfo, clearCustomerInfo, timeToLive, numberOfUse);
```
### Field Data Validation

The **createPaymentToken** function will validate field values. If any of the values do not pass validation, no token will be created. If that is the case, the system will **generate a violations array** indicating which fields are invalid.
|      Parameter      |      Requirement                                                                                                         |
|:---------------------|--------------------------------------------------------------------------------------------------------------------------|
|     clientAccnum    |     A range of 90000-999999 and must be a number                                                                         |
|     clientSubacc    |     A range of 0-9999 and must   be a number                                                                             |
|     timeToLive      |     A range of 0-2147483647 and must be a number, if max value is   desired then just leave this out (or pass null)      |
|     numberOfUse     |     A range of 0-2147483647 and   must be a number, if max value is desired then just leave this out (or pass   null)    |
|     cardNumber      |     Must be a valid credit card number                                                                                   |
|     expMonth        |     A range of 1-12, is   required and must be a number                                                                  |
|     expYear         |     A range of 2018-2100, is required and must be a number                                                               |
|     firstName       |     Required                                                                                                             |
|     lastName        |     Required                                                                                                             |
|     address1        |     Required                                                                                                             |
|     city            |     Required                                                                                                             |
|     country         |     Required                                                                                                             |
|     state           |     Required ("XX" if not available)                                                                                     |
|     postalCode      |     Must be a valid postal code   for the country provided                                                               |
|     phoneNumber     |     if provided, must be a valid telephone number                                                                        |
|     email           |     Must be a valid email                                                                                                |
|     ipAddress       |     Must be a valid IP                                                                                                   |

The violations object is an array of the following object: **{target: Object, message: string, propertyName: string}**

## How to Integrate with the CCBill Advanced Widget

The following instructions describe how to setup and use the Advanced Widget library.

### Step 1

Add a preload link and script html elements to the client HTML page that will connect to the CCBill Advanced Widget:
```
<link rel="preload" href="https://js.ccbill.com/v1/ccbill-advanced-widget.js" as="script"/>

<script type="text/javascript" src="https://js.ccbill.com/v1/ccbill-advanced-widget.js"></script>
```
### Step 2

The Advanced Widget library extracts values for the fields based on the ID of the fields. The CCBill Advanced Widget has a set a default IDs:  
|      Field ID     |      Field                                                                                                     |
|:--------------:|:----------------------------------------------------------------------------------------------------------------:|
|         1.        |     _ccbillId_cardNumber                                                                                       |
|         2.        |     _ccbillId_expMonth                                                                                         |
|         3.        |     _ccbillId_expYear                                                                                          |
|         4.        |     _ccbillId_firstName                                                                                        |
|         5.        |     _ccbillId_lastName                                                                                         |
|         6.        |     _ccbillId_address1                                                                                         |
|         7.        |     _ccbillId_address2 (optional)                                                                              |
|         8.        |     _ccbillId_city                                                                                             |
|         9.        |     _ccbillId_country                                                                                          |
|         10.       |     _ccbillId_state   (State/Province, Must pass "XX" if not specified)                                        |
|         11.       |     _ccbillId_postalCode                                                                                       |
|         12.       |     _ccbillId_phoneNumber   (optional)                                                                         |
|         13.       |     _ccbillId_email                                                                                            |
|         14.       |     _ccbillId_ipAddress   (recommended hidden field auto-populated by JavaScript)                              |
|         15.       |     _ccbillId_browserHttpAccept (optional, recommended hidden field   auto-populated by JavaScript)            |
|         16.       |     _ccbillId_browserHttpAcceptEncoding   (optional, recommended hidden field auto-populated by JavaScript)    |
|         17.       |     _ccbillId_browserHttpAcceptLanguage (optional, recommended   hidden field auto-populated by JavaScript)    |
|         18.       |     _ccbillId_browserHttpUserAgent   (optional, recommended hidden field auto-populated by JavaScript)         |

  
However, you can also customize the default set to use custom IDs. To specify custom IDs, the widget must have the custom IDs passed in thru the constructor or call a function that changes the expected ID.  
  
The following keywords are needed to change the expected ID of associated field:

-   firstName
-   lastName
-   address1
-   address2
-   city
-   state
-   country
-   postalCode
-   ipAddress
-   email
-   phoneNumber
-   browserHttpUserAgent
-   browserHttpAccept
-   browserHttpAcceptEncoding
-   browserHttpAcceptLanguage
-   cardNumber
-   expYear
-   expMonth
-   nameOnCard
```
var customIds = {firstName: "firstNameId", lastName: "lNameId", ipAddress:"ipId"};

var widget = new ccbill.MerchantConnectWidget(applicationId, customIds);
```
  
**OR**
```
var widget = new ccbill.CCBillAdvancedWidget(applicationId, customIds); 

widget.changeDefaultFieldId("phoneNumber", "phoneNumberId");

widget.changeDefaultFieldId("cardNumber", "creditCardId");
```
### Step 3

Create a JavaScript method that will call the CCBill Advanced Widget createPaymentToken() function. The following example can be modified as required:
```
var widget = new ccbill.CCBillAdvancedWidget(applicationId);

try {

 var result = widget.createPaymentToken(oauthToken, clientAccnum, clientSubacc, clearPaymentInfo, clearCustomerInfo, timeToLive, numberOfUse);

 result.then((data) => {

 console.log("SUCCESS");

 return data.json();  },

 (error) => {

 console.log("ERROR");

 return error.json();

 }).then(json => { 

 updateResult(json);

 }).catch((error) => {

 console.error("ERROR2 [" + error + "]");

 });

} catch (error) {

 console.log(`FINISHED`);

 } catch (error) {

 var errors = [];

 error.forEach(function(item) {

 var msg = item.message.split(".");

 errors.push(msg[1]);

});

 console.error(`ERROR ` + JSON.stringify(errors));

 alert("ERROR: Unable to generate Payment Token: " + JSON.stringify(errors));

 }
```
### Payment Token Generated

The result will contain the newly created payment token which can be stringified into JSON. It may also contain validation violations that occurred on the data being passed from the form or indicate any errors that might have happened during the process of generating the Payment Token.  
  
At this time the Client Page will need to either resolve any errors, display the proper message on which fields were invalid or make use of the new payment token provided.  

## Charge Payment Token ID:

After you have generated a new token bearer using the second set of credentials and have generated a payment token ID. You’ll then use those two new tokens to charge the consumer’s credit card. Once the payment token has been charged a [webhooks HTTP POST notification](https://kb.ccbill.com/Webhooks+User+Guide) will be triggered so that you may capture the transaction information. This webhooks event will be a “[UpSaleSuccess](https://kb.ccbill.com/Webhooks+User+Guide#UpSaleSuccess)”.

### End point URL:

* [https://api.ccbill.com/transactions/payment-tokens/{Payment_Token_ID}](https://api.ccbill.com/transactions/payment-tokens/%7bPayment_Token_ID%7d)

### Headers:

* **Accept: application/vnd.mcn.transaction-service.api.v.1+json**

### Request Parameters:
|     Parameter                    |     Type          |     Description                                                         |
|:----------------------------------|:-------------------|:-------------------------------------------------------------------------|
|     clientAccnum   (required)    |     integer       |     clientAccnum   of the merchant                                      |
|     clientSubacc   (required)    |     integer       |     clientSubacc   of the merchant                                      |
|     initialPrice   (required)    |     number        |     price of   initial transaction                                      |
|     initialPeriod(required)      |     integer       |     time   period of initial transaction                                |
|     recurringPrice               |     number        |     price of   recurrent transaction                                    |
|     recurringPeriod              |     integer       |     time   period of recurrent transactions                             |
|     rebills                      |     integer       |     number   of times recurrent transactions can occur                  |
|     currencyCode                 |     integer       |     numerical   representation of the currency used in a transaction    |
|     lifeTimeSubscription         |     boolean       |     subscription   is eternal or not                                    |
|     createNewPaymentToken        |     boolean       |     return   new payment token for subsequent transactions or not       |
|     passThroughInfo              |     Array[any]    |     Paired   Information Passed Through to the Transaction Service      |

### Example Request:
```
{

"clientAccnum":99999,

"clientSubacc":0,

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
### Response Parameters:


|     Parameter            |     Type       |     Description                                           |
|:--------------------------|:----------------|:-----------------------------------------------------------|
|     declineCode          |     integer    |     Error   condition value of the transaction            |
|     declineText          |     boolean    |     Description of decline                                |
|     declineId            |     string     |     Randomly   generated GUID                             |
|     approved             |     boolean    |     Approval   status of the transaction                  |
|     paymentUniqueId      |     string     |     Unique   key connected to payment account             |
|     sessionId            |     string     |     Unique   session ID value for transaction             |
|     subscriptionId       |     integer    |     Subscription   ID to which the transaction belongs    |
|     newPaymentTokenId    |     string     |     New   payment token for subsequent transactions       |

### Example Response:
```
{"declineCode":null,"declineText":null,"denialId":null,"approved":true,"paymentUniqueId":"dG4P1t8dL58pA3rNxE+Phw","sessionId":null,"subscriptionId":121095101000018190,"newPaymentTokenId":null}
```
## Read Payment Token ID:

Use this API call to obtain data on a previously made charge. You will need to identify the charge by the payment token ID.

### End point URL:

* [https://api.ccbill.com/payment-tokens/{paymentTokenId}](https://api.ccbill.com/payment-tokens/%7bpaymentTokenId%7d)

### Headers:

* **Accept: application/vnd.mcn.transaction-service.api.v.2+json**

### Example Request:
```
GET 'https://api.ccbill.com/payment-tokens/{{PAYMENT_TOKEN_ID}}' \

--header 'Authorization: Bearer {{BACKEND_ACCESS_TOKEN}}' \

--header 'Accept: application/vnd.mcn.transaction-service.api.v.2+json'
```
### Response Parameters:


|     Parameter                 |     Type             |     Description                                                             |
|:-------------------------------|:----------------------|:-----------------------------------------------------------------------------|
|     paymentTokenId            |     string           |     Complex   representation of Payment Token Id                            |
|     programParticipationId    |     integer          |     The   Program connected to the Payment Token                            |
|     originalPaymentTokenId    |     string           |     Reference   to a previous Token Id                                      |
|     clientAccnum              |     integer          |     Merchant   account number                                               |
|     clientSubacc              |     integer          |     Merchant   subaccount number                                            |
|     createdDatetime           |     datetime-only    |     Date and   time of creation of the Payment Token.                       |
|     timeToLive                |     integer          |     Time for   the token to exist (hours)                                   |
|     validNumberOfUse          |     integer          |     Total   number of times the Payment Token can be used for purchases     |
|     paymentInfoId             |     string           |     Information   associated with the payment                               |
|     subscriptionId            |     integer          |     Identification   of the subscription associated with the transaction    |
|     cvv2Response              |     string           |     The   result of CVV2 verification                                       |
|     avsResponse               |     string           |     The result   of AVS verification                                        |

### Example Response:
```
{"paymentTokenId":"01390f2aae864749a6437e007936529b","programParticipationId":null,"originalPaymentTokenId":null,"clientAccnum":999999,"clientSubacc":0,"createdDatetime":"2021-04-02T23:09:02","timeToLive":30,"validNumberOfUse":3,"paymentInfoId":null,"errors":null,"subscriptionId":"121092501000146223","cvv2Response":M,"avsResponse":M}
```


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
