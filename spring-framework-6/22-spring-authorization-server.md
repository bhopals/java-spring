## Spring Authorization Server

- Refer

  - https://spring.io/projects/spring-authorization-server/
  - https://docs.spring.io/spring-authorization-server/reference/getting-started.html

- The OAuth2.0 Authorization Framework

  - Authorization Code Flow
  - Client Credentials Grant (This is the one we are going to explore and use in our REST App.)
  - Refresh Token Grant

- OpenID connect Core 1.0

  - Authorization Code Flow

- Client: Application requesting access to a protected resource on behalf of the Resource Owner.
- Resource Server : Server hosting the protected resources. This is the API you want to access.
- Authorization Server : Server that authenticates the Resource Owner and issues Access Tokens
  after getting proper authorization.

- Spring Authorization Server is a framework that provides implementations of the OAuth 2.1 and
  OpenID Connect 1.0 specifications

### Overview of OAuth 2

- OAuth 2 is an authorization framework
- OAuth 2 is used to grant limited access to resources without full access to the account
- OAuth 2 is used by organizations such as Google, Facebook, and GitHub
  - “Sign in With” - is using OAuth 2
  - Allows you to grant access to a third party application to act on your behalf

### OAuth Roles

- Resource Owner - the user who wishes to grant an application (client) access
- Client - The application requesting access
- Resource Server - The resource to access
- Authorization Server - Verifies the identity of the user then issues access tokens to the application

### Abstract Protocol Flow

Application (Client) ----Authorization Request---> Resource Owner (User)
Application (Client) <----Authorization Grant--- Resource Owner (User)

Application (Client) ----Authorization Grant---> Authorization Server
Application (Client) <----Access Token--- Authorization Server

Application (Client) ----Access Token----> Resource Server
Application (Client) <----Access to Resource--- Resource Server

### Types of OAuth Authorization Flows

- Authorization Code Flow - Server Side web applications where source code is not exposed publicly
- Client Credentials Flow - Used by services, where the “user” is a service role
  - Mostly been used. Works for REST
- Resource Owner Password Flow - Used by highly trusted applications when redirects cannot be used
- Implicit Flow - User grants access with redirects
- Hybrid Flow - similar to Client Credentials, but for long running applications
- Device Authorization Flow - Used by input constrained devices
- Authorization Code Flow with PKCE - Flow using proof Key Exchange

### Client Credentials Flow w/JWT

```
Application (Client) ----Authorization Request---> Resource Owner (User)
Application (Client) <----Authorization Grant--- Resource Owner (User)

Application (Client) ----Authorization Grant---> Authorization Server
Application (Client) <----Access Token--- Authorization Server
                                                    ^
                                                    |
                                                    Validate Token
                                                    |
                                                    |
Application (Client) ----Access Token----> Resource Server
Application (Client) <----Access to Resource--- Resource Server
```

### JWT Tokens

- JWT - JSON Web Token - often pronounced Ja-oot
- RFC 7519 - IETF Specification for JWT, defines how JWT is structured
- HTTP / REST are stateless - each request is self contained
- Unlike Web Applications which often use session id’s stored in cookies
- JWT token has user information and authorized roles (scopes)
- JWT has three parts - Header, Payload (data), and Signature
- The 3 parts are tokenized using Base 64 encoding

### JWT Token Signing

- JWT’s are signed - which prevents clients from altering the contents of the JWT token
- JWT’s maybe signed using a number of techniques
- Symmetric encryption - Uses single key to sign, requires key to be shared
- Asymmetric encryption - Uses public & private key (known as key pair)
  - Private Key is used to generate signature and is not shared
  - Public Key is shared and is used to verify signature

### JWT Token Verification

- The Authorization server signs the JWT token using the private key
- The Resource Server requests the public key from the Authorization Server
- Using the public key the resource server verifies the signature of the JWT token
- The resource server can cache the public key for verification of future requests
- Once the resource server has the public key, JWT tokens can be validated without additional
  requests from the Authorization Server

### OAuth vs HTTP Basic

- HTTP Basic Authentication requires user credentials to be shared with every resource
- HTTP Basic Authentication sends user credentials unencrypted in HTTP Header and can be compromised
- With OAuth user credentials are only shared with Authentication Server
- User credentials cannot be obtained from authorization token
- HTTP Basic Authentication has no concept of security roles
- With OAuth 2 security roles are defined in scopes and passed in token

### Init Project

- Create new Spring boot project selecting below:

  - Spring Security
  - H2 Database
  - JDBC API

- Add `OAuth Server dependencies`

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>1.2.1</version>
</dependency>
```

### All 8 Steps Config Process

This is a minimal configuration for getting started quickly. To understand what each component is used for, see the following descriptions:

- 1. A Spring Security filter chain for the Protocol Endpoints.
- 2. A Spring Security filter chain for authentication.
- 3. An instance of UserDetailsService for retrieving users to authenticate.
- 4. An instance of RegisteredClientRepository for managing clients.
- 5. An instance of com.nimbusds.jose.jwk.source.JWKSource for signing access tokens.
- 6. An instance of java.security.KeyPair with keys generated on startup used to create the JWKSource.
- 7. An instance of JwtDecoder for decoding signed access tokens.
- 8. An instance of AuthorizationServerSettings to configure Spring Authorization Server.

### 1. Add Authorization Server Filter Chain

- This Sets up OAuth Authorization Server with default settings
- Grant specific Endpoints for use

```
@Configuration
@EnableWebSecurity
public class SecurityConfig {

