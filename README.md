<h1 align="center">
  <br>
  <a href="https://kb.ccbill.com/CCBill+API"><img src="https://user-images.githubusercontent.com/81383705/118308832-b791cf80-b4ec-11eb-95a9-14a9363c56e4.png" alt="ccbill-restful-api" width="300"></a>
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

## CCBill Community

Become part of the CCBill community to get updates on new features, help us improve the platform, and engage with other merchants and developers.

* Follow [@CCBillBIZ on Twitter](https://twitter.com/CCBillBIZ).
* Visit [CCBill's Blog](https://ccbill.com/blog) and read about latest developments in payment processing.

### Resources

* [API Documentation](https://kb.ccbill.com/CCBill+API)
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
