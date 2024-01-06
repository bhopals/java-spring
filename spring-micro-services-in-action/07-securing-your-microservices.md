## Securing your Microservices

- Spring Cloud security and the OAuth2 (Open Authentication)

- A secure application involves multiple layers of protection, including:

  - Ensuring that the proper user controls are in place so that you can validate that a user is who they say they are and
    that they have permission to do what they’re trying to do.

  - Keeping the infrastructure the service is running on patched and up-to-date to minimize the risk of vulnerabilities.

  - Implementing network access controls so that a service is only accessible through well-defined ports and accessible to
    a small number of authorized servers.

- OAuth2 is a token-based security framework that allows users to authenticate themselves with a third-party authentication service.

- The real power behind OAuth2 is that it allows application developers to easily integrate with third-party cloud providers
  and do user authentication and authorization with those services without having to constantly pass the user’s credentials to the thirdparty service. Cloud providers such as Facebook, GitHub, and Salesforce all support OAuth2 as a standard.

### Introduction to OAuth2

- OAuth2 is a token-based security authentication and authorization framework that breaks security down into four components.
- These four components are:

  - 1. Protected resource
  - 2. Resource Owner
  - 3. Application
  - 4. OAuth2 Authentication Server

- OAuth2 allows a user to authenticate without constantly having to present credentials.

  - 1. The service we want to protect
  - 2. The resource owner grants which applications/users can access the resource via the OAuth2 service.
  - 3. When the user tries to access a protected service they must authenticate and obtain a token from the OAuth2 service.
  - 4. The OAuth2 server authenticates the user and validates tokens presented to it.

- The OAuth2 authentication server is the intermediary between the application and the services being consumed
- OAuth2 allows you to protect your REST-based services across these different scenarios through different authentication
  schemes called grants. The OAuth2 specification has four types of grants:

  - Password
  - Client credential
  - Authorization code
  - Implicit

- “grant type” refers to the way an application gets an access token
- OAuth `flows`/`grant types`

### JavaScript Web Tokens and OAuth2

OAuth2 is a token-based authentication framework, but ironically it doesn’t provide any standards for how the tokens in its specification are to be defined. To rectify the lack of standards around OAuth2 tokens, a new standard is emerging called JavaScript
Web Tokens (JWT). JWT is an open standard (RFC-7519) proposed by the Internet Engineering Task Force (IETF) that attempts to provide a standard structure for OAuth2 tokens.

- JWT tokens are:
  - Small—JWT
  - Cryptographically signed
  - Self-contained
  - Extensible

### Some closing thoughts on microservice security

- 1. Use HTTPS/Secure Sockets Layer (SSL) for all service communication.
- 2. All service calls should go through an API gateway.
- 3. Zone your services into a public API and private API.
- 4. Limit the attack surface of your microservices by locking down unneeded network ports.

### Summary

- OAuth2 is a token-based authentication framework to authenticate users.

- OAuth2 ensures that each microservice carrying out a user request doesn’t need to be presented with user credentials
  with every call.

- OAuth2 offers different mechanisms for protecting web services calls. These mechanisms are called grants.

- To use OAuth2 in Spring, you need to set up an OAuth2-based authentication service.

- Each application that wants to call your services needs to be registered with your OAuth2 authentication service.

- Each application will have its own application name and secret key.

- User credentials and roles are in memory or a data store and accessed via Spring security.

- Each service must define what actions a role can take.

- Spring Cloud Security supports the JavaScript Web Token (JWT) specification.

- JWT defines a signed, JavaScript standard for generating OAuth2 tokens.

- With JWT, you can inject custom fields into the specification.

- Securing your microservices involves more than just using OAuth2.

- You should use HTTPS to encrypt all calls between services.

- Use a services gateway to narrow the number of access points a service can be reached through.

- Limit the attack surface for a service by limiting the number of inbound and outbound ports on the operating system
  that the service is running on.

### KEY TERMS

- Multiple layers of protection
- Ensuring proper User controls are in place
- Minimize risk of vulnerabilities
- Implementing Network Access Controls

- Spring Cloud security and the OAuth2 (Open Authentication)
- OAuth2 is a token-based security framework
- Third-party authentication service
- Third-party cloud providers

- OAuth2 is a token-based security authentication and authorization framework

- Protected resource
- Resource Owner
- Application
- OAuth2 Authentication Server
- OAuth2 Token-based security framework

- JWT is an open standard (RFC-7519) proposed by the Internet Engineering Task Force (IETF) that attempts to provide
  a standard structure for OAuth2 tokens

- OAuth2-based authentication service
- Limit the attack surface
