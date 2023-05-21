##### An API’s authentication process is meant to validate that users are who they claim to be. 

##### An API's authorization is meant to allow users to access the data they are permitted to access.

 API providers have been pretty good about requiring authentication when necessary, but there has been a tendency to overlook controls beyond the hurdle of authentication

#### Authorization vulnerabilities are so common for APIs that the OWASP security project included two authorization vulnerabilities on its top ten list, Broken Object Level Authorization (BOLA) and Broken Function Level Authorization (BFLA).

#### AUTHENTICATION PROCESS IN RESTFUL API
RESTful APIs are stateless, so when a consumer authenticates to these APIs, *** no session is created between the client and server. *** Instead, the API consumer must prove their identity within every request sent to the API provider’s web server.

Authorization weaknesses are present *** within the access control mechanisms of an API. ***  An API consumer should only have access to the resources they are authorized to access.


###  BOLA vulnerabilities occur when an API provider does not restrict access to access to resources.

### BFLA vulnerabilities are present when an API provider does not restrict the actions that can be used to manipulate the resources of other users.

## BOLA is the ability for UserA to see UserB's bank account balance and BFLA is the ability to for UserA to transfer funds from UserB's account back to UserA.


## Which Ingredients are required for BOLA ?

When hunting for BOLA there are three ingredients needed for successful exploitation.

I.  Resource ID: a resource identifier will be the value used to specify a unique resource. This could be as simple as a number, but will often be more complicated.

II.  Requests that access resources. In order to test if you can access another user's resource, you will need to know the requests that are necessary to obtain resources that your account should not be authorized to access.

III.  Missing or flawed access controls. In order to exploit this weakness, the API provider must not have access controls in place. This may seem obvious, but just because resource IDs are predictable, does not mean there is an authorization vulnerability present.

## Finding Resource IDs and Requests

You can test for authorization weaknesses by understanding how an API’s resources are structured and then attempting to access resources you shouldn’t be able to access.

By detecting patterns within API paths and parameters, you might be able to predict other potential resources.

-   GET /api/resource/**1**
-   POST /company/account/**Apple**/balance
-   POST /admin/pwreset/account/**90**
In these instances, you can probably guess other potential resources

## Authorization Testing Strategy

When searching for authorization vulnerabilities the most effective way to find authorization weaknesses is to create two accounts and perform A-B testing. The A-B testing process consists of:

1.  Create a UserA account.
2.  Use the API and discover requests that involve resource IDs as UserA.
3.  Document requests that include resource IDs and should require authorization.
4.  Create a UserB account.
5.  Obtaining a valid UserB token and attempt to access UserA's resources.


## BFLA

Where BOLA is all about accessing resources that do not belong to you, BFLA is all about performing unauthorized actions.

BFLA vulnerabilities are common for requests that perform actions of other users.

### LATERAL ACTION & ESCALATED ACTION

These requests could be lateral actions or escalated actions. 
Lateral actions are requests that perform actions of users that are the same role or privilege level.
Escalated actions are requests that perform actions that are of an escalated role like an administrator.


The main difference between hunting for BFLA is that you are looking for functional requests. This means that you will be testing for various HTTP methods, seeking out actions of other users that you should not be able to perform.

 For BFLA we will be hunting for very similar requests to BOLA.

1.  Resource ID: a resource identifier will be the value used to specify a unique resource. 
2.  Requests that perform authorized actions. In order to test if you can access another update, delete, or otherwise alter other the resources of other users.
3.  Missing or flawed access controls. In order to exploit this weakness, the API provider must not have access controls in place.

 When we are thinking of CRUD (create, read, update, and delete), BFLA will mainly concern requests that are used to update, delete, and create resources that we should not be authorized to.

 For APIs that means that we should scrutinize requests that utilize POST, PUT, DELETE, and potentially GET with parameters.  We will need to search through the API documentation and/or collection for requests that involve altering the resources of other users. So, if we can find requests that create, update, and delete resources specified by a resource ID then we will be on the right track.

### HOW TO PERFORM?

For BFLA we will perform A-B-A testing. The reason is with BFLA there is a potential to alter another user's resources.
So when performing testing there is a chance that we receive a successful response indicating that we have altered another user's resources, but to have a stronger PoC we will want to verify with the victim's account.
So, we make valid requests as UserA, switch out to our UserB token, attempt to make requests altering UserA's resources, and return to UserA's account to see if we were successful.

*** When successful, BFLA attacks can alter the data of other users. This means that accounts and documents that are important to the organization you are testing could be on the line. DO NOT brute force BFLA attacks, instead, use your secondary account to safely attack your own resources. Deleting other users' resources in a production environment will likely be a violation of most rules of engagement for bug bounty programs and penetration tests. ***






