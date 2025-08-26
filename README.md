<h1 align="center">
  <br>
  <a href="https://kb.ccbill.com/CCBill+API"><img src="https://user-images.githubusercontent.com/81383705/118320543-b10b5400-b4fc-11eb-8290-98b0d7351ca8.png" alt="ccbill-restful-api" width="300"></a>
  <br>
  CCBill RESTful API
  <br>
</h1>

<p align="center">
This document introduces the CCBill Transaction RESTful API service and the CCBill Advanced Widget, a JavaScript library for integrating API functionality into web pages. CCBill Merchants can use these tools to process payments with tokens. </p>
<p align="center">The content is intended for developers, technicians, and others with programming experience.
</p>

<p align="center">
  <a href="https://ccbill.com/doc/ccbill-restful-transaction-api">API</a> •
  <a href="https://ccbill.com/kb/">Knowledge Base</a> •
  <a href="https://ccbill.com/contact">Support</a>
</p>

## Terminology

* **Merchant Account:**. Each CCBill merchant receives a unique account number for tracking purposes. The standard format is 9xxxxx-xxxx, where 9xxxxx is the six (6) digit main account (e.g., **999999**).
* **Merchant Sub-account:**. Merchants may create one or more sub-accounts. A sub-account is a four (4) digit number (e.g., **1234**) and is tied to the main account.
* **[Payment Token](https://ccbill.com/kb/credit-card-tokenization)**. Identifies a billable entity within the system.
* **Subscription ID:**  Transaction subscription identification number.
* **Merchant Application ID:**  The client ID assigned when signing up to use the CCBill RESTful API.
* **Merchant Secret:**  The client password paired with the Application ID, used to authenticate with the CCBill RESTful API.
* **CCBill Advanced Widget:**  A JavaScript library for simplifying the integration of CCBill's payment system into your website or application.
* **[Strong Customer Authentication (SCA)](https://ccbill.com/kb/strong-customer-authentication):** European regulations ([PSD2](https://ccbill.com/blog/what-is-psd2)) require the use of SCA, such as the [3DS](https://ccbill.com/kb/3d-secure-2) protocol, for online payment processing. When an EU-based cardholder makes a payment online, SCA is initiated. Merchants can use CCBill's Advanced Widget to handle these authentication flows.

## Requirements

* The CCBill RESTful API supports TLS 1.2 only.
* A [CCBill Account](https://admin.ccbill.com/loginMM.cgi) with a client account number and two subaccounts used to generate payment tokens for 3DS and non-3DS transactions.
* Two sets of [API credentials](https://ccbill.com/contact) provided by CCBill:
   - **Frontend credentials** — used to obtain a **frontend bearer token** for the CCBill Advanced Widget (executed in the browser).  
  - **Backend credentials** — used to obtain a **backend bearer token** for server-to-server REST API authentication.   
* A whitelisted domain (contact [CCBill Support](https://ccbill.com/contact) for setup).
* Experience with **RESTful Web Services** and **JSON formats**.

## Payment Flow Overview

To integrate with CCBill's payment system and charge a customer you need to:

1. **Collect payment information** via a payment form.
2. **Generate frontend and backend OAuth tokens** using the credentials provided by CCBill.
3. **Create a payment token** from the captured card data.
4. **Charge the payment token** to finalize the transaction.

All of these steps (form handling, validation, token creation, and charging) can be completed through direct API calls to [CCBill’s RESTful API endpoints](https://ccbill.com/doc/ccbill-restful-api-resources).

However, we recommend you use the CCBill Advanced Widget. The JavaScript library serves as a convenience layer to automate Step 1 (form validation) and Step 3 (payment token creation).

## CCBill Advanced Widget

When using the CCBill Advanced Widget, your role is to provide a payment form with the correct attributes. The widget will attach logic to that form, validate the data, tokenize it, and return a ```paymentToken``` value you can subsequently charge.

Use the CCBill Advanced Widget because it:
* Automatically maps form fields to CCBill's API.
* Validates card numbers, expiration dates, and CVVs before sending.
* Submits card details directly to CCBill (never touching your server), then returns a payment token.
* Completes SCA directly within the browser, ensuring compliance with regulations like PSD2.

💡 **PCI Compliance Note:** Load CCBill JavaScript libraries only from `https://js.ccbill.com`. Do **not** bundle or self-host these scripts.

You can use the CCBill Advanced Widget to integrate both non-3DS and 3DS payment flows.

### Non-3DS Payment Flow

Use this flow when Strong Customer Authentication (SCA) is not required. It tokenizes payment details without 3D Secure (3DS) authentication and enables frictionless one-click payments.

Expand the tab for a detailed guide and code examples.

<details><summary><strong>👉 Create Payment Token (Non-3DS)</strong></summary><br>

To set up the **CCBill Advanced Widget** and create payment tokens **without 3DS authentication** you need to:

1.  Include the Widget on your page.
2.  Provide payment details.
3.  Generate the **frontend OAuth bearer token**.
4.  Utilize the payment details and bearer token to create  a **payment token**.
5.  Use the payment token and **backend OAuth bearer token** to process a transaction securely.

The diagram below shows the full flow:

<a href="https://github.com/user-attachments/assets/476eefe0-f1ee-4112-b893-764b20865c0d" target="_blank" rel="noopener"><img src="https://github.com/user-attachments/assets/476eefe0-f1ee-4112-b893-764b20865c0d" width="300"></a>

#### 1. Include the Widget in Your Page 

Add the following **preload link** and **script** elements to your HTML page:

```
<link rel="preload" href="https://js.ccbill.com/v1.13.1/ccbill-advanced-widget.js" as="script"/>

<script type="text/javascript" src="https://js.ccbill.com/v1.13.1/ccbill-advanced-widget.js"></script>
```
Pay special attention to the Widget version (**v1.13.1**) in the URI path, as the version number may be subject to change.

#### 2. Collect Customer and Payment Data

The widget extracts values from form fields. You can provide them in three ways:

<details><summary>👉 (Recommended) Use <code>data-ccbill</code> HTML data attributes.</summary><br>

Using <code>data-ccbill</code> data attributes is non-intrusive and provides more flexibility. You can map form inputs directly without modifying existing <code>id</code> attributes.
```
<form id="payment-form"> 
    <input data-ccbill="firstName" />
    <input data-ccbill="lastName" /> 
    <input data-ccbill="postalCode" />
    <input data-ccbill="country" /> 
    <input data-ccbill="email" /> 
    <input data-ccbill="cardNumber" /> 
    <input data-ccbill="expYear" /> 
    <input data-ccbill="expMonth" /> 
    <input data-ccbill="nameOnCard" /> 
    <input data-ccbill="cvv2" /> 
</form>
```
</details>
<details><summary>👉 Use default <code>_ccbillId_FieldName</code> ID attributes.</summary><br>

If you cannot modify your HTML to include <code>data-ccbill</code> attributes, use the default <code>_ccbillId_</code> attributes instead. They are less flexible because the field names must match CCBill's predefined format.
```
<form id="payment-form">
    <input id="_ccbillId_firstName" />
    <input id="_ccbillId_lastName" />
    <input id="_ccbillId_postalCode" />
    <input id="_ccbillId_country" />
    <input id="_ccbillId_email" />
    <input id="_ccbillId_cardNumber" />
    <input id="_ccbillId_expYear" />
    <input id="_ccbillId_expMonth" />
    <input id="_ccbillId_nameOnCard" />
    <input id="_ccbillId_cvv2" />
</form>
```
</details>
<details><summary>👉 Use custom ID attributes (requires additional mapping).</summary><br>

You can also map custom IDs to corresponding input fields using the <code>customIds</code> parameter in the Widget <code>constructor</code>.
```
<form id="payment-form">
    <input id="custom_firstName_id" />
    <input id="custom_lastName_id" />
    <input id="custom_postalCode_id" />
    <input id="custom_country_id" /> 
    <input id="custom_email_id" /> 
    <input id="custom_cardNumber_id" /> 
    <input id="custom_expYear_id" /> 
    <input id="custom_expMonth_id" /> 
    <input id="custom_nameOnCard_id" /> 
    <input id="custom_cvv2_id" /> 
</form>
<script>
// map custom ids to relevant fields
const customIds = {
    firstName: "custom_firstName_id",
    lastName: "custom_lastName_id",
    postalCode: "custom_postalCode_id",
    country: "custom_country_id",
    email: "custom_email_id",
    cardNumber: "custom_cardNumber_id",
    expYear: "custom_expYear_id", 
    expMonth: "custom_expMonth_id", 
    nameOnCard: "custom_nameOnCard_id",
    cvv2: "custom_cvv2_id"
};

// pass custom ids to Widget constructor
const widget = new ccbill.CCBillAdvancedWidget("application_id", customIds);

// call the desired Widget method

</script>
```
</details>

##### All Supported Form Fields

| **Name**                                        | **Required**                     | **Description**                                                         |
|-------------------------------------------------|---------------------------------|--------------------------------------------------------------------------|
| **firstName**                                   | Yes                             | Customer's first name.                                                  |
| **lastName**                                    | Yes                             | Customer's last name.                                           |
| **address1**                                    | No                              | Customer's billing address. If provided, it should be between 1 and 50 characters long.                                        |
| **address2**                                    | No                              | Customer's address (line 2). If provided, it should be between 1 and 50 characters long.                                        |
| **postalCode**                                  | Yes                             | Customer's billing zip code. It should be a valid zip code between 1 and 16 characters long.                                  |
| **city**                                        | No                              | Customer's billing city. If provided, it should be between 1 and 50 characters long.                                           |
| **state**                                       | No                              | Customer's billing state. If provided, it should be between 1 and 3 characters long.                                 |
| **country**                                     | Yes                             | Customer's billing country. Should be a two-letter country code as defined in ISO 3166-1.                             |
| **email**                                       | Yes                             | Customer's email. Should be a well-formed email address, max 254 characters long.                      |
| **phoneNumber**                                 | No                              | Customer's phone number. If provided, it should be a well-formed phone number.                                         |
| **ipAddress**                                   | No                              | Customer's IP address.                                                                                              |
| **browserHttpUserAgent**                        | No                              | Browser User-Agent header value.                                                                                    |
| **browserHttpAccept**                           | No                              | Browser Accept header value.                                                                                           |  
| **browserHttpAcceptEncoding**                   | No                              | Browser Accept Encoding header value.                                                                                           |                   
| **browserHttpAcceptLanguate**                   | No                              | Browser Accept Language header value.                        |
| **cardNumber**                                  | Yes                             | A valid credit card number.                                                                                            |
| **expMonth**                                    | Yes                             | Credit card expiration month in mm format. Should be a value between 1 and 12. |
| **expYear**                                     | Yes                             | Credit card expiration year in yyyy format. Should be a value between current year and 2100.                           |
| **cvv2**                                        | Yes                             | Card security code. Should be a 3-4 digit value.               |
| **nameOnCard**                                  | Yes                             | Name displayed on the credit card. Should be between 2 and 45 characters long.                   |

#### 3. Generate CCBill OAuth Bearer Token

The CCBill RESTful API uses [OAuth-based](https://ccbill.com/kb/what-is-oauth) authentication and authorization. Use the **frontend credentials** (Base64 encoded **`Merchant Application ID`** and **`Secret Key`**) you received from Merchant Support to generate a **frontend bearer token**.

You must include this token in the Authorization header of API requests when creating payment tokens. Use the following examples and adjust the necessary parameters to obtain a **frontend bearer token:**

<details><summary>👉 cURL</summary>

```
curl -X POST 'https://api.ccbill.com/ccbill-auth/oauth/token' \
  -u '[Frontend_Merchant_Application_ID]:[Frontend_Secret_Key]' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials'  
```

</details>
<details><summary>👉 Java</summary>

```
String getOAuthToken() {
    String credentials = Base64.getEncoder()
        .encodeToString(("[Frontend_Merchant_Application_ID]" + ":" + "[Frontend_Secret_Key]")
        .getBytes(StandardCharsets.UTF_8));
    String requestBody = "grant_type=client_credentials";

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.ccbill.com/ccbill-auth/oauth/token"))
           .header("Authorization", "Basic " + credentials)
            .header("Content-Type", "application/x-www-form-urlencoded")
           .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
            .build();

    try {
        HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
        return extractAccessToken(response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
        return null;
    }
}
```
</details>
<details><summary>👉 PHP</summary>

```
<?php

function getOAuthToken() {
    $url = "https://api.ccbill.com/ccbill-auth/oauth/token";
    $merchantAppId = "[Frontend_Merchant_Application_ID]";
    $secretKey = "[Frontend_Secret_Key]";
    $data = http_build_query(["grant_type" => "client_credentials"]);

    try {
        $httpRequest = new HttpRequest();
        $httpRequest->setUrl($url);
        $httpRequest->setMethod(HTTP_METH_POST);
        $httpRequest->setHeaders([
            "Authorization" => "Basic " . base64_encode("$merchantAppId:$secretKey"),
            "Content-Type" => "application/x-www-form-urlencoded"
        ]);
        $httpRequest->setBody($data);

        $httpClient = new HttpClient();
        $response = $httpClient->send($httpRequest);
        
        $responseData = json_decode($response->getBody(), true);
        return $responseData['access_token'] ?? die("Error: Invalid OAuth response.");
    } catch (HttpException $ex) {
        die("Error fetching OAuth token: " . $ex->getMessage());
    }
}

?>
```
</details>

⚠️**Important Note**

-   **Never expose API credentials on the front end.** Always store your Merchant Application ID and Secret Key securely in server-side environment variables.
-   **This request must be sent from your backend.** OAuth token requests cannot be made from a web browser for security reasons.
-   **OAuth access tokens are temporary.** Each token remains valid for a single request or until it expires.
-   **Reduce API token attack surface**. Execute calls to create an OAuth token and a payment token in quick succession to minimize the risk of the access token being exposed to attackers.
-   **Use CSRF tokens for your front-end** **payment forms**. Protect your front-end forms with CSRF tokens to prevent unauthorized form submissions.

#### 4. Generate Payment Token

Call the widget's [createPaymentToken()](https://ccbill.com/doc/method-reference#ftoc-heading-1) method with the **frontend token**, ```clientAccnum```, and ```clientSubacc```.

💻**Code Example**

```
async function createPaymentToken() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const paymentTokenResponse = await widget.createPaymentToken(
        "[Frondent_Access_Token]",
        [Your_Client_Account_Number],
        [Your_Client_Subaccount_Number]
    );
    return await paymentTokenResponse.json();
}
```

The **`createPaymentToken()`** function automatically validates all field values before generating a token:

-   A successful response returns a **payment token ID**, which is required to continue the payment flow.
-   If validation fails, the client page must display an appropriate error message and prompt them to resolve the invalid input before resubmitting the request.


#### 5. Charge Payment Token

To finalize a payment, send a request to charge the Payment Token through the backend. Generate a new **backend bearer token** using your Base64 encoded backend credentials. Then, pass the **backend bearer token** and **payment token ID** to the API endpoint and charge the customer's credit card.

💻**Code Examples**
<details><summary>👉 cURL</summary>
  
```
curl -X POST 'https://api.ccbill.com/transactions/payment-tokens/[payment_token_id]' \
  -H 'Accept: application/vnd.mcn.transaction-service.api.v.2+json' \
  -H 'Authorization: Bearer [Backend_Access_Token]' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
    "clientAccnum": [Your_Client_Account_Number],
    "clientSubacc": [Your_Client_Subaccount_Number],
    "initialPrice": 9.99,
    "initialPeriod": 30,
    "currencyCode": 840
  }'
```
</details>
  
<details><summary>👉 Java</summary>

```
public ResponseEntity<String> processPurchase() {
    String requestBody = """
        {
            "clientAccnum": [Your_Client_Account_Number],
            "clientSubacc": [Your_Client_Subaccount_Number],
            "initialPrice": 9.99,
            "initialPeriod": 30,
            "currencyCode": 840
        }""";

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.ccbill.com/transactions/payment-tokens/[payment_token_id]"))
            .header("Accept", "application/vnd.mcn.transaction-service.api.v.2+json")
            .header("Authorization", "Bearer [Backend_Access_Token]")
            .header("Cache-Control", "no-cache")
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
            .build();

    try {
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        return ResponseEntity.ok(response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
        return ResponseEntity.status(500).body("Error processing payment");
    }
}
```

</details>
<details><summary>👉 PHP</summary>

```
<?php

function processPurchase() {
    $url = "https://api.ccbill.com/transactions/payment-tokens/[payment_token_id]";
    $paymentData = json_encode([
        "clientAccnum" => [Your_Client_Account_Number],
        "clientSubacc" => [Your_Client_Subaccount_Number],
        "initialPrice" => 9.99,
        "initialPeriod" => 30,
        "currencyCode" => 840,
    ]);

    try {
        $httpRequest = new HttpRequest();
        $httpRequest->setUrl($url);
        $httpRequest->setMethod(HTTP_METH_POST);
        $httpRequest->setHeaders([
            "Accept" => "application/vnd.mcn.transaction-service.api.v.2+json",
            "Authorization" => "Bearer [Backend_Access_Token]",
            "Cache-Control" => "no-cache",
            "Content-Type" => "application/json"
        ]);
        $httpRequest->setBody($paymentData);

        $httpClient = new HttpClient();
        $response = $httpClient->send($httpRequest);
        
        return $response->getBody();
    } catch (HttpException $ex) {
        die("Error charging payment token: " . $ex->getMessage();
    }
}

?>
```
</details>

The endpoint validates the Payment Token and processes the transaction:

-   A successful charge returns a direct API response with transaction details (such as transaction ID, status, amount, and timestamps).
-   If the charge fails, the response will include an [error code](https://ccbill.com/doc/error-codes) and message explaining the reason for the failure. Use backend logic to handle the error and return a user-friendly message or trigger corrective actions.

#### Full Integration Example (Non-3DS)

This is a complete working example that demonstrates how to use the CCBill Advanced Widget on the front and back end. The example includes:

-   A **JavaScript frontend** that initializes the widget, generates a payment token and submits it to your backend.
-   A **Java-based backend** that handles the authorization and performs a server-side charge request.

Replace the placeholder values within the examples with your own application credentials and parameters.

<details><summary>🌐 JavaScript Frontend</summary>

```
async function fetchOAuthToken() {
    return (await (await fetch('https://your-website.com/api/auth-token')).json()).token;
}

async function createPaymentToken(widget, authToken, clientAccnum, clientSubacc) {
    const paymentTokenResponse = await widget.createPaymentToken(
        authToken,
        clientAccnum,
        clientSubacc
    );
    return await paymentTokenResponse.json();
}

async function chargePaymentToken(paymentToken) {
    return await (await (fetch('https://your-website.com/api/purchase', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            paymentToken,
            amount: 9.99,
            currency: 840
        })
    }))).json();
}

async function purchase() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    try {
        // create the payment token to be submitted to the merchant owned endpoint
        const paymentToken = await createPaymentToken(widget, 
            await fetchOAuthToken(), 
            [Your_Client_Account_Number], 
            [Your_Client_Subaccount_Number]);

        // submit the payment token to be charged to an endpoint implementing backend charging of the token
        const chargeCallResponse = await chargePaymentToken(paymentToken);
        return Promise.resolve(chargeCallResponse);
    } catch (error) {
        // react to any errors that may occur during the process
        return Promise.reject({error});
    }
}

let result = await purchase();
```
</details>
<details><summary>⚙️ Java Backend</summary>

```
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;
import java.time.Duration;
import java.util.Base64;
import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

@RestController
@RequestMapping("/api")
public class ApiController {

    private static final HttpClient HTTP_CLIENT = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .build();

    @PostMapping("/auth-token")
    public ResponseEntity<AuthTokenResponse> getAuthToken() {
        String accessToken = fetchOAuthToken("[Frontend_Merchant_Application_ID]", "[Frontend_Secret_Key]");
        if (accessToken != null) {
            return ResponseEntity.ok(new AuthTokenResponse(accessToken));
        } else {
            return ResponseEntity.status(500).body(new AuthTokenResponse(""));
        }
    }

    @PostMapping("/purchase")
    public ResponseEntity<String> processPurchase(@RequestBody PurchaseRequest purchaseRequest) {
        String requestBody = String.format(
                """
                {
                  "clientAccnum": %d,
                  "clientSubacc": %d,
                  "initialPrice": %.2f,
                  "initialPeriod": 30,
                  "currencyCode": %d
                }
                """,
                purchaseRequest.paymentToken().clientAccnum(),
                purchaseRequest.paymentToken().clientSubacc(),
                purchaseRequest.amount(),
                purchaseRequest.currency()
        );

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.ccbill.com/transactions/payment-tokens/" 
                    + purchaseRequest.paymentToken().paymentTokenId()))
                .header("Accept", "application/vnd.mcn.transaction-service.api.v.2+json")
                .header("Authorization", "Bearer " 
                    + fetchOAuthToken("[Backend_Merchant_Application_ID]", "[Backend_Secret_Key]"))
                .header("Cache-Control", "no-cache")
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
                .build();

        try {
            HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
            return ResponseEntity.ok(response.body());
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body("Error processing payment");
        }
    }

    private static String fetchOAuthToken(String merchantAppId, String sercretKey) {
        String credentials = Base64.getEncoder()
            .encodeToString((merchantAppId + ":" + sercretKey).getBytes(StandardCharsets.UTF_8));
        String requestBody = "grant_type=client_credentials";

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.ccbill.com/ccbill-auth/oauth/token"))
                .header("Authorization", "Basic " + credentials)
                .header("Content-Type", "application/x-www-form-urlencoded")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
                .build();

        try {
            HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
            return extractAccessToken(response.body());
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
            return null;
        }
    }

    private static String extractAccessToken(String responseBody) {
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(responseBody);
            return jsonNode.has("access_token") ? jsonNode.get("access_token").asText() : null;
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    private record AuthTokenResponse(String token) {}
    private record PurchaseRequest(double amount, String currency, PaymentToken paymentToken) {}
    private record PaymentToken(String paymentTokenId, Integer clientAccnum, Integer clientSubacc) {}
}
```
</details>
</details>

### 3DS Payment Flows

[3D Secure](https://ccbill.com/kb/3d-secure-2) is the industry standard for strong customer authentication. CCBill supports 3DS across its payment systems and is fully compliant with [PSD2 regulations](https://ccbill.com/kb/psd2-sca).

Select the 3DS payment flow that best matches your business model and planned checkout process.

<details><summary><strong>👉 Create Payment Token (3DS)</strong></summary><br>

To perform 3DS authentication based on the customer's payment details, create a **payment token** and then use the token to charge the customer:

1.  Include the Widget on your page.
2.  Provide payment details.
3.  Generate a **frontend OAuth Bearer Token**.
4.  Check whether 3DS authentication is required based on customer data (or a pre-existing payment token).
5.  Authenticate the customer.
6.  Utilize the payment details and frontend bearer token to create  a **payment token**.
7.  Use the payment token, authentication results, and **backend OAuth bearer token** to process the transaction securely.

Diagram of the flow:

<a href="https://github.com/user-attachments/assets/852d8560-551c-4768-9dc3-b8cee38a1d4c" target="_blank" rel="noopener"><img src="https://github.com/user-attachments/assets/852d8560-551c-4768-9dc3-b8cee38a1d4c" width="300"></a>

#### 1. Include the Widget in Your Page 

Add the following **preload link** and **script** elements to your HTML page:

```
<link rel="preload" href="https://js.ccbill.com/v1.13.1/ccbill-advanced-widget.js" as="script"/>

<script type="text/javascript" src="https://js.ccbill.com/v1.13.1/ccbill-advanced-widget.js"></script>
```
Pay special attention to the Widget version (**v1.13.1**) in the URI path, as the version number may be subject to change.

#### 2. Collect Customer and Payment Data

The Advanced Widget automatically extracts values from form fields. The required fields can be provided in three ways:

<details><summary>👉 (Recommended) Use <code>data-ccbill</code> HTML data attributes.</summary>

Using <code>data-ccbill</code> data attributes is non-intrusive and provides more flexibility. You can map form inputs directly without modifying existing <code>id</code> attributes.
```
<form id="payment-form"> 
    <input data-ccbill="firstName" />
    <input data-ccbill="lastName" /> 
    <input data-ccbill="postalCode" />
	<input data-ccbill="amount" />
    <input data-ccbill="country" /> 
    <input data-ccbill="email" /> 
    <input data-ccbill="cardNumber" />
	<input data-ccbill="currencyCode" />
    <input data-ccbill="expYear" /> 
    <input data-ccbill="expMonth" /> 
    <input data-ccbill="nameOnCard" /> 
    <input data-ccbill="cvv2" /> 
</form>
```

</details>

<details><summary>👉 Use default <code>_ccbillId_FieldName</code> ID attributes.</summary>

If you cannot modify your HTML to include <code>data-ccbill</code> attributes, use the default <code>_ccbillId_</code> attributes instead. The field names must match CCBill's predefined format.
```
<form id="payment-form">
    <input id="_ccbillId_firstName" />
    <input id="_ccbillId_lastName" />
    <input id="_ccbillId_postalCode" />
	<input id="_ccbillId_amount" />
    <input id="_ccbillId_country" />
    <input id="_ccbillId_email" />
    <input id="_ccbillId_cardNumber" />
	<input id="_ccbillId_currencyCode" />
    <input id="_ccbillId_expYear" />
    <input id="_ccbillId_expMonth" />
    <input id="_ccbillId_nameOnCard" />
    <input id="_ccbillId_cvv2" />
</form>
```
</details>

<details><summary>👉 Use custom ID attributes (requires additional mapping).</summary>

If you prefer custom IDs, map them to corresponding input fields using the <code>customIds</code> parameter in the Widget <code>constructor</code>.
```
<form id="payment-form">
    <input id="custom_firstName_id" />
    <input id="custom_lastName_id" />
    <input id="custom_postalCode_id" />
    <input id="custom_amount_id" /> 
    <input id="custom_country_id" /> 
    <input id="custom_email_id" /> 
    <input id="custom_cardNumber_id" />
    <input id="custom_currencyCode_id" /> 
    <input id="custom_expYear_id" /> 
    <input id="custom_expMonth_id" /> 
    <input id="custom_nameOnCard_id" /> 
    <input id="custom_cvv2_id" /> 
</form>
<script>
// map custom ids to relevant fields
const customIds = {
    firstName: "custom_firstName_id",
    lastName: "custom_lastName_id",
    postalCode: "custom_postalCode_id",
    amount: "custom_amount_id",
    country: "custom_country_id",
    email: "custom_email_id",
    currencyCode: "custom_currencyCode_id",
    cardNumber: "custom_cardNumber_id",
    expYear: "custom_expYear_id", 
    expMonth: "custom_expMonth_id", 
    nameOnCard: "custom_nameOnCard_id",
    cvv2: "custom_cvv2_id"
};

// pass custom ids to Widget constructor
const widget = new ccbill.CCBillAdvancedWidget("application_id", customIds);

// call the desired Widget method

</script>
```
</details>

##### All Supported Form Fields


| **Name**                                        | **Required**                     | **Description**                                                         |
|-------------------------------------------------|---------------------------------|--------------------------------------------------------------------------|
| **amount**                                      | Yes                             | Transaction total. Should be a value greater than 0.                      |
| **currencyCode**                                | Yes                             | A three-digit currency code ([ISO 4217 standard](https://www.iso.org/obp/ui/#search/code/)) for the currency used in the transaction. |
| **firstName**                                   | Yes                             | Customer's first name.                                                  |
| **lastName**                                    | Yes                             | Customer's last name.                                           |
| **address1**                                    | No                              | Customer's billing address. If provided, it should be between 1 and 50 characters long.                                        |
| **address2**                                    | No                              | Customer's address (line 2). If provided, it should be between 1 and 50 characters long.                                        |
| **postalCode**                                  | Yes                             | Customer's billing zip code. It should be a valid zip code between 1 and 16 characters long.                                  |
| **city**                                        | No                              | Customer's billing city. If provided, it should be between 1 and 50 characters long.                                           |
| **state**                                       | No                              | Customer's billing state. If provided, it should be between 1 and 3 characters long.                                 |
| **country**                                     | Yes                             | Customer's billing country. Should be a two-letter country code as defined in ISO 3166-1.                             |
| **email**                                       | Yes                             | Customer's email. Should be a well-formed email address, max 254 characters long.                      |
| **phoneNumber**                                 | No                              | Customer's phone number. If provided, it should be a well-formed phone number.                                         |
| **ipAddress**                                   | No                              | Customer's IP address.                                                                                              |
| **browserHttpUserAgent**                        | No                              | Browser User-Agent header value.                                                                                    |
| **browserHttpAccept**                           | No                              | Browser Accept header value.                                                                                           |  
| **browserHttpAcceptEncoding**                   | No                              | Browser Accept Encoding header value.                                                                                           |                   
| **browserHttpAcceptLanguate**                   | No                              | Browser Accept Language header value.                        |
| **cardNumber**                                  | Yes                             | A valid credit card number.                                                                                            |
| **expMonth**                                    | Yes                             | Credit card expiration month in mm format. Should be a value between 1 and 12. |
| **expYear**                                     | Yes                             | Credit card expiration year in yyyy format. Should be a value between current year and 2100.                           |
| **cvv2**                                        | Yes                             | Card security code. Should be a 3-4 digit value.               |
| **nameOnCard**                                  | Yes                             | Name displayed on the credit card. Should be between 2 and 45 characters long.                   |

#### 3. Generate CCBill OAuth Bearer Token

The CCBill RESTful API uses [OAuth-based](https://ccbill.com/kb/what-is-oauth) authentication and authorization. Use the **frontend credentials** (Base64 encoded **`Merchant Application ID`** and **`Secret Key`**) you received from Merchant Support to generate a **frontend bearer token**.

Include this token in the Authorization header of API requests when creating payment tokens. Use the following examples and adjust the necessary parameters to obtain a **frontend bearer token:**

<details><summary>👉 cURL</summary>

```
curl -X POST 'https://api.ccbill.com/ccbill-auth/oauth/token' \
  -u '[Frontend_Merchant_Application_ID]:[Frontend_Secret_Key]' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials'  
```
</details>
<details><summary>👉 Java</summary>
  
```
String getOAuthToken() {
    String credentials = Base64.getEncoder()
        .encodeToString(("[Frontend_Merchant_Application_ID]" + ":" + "[Frontend_Secret_Key]")
        .getBytes(StandardCharsets.UTF_8));
    String requestBody = "grant_type=client_credentials";

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.ccbill.com/ccbill-auth/oauth/token"))
           .header("Authorization", "Basic " + credentials)
            .header("Content-Type", "application/x-www-form-urlencoded")
           .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
            .build();

    try {
        HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
        return extractAccessToken(response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
        return null;
    }
}
```
</details>
<details><summary>👉 PHP</summary>
  
```
<?php

function getOAuthToken() {
    $url = "https://api.ccbill.com/ccbill-auth/oauth/token";
    $merchantAppId = "[Frontend_Merchant_Application_ID]";
    $secretKey = "[Frontend_Secret_Key]";
    $data = http_build_query(["grant_type" => "client_credentials"]);

    try {
        $httpRequest = new HttpRequest();
        $httpRequest->setUrl($url);
        $httpRequest->setMethod(HTTP_METH_POST);
        $httpRequest->setHeaders([
            "Authorization" => "Basic " . base64_encode("$merchantAppId:$secretKey"),
            "Content-Type" => "application/x-www-form-urlencoded"
        ]);
        $httpRequest->setBody($data);

        $httpClient = new HttpClient();
        $response = $httpClient->send($httpRequest);
        
        $responseData = json_decode($response->getBody(), true);
        return $responseData['access_token'] ?? die("Error: Invalid OAuth response.");
    } catch (HttpException $ex) {
        die("Error fetching OAuth token: " . $ex->getMessage());
    }
}

?>
```
</details>

⚠️**Important Note**

-   **Never expose API credentials on the front end.** Always store your Merchant Application ID and Secret Key securely in server-side environment variables.
-   **This request must be sent from your backend.** OAuth token requests cannot be made from a web browser for security reasons.
-   **OAuth access tokens are temporary.** Each token remains valid for a single request or until it expires.
-   **Reduce API token attack surface**. Execute calls to create an OAuth token and a payment token in quick succession to minimize the risk of the access token being exposed to attackers.
-   **Use CSRF tokens for your front-end** **payment forms**. Protect your front-end forms with CSRF tokens to prevent unauthorized form submissions.

#### 4. Check If SCA Is Required

Use the [isScaRequired()](https://ccbill.com/doc/method-reference#ftoc-heading-2) function to determine whether strong customer authentication is required before generating a payment token. The system checks the provided **credit card number**, **merchant account number**, **subaccount**, and **currency code**.

💻**Code Example**

```
async function checkIfScaRequired() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const scaRequiredResponse = await widget.isScaRequired(
        "[Frondent_Access_Token]", 
        [Your_Client_Account_Number], 
        [Your_3DS_Client_Subaccount_Number]);
    return await scaRequiredResponse.json();
}
```
<details><summary>Alternatively Check If 3DS Is Required Based on Existing Token</summary><br>

Merchants who have already stored payment information as a token (Payment Token ID) can use the [isScaRequiredForPaymentToken()](https://ccbill.com/doc/method-reference#ftoc-heading-3) function to determine if SCA is required before processing a charge.

💻**Code Example**

```
async function checkIfScaRequired() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const scaRequiredResponse = await widget.isScaRequiredForPaymentToken(
        "[Frondent_Access_Token]", 
        "[payment_token_id]");
    return await scaRequiredResponse.json();
}
```
</details>

The function automatically checks the transaction parameters to determine if strong customer authentication (SCA) is required:

- A successful response returns a Boolean value that indicates whether 3DS is required for the transaction. Use the result to dynamically route customers through a 3DS flow only when required. This ensures a better user experience and compliance with SCA regulations.
- If validation fails (e.g., invalid credentials), the response will show an error message to describe the issue.


#### 5. Authenticate Customer

If 3DS is required, call [authenticateCustomer()](https://ccbill.com/doc/method-reference#ftoc-heading-4).

💻**Code Example**

```
async function authenticate() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    return await widget.authenticateCustomer(
        "[Frondent_Access_Token]", 
        [Your_Client_Account_Number], 
        [Your_3DS_Client_Subaccount_Number]);
}
```

<details><summary>Authenticate Customer Based on Existing Token</summary><br>

Merchants who have already stored payment information as a token (**`paymentTokenID`**) can call the **`authenticateCustomer()`** function to authenticate a customer before processing a charge.

💻**Code Example**

```
async function authenticate() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    return await widget.authenticateCustomer(
        "[Frondent_Access_Token]", 
        [Your_Client_Account_Number], 
        [Your_Client_Subaccount_Number],
        null, null,
        "[payment_token_id]");
}
```

</details>

The function initiates the 3DS authentication flow and returns:

- A successful response that includes authentication data, which is required to proceed with a 3DS transaction.
- A relevant error code and description in case of failure. In this case, prompt the user to address the error and retry the authentication.


#### 6. Generate Payment Token

The [createPaymentToken()](https://ccbill.com/doc/method-reference#ftoc-heading-1) function is the primary method for generating a **Payment Token**. Call it to initiate a payment flow after collecting and validating customer data and generating a **frontend bearer token** using your frontend credentials.

💻**Code Example**

```
async function createPaymentToken(scaRequired) {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const clientSubacc = scaRequired ? [Your_3DS_Client_Subaccount_Number] : [Your_Client_Subaccount_Number];
    const paymentTokenResponse = await widget.createPaymentToken(
        "[Frondent_Access_Token]",
        [Your_Client_Account_Number],
        clientSubacc
    );
    return await paymentTokenResponse.json();
}
```

The **`createPaymentToken()`** function automatically validates all field values before generating a token:

-   A successful response returns a **payment token ID**, which is required to continue the payment flow.
-   If validation fails, the client page must display an appropriate error message and prompt them to resolve the invalid input before resubmitting the request.

#### 7. Charge Payment Token

Use the Payment Token ID and backend bearer token to charge a customer's credit card through a 3DS-secured payment flow. Generate a new backend bearer token using your Base64 encoded backend credentials.

Ensure the Payment Token passed the required 3DS authentication flow and the required 3DS values are collected.

💻**Code Examples**
<details><summary>👉 cURL</summary>
  
```
curl -X POST 'https://api.ccbill.com/transactions/payment-tokens/threeds/[payment_token_id]' \
  -H 'Accept: application/vnd.mcn.transaction-service.api.v.2+json' \
  -H 'Authorization: Bearer [Backend_Access_Token]' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
    "clientAccnum": [Your_Client_Account_Number],
    "clientSubacc": [Your_Client_Subaccount_Number],
    "initialPrice": 9.99,
    "initialPeriod": 30,
    "currencyCode": 978,
    "threedsEci": "05",
    "threedsStatus": "Y",
    "threedsSuccess": true,
    "threedsVersion": "2.2.0",
    "threedsAmount": 9.99,
    "threedsClientTransactionId": "id-wl9r6duc5zj",
    "threedsCurrency": "840",
    "threedsSdkTransId": "d535b6d1-19f9-11f0-92b9-0242ac110005",
    "threedsAcsTransId": "ca5f9649-b865-47ce-be6f-54422a0fce47",
    "threedsDsTransId": "e3693b86-8217-48c6-9628-2e8852dc60d4",
    "threedsAuthenticationType": "",
    "threedsAuthenticationValue": "Pes4aJnpT+1mjhUoBynC92iQbeg="
  }'
 ```

</details>
  
<details><summary>👉 Java</summary>

```
public ResponseEntity<String> processPurchase3ds() {
    String requestBody = """
        {
            "clientAccnum": [Your_Client_Account_Number],
            "clientSubacc": [Your_Client_Subaccount_Number],
            "initialPrice": 9.99,
            "initialPeriod": 30,
            "currencyCode": 978,
            "threedsEci": "05",
            "threedsStatus": "Y",
            "threedsSuccess": true,
            "threedsVersion": "2.2.0",
            "threedsAmount": 9.99,
            "threedsClientTransactionId": "id-wl9r6duc5zj",
            "threedsCurrency": "840",
            "threedsSdkTransId": "d535b6d1-19f9-11f0-92b9-0242ac110005",
            "threedsAcsTransId": "ca5f9649-b865-47ce-be6f-54422a0fce47",
            "threedsDsTransId": "e3693b86-8217-48c6-9628-2e8852dc60d4",
            "threedsAuthenticationType": "",
            "threedsAuthenticationValue": "Pes4aJnpT+1mjhUoBynC92iQbeg="
        }""";

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.ccbill.com/transactions/payment-tokens/threeds/[payment_token_id]"))
            .header("Accept", "application/vnd.mcn.transaction-service.api.v.2+json")
            .header("Authorization", "Bearer [Backend_Access_Token]")
            .header("Cache-Control", "no-cache")
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
            .build();

    try {
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        return ResponseEntity.ok(response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
        return ResponseEntity.status(500).body("Error processing payment");
    }
}

```
</details>
<details><summary>👉 PHP</summary>

```
<?php

function processPurchase3ds() {
    $url = "https://api.ccbill.com/transactions/payment-tokens/threeds/[payment_token_id]";
    $paymentData = json_encode([
        "clientAccnum" => [Your_Client_Account_Number],
        "clientSubacc" => [Your_Client_Subaccount_Number],
        "initialPrice" => 9.99,
        "initialPeriod" => 30,
        "threedsEci" => "05",
        "threedsStatus" => "Y",
        "threedsSuccess" => true,
        "threedsVersion" => "2.2.0",
        "threedsAmount" => 9.99,
        "threedsClientTransactionId" => "id-wl9r6duc5zj",
        "threedsCurrency" => "840",
        "threedsSdkTransId" => "d535b6d1-19f9-11f0-92b9-0242ac110005",
        "threedsAcsTransId" => "ca5f9649-b865-47ce-be6f-54422a0fce47",
        "threedsDsTransId" => "e3693b86-8217-48c6-9628-2e8852dc60d4",
        "threedsAuthenticationType" => "",
        "threedsAuthenticationValue" => "Pes4aJnpT+1mjhUoBynC92iQbeg="
    ]);

    try {
        $httpRequest = new HttpRequest();
        $httpRequest->setUrl($url);
        $httpRequest->setMethod(HTTP_METH_POST);
        $httpRequest->setHeaders([
            "Accept" => "application/vnd.mcn.transaction-service.api.v.2+json",
            "Authorization" => "Bearer [Backend_Access_Token]",
            "Cache-Control" => "no-cache",
            "Content-Type" => "application/json"
        ]);
        $httpRequest->setBody($paymentData);

        $httpClient = new HttpClient();
        $response = $httpClient->send($httpRequest);
        
        return $response->getBody();
    } catch (HttpException $ex) {
        die("Error charging payment token: " . $ex->getMessage();
    }
}

?>
```
</details>

The API endpoint handles the transaction:

 -  A successful charge returns a response with transaction details.
 -  If the charge fails, the response includes an error code and a descriptive message.


#### Full Integration Example (Non-3DS)

This is a full working example that shows how to implement a 3DS-compliant transaction. The example has:

- A JavaScript frontend that initializes the widget, collects payment data, and triggers the 3DS authentication flow.
- A backend in Java that handles bearer token generation, receives the Payment Token, and submits a 3DS charge request using the required data.

Replace all placeholder values with actual client account details, bearer tokens, and 3DS credentials.

<details><summary>🌐 JavaScript Frontend</summary>

```
async function fetchOAuthToken() {
    return (await (await fetch('https://your-website.com/api/auth-token')).json()).token;
}

async function checkIfScaRequired(widget, authToken, clientAccnum, clientSubacc) {
    const scaRequiredResponse = await widget.isScaRequired(authToken, clientAccnum, clientSubacc);
    return await scaRequiredResponse.json();
}

async function authenticate(widget, authToken, clientAccnum, clientSubacc) {
    return await widget.authenticateCustomer(authToken, clientAccnum, clientSubacc);
}

async function createPaymentToken(widget, authToken, clientAccnum, clientSubacc) {
    const paymentTokenResponse = await widget.createPaymentToken(
        authToken,
        clientAccnum,
        clientSubacc
    );
    return await paymentTokenResponse.json();
}

async function chargePaymentToken(paymentToken) {
    return await (await (fetch('https://your-website.com/api/purchase', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            paymentToken,
            amount: 9.99,
            currency: 840
        })
    }))).json();
}

async function chargePaymentToken3ds(paymentToken, threedsInformation) {
    return await (await (fetch('https://your-website.com/api/purchase-3ds', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            paymentToken,
            threedsInformation,
            amount: 9.99,
            currency: 840
        })
    }))).json();
}

async function authenticateAndPurchase() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const clientAccnum = [Your_Client_Account_Number];
    const clientSubacc = [Your_3DS_Client_Subaccount_Number];
 
    try {
        // retrieval of the auth token from merchant provided endpoint
        // this should be done as late in the submission process as possible to avoid potential exploit.
        const authToken = await fetchOAuthToken();
 
        let threedsInformation;
        // check if 3DS is required and process the 3DS flow with the client if necessary
        const scaRequired = await checkIfScaRequired(widget, authToken, clientAccnum, clientSubacc);
        if (scaRequired) {
   // go through 3DS flow
            threedsInformation = await authenticate(widget, authToken, clientAccnum, clientSubacc);
        }
 
        // create the payment token to be submitted to the merchant owned endpoint
        const paymentToken = await createPaymentToken(widget, authToken, clientAccnum, scaRequired ? clientSubacc : [Your_Client_Subaccount_Number]);
 
        // submit the payment token and 3DS information to the back-end endpoint implementing charging of the token
        const chargeCallResponse = scaRequired ? await chargePaymentToken3ds(paymentToken, threedsInformation)
            : await chargePaymentToken(paymentToken);
        return Promise.resolve(chargeCallResponse);
    } catch (error) {
        // react to any errors that may occur during the process
        return Promise.reject({error});
    }
}

let result = await authenticateAndPurchase();
```
</details>
<details><summary>⚙️ Java Backend</summary>

```
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;
import java.time.Duration;
import java.util.Base64;
import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

@RestController
@RequestMapping("/api")
public class ApiController {

    private static final HttpClient HTTP_CLIENT = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .build();

    @PostMapping("/auth-token")
    public ResponseEntity<AuthTokenResponse> getAuthToken() {
        String accessToken = fetchOAuthToken("[Frontend_Merchant_Application_ID]", "[Frontend_Secret_Key]");
        if (accessToken != null) {
            return ResponseEntity.ok(new AuthTokenResponse(accessToken));
        } else {
            return ResponseEntity.status(500).body(new AuthTokenResponse(""));
        }
    }
    
    @PostMapping("/purchase")
    public ResponseEntity<String> processPurchase(@RequestBody PurchaseRequest purchaseRequest) {
        String requestBody = String.format(
                """
                {
                  "clientAccnum": %d,
                  "clientSubacc": %d,
                  "initialPrice": %.2f,
                  "initialPeriod": 30,
                  "currencyCode": %d
                }
                """,
                purchaseRequest.paymentToken().clientAccnum(),
                purchaseRequest.paymentToken().clientSubacc(),
                purchaseRequest.amount(),
                purchaseRequest.currency()
        );

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.ccbill.com/transactions/payment-tokens/" 
                    + purchaseRequest.paymentToken().paymentTokenId()))
                .header("Accept", "application/vnd.mcn.transaction-service.api.v.2+json")
                .header("Authorization", "Bearer " 
                    + fetchOAuthToken("[Backend_Merchant_Application_ID]", "[Backend_Secret_Key]"))
                .header("Cache-Control", "no-cache")
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
                .build();

        try {
            HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
            return ResponseEntity.ok(response.body());
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body("Error processing payment");
        }
    }

    @PostMapping("/purchase-3ds")
    public ResponseEntity<String> processPurchase3ds(@RequestBody PurchaseRequest3ds purchaseRequest3ds) {
        String requestBody = String.format(
                """
                {
                  "clientAccnum": %d,
                  "clientSubacc": %d,
                  "initialPrice": %.2f,
                  "initialPeriod": 10,
                  "currencyCode": "%s",
                  "threedsEci": "%s",
                  "threedsStatus": "%s",
                  "threedsSuccess": %b,
                  "threedsVersion": "%s",
                  "threedsAmount": %.2f,
                  "threedsClientTransactionId": "%s",
                  "threedsCurrency": "%s",
                  "threedsSdkTransId": "%s",
                  "threedsAcsTransId": "%s",
                  "threedsDsTransId": "%s",
                  "threedsAuthenticationType": "%s",
                  "threedsAuthenticationValue": "%s"
                }
                """,
                purchaseRequest3ds.paymentToken().clientAccnum(),
                purchaseRequest3ds.paymentToken().clientSubacc(),
                purchaseRequest3ds.amount(),
                purchaseRequest3ds.currency(),
                purchaseRequest3ds.threedsInformation().eci(),
                purchaseRequest3ds.threedsInformation().status(),
                purchaseRequest3ds.threedsInformation().success(),
                purchaseRequest3ds.threedsInformation().protocolVersion(),
                purchaseRequest3ds.threedsInformation().amount(),
                purchaseRequest3ds.threedsInformation().clientTransactionId(),
                purchaseRequest3ds.threedsInformation().currency(),
                purchaseRequest3ds.threedsInformation().sdkTransId(),
                purchaseRequest3ds.threedsInformation().acsTransId(),
                purchaseRequest3ds.threedsInformation().dsTransId(),
                purchaseRequest3ds.threedsInformation().authenticationType(),
                purchaseRequest3ds.threedsInformation().authenticationValue()
        );

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.ccbill.com/transactions/payment-tokens/threeds/"
                    + purchaseRequest.paymentToken().paymentTokenId()))
                .header("Accept", "application/vnd.mcn.transaction-service.api.v.2+json")
                .header("Authorization", "Bearer " 
                    + fetchOAuthToken("[Backend_Merchant_Application_ID]", "[Backend_Secret_Key]"))
                .header("Cache-Control", "no-cache")
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
                .build();

        try {
            HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
            return ResponseEntity.ok(response.body());
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body("Error processing payment");
        }
    }

    private static String fetchOAuthToken(String merchantAppId, String sercretKey) {
        String credentials = Base64.getEncoder()
            .encodeToString((merchantAppId + ":" + sercretKey).getBytes(StandardCharsets.UTF_8));
        String requestBody = "grant_type=client_credentials";

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.ccbill.com/ccbill-auth/oauth/token"))
                .header("Authorization", "Basic " + credentials)
                .header("Content-Type", "application/x-www-form-urlencoded")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
                .build();

        try {
            HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
            return extractAccessToken(response.body());
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
            return null;
        }
    }

    private static String extractAccessToken(String responseBody) {
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(responseBody);
            return jsonNode.has("access_token") ? jsonNode.get("access_token").asText() : null;
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    private record AuthTokenResponse(String token) {}
    private record PurchaseRequest(double amount, String currency, PaymentToken paymentToken) {}
    private record PurchaseRequest3ds(double amount, String currency, PaymentToken paymentToken, 
        ThreedsInformation threedsInformation) {}
    private record PaymentToken(String paymentTokenId, Integer clientAccnum, Integer clientSubacc) {}
    private record ThreedsInformation(String eci, String status, boolean success, String protocolVersion, 
        double amount, String clientTransactionId, String currency, String sdkTransId, String acsTransId, 
        String dsTransId, String authenticationType, String authenticationValue) {}
}
```
</details>
</details>

Choose this flow if you run an **e-commerce store** or offer **instant access to digital content or services**. It supports one-click payments and lets you charge customers in real time while they’re still in session.

<details><summary><strong>👉 Authenticate Customer and Create Payment Token (One Step)</strong></summary><br>

To simultaneously perform a 3DS check on customer payment details and generate a payment token:

1.  Include the Widget on your page.
2.  Provide payment details.
3.  Generate the **frontend OAuth bearer token**.
4.  Check whether the 3DS authentication is required based on customer data (or a pre-existing payment token).
5.  Authenticate the customer and create a payment token in a single step.
6.  Use the payment token, authentication results, and **backend OAuth bearer token** to process a transaction securely.

The diagram below shows the full flow:

<a href="https://github.com/user-attachments/assets/852d8560-551c-4768-9dc3-b8cee38a1d4c" target="_blank" rel="noopener"><img src="https://github.com/user-attachments/assets/852d8560-551c-4768-9dc3-b8cee38a1d4c" width="300"></a>

#### 1. Include the Widget in Your Page 

To use the CCBill Advanced Widget, add the following **preload link** and **script** elements to your HTML page:

```
<link rel="preload" href="https://js.ccbill.com/v1.13.1/ccbill-advanced-widget.js" as="script"/>

<script type="text/javascript" src="https://js.ccbill.com/v1.13.1/ccbill-advanced-widget.js"></script>
```
Pay special attention to the Widget version (**v1.13.1**) in the URI path, as the version number may be subject to change.

#### 2. Collect Customer and Payment Data

The Advanced Widget automatically extracts values from form fields. The required fields can be provided in three ways:

<details><summary>👉 (Recommended) Use <code>data-ccbill</code> HTML data attributes.</summary><br>

Using <code>data-ccbill</code> data attributes is non-intrusive and provides more flexibility. You can map form inputs directly without modifying existing <code>id</code> attributes.
```
<form id="payment-form"> 
    <input data-ccbill="firstName" />
    <input data-ccbill="lastName" />
    <input data-ccbill="postalCode" />
    <input data-ccbill="amount" /> 
    <input data-ccbill="country" /> 
    <input data-ccbill="email" /> 
    <input data-ccbill="cardNumber" /> 
    <input data-ccbill="currencyCode" /> 
    <input data-ccbill="expYear" /> 
    <input data-ccbill="expMonth" /> 
    <input data-ccbill="nameOnCard" /> 
    <input data-ccbill="cvv2" /> 
</form>
```

</details>

<details><summary>👉 Use default <code>_ccbillId_FieldName</code> ID attributes.</summary><br>

If you cannot modify your HTML to include <code>data-ccbill</code> attributes, use the default <code>_ccbillId_</code> attributes instead. The field names must match CCBill's predefined format.
```
<form id="payment-form">
    <input id="_ccbillId_firstName" />
    <input id="_ccbillId_lastName" />
    <input id="_ccbillId_postalCode" />
    <input id="_ccbillId_amount" />
    <input id="_ccbillId_country" />
    <input id="_ccbillId_email" />
    <input id="_ccbillId_cardNumber" />
    <input id="_ccbillId_expYear" />
    <input id="_ccbillId_currencyCode" /> 
    <input id="_ccbillId_expMonth" />
    <input id="_ccbillId_nameOnCard" />
    <input id="_ccbillId_cvv2" />
</form>
```
</details>

<details><summary>👉 Use custom ID attributes (requires additional mapping).</summary><br>

Map custom IDs to corresponding input fields using the <code>customIds</code> parameter in the Widget <code>constructor</code>.
```
<form id="payment-form">
    <input id="custom_firstName_id" />
    <input id="custom_lastName_id" />
    <input id="custom_postalCode_id" />
    <input id="custom_amount_id" /> 
    <input id="custom_country_id" /> 
    <input id="custom_email_id" /> 
    <input id="custom_cardNumber_id" />
    <input id="custom_currencyCode_id" /> 
    <input id="custom_expYear_id" /> 
    <input id="custom_expMonth_id" /> 
    <input id="custom_nameOnCard_id" /> 
    <input id="custom_cvv2_id" /> 
</form>
<script>
// map custom ids to relevant fields
const customIds = {
    firstName: "custom_firstName_id",
    lastName: "custom_lastName_id",
    postalCode: "custom_postalCode_id",
    amount: "custom_amount_id",
    country: "custom_country_id",
    email: "custom_email_id",
    currencyCode: "custom_currencyCode_id",
    cardNumber: "custom_cardNumber_id",
    expYear: "custom_expYear_id", 
    expMonth: "custom_expMonth_id", 
    nameOnCard: "custom_nameOnCard_id",
    cvv2: "custom_cvv2_id"
};

// pass custom ids to Widget constructor
const widget = new ccbill.CCBillAdvancedWidget("application_id", customIds);

// call the desired Widget method

</script>
```
</details>

##### All Form Fields

| **Name**                                        | **Required**                     | **Description**                                                         |
|-------------------------------------------------|---------------------------------|--------------------------------------------------------------------------|
| **amount**                                      | Yes                             | Transaction total. Should be a value greater than 0.                      |
| **currencyCode**                                | Yes                             | A three-digit currency code ([ISO 4217 standard](https://www.iso.org/obp/ui/#search/code/)) for the currency used in the transaction. |
| **firstName**                                   | Yes                             | Customer's first name.                                                  |
| **lastName**                                    | Yes                             | Customer's last name.                                           |
| **address1**                                    | No                              | Customer's billing address. If provided, it should be between 1 and 50 characters long.                                        |
| **address2**                                    | No                              | Customer's address (line 2). If provided, it should be between 1 and 50 characters long.                                        |
| **postalCode**                                  | Yes                             | Customer's billing zip code. It should be a valid zip code between 1 and 16 characters long.                                  |
| **city**                                        | No                              | Customer's billing city. If provided, it should be between 1 and 50 characters long.                                           |
| **state**                                       | No                              | Customer's billing state. If provided, it should be between 1 and 3 characters long.                                 |
| **country**                                     | Yes                             | Customer's billing country. Should be a two-letter country code as defined in ISO 3166-1.                             |
| **email**                                       | Yes                             | Customer's email. Should be a well-formed email address, max 254 characters long.                      |
| **phoneNumber**                                 | No                              | Customer's phone number. If provided, it should be a well-formed phone number.                                         |
| **ipAddress**                                   | No                              | Customer's IP address.                                                                                              |
| **browserHttpUserAgent**                        | No                              | Browser User-Agent header value.                                                                                    |
| **browserHttpAccept**                           | No                              | Browser Accept header value.                                                                                           |  
| **browserHttpAcceptEncoding**                   | No                              | Browser Accept Encoding header value.                                                                                           |                   
| **browserHttpAcceptLanguate**                   | No                              | Browser Accept Language header value.                        |
| **cardNumber**                                  | Yes                             | A valid credit card number.                                                                                            |
| **expMonth**                                    | Yes                             | Credit card expiration month in mm format. Should be a value between 1 and 12. |
| **expYear**                                     | Yes                             | Credit card expiration year in yyyy format. Should be a value between current year and 2100.                           |
| **cvv2**                                        | Yes                             | Card security code. Should be a 3-4 digit value.               |
| **nameOnCard**                                  | Yes                             | Name displayed on the credit card. Should be between 2 and 45 characters long.                   |

#### 3. Generate CCBill OAuth Bearer Token

The CCBill RESTful API uses [OAuth-based](https://ccbill.com/kb/what-is-oauth) authentication and authorization. Use the **frontend credentials** (Base64 encoded **`Merchant Application ID`** and **`Secret Key`**) you received from Merchant Support to generate a **frontend bearer token**.

You must include this token in the Authorization header of API requests when creating payment tokens. Use the following examples and adjust the necessary parameters to obtain a **frontend bearer token:**

<details><summary>👉 cURL</summary>

```
curl -X POST 'https://api.ccbill.com/ccbill-auth/oauth/token' \
  -u '[Frontend_Merchant_Application_ID]:[Frontend_Secret_Key]' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials' 
```
</details>
<details><summary>👉 Java</summary>

```
String getOAuthToken() {
    String credentials = Base64.getEncoder()
        .encodeToString(("[Frontend_Merchant_Application_ID]" + ":" + "[Frontend_Secret_Key]")
        .getBytes(StandardCharsets.UTF_8));
    String requestBody = "grant_type=client_credentials";

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.ccbill.com/ccbill-auth/oauth/token"))
           .header("Authorization", "Basic " + credentials)
            .header("Content-Type", "application/x-www-form-urlencoded")
           .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
            .build();

    try {
        HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
        return extractAccessToken(response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
        return null;
    }
}
```
</details>
<details><summary>👉 PHP</summary>

```
<?php

function getOAuthToken() {
    $url = "https://api.ccbill.com/ccbill-auth/oauth/token";
    $merchantAppId = "[Frontend_Merchant_Application_ID]";
    $secretKey = "[Frontend_Secret_Key]";
    $data = http_build_query(["grant_type" => "client_credentials"]);

    try {
        $httpRequest = new HttpRequest();
        $httpRequest->setUrl($url);
        $httpRequest->setMethod(HTTP_METH_POST);
        $httpRequest->setHeaders([
            "Authorization" => "Basic " . base64_encode("$merchantAppId:$secretKey"),
            "Content-Type" => "application/x-www-form-urlencoded"
        ]);
        $httpRequest->setBody($data);

        $httpClient = new HttpClient();
        $response = $httpClient->send($httpRequest);
        
        $responseData = json_decode($response->getBody(), true);
        return $responseData['access_token'] ?? die("Error: Invalid OAuth response.");
    } catch (HttpException $ex) {
        die("Error fetching OAuth token: " . $ex->getMessage());
    }
}

?>
```
</details>

⚠️**Important Notes**

-   **Never expose API credentials on the front end.** Always store your Merchant Application ID and Secret Key securely in server-side environment variables.
-   **This request must be sent from your backend.** OAuth token requests cannot be made from a web browser for security reasons.
-   **OAuth access tokens are temporary.** Each token remains valid for a single request or until it expires.
-   **Reduce API token attack surface**. Execute calls to create an Oauth token and a payment token in quick succession to minimize the risk of the access token being exposed to attackers.
-   **Use CSRF tokens for your front-end** **payment forms**. Protect your front-end forms with CSRF tokens to prevent unauthorized form submissions.

#### 4. Check If SCA Is Required

The [isScaRequired()](https://ccbill.com/doc/method-reference#ftoc-heading-2) function determines whether strong customer authentication is required before generating a payment token. The system checks the provided **credit card number**, **merchant account number**, **subaccount**, and **currency code**.

💻**Code Example**

```
async function checkIfScaRequired() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const scaRequiredResponse = await widget.isScaRequired(
        "[Frondent_Access_Token]", 
        [Your_Client_Account_Number], 
        [Your_3DS_Client_Subaccount_Number]);
    return await scaRequiredResponse.json();
}
```
<details><summary>Alternatively Check If 3DS Is Required Based on Existing Token</summary><br>

Use [isScaRequiredForPaymentToken()](https://ccbill.com/doc/method-reference#ftoc-heading-3) to determine whether strong customer authentication (3DS) is required for a **pre-existing Payment Token**.

💻**Code Example**

```
async function checkIfScaRequired() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const scaRequiredResponse = await widget.isScaRequiredForPaymentToken(
        "[Frondent_Access_Token]", 
        "[payment_token_id]");
    return await scaRequiredResponse.json();
}

```
</details>

The function automatically checks the transaction parameters to determine if strong customer authentication (SCA) is required:

- A successful response returns a Boolean value that indicates whether 3DS is required for the transaction.
- If validation fails (e.g., invalid credentials), the response will show an error message to describe the issue.

#### 5. Authenticate and Create Payment Token in One Step

The [authenticateCustomerAndCreatePaymentToken()](https://ccbill.com/doc/method-reference#ftoc-heading-5) function combines 3DS authentication and payment token creation in a single call. This integration simplifies the workflow by:

-   Initiating Strong Customer Authentication (SCA) through the 3DS flow.
-   Generating a reusable Payment Token for the authenticated customer.
-   Returning an object containing both the 3DS authentication results and the Payment Token.

💻**Code Example**

```
async function authenticateCustomerAndCreatePaymentToken() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    return await widget.authenticateCustomerAndCreatePaymentToken(
    "[Frondent_Access_Token]",
    [Your_Client_Account_Number], 
    [Your_3DS_Client_Subaccount_Number]);
}
```
The function automatically handles 3DS authentication and Payment Token generation:

-   A successful authentication returns an object with two parts: the result of the 3DS authentication process and Payment Token details.
-   If the process fails, the response includes error details to help troubleshoot the issue.

#### 6. Charge Payment Token

After you receive a **payment token ID**, generate a new **backend bearer token** using your Base64 encoded backend credentials. Then, use both tokens to charge the customer's credit card.

💻**Code Examples**

<details><summary>👉 cURL</summary>
  
```
curl -X POST 'https://api.ccbill.com/transactions/payment-tokens/threeds/[payment_token_id]' \
  -H 'Accept: application/vnd.mcn.transaction-service.api.v.2+json' \
  -H 'Authorization: Bearer [Backend_Access_Token]' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
    "clientAccnum": [Your_Client_Account_Number],
    "clientSubacc": [Your_Client_Subaccount_Number],
    "initialPrice": 9.99,
    "initialPeriod": 30,
    "currencyCode": 978,
    "threedsEci": "05",
    "threedsStatus": "Y",
    "threedsSuccess": true,
    "threedsVersion": "2.2.0",
    "threedsAmount": 9.99,
    "threedsClientTransactionId": "id-wl9r6duc5zj",
    "threedsCurrency": "840",
    "threedsSdkTransId": "d535b6d1-19f9-11f0-92b9-0242ac110005",
    "threedsAcsTransId": "ca5f9649-b865-47ce-be6f-54422a0fce47",
    "threedsDsTransId": "e3693b86-8217-48c6-9628-2e8852dc60d4",
    "threedsAuthenticationType": "",
    "threedsAuthenticationValue": "Pes4aJnpT+1mjhUoBynC92iQbeg="
  }'
```

</details>
  
<details><summary>👉 Java</summary>

```
public ResponseEntity<String> processPurchase3ds() {
    String requestBody = """
        {
            "clientAccnum": [Your_Client_Account_Number],
            "clientSubacc": [Your_Client_Subaccount_Number],
            "initialPrice": 9.99,
            "initialPeriod": 30,
            "currencyCode": 978,
            "threedsEci": "05",
            "threedsStatus": "Y",
            "threedsSuccess": true,
            "threedsVersion": "2.2.0",
            "threedsAmount": 9.99,
            "threedsClientTransactionId": "id-wl9r6duc5zj",
            "threedsCurrency": "840",
            "threedsSdkTransId": "d535b6d1-19f9-11f0-92b9-0242ac110005",
            "threedsAcsTransId": "ca5f9649-b865-47ce-be6f-54422a0fce47",
            "threedsDsTransId": "e3693b86-8217-48c6-9628-2e8852dc60d4",
            "threedsAuthenticationType": "",
            "threedsAuthenticationValue": "Pes4aJnpT+1mjhUoBynC92iQbeg="
        }""";

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.ccbill.com/transactions/payment-tokens/threeds/[payment_token_id]"))
            .header("Accept", "application/vnd.mcn.transaction-service.api.v.2+json")
            .header("Authorization", "Bearer [Backend_Access_Token]")
            .header("Cache-Control", "no-cache")
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
            .build();

    try {
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        return ResponseEntity.ok(response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
        return ResponseEntity.status(500).body("Error processing payment");
    }
}

```

</details>
<details><summary>👉 PHP</summary>

```
<?php

function processPurchase3ds() {
    $url = "https://api.ccbill.com/transactions/payment-tokens/threeds/[payment_token_id]";
    $paymentData = json_encode([
        "clientAccnum" => [Your_Client_Account_Number],
        "clientSubacc" => [Your_Client_Subaccount_Number],
        "initialPrice" => 9.99,
        "initialPeriod" => 30,
        "threedsEci" => "05",
        "threedsStatus" => "Y",
        "threedsSuccess" => true,
        "threedsVersion" => "2.2.0",
        "threedsAmount" => 9.99,
        "threedsClientTransactionId" => "id-wl9r6duc5zj",
        "threedsCurrency" => "840",
        "threedsSdkTransId" => "d535b6d1-19f9-11f0-92b9-0242ac110005",
        "threedsAcsTransId" => "ca5f9649-b865-47ce-be6f-54422a0fce47",
        "threedsDsTransId" => "e3693b86-8217-48c6-9628-2e8852dc60d4",
        "threedsAuthenticationType" => "",
        "threedsAuthenticationValue" => "Pes4aJnpT+1mjhUoBynC92iQbeg="
    ]);

    try {
        $httpRequest = new HttpRequest();
        $httpRequest->setUrl($url);
        $httpRequest->setMethod(HTTP_METH_POST);
        $httpRequest->setHeaders([
            "Accept" => "application/vnd.mcn.transaction-service.api.v.2+json",
            "Authorization" => "Bearer [Backend_Access_Token]",
            "Cache-Control" => "no-cache",
            "Content-Type" => "application/json"
        ]);
        $httpRequest->setBody($paymentData);

        $httpClient = new HttpClient();
        $response = $httpClient->send($httpRequest);
        
        return $response->getBody();
    } catch (HttpException $ex) {
        die("Error charging payment token: " . $ex->getMessage();
    }
}

?>
```
</details>

The server returns a standard response with the transaction status:

-   A successful response includes the transaction details, such as the amount, transaction ID, and status.
-   A failed request includes an [error code](https://ccbill.com/doc/error-codes) and an explanation for the error (e.g., authentication failure).


#### Full Integration Example (Non-3DS)

To simplify the 3DS transaction flow, the example below shows how to authenticate a customer and create a Payment Token using the above steps. The example uses:

-   A JavaScript frontend to initialize the widget, collect payment and customer data, and trigger 3DS authentication.
-   A Java backend to generate a bearer token, receive a Payment Token request, and create a 3DS-ready Payment Token using the provided data.

All placeholder values should be replaced with the actual client account number, subaccount number, bearer token, and 3DS credentials.

<details><summary>🌐 JavaScript Frontend</summary>

```
async function fetchOAuthToken() {
    return (await (await fetch('https://your-website.com/api/auth-token')).json()).token;
}

async function checkIfScaRequired(widget, authToken, clientAccnum, clientSubacc) {
    const scaRequiredResponse = await widget.isScaRequired(authToken, clientAccnum, clientSubacc);
    return await scaRequiredResponse.json();
}

async function authenticateCustomerAndCreatePaymentToken(widget, authToken, clientAccnum, clientSubacc) {
    return await widget.authenticateCustomerAndCreatePaymentToken(authToken, clientAccnum, clientSubacc);
}

async function createPaymentToken(widget, authToken, clientAccnum, clientSubacc) {
    const paymentTokenResponse = await widget.createPaymentToken(
        authToken,
        clientAccnum,
        clientSubacc
    );
    return await paymentTokenResponse.json();
}

async function chargePaymentToken(paymentToken) {
    return await (await (fetch('https://your-website.com/api/purchase', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            paymentToken,
            amount: 9.99,
            currency: 840
        })
    }))).json();
}

async function chargePaymentToken3ds(paymentToken, threedsInformation) {
    return await (await (fetch('https://your-website.com/api/purchase-3ds', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            paymentToken,
            threedsInformation,
            amount: 9.99,
            currency: 840
        })
    }))).json();
}

async function authenticateAndPurchase() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const clientAccnum = [Your_Client_Account_Number];
    const clientSubacc = [Your_3DS_Client_Subaccount_Number];



    try {
        // retrieval of the auth token from merchant provided endpoint
        // this should be done as late in the submission process as possible to avoid potential exploit.
        const authToken = await fetchOAuthToken();



        let chargeCallResponse;
        
        // check if 3DS is required and process the 3DS flow with the client if necessary
        const scaRequired = await checkIfScaRequired(widget, authToken, clientAccnum, clientSubacc);
        if (scaRequired) {
            // go through 3DS flow and create payment token in a single API call.
            // The resulting object will hold both payment token and SCA results,
            // which should be submitted to merchant owned endpoint and charged 
            /// via /transactions/payment-tokens/threeds/{paymentTokenId}.
            const response = await authenticateCustomerAndCreatePaymentToken(widget, 
                authToken, clientAccnum, clientSubacc);
            // submit the payment token and 3DS information to the back-end endpoint implementing 
            // charging of the token
            chargeCallResponse = await chargePaymentToken3ds(response.paymentToken, response.threedsInformation);
        } else {
            // create the payment token to be submitted to the merchant owned endpoint
            // and charged via /transactions/payment-tokens/{paymentTokenId}.
            const paymentToken = await createPaymentToken(widget, authToken, clientAccnum, [Your_Client_Subaccount_Number]);
            // submit the payment token to be charged to an endpoint implementing backend charging of the token
            chargeCallResponse = await chargePaymentToken(paymentToken);
        }
        
        return Promise.resolve(chargeCallResponse);
    } catch (error) {
        // react to any errors that may occur during the process
        return Promise.reject({error});
    }
}

let result = await authenticateAndPurchase();
```
</details>
<details><summary>⚙️ Java Backend</summary>

```
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;
import java.time.Duration;
import java.util.Base64;
import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

@RestController
@RequestMapping("/api")
public class ApiController {

    private static final HttpClient HTTP_CLIENT = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .build();

    @PostMapping("/auth-token")
    public ResponseEntity<AuthTokenResponse> getAuthToken() {
        String accessToken = fetchOAuthToken("[Frontend_Merchant_Application_ID]", "[Frontend_Secret_Key]");
        if (accessToken != null) {
            return ResponseEntity.ok(new AuthTokenResponse(accessToken));
        } else {
            return ResponseEntity.status(500).body(new AuthTokenResponse(""));
        }
    }
    
    @PostMapping("/purchase")
    public ResponseEntity<String> processPurchase(@RequestBody PurchaseRequest purchaseRequest) {
        String requestBody = String.format(
                """
                {
                  "clientAccnum": %d,
                  "clientSubacc": %d,
                  "initialPrice": %.2f,
                  "initialPeriod": 30,
                  "currencyCode": %d
                }
                """,
                purchaseRequest.paymentToken().clientAccnum(),
                purchaseRequest.paymentToken().clientSubacc(),
                purchaseRequest.amount(),
                purchaseRequest.currency()
        );

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.ccbill.com/transactions/payment-tokens/" 
                    + purchaseRequest.paymentToken().paymentTokenId()))
                .header("Accept", "application/vnd.mcn.transaction-service.api.v.2+json")
                .header("Authorization", "Bearer " 
                    + fetchOAuthToken("[Backend_Merchant_Application_ID]", "[Backend_Secret_Key]"))
                .header("Cache-Control", "no-cache")
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
                .build();

        try {
            HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
            return ResponseEntity.ok(response.body());
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body("Error processing payment");
        }
    }

    @PostMapping("/purchase-3ds")
    public ResponseEntity<String> processPurchase3ds(@RequestBody PurchaseRequest3ds purchaseRequest3ds) {
        String requestBody = String.format(
                """
                {
                  "clientAccnum": %d,
                  "clientSubacc": %d,
                  "initialPrice": %.2f,
                  "initialPeriod": 10,
                  "currencyCode": "%s",
                  "threedsEci": "%s",
                  "threedsStatus": "%s",
                  "threedsSuccess": %b,
                  "threedsVersion": "%s",
                  "threedsAmount": %.2f,
                  "threedsClientTransactionId": "%s",
                  "threedsCurrency": "%s",
                  "threedsSdkTransId": "%s",
                  "threedsAcsTransId": "%s",
                  "threedsDsTransId": "%s",
                  "threedsAuthenticationType": "%s",
                  "threedsAuthenticationValue": "%s"
                }
                """,
                purchaseRequest3ds.paymentToken().clientAccnum(),
                purchaseRequest3ds.paymentToken().clientSubacc(),
                purchaseRequest3ds.amount(),
                purchaseRequest3ds.currency(),
                purchaseRequest3ds.threedsInformation().eci(),
                purchaseRequest3ds.threedsInformation().status(),
                purchaseRequest3ds.threedsInformation().success(),
                purchaseRequest3ds.threedsInformation().protocolVersion(),
                purchaseRequest3ds.threedsInformation().amount(),
                purchaseRequest3ds.threedsInformation().clientTransactionId(),
                purchaseRequest3ds.threedsInformation().currency(),
                purchaseRequest3ds.threedsInformation().sdkTransId(),
                purchaseRequest3ds.threedsInformation().acsTransId(),
                purchaseRequest3ds.threedsInformation().dsTransId(),
                purchaseRequest3ds.threedsInformation().authenticationType(),
                purchaseRequest3ds.threedsInformation().authenticationValue()
        );

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.ccbill.com/transactions/payment-tokens/threeds/"
                    + purchaseRequest.paymentToken().paymentTokenId()))
                .header("Accept", "application/vnd.mcn.transaction-service.api.v.2+json")
                .header("Authorization", "Bearer " 
                    + fetchOAuthToken("[Backend_Merchant_Application_ID]", "[Backend_Secret_Key]"))
                .header("Cache-Control", "no-cache")
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
                .build();

        try {
            HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
            return ResponseEntity.ok(response.body());
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body("Error processing payment");
        }
    }

    private static String fetchOAuthToken(String merchantAppId, String sercretKey) {
        String credentials = Base64.getEncoder()
            .encodeToString((merchantAppId + ":" + sercretKey).getBytes(StandardCharsets.UTF_8));
        String requestBody = "grant_type=client_credentials";

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.ccbill.com/ccbill-auth/oauth/token"))
                .header("Authorization", "Basic " + credentials)
                .header("Content-Type", "application/x-www-form-urlencoded")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
                .build();

        try {
            HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
            return extractAccessToken(response.body());
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
            return null;
        }
    }

    private static String extractAccessToken(String responseBody) {
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(responseBody);
            return jsonNode.has("access_token") ? jsonNode.get("access_token").asText() : null;
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    private record AuthTokenResponse(String token) {}
    private record PurchaseRequest(double amount, String currency, PaymentToken paymentToken) {}
    private record PurchaseRequest3ds(double amount, String currency, PaymentToken paymentToken, 
        ThreedsInformation threedsInformation) {}
    private record PaymentToken(String paymentTokenId, Integer clientAccnum, Integer clientSubacc) {}
    private record ThreedsInformation(String eci, String status, boolean success, String protocolVersion, 
        double amount, String clientTransactionId, String currency, String sdkTransId, String acsTransId, 
        String dsTransId, String authenticationType, String authenticationValue) {}
}
```
</details>
</details>

This flow combines 3DS authentication and payment token creation in a single step. It is similar to the Create Payment Token (3DS) flow and is ideal for real-time transactions when the customer is in session and the charge amount is known upfront. 

Use it to reduce the number of API calls your system needs to make.

<details><summary><strong>👉 Create Payment Token For Deferred Charges</strong></summary><br>

To perform a 3DS check based on customer payment details, create a payment token, and use it to charge a customer at a later time, follow these steps:

1.  Include the Widget on your page.
2.  Provide payment details.
3.  Generate the **frontend OAuth bearer token**.
4.  Check whether the 3DS authentication is required based on customer data (or a pre-existing payment token).
5.  Authenticate the customer and create  a **payment token**.
6.  Use the payment token and **backend OAuth bearer token** to process a transaction securely.

The diagram below shows the full flow:

<a href="https://github.com/user-attachments/assets/852d8560-551c-4768-9dc3-b8cee38a1d4c" target="_blank" rel="noopener"><img src="https://github.com/user-attachments/assets/852d8560-551c-4768-9dc3-b8cee38a1d4c" width="300"></a>

#### 1. Include the Widget in Your Page 

Add the following **preload link** and **script** elements to your HTML page:

```
<link rel="preload" href="https://js.ccbill.com/v1.13.1/ccbill-advanced-widget.js" as="script"/>

<script type="text/javascript" src="https://js.ccbill.com/v1.13.1/ccbill-advanced-widget.js"></script>
```
Pay special attention to the Widget version (**v1.13.1**) in the URI path, as the version number may be subject to change.

#### 2. Collect Customer and Payment Data

The Advanced Widget automatically extracts values from form fields. The required fields can be provided in three ways:

<details><summary>👉 (Recommended) Use <code>data-ccbill</code> HTML data attributes.</summary><br>

Using <code>data-ccbill</code> data attributes is non-intrusive and provides more flexibility. You can map form inputs directly without modifying existing <code>id</code> attributes.
```
<form id="payment-form"> 
    <input data-ccbill="firstName" />
    <input data-ccbill="lastName" /> 
    <input data-ccbill="postalCode" /> 
    <input data-ccbill="country" /> 
    <input data-ccbill="email" /> 
    <input data-ccbill="cardNumber" />
    <input data-ccbill="currencyCode" /> 
    <input data-ccbill="expYear" /> 
    <input data-ccbill="expMonth" /> 
    <input data-ccbill="nameOnCard" /> 
    <input data-ccbill="cvv2" /> 
</form>
```

</details>

<details><summary>👉 Use default <code>_ccbillId_FieldName</code> ID attributes.</summary><br>

If you cannot modify your HTML to include <code>data-ccbill</code> attributes, use the default <code>_ccbillId_</code> attributes instead. The field names must match CCBill's predefined format.
```
<form id="payment-form">
    <input id="_ccbillId_firstName" />
    <input id="_ccbillId_lastName" />
    <input id="_ccbillId_postalCode" />
    <input id="_ccbillId_country" />
    <input id="_ccbillId_email" />
    <input id="_ccbillId_cardNumber" />
    <input id="_ccbillId_expYear" />
    <input id="_ccbillId_currencyCode" /> 
    <input id="_ccbillId_expMonth" />
    <input id="_ccbillId_nameOnCard" />
    <input id="_ccbillId_cvv2" />
</form>
```
</details>

<details><summary>👉 Use custom ID attributes (requires additional mapping).</summary><br>

Map custom IDs to corresponding input fields using the <code>customIds</code> parameter in the Widget <code>constructor</code>.
```
<form id="payment-form">
    <input id="custom_firstName_id" />
    <input id="custom_lastName_id" />
    <input id="custom_postalCode_id" /> 
    <input id="custom_country_id" /> 
    <input id="custom_email_id" /> 
    <input id="custom_cardNumber_id" />
    <input id="custom_currencyCode_id" /> 
    <input id="custom_expYear_id" /> 
    <input id="custom_expMonth_id" /> 
    <input id="custom_nameOnCard_id" /> 
    <input id="custom_cvv2_id" /> 
</form>
<script>
// map custom ids to relevant fields
const customIds = {
    firstName: "custom_firstName_id",
    lastName: "custom_lastName_id",
    postalCode: "custom_postalCode_id",
    country: "custom_country_id",
    email: "custom_email_id",
    currencyCode: "custom_currencyCode_id",
    cardNumber: "custom_cardNumber_id",
    expYear: "custom_expYear_id", 
    expMonth: "custom_expMonth_id", 
    nameOnCard: "custom_nameOnCard_id",
    cvv2: "custom_cvv2_id"
};

// pass custom ids to Widget constructor
const widget = new ccbill.CCBillAdvancedWidget("application_id", customIds);

// call the desired Widget method

</script>
```
</details>

##### All Supported Form Fields

| **Name**                                        | **Required**                     | **Description**                                                         |
|-------------------------------------------------|---------------------------------|--------------------------------------------------------------------------|
| **currencyCode**                                | Yes                             | A three-digit currency code ([ISO 4217 standard](https://www.iso.org/obp/ui/#search/code/)) for the currency used in the transaction. |
| **firstName**                                   | Yes                             | Customer's first name.                                                  |
| **lastName**                                    | Yes                             | Customer's last name.                                           |
| **address1**                                    | No                              | Customer's billing address. If provided, it should be between 1 and 50 characters long.                                        |
| **address2**                                    | No                              | Customer's address (line 2). If provided, it should be between 1 and 50 characters long.                                        |
| **address3**                                    | No                              | Customer's address (line 3). If provided, it should be between 1 and 50 characters long.
| **postalCode**                                  | Yes                             | Customer's billing zip code. It should be a valid zip code between 1 and 16 characters long.                                  |
| **city**                                        | No                              | Customer's billing city. If provided, it should be between 1 and 50 characters long.                                           |
| **state**                                       | No                              | Customer's billing state. If provided, it should be between 1 and 3 characters long.                                 |
| **country**                                     | Yes                             | Customer's billing country. Should be a two-letter country code as defined in ISO 3166-1.                             |
| **email**                                       | Yes                             | Customer's email. Should be a well-formed email address, max 254 characters long.                      |
| **phoneNumber**                                 | No                              | Customer's phone number. If provided, it should be a well-formed phone number.                                         |
| **ipAddress**                                   | No                              | Customer's IP address.                                                                                              |
| **browserHttpUserAgent**                        | No                              | Browser User-Agent header value.                                                                                    |
| **browserHttpAccept**                           | No                              | Browser Accept header value.                                                                                           |  
| **browserHttpAcceptEncoding**                   | No                              | Browser Accept Encoding header value.                                                                                           |                   
| **browserHttpAcceptLanguate**                   | No                              | Browser Accept Language header value.                        |
| **cardNumber**                                  | Yes                             | A valid credit card number.                                                                                            |
| **expMonth**                                    | Yes                             | Credit card expiration month in mm format. Should be a value between 1 and 12. |
| **expYear**                                     | Yes                             | Credit card expiration year in yyyy format. Should be a value between current year and 2100.                           |
| **cvv2**                                        | Yes                             | Card security code. Should be a 3-4 digit value.               |
| **nameOnCard**                                  | Yes                             | Name displayed on the credit card. Should be between 2 and 45 characters long.                   |

#### 3. Generate CCBill OAuth Bearer Token

The CCBill RESTful API uses [OAuth-based](https://ccbill.com/kb/what-is-oauth) authentication and authorization. Use the **frontend credentials** (Base64 encoded **`Merchant Application ID`** and **`Secret Key`**) you received from Merchant Support to generate a **frontend bearer token**.

Include this token in the Authorization header of API requests when creating payment tokens. Use the following examples and adjust the necessary parameters to obtain a **frontend bearer token:**

<details><summary>👉 cURL</summary>

```
curl -X POST 'https://api.ccbill.com/ccbill-auth/oauth/token' \
  -u '[Frontend_Merchant_Application_ID]:[Frontend_Secret_Key]' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials'
  
```
</details>
<details><summary>👉 Java</summary>

```

String getOAuthToken() {
    String credentials = Base64.getEncoder()
        .encodeToString(("[Frontend_Merchant_Application_ID]" + ":" + "[Frontend_Secret_Key]")
        .getBytes(StandardCharsets.UTF_8));
    String requestBody = "grant_type=client_credentials";

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.ccbill.com/ccbill-auth/oauth/token"))
           .header("Authorization", "Basic " + credentials)
            .header("Content-Type", "application/x-www-form-urlencoded")
           .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
            .build();

    try {
        HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
        return extractAccessToken(response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
        return null;
    }
}
```
</details>
<details><summary>👉 PHP</summary>

```
<?php

function getOAuthToken() {
    $url = "https://api.ccbill.com/ccbill-auth/oauth/token";
    $merchantAppId = "[Frontend_Merchant_Application_ID]";
    $secretKey = "[Frontend_Secret_Key]";
    $data = http_build_query(["grant_type" => "client_credentials"]);

    try {
        $httpRequest = new HttpRequest();
        $httpRequest->setUrl($url);
        $httpRequest->setMethod(HTTP_METH_POST);
        $httpRequest->setHeaders([
            "Authorization" => "Basic " . base64_encode("$merchantAppId:$secretKey"),
            "Content-Type" => "application/x-www-form-urlencoded"
        ]);
        $httpRequest->setBody($data);

        $httpClient = new HttpClient();
        $response = $httpClient->send($httpRequest);
        
        $responseData = json_decode($response->getBody(), true);
        return $responseData['access_token'] ?? die("Error: Invalid OAuth response.");
    } catch (HttpException $ex) {
        die("Error fetching OAuth token: " . $ex->getMessage());
    }
}

?>

```
</details>

⚠️**Important Notes**

-   **Never expose API credentials on the front end.** Always store your Merchant Application ID and Secret Key securely in server-side environment variables.
-   **This request must be sent from your backend.** OAuth token requests cannot be made from a web browser for security reasons.
-   **OAuth access tokens are temporary.** Each token remains valid for a single request or until it expires.
-   **Reduce API token attack surface**. Execute calls to create an OAuth token and a payment token in quick succession to minimize the risk of the access token being exposed to attackers.
-   **Use CSRF tokens for your front-end** **payment forms**. Protect your front-end forms with CSRF tokens to prevent unauthorized form submissions.

#### 4. Check If SCA Is Required

The [isScaRequired()](https://ccbill.com/doc/method-reference#ftoc-heading-2) function determines whether strong customer authentication is required before generating a payment token. The system checks the provided **credit card number**, **merchant account number**, **subaccount**, and **currency code**.

💻**Code Example**

```
async function checkIfScaRequired() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const scaRequiredResponse = await widget.isScaRequired(
        "[Frondent_Access_Token]", 
        [Your_Client_Account_Number], 
        [Your_3DS_Client_Subaccount_Number]);
    return await scaRequiredResponse.json();
}
```
<details><summary>Alternatively Check If 3DS Is Required Based on Existing Token</summary><br>

The [isScaRequiredForPaymentToken()](https://ccbill.com/doc/method-reference#ftoc-heading-3) function determines whether strong customer authentication (3DS) is required for a **pre-existing Payment Token**.

💻**Code Example**

```
async function checkIfScaRequired() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const scaRequiredResponse = await widget.isScaRequiredForPaymentToken(
        "[Frondent_Access_Token]", 
        "[payment_token_id]");
    return await scaRequiredResponse.json();
}

```
</details>

The function automatically checks the transaction parameters to determine if strong customer authentication (SCA) is required:

-   A successful response returns a Boolean value that indicates whether 3DS is required for the transaction.
-   If validation fails (e.g., invalid credentials), the response will show an error message to describe the issue.

#### 5. Create a Payment Token With 3DS Authentication For Deferred Charges

Use the [createPaymentToken3DS()](https://ccbill.com/doc/method-reference#ftoc-heading-6) method to authenticate the customer once, create a payment token, and charge them later without triggering another 3DS flow.

💻**Code Example**

```
async function createPaymentToken3ds(widget, authToken, clientAccnum, clientSubacc) {
    return await widget.createPaymentToken3DS(authToken, clientAccnum, clientSubacc);
}
```

The function returns a response with the outcome of the 3DS authentication and token creation process:

-   If the request is successful, the function returns an object with the Payment Token and additional metadata.
-   If the request fails, the response contains an [error code](https://ccbill.com/doc/error-codes) and a message explaining the issue (e.g., invalid credentials or authentication failure).

#### 6. Charge Payment Token

Use the Payment Token ID and backend bearer token to charge a customer's credit card through a 3DS-secured payment flow. Generate a new **backend bearer token** using your Base64 encoded backend credentials.

💻**Code Examples**
<details><summary>👉 cURL</summary>
  
```
curl -X POST 'https://api.ccbill.com/transactions/payment-tokens/[payment_token_id]' \
  -H 'Accept: application/vnd.mcn.transaction-service.api.v.2+json' \
  -H 'Authorization: Bearer [Backend_Access_Token]' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
    "clientAccnum": [Your_Client_Account_Number],
    "clientSubacc": [Your_Client_Subaccount_Number],
    "initialPrice": 9.99,
    "initialPeriod": 30,
    "currencyCode": 840
  }'
```

</details>
  
<details><summary>👉 Java</summary>

```
public ResponseEntity<String> processPurchase() {
    String requestBody = """
        {
            "clientAccnum": [Your_Client_Account_Number],
            "clientSubacc": [Your_Client_Subaccount_Number],
            "initialPrice": 9.99,
            "initialPeriod": 30,
            "currencyCode": 840
        }""";

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.ccbill.com/transactions/payment-tokens/[payment_token_id]"))
            .header("Accept", "application/vnd.mcn.transaction-service.api.v.2+json")
            .header("Authorization", "Bearer [Backend_Access_Token]")
            .header("Cache-Control", "no-cache")
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
            .build();

    try {
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        return ResponseEntity.ok(response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
        return ResponseEntity.status(500).body("Error processing payment");
    }
}
```

</details>
<details><summary>👉 PHP</summary>

```
<?php

function processPurchase() {
    $url = "https://api.ccbill.com/transactions/payment-tokens/[payment_token_id]";
    $paymentData = json_encode([
        "clientAccnum" => [Your_Client_Account_Number],
        "clientSubacc" => [Your_Client_Subaccount_Number],
        "initialPrice" => 9.99,
        "initialPeriod" => 30,
        "currencyCode" => 840,
    ]);

    try {
        $httpRequest = new HttpRequest();
        $httpRequest->setUrl($url);
        $httpRequest->setMethod(HTTP_METH_POST);
        $httpRequest->setHeaders([
            "Accept" => "application/vnd.mcn.transaction-service.api.v.2+json",
            "Authorization" => "Bearer [Backend_Access_Token]",
            "Cache-Control" => "no-cache",
            "Content-Type" => "application/json"
        ]);
        $httpRequest->setBody($paymentData);

        $httpClient = new HttpClient();
        $response = $httpClient->send($httpRequest);
        
        return $response->getBody();
    } catch (HttpException $ex) {
        die("Error charging payment token: " . $ex->getMessage();
    }
}

?>
```
</details>

The API endpoint handles the transaction:

-   A successful charge returns a response with transaction details.

-   If the charge fails, the response includes an [error code](https://ccbill.com/doc/error-codes) and a descriptive message


#### Full Integration Example (Non-3DS)

Use the following example to implement a 3DS-authenticated tokenization flow that authenticates the cardholder up front and charges the stored token later.

The example has:

-   A JavaScript frontend that initializes the widget, collects payment data, and triggers the 3DS authentication flow to create a Payment Token.
-   A Java backend that generates the bearer token, receives the Payment Token from the frontend, and securely stores it for future use in deferred billing scenarios.

Replace all placeholder values with actual data in your integration.

<details><summary>🌐 JavaScript Frontend</summary>

```
async function fetchOAuthToken() {
    return (await (await fetch('https://your-website.com/api/auth-token')).json()).token;
}

async function createPaymentToken(widget, authToken, clientAccnum, clientSubacc) {
    const paymentTokenResponse = await widget.createPaymentToken(
        authToken,
        clientAccnum,
        clientSubacc
    );
    return await paymentTokenResponse.json();
}

async function createPaymentToken3ds(widget, authToken, clientAccnum, clientSubacc) {
    return await widget.createPaymentToken3DS(authToken, clientAccnum, clientSubacc);
}

async function chargePaymentToken(paymentToken) {
    return await (await (fetch('https://your-website.com/api/purchase', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            paymentToken,
            amount: 9.99,
            currency: 840
        })
    }))).json();
}
async function authenticateAndPurchaseLater() {
    const widget = new ccbill.CCBillAdvancedWidget('your-application-id');
    const clientAccnum = [Your_Client_Account_Number];
    const clientSubacc = [Your_Client_Subaccount_Number];

    try {
        // retrieval of the auth token from merchant provided endpoint
        // this should be done as late in the submission process as possible to avoid potential exploit.
        const authToken = await fetchOAuthToken();

        let paymentToken;

        // check if 3DS is required and process the 3DS flow with the client if necessary
        const scaRequired = await checkIfScaRequired(widget, authToken, clientAccnum, clientSubacc);
        if (scaRequired) {
            // go through 3DS flow and create payment token in a single API call.
            // The resulting 3DS payment token should be submitted to the merchant owned endpoint
            // and can be charged at some point in future via /transactions/payment-tokens/{paymentTokenId}.
            paymentToken = await createPaymentToken3ds(widget, authToken, clientAccnum, clientSubacc);
        } else {
            // create the non-3DS payment token to be submitted to the merchant owned endpoint
            // and charged via /transactions/payment-tokens/{paymentTokenId}.
            paymentToken = await createPaymentToken(widget, authToken, clientAccnum, clientSubacc);
        }

        // submit the payment token to be charged to an endpoint implementing backend charging of the token
        const chargeCallResponse = await chargePaymentToken(paymentToken);
        return Promise.resolve(chargeCallResponse);
    } catch (error) {
        // react to any errors that may occur during the process
        return Promise.reject({ error });
    }
}

let result = await authenticateAndPurchaseLater();
```
</details>
<details><summary>⚙️ Java Backend</summary>

```
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;
import java.time.Duration;
import java.util.Base64;
import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

@RestController
@RequestMapping("/api")
public class ApiController {

    private static final HttpClient HTTP_CLIENT = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .build();

    @PostMapping("/auth-token")
    public ResponseEntity<AuthTokenResponse> getAuthToken() {
        String accessToken = fetchOAuthToken("[Frontend_Merchant_Application_ID]", "[Frontend_Secret_Key]");
        if (accessToken != null) {
            return ResponseEntity.ok(new AuthTokenResponse(accessToken));
        } else {
            return ResponseEntity.status(500).body(new AuthTokenResponse(""));
        }
    }

    @PostMapping("/purchase")
    public ResponseEntity<String> processPurchase(@RequestBody PurchaseRequest purchaseRequest) {
        String requestBody = String.format(
                """
                {
                  "clientAccnum": %d,
                  "clientSubacc": %d,
                  "initialPrice": %.2f,
                  "initialPeriod": 30,
                  "currencyCode": %d
                }
                """,
                purchaseRequest.paymentToken().clientAccnum(),
                purchaseRequest.paymentToken().clientSubacc(),
                purchaseRequest.amount(),
                purchaseRequest.currency()
        );

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.ccbill.com/transactions/payment-tokens/" 
                    + purchaseRequest.paymentToken().paymentTokenId()))
                .header("Accept", "application/vnd.mcn.transaction-service.api.v.2+json")
                .header("Authorization", "Bearer " 
                    + fetchOAuthToken("[Backend_Merchant_Application_ID]", "[Backend_Secret_Key]"))
                .header("Cache-Control", "no-cache")
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
                .build();

        try {
            HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
            return ResponseEntity.ok(response.body());
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body("Error processing payment");
        }
    }

    private static String fetchOAuthToken(String merchantAppId, String sercretKey) {
        String credentials = Base64.getEncoder()
            .encodeToString((merchantAppId + ":" + sercretKey).getBytes(StandardCharsets.UTF_8));
        String requestBody = "grant_type=client_credentials";

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.ccbill.com/ccbill-auth/oauth/token"))
                .header("Authorization", "Basic " + credentials)
                .header("Content-Type", "application/x-www-form-urlencoded")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody, StandardCharsets.UTF_8))
                .build();

        try {
            HttpResponse<String> response = HTTP_CLIENT.send(request, HttpResponse.BodyHandlers.ofString());
            return extractAccessToken(response.body());
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
            return null;
        }
    }

    private static String extractAccessToken(String responseBody) {
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(responseBody);
            return jsonNode.has("access_token") ? jsonNode.get("access_token").asText() : null;
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    private record AuthTokenResponse(String token) {}
    private record PurchaseRequest(double amount, String currency, PaymentToken paymentToken) {}
    private record PaymentToken(String paymentTokenId, Integer clientAccnum, Integer clientSubacc) {}
}
```
</details>
</details>

This method authenticates the cardholder and creates a payment token. The customer does not have to be in session when the token is later charged.

For example, you can generate a token when customers register on a website (i.e., for a free trial) and use it later to charge them for recurring subscriptions or one-time purchases.

## Resources

* [CCBill Advanced Widget API Reference](https://ccbill.com/doc/method-reference)
* [CCBill RESTful API Resources](https://ccbill.com/doc/ccbill-restful-api-resources)
* [API Documentation](https://ccbill.com/doc/ccbill-restful-transaction-api)
* [Webhooks User Guide](https://ccbill.com/doc/webhooks-user-guide)
* [Error Codes](https://ccbill.com/doc/error-codes)

## CCBill Community

Become part of the CCBill community to get updates on the new features, help us improve the platform, and engage with other merchants and developers.

* Follow [@CCBillBIZ on Twitter](https://twitter.com/CCBillBIZ).
* Visit [CCBill's Blog](https://ccbill.com/blog) and read about latest developments in payment processing.

## Contact CCBill

Get in touch with us if you have questions or need help with the CCBill RESTful API.

<p align="left">
  <a href="https://twitter.com/CCBillBIZ">Twitter</a> •
  <a href="https://www.facebook.com/ccbillBIZ/">Facebook</a> •
  <a href="https://www.linkedin.com/company/ccbill">LinkedIn</a> •
  <a href="https://www.instagram.com/ccbillbiz/">Instagram</a> •
  <a href="https://www.youtube.com/c/CCBillBiz/featured">YouTube</a> •
  <a href="https://ccbill.com/contact">Support</a>
</p>

