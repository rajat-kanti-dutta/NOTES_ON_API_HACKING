Testing for Improper Assets Management is all about discovering unsupported and non-production versions of an API.

Often times an API provider will update services and the newer version of the API will be available over a new path like the following:

-   api.target.com/v3
-   /api/v2/accounts
-  /api/v3/accounts
-  /v2/accounts
API versioning could also be maintained as a header:

-   _Accept: version=2.0_
In addition versioning could also be set within a query parameter or request body.

-   /api/accounts?ver=2

-   POST /api/accounts  
      
    {  
    "ver":1.0,  
    "user":"hapihacker"  
    }
In these instances, earlier versions of the API may no longer be patched or updated.   Since the older versions lack this support, they may expose the API to additional vulnerabilities.

Non-production versions of an API include any version of the API that was not meant for end-user consumption. Non-production versions could include:

-   api.test.target.com
-   api.uat.target.com
-   beta.api.com
-  /api/private
-  /api/partner
- /api/test

The discovery of non-production versions of an API might not be treated with the same security controls as the production version. Once we have discovered an unsupported version of the API, we will test for additional weaknesses.

## Finding Improper Assets Management Vulnerabilities

You can discover mass assignment vulnerabilities by finding interesting parameters in API documentation and then adding those parameters to requests.

Look for parameters involved in user account properties, critical functions, and administrative actions. Intercepting API requests and responses could also reveal parameters worthy of testing. Additionally, you can guess parameters or fuzz them in API requests that accept user input.

