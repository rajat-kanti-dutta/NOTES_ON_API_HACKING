#### What It is?

OAuth allows the user to grant this access without exposing their login credentials to the requesting application. This means users can fine-tune which data they want to share rather than having to hand over full control of their account to a third party.

## How Does OAuth2.0 Works?

OAuth 2.0 was originally developed as a way of sharing access to specific data between applications.

- Client Application - The website or application that want to access user data
- Resource Owner - The user whose data the client application wants to access.
- OAuth Service Provider - The website or application that controls the user's data and access to it. They support OAuth by providing an API for interacting with both an authorization server and a resource server.


### Grant Types

- The client application requests access to a subset of the user's data, specifying which grant type they want to use and what kind of access they want.
- The user is prompted to log in to the OAuth service and explicitly give their consent for the requested access.
- The client application receives a unique access token that proves they have permission from the user to access the requested data. Exactly how this happens varies significantly depending on the grant type.
- The client application uses this access token to make API calls fetching the relevant data from the resource server.

## OAuth Grant Types

#### What is OAuth Grant Type?
The OAuth grant type determines the exact sequence of steps that are involved in the OAuth process. The grant type also affects how the client application communicates with the OAuth service at each stage, including how the access token itself is sent. For this reason, grant types are often referred to as "OAuth flows".

An OAuth service must be configured to support a particular grant type before a client application can initiate the corresponding flow.

### OAuth scopes

For any OAuth grant type, the client application has to specify which data it wants to access and what kind of operations it wants to perform. It does this using the `scope` parameter of the authorization request it sends to the OAuth service.

Example:

```
scope=contacts scope=contacts.read scope=contact-list-r scope=https://oauth-authorization-server.com/auth/scopes/user/contacts.readonly
```


##### When OAuth is used for authentication, however, the standardized OpenID Connect scopes are often used instead.

### BASICS OF Authorization Code GRANT

- the client application and OAuth service first use redirects to exchange a series of browser-based HTTP requests that initiate the flow.
- The user is asked whether they consent to the requested access. If they accept, the client application is granted an "authorization code". 
- The client application then exchanges this code with the OAuth service to receive an "access token", which they can use to make API calls to fetch the relevant user data.
- All communication that takes place from the code/token exchange onward is sent server-to-server over a secure, preconfigured back-channel and is, therefore, invisible to the end user.
-  *** This secure channel is established when the client application first registers with the OAuth service. At this time, a `client_secret` is also generated, which the client application must use to authenticate itself when sending these server-to-server requests. ***
- As the most sensitive data (the access token and user data) is not sent via the browser, this grant type is arguably the most secure. Server-side applications should ideally always use this grant type if possible.
![[Pasted image 20230519190341.png]]

### Explanations

1. Authorization Request: 
The client application sends a request to the OAuth service's `/authorization` endpoint asking for permission to access specific user data.

** This endpoint may vary b/w providers **

**  However, you should always be able to identify the endpoint based on the parameters used in the request. **

Example:
```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1 Host: oauth-authorization-server.com
```

#### Notes for Parameters

