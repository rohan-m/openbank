# [![apigee.com](http://apigee.com/about/sites/all/themes/apigee_themes/apigee_mktg/images/logo.png)](http://apigee.com) Open bank

The Apigee Open Banking APIx solution simplifies and accelerates the process of delivering open banking by providing secure, ready-to-use APIs along with the computing infrastructure to support internal and external innovation.


This document is organized into the following sections

[Overview](#overview)

[Design](#design)
- [Architecture](#architecture)
- [Security](#security)
- [APIs](#functional-apis)  

[Setup](#setup)
- [Getting Started](#getting-started)
- [Installation](#installation)
- [Test](#Test)

[Developer Portal](#developer-portal)

- [Developer Portal Setup](#developer-portal-setup) 

[Data](#data)

[Notes for Implementors](#notes-for-implementors)

- [Client App Developers](#client-app-developers)
- [API Deployment](#api-deployment)

[Changelog](#changelog)

## Overview

The OpenBank solution is built on Apigee Edge API Management Platform, and features the following APIs:

**Account Access APIs**
  - Account Request 
  - Account Information 
  - Transactions
  - Balances
  - Direct Debits
  - Standing Orders
  - Products
  
**Payment Initiation APIs**
  - Single Immediate Payment
  - Payment Submission
  
**Security APIs**
  - OAuth
  - UserInfo

![enter image description
here](http://openbank.apigee.com/sites/default/files/openbank_architecture.png)

The Account Access APIs follow the Open Banking [Account and Transaction API Specification - v1.0.0](https://www.openbanking.org.uk/read-write-apis/account-transaction-api/v1-0-0/) and Payment Initiation APIs follow [Payment Initiation API Specification - v1.0.0](https://www.openbanking.org.uk/read-write-apis/payment-initiation-api/v1-0-0/)

It also provides an implementation of [Open Banking Security Profile v1.0.0](https://www.openbanking.org.uk/read-write-apis/security-profile/id1-0-1/). It is  **OpenID** and **oAuth** based authentication, having **consent** and **two-factor authentication** using phone number and PIN based authentication using external SMS API.

### Repository Overview

This repository contains the necessary artifacts that will allow one to pull up
a complete set of **Banking APIs** that comply with CMA 1.0 specification (that in turn comply with _Openbanking_ and _PSD2_ )
regulations. In addition this will also allow one to build a _sandbox_ complete
with a **Developer Portal**, dummy backend and a sample app.

## Design

The APIs provided are configurable to connect to your Core Banking systems and 
for the use of developer applications, and to obtain end user consent. The following sections will help you
understand this solution so that you can go about this on your own.


### Architecture

![API Architecture](http://imageshack.com/a/img922/3760/tCOiYq.png)

The Banking APIs are designed such that each API is chained together as Northbound <-> Southbound APIs.

As shown in the architecture diagram above, the message flow is as follows: 
End user Application <-> _Northbound APIs_ <-> _Southbound APIs_ <-> Backend Systems (Core Banking / Payment etc)


The Northbound API provides a fixed set of interfaces, complying with respective Open Banking specification, that can be used by
the external consumers. In order to minimize changes to the contract, and there by to applications that connect to it, this API may not be required to change as often.


The Southbound API connects to the actual backend of the bank. We have used a Sandbox
backend as a default as part of this solution build out. Therefore you may want to modify the southbound interfaces to suite your specific backend needs. 

All **Southbound APIs** end with the suffix _'-connector'_

In addition, there are some internal APIs which are not exposed outside, but
which are used internally from the other APIs and provide common services like
sending out SMS.


Each API deployed in Apigee Edge is encapsulated withing a unit of deployment called a [Proxy](http://docs.apigee.com/api-services/content/understanding-apis-and-api-proxies).
To Learn more on the basic concepts of how to manage these within Apigee Edge, please refer to : 
http://docs.apigee.com/api-services/content/what-apigee-edge

Each of the following entities in the sequence diagram below, such as oAuth, consent-app , session,authentication-connector are examples of proxies.  

There are two broad sets of proxy definitions in the solution. One set, **Security**,  helps manage the security around the APIs while the other is the set of **Functional APIs** that a bank would like to expose. For example: accounts, payments, etc.


### Security 
In this solution, access to all APIs are protected via a security mechanism, that requires explicit end user authentication and authorization to make a successful API call. We have broadly used the OAuth 2.0 framework to secure these APIs, with integrated consent management application to manage end user authorization. Therefore, one has to obtain a valid **Access Token** or a one time Token via the Security APIs before making a call to the Functional APIs.
In order to enable **two-factor authentication** for the functional APIs, OTP(one time password) mechanism via SMS has been enabled for each of the APIs, which can be disabled on demand. The sequence diagram below gives an overview on the call flow between various security related proxies to fetch a valid Access Token with two-factor authentication enabled.
#### OAuth API Flow

![OAuth API
Interaction](./images/oauth-consent-flow.png)

#### Login App

The Login app provides a basic user authentication functionality. It uses User Management API to verify the user credentials.  The User Management APIs connect to a Sandbox user store. The banks might want to replace the Login App with their corporate login page

#### Consent App

The consent app uses login app for the user securely authenticate with the bank. The consent app is a trusted app of the bank which will allow the user to login and subsequently provide consent information.

The consent app will talk to the following APIs in order to
fulfill its functionality : Consent API, SMS API, Accounts-connector API, Payment-connector API, Authentication-connector API and Customer Management API.
For more details on each of these APIs, refer to the README.md of the respective proxy which exposes these APIs.

### Functional APIs

The Functional APIs deployed and available as part of OpenBank solution are broadly classified as follow:

#### **1. Accounts Information APIs** 
Account Information APIs provide account information for accounts held by the consumer. Information is categorized into:

 - Request (for Account Information)
 - Information
 - Balance
 - Transactions 
 - Direct debits
 - Standing Orders
 - Products

An API end point is provided for each type of information.
These Account Information APIs are secured with **oAuth 2.0**  and need a **valid Access token** for making API calls. 


#### **2. Payment Initiation APIs**
Payment Initiation APIs provide a simple one time payment functionality. It has two sets of APIs:
 - Payment request  - (CREATE, GET and DELETE) to request for a one time payment  
 - Payment submission  - (CREATE, GET) to submit payment request for actual payment.

The Payment Initiation APIs are also secured with **oAuth 2.0** and need a **valid Access token**. 

Banking APIs provide developers with the information needed to create innovative Fintech apps for consumers.There are a few obvious use cases worth mentioning:

 - Aggregation of financial metrics such as net worth and savings across multiple accounts.
 - Analysis and recommendations for better money management.
 - Recommendation of products and deals based on monthly statements.


## Apigee Edge Setup

### Getting Started

+   Create an [Apigee API Management Developer Account](https://enterprise.apigee.com)
+   Create an [Apigee BaaS Account](https://apibaas.apigee.com)
+   Request For [Apigee Developer Portal](https://goo.gl/j8Vbew), if you want to use portal

To Learn more on the basic concepts of Apigee Edge, please refer to : 
http://docs.apigee.com/api-services/content/what-apigee-edge


To deploy the APIs and its dependencies on your own org please run the following
script from the root folder of the cloned repo.

### Installation 

#### Pre-requisites
+ node.js 
+ npm

#### Instructions

Install gulp 
```
npm install --global gulp-cli
```

Pull node modules
```
npm install
```

Run the deploy command
```
gulp deploy 
```

This will interactively prompt you for following details, and will then create / deploy all relevants bundles and artifacts and will provision the **OpenBank Sandbox** on your own Org.

+ Edge Org name
+ Edge Username
+ Edge Password
+ Edge Env for deployment
+ BaaS Org Name
+ BaaS App Name
+ BaaS Org Client Id
+ BaaS Org Client Secret 
+ Consent Session Key for signing the Session Cookie
+ Login App Key for signing the user details 


### Test

Once the deploy script is complete, run the following command to do a basic sanity test that the APIs are working

change to test folder
```
cd test
```

install node modules
```
npm install
```

run tests
```
gulp test
```


## Developer Portal
Every API provider must be able to educate developers and successfully expose their APIs. A developer portal is the face of your API program, providing everything that internal, partner, and third party developers need. 

Developers need to interact with the Banks and with each other. Enable your developer community to provide feedback, make support and feature requests, and submit their own content that can be accessed by other developers with the right developer portal.

Apigee Edge provides with a Developer Services portal that you can use to build and launch your own customized website to provide all of these services to your development community. One has the option to create their own developer portal, either in the cloud or on-premises.

The below picture depicts how a dev portal looks like

![developer-portal](images/openbank.png)

### Developer Portal Setup
The detailed instructions for developer portal setup for openbank solution can be found [Here](./src/devportal/README.md).

## Data
The dummy Backend system is created by the deploy script for this OpenBank solution and is hosted on [Baas 2.0](http://apibaas.apigee.com/) in your org.  You can find the dummy data under `./setup/data` folder

## Notes for Implementors

### Client App Developers

- The APIs use Public/Private Key pair for doing JWS signing of the Payload. The Public Key of the sample bank and Private Key for the sample TPP (Client App) are present in `./test` folder.

### API Deployment

- You can find two sets of Public/Private Key Pair under `./test` folder; you could use it for configuring the APIs to use them for signing/verifying the responses/requests.
- Private key for the bank has to be provided during deployment. It is recommended to define a Prompt in config.yml and use it as value for the private key.
- For Production access, a Mutual TSL connectivity needs to be configured as defined [here](http://docs.apigee.com/api-services/content/creating-virtual-host).
- While running `gulp deploy` please do make sure there are no custom APIs defined with the same names; otherwise those APIs will be overwritten with a new revision.   


## Changelog

#### 2017/09/13
* APIs / API Spec
    * Payment Initiation APIs added

#### 2017/08/24
* APIs / API Spec
    * Account APIs are made compliant with OpenBanking v1.0.0
    * Support for Open Bank Security Profile v1.0.0 - hybrid flow


#### 2017/06/20

* APIs / API Spec
    * APIs are made compliant with CMA 0.1 spec 
    * Support for CMA 0.1 consent model for Accounts and Payments APIs


#### 2017/02/20

* APIs / API Spec
    * Refined Products API and added to Developer Portal Smartdocs
    * Updated OpenAPI spec for Products API

* Installation
    * New node-based deployment script
    * New script for easier portal setup
    
* Documentation
    * New video overview of the OpenBank APIs and Installation - http://www.youtube.com/watch?v=8eecvL75B5w 
    * Updated API documenation
    * Updated installation instructions for APIs and Developer Portal


#### 2016/11/03

*   Added Products API
*   Changed URL of OpenData APIs (/atms and /branches)
    *   /apis/v1/opendata/locations/atms -> /apis/v1/locations/atms
    *   /apis/v1/opendata/locations/branches -> /apis/v1/locations/branches
