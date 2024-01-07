## OAuth2 Grant Types

- OAuth2 grant types

- An authentication service that checks a user’s credentials and issues a token back to the user. The token can, in turn,
  be presented every time the user wants to call a service protected by the OAuth2 server.

- OAuth2 is a flexible authorization framework that provides multiple mechanisms for applications to authenticate and
  authorize users without forcing them to share credentials.

- These authentication mechanisms are called authentication grants. OAuth2 has four forms of authentication grants that
  client applications can use to authenticate users, receive an access token, and then validate that token.
  These grants are:

  - Password
  - Client credential
  - Authorization code
  - Implicit

- EagleEye is our example application name that has 2 services
  - Organization service
  - Licensing service

### Overview

- 1. OAuth2 Password grant
- 2. OAuth2 Client credentials grant
- 3. OAuth2 Authorization code grant
- 4. OAuth2 Implicit credentials grant
- 5. OAuth2 Token Refreshing

### 01. Password grants

An OAuth2 password grant is probably the most straightforward grant type to understand. This grant type is used when both the application and the services explicitly trust one another.

- 1. Application owner registers application name with OAuth2 service, which provides a secret key
- 2. User logs into EagleEye, which passes user credentials with application name and key to OAuth2 service
- 3. OAuth2 authenticates user and application and provides access token
- 4. EagleEye attaches access token to any service calls from user
- 5. Protected services call OAuth2 to validate access token

### 02. Client credentials grant

The client credentials grant is typically used when an application needs to access an OAuth2 protected resource, but no human
being is involved in the transaction.

With the client credentials grant type, the OAuth2 server only authenticates based on application name and the secret key provided by the owner of the resource. Again, the client credential task is usually used when both applications are owned by the same
company.

- 1. Application owner registers data analytics job with OAuth2
- 2. When the data analytics job runs, EagleEye passes application name and key to OAuth2
- 3. OAuth2 authenticates application and provides access token
- 4. EagleEye attaches access token to any service calls

The difference between the password grant and the client credential grant is that a client credential grant authenticates by only using the registered application name and the secret key.

The client credential grant is for “no-user-involved” application authentication and
authorization.

### 03. Authorization code grant

The authorization code grant is by far the most complicated of the OAuth2 grants, but it’s also the most common flow used because it allows different applications from different vendors to share data and services without having to expose a user’s credentials across multiple applications. It also enforces an extra layer of checking by not letting a calling application immediately get an OAuth2 access token, but rather a “pre-access” authorization code.

- 1. EagleEye user registers Salesforce application with OAuth2, obtains secret key and a callback URL to return users
     from EagleEye login to Salesforce.com.
- 2. User configures Salesforce app with name, secret key, and a URL for the EagleEye OAuth2 login page.
- 3. Potential Salesforce app users now directed to EagleEye login page; authenticated users return to Salesforce.com
     through callback URL (with authorization code).
- 4. Salesforce app passes authorization code along with secret key to OAuth2 and obtains access token.
- 5. Salesforce app attaches access token to any service calls.
- 6. Protected services call OAuth2 to validate access token.

The authentication code grant allows applications to share data without exposing user credentials.

### 04. Implicit credentials grant

The authorization grant is used when you’re running a web application through a traditional server-side web programming environment like Java or .NET. What happens if your client application is a pure JavaScript application or a mobile application that
runs completely in a web browser and doesn’t rely on server-side calls to invoke thirdparty services?
This is where the last grant type, the implicit grant, comes into play

- 1. JavaScript application owner registers application name and a callback URL.
- 2. Application user forced to authenticate by OAuth2 service.
- 3. OAuth2 redirects authenticated user to the callback URL (with access token as query parameter)
- 4. JavaScript app parses and stores the access token.
- 5. JavaScript app attaches access token to any service calls.
- 6. Protected services call OAuth2 to validate access token.

- The implicit grant is used in a browser-based Single-Page Application (SPA) JavaScript application.

### Token Refreshing

When an OAuth2 access token is issued, it has a limited amount of time that it’s valid and will eventually expire. When the token expires, the calling application (and user) will need to re-authenticate with the OAuth2 service. However, in most of the Oauth2
grant flows, the OAuth2 server will issue both an access token and a refresh token. A client can present the refresh token to the OAuth2 authentication service and the service will validate the refresh token and then issue a new OAuth2 access token.

- 1. User is already logged into application when their access token expires.
- 2. Application attaches expired token to next service call (to organization service).
- 3. Organization service calls OAuth2, gets response that token is no longer valid, passes response back to application.
- 4. Application calls OAuth2 with refresh token and receives new access token.

- The refresh token flow allows an application to get a new access token without forcing the user to re-authenticate.

### KEY TERMS

- Multiple Authentication Mechanisms
- Four forms of Authentication Grants

- 1. OAuth2 Password grant
  - This grant type is used when both the application and the services explicitly trust one another.
- 2. OAuth2 Client credentials grant -
  - This grant is typically used when an application needs to access an OAuth2 protected resource
- 3. OAuth2 Authorization code grant
  - When different applications from different vendors to share data and services without having to expose a user’s credentials
- 4. OAuth2 Implicit credentials grant
  - This grant is used when you’re running a web application through a traditional server-side web environment like Java or .NET.
