
In order to target APIs, you must first be able to find them.

## What is Public API?

(i) Public APIs are meant to be easily found and used by end-users. 
(ii) Public APIs may be entirely public without authentication or be meant for use by authenticated users.
(iii) Authentication for a public API primarily depends on the sensitivity of the data that is handled
(iv) When a public API handles public information then, authentication not required in all other instances authentication required.
(v)  Public APIs are meant to be consumed by end-users.  API providers share documentation that serves as an instruction manual for a given API. This documentation should be end-user friendly and relatively straightforward to find.

## What is Partner API?

(i) are intended to be used exclusively by partners of the provider. So, harder to find, if you are not a partner.
(ii) Partner APIs may be documented but that is limited to partners.

## Private API
(i) are intended for use, privately, within an organization.
(ii) Less documented than Partner APIs.


## WEB API INDICATOR

### What is the Goal? 
The goal here is to find APIs to attack and this can be accomplished by discovering the API itself or the API documentation

(i) To get API documentation (if possible)
(ii) To look around the target's landing page , to get an API (links to API or development portal)
(iii) URL Naming scheme which will be useful for searching API endpoint.
	[https://target-name.com/api/v1](https://target-name.com/api/v1) 

[https://api.target-name.com/v1](https://api.target-name.com/v1) 

[https://target-name.com/docs](https://target-name.com/v1)

[https://dev.target-name.com/rest](https://target-name.com/v1)

(iv) Look for API indicators within directory names like:  
_/api, /api/v1, /v1, /v2, /v3, /rest, /swagger, /swagger.json, /doc, /docs, /graphql, /graphiql, /altair, /playground_

(v) Subdomains are also indicator of Web APIs.
```

api.target-name.com

uat.target-name.com

dev.target-name.com

developer.target-name.com

test.target-name.com

```

(vi) Another indicator of web API is the HTTP request and response headers. The Use of ** JSON or XML ** could be a good indicator that you've discovered an API

```
_HTTP Request and Response Headers containing "Content-Type: application/json, application/xml"_

_Also, watch for HTTP Responses that include statements like:  
{"message": "Missing Authorization token"}_
```

(vii) One of the most obvious indicators of an API would be through information gathered using third-Party Sources like Github and API directories.

Gitub: [https://github.com/](https://github.com/) 

Postman Explore: [https://www.postman.com/explore/apis](https://www.postman.com/explore/apis)

ProgrammableWeb API Directory: [https://www.programmableweb.com/apis/directory](https://www.programmableweb.com/apis/directory) 

APIs Guru: [https://apis.guru/](https://apis.guru/) 

Public APIs Github Project: [https://github.com/public-apis/public-apis](https://github.com/public-apis/public-apis) 

RapidAPI Hub: [https://rapidapi.com/search/](https://rapidapi.com/search/)