	@Bean
	@Order(1)
	public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http)
			throws Exception {
		OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
		http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
			.oidc(Customizer.withDefaults());	// Enable OpenID Connect 1.0
		http
			// Redirect to the login page when not authenticated from the
			// authorization endpoint
			.exceptionHandling((exceptions) -> exceptions
				.defaultAuthenticationEntryPointFor(
					new LoginUrlAuthenticationEntryPoint("/login"),
					new MediaTypeRequestMatcher(MediaType.TEXT_HTML)
				)
			)
			// Accept access tokens for User Info and/or Client Registration
			.oauth2ResourceServer((resourceServer) -> resourceServer
				.jwt(Customizer.withDefaults()));

		return http.build();
	}
}
```

### 2. Add Default Security Filter Chain

- Authorize all the request comes to Auth Server for Authentication
- If not authenticated, redirects to the login page

```
@Bean
	@Order(2)
	public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http)
			throws Exception {
		http
			.authorizeHttpRequests((authorize) -> authorize
				.anyRequest().authenticated()
			)
			// Form login handles the redirect to the login page from the
			// authorization server filter chain
			.formLogin(Customizer.withDefaults());

		return http.build();
	}
```

### 3. Create User Details Service

- Create User Details

```
@Bean
	public UserDetailsService userDetailsService() {
		UserDetails userDetails = User.withDefaultPasswordEncoder()
				.username("user")
				.password("password")
				.roles("USER")
				.build();

		return new InMemoryUserDetailsManager(userDetails);
	}
```

### 4. Add Registered Client Repository

- Sets up Registered Client Repository

```
@Bean
	public RegisteredClientRepository registeredClientRepository() {
		RegisteredClient oidcClient = RegisteredClient.withId(UUID.randomUUID().toString())
				.clientId("oidc-client")
				.clientSecret("{noop}secret")
				.clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
				.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
				.authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
				.redirectUri("http://127.0.0.1:8080/login/oauth2/code/oidc-client")
				.postLogoutRedirectUri("http://127.0.0.1:8080/")
				.scope(OidcScopes.OPENID)
				.scope(OidcScopes.PROFILE)
				.clientSettings(ClientSettings.builder().requireAuthorizationConsent(true).build())
				.build();

		return new InMemoryRegisteredClientRepository(oidcClient);
	}
```

### 5. Create JWK Source

- JWK (JSON Web Key Set)
- Setting up Keys to sign the JWT token
- Validate JWT token using Public Key

```
@Bean
	public JWKSource<SecurityContext> jwkSource() {
		KeyPair keyPair = generateRsaKey();
		RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
		RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
		RSAKey rsaKey = new RSAKey.Builder(publicKey)
				.privateKey(privateKey)
				.keyID(UUID.randomUUID().toString())
				.build();
		JWKSet jwkSet = new JWKSet(rsaKey);
		return new ImmutableJWKSet<>(jwkSet);
	}
```

### 6. Generate java.security.KeyPair

```
private static KeyPair generateRsaKey() {
		KeyPair keyPair;
		try {
			KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
			keyPairGenerator.initialize(2048);
			keyPair = keyPairGenerator.generateKeyPair();
		}
		catch (Exception ex) {
			throw new IllegalStateException(ex);
		}
		return keyPair;
	}
```

### 7. Create JWT Decoder

- To decode a JWT (Json Web Token)

```
@Bean
	public JwtDecoder jwtDecoder(JWKSource<SecurityContext> jwkSource) {
		return OAuth2AuthorizationServerConfiguration.jwtDecoder(jwkSource);
	}
```

### 8. Set Authorization Server Setting

```
@Bean
	public AuthorizationServerSettings authorizationServerSettings() {
		return AuthorizationServerSettings.builder().build();
	}
```

The above config is going to create below

```
public final class AuthorizationServerSettings extends AbstractSettings {

	...

	public static Builder builder() {
		return new Builder()
			.authorizationEndpoint("/oauth2/authorize")
			.deviceAuthorizationEndpoint("/oauth2/device_authorization")
			.deviceVerificationEndpoint("/oauth2/device_verification")
			.tokenEndpoint("/oauth2/token")
			.tokenIntrospectionEndpoint("/oauth2/introspect")
			.tokenRevocationEndpoint("/oauth2/revoke")
			.jwkSetEndpoint("/oauth2/jwks")
			.oidcLogoutEndpoint("/connect/logout")
			.oidcUserInfoEndpoint("/userinfo")
			.oidcClientRegistrationEndpoint("/connect/register");
	}

	...

}
```

### Summary

- Run the spring applicaton (on different port - 9000)
- Got to Postman
  - Select `Authorization` section of the Request - `http://localhost:8080/api/v1/beer`
  - http://localhost:9000/oauth2/token
  - Select Grant type - `Client Credentials`
  - Enter `clientId` and `clientSecret`
  - Enter Scope details - `message.read message.write`
  - Click on `Get New Access Token`
  - Copy the returned token
  - Select `OAuth2.0` from Authorization Type
  - Select Add Auth. data to Request Header
  - Enter Access Token to make a request to Resource Server

### KEY TERMS

- Authorization/Authentication Server
- Resource Server
- Resource Owner
- Client

- OAuth Authorization Flow - Client Credential Flow
- Symmetric Encryption
- Assymetric Encryption

- JWT (Json Web Token)
- JWK (JSON Web Key Set)