- client_id : Mandatory parameter containing the unique identifier of the client application. This value is generated when the client application registers with the OAuth service.
- redirect_uri : The URI to which the user's browser should be redirected when sending the authorization code to the client application. This is also known as the "callback URI" or "callback endpoint". *** many OAuth exploits are based upon validation of this parameter ***
- response_type : Determines which kind of response the client application is expecting and, therefore, which flow it wants to initiate. For the authorization code grant type, the value should be `code`.
- scope : Used to specify which subset of the user's data the client application wants to access.
- state: Stores a unique, unguessable value that is tied to the current session on the client application. The OAuth service should return this exact value in the response, along with the authorization code. *** This parameter serves as a form of [CSRF](https://portswigger.net/web-security/csrf) token for the client application by making sure that the request to its `/callback` endpoint is from the same person who initiated the OAuth flow. ***

2. User Login & concent
When the authorization server receives the initial request, it will redirect the user to a login page, where they will be prompted to log in to their account with the OAuth provider.

They will then be presented with a list of data that the client application wants to access. This is based on the scopes defined in the authorization request. The user can choose whether or not to consent to this access.

The first time the user selects "Log in with social media", they will need to manually log in and give their consent, but if they revisit the client application later, they will often be able to log back in with a single click.

3. Authorization Code Grant
If the user consents to the requested access, their browser will be redirected to the `/callback` endpoint that was specified in the `redirect_uri` parameter of the authorization request.
The resulting `GET` request will contain the authorization code as a query parameter. Depending on the configuration, it may also send the `state` parameter with the same value as in the authorization request.

```
GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1 
Host: client-app.com
```

4. Access Token Request
Once the client application receives the authorization code, it needs to exchange it for an access token.

##### To do this, it sends a server-to-server `POST` request to the OAuth service's `/token` endpoint. All communication from this point on takes place in a secure back-channel and, therefore, cannot usually be observed or controlled by an attacker.

```
POST /token HTTP/1.1 
Host: oauth-authorization-server.com 
… 
client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8
```

- client_secret : The client application must authenticate itself by including the secret key that it was assigned when registering with the OAuth service.
- grant_type : Used to make sure the new endpoint knows which grant type the client application wants to use. In this case, this should be set to `authorization_code`.

5. Access Token Grant
The OAuth service will validate the access token request. If everything is as expected, the server responds by granting the client application an access token with the requested scope.

```
{ "access_token": "z0y9x8w7v6u5",
"token_type": "Bearer",
"expires_in": 3600,
"scope": "openid profile",
…
}

```

6. API Call
Now the client application has the access code, it can finally fetch the user's data from the resource server. To do this, it makes an API call to the OAuth service's `/userinfo` endpoint.

```
GET /userinfo HTTP/1.1
Host: oauth-resource-server.com 
Authorization: Bearer z0y9x8w7v6u5
```

7. Resource Grant
The resource server should verify that the token is valid and that it belongs to the current client application. If so, it will respond by sending the requested resource i.e. the user's data based on the scope of the access token.


## Implicit Grant Type

The implicit grant type is much simpler.
#### It is far less secure 

##### Rather than first obtaining an authorization code and then exchanging it for an access token, the client application receives the access token immediately after the user gives their consent.

##### When using the implicit grant type, all communication happens via browser redirects - there is no secure back-channel like in the authorization code flow. This means that the sensitive access token and the user's data are more exposed to potential attacks.

*** Implicit Grant Type is suited to Single-Page Application ***

![[Pasted image 20230519194525.png]]


1. Authorization Request
The implicit flow starts in much the same way as the authorization code flow. The only major difference is that the `response_type` parameter must be set to `token`.

```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```

2. User Login and Consent
The user logs in and decides whether to consent to the requested permissions or not. This process is exactly the same as for the authorization code flow.

3. Access Token Grant
If the user gives their consent to the requested access, this is where things start to differ.
###### The OAuth service will redirect the user's browser to the `redirect_uri` specified in the authorization request. However, instead of sending a query parameter containing an authorization code, it will send the access token and other token-specific data as a URL fragment.

```
GET /callback#access_token=z0y9x8w7v6u5&token_type=Bearer&expires_in=5000&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1 Host: client-app.com
```
*** As the access token is sent in a URL fragment, it is never sent directly to the client application. Instead, the client application must use a suitable script to extract the fragment and store it. ***

4. API Call
Once the client application has successfully extracted the access token from the URL fragment, it can use it to make API calls to the OAuth service's `/userinfo` endpoint.
##### Unlike in the authorization code flow, this also happens via the browser.

```
GET /userinfo HTTP/1.1 
Host: oauth-resource-server.com 
Authorization: Bearer z0y9x8w7v6u5
```

5. Resource Call
The resource server should verify that the token is valid and that it belongs to the current client application. If so, it will respond by sending the requested resource i.e. the user's data based on the scope associated with the access token.

OAuth Vulnerabilities ()