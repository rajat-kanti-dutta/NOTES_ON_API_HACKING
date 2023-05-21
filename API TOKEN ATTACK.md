
### TOKEN ANALYSIS

 Using this process can help you identify predictable tokens and aid in ** token forgery attacks. **
The tool: sequencer of burp suite.

This analysis using sequencer could demonstrate that a weak token creation process is in use.

We can use live capture or manual load. => character level analysis

#### Sequencer is great at showing that some complex tokens are actually very predictable. If an API provider is generating tokens sequentially then even if the token were 20 plus characters long, it could be the case that many of the characters in the token do not actually change. Making it easy to predict and create our own valid tokens.


## JWT 


### WHY IT IS USED?

(i) -   **Authorization**: This is the most common scenario for using JWT. Once the user is logged in, each subsequent request will include the JWT, allowing the user to access routes, services, and resources that are permitted with that token. Single Sign On is a feature that widely uses JWT nowadays, because of its small overhead and its ability to be easily used across different domains.

(ii) -   **Information Exchange**: JSON Web Tokens are a good way of securely transmitting information between parties. Because JWTs can be signed—for example, using public/private key pairs—you can be sure the senders are who they say they are. Additionally, as the signature is calculated using the header and the payload, you can also verify that the content hasn't been tampered with.

## JWT ATTACK

*** These tokens are susceptible to all sorts of misconfiguration mistakes that can leave the tokens vulnerable to several additional attacks. These attacks could provide you with sensitive information, grant you basic unauthorized access, or even administrative access to an API. *** 

JWTs consist of three parts, all of which are base64 encoded and separated by periods: the header, payload, and signature.

jwt.io is a online free jwt debugger.

They begin with "ey" because that is what happens when you base64 encode a curly bracket followed by a quote, which is the way that a decoded JWT always begins.

### header

```
{"alg":"HS256","typ":"JWT"}
```

### payload

```
{"sub":"1234567890","name":"hapihacker","iat":1516239022}
```

 A JWT body can be a great source of information disclosure. Consider a JWT that contains the username, email, password, and role all hiding behind one base64 decode.

Finally, the signature is the output of HMAC used for token validation and generated with the algorithm specified in the header. To create the signature, the API base64-encodes the header and payload and then applies the hashing algorithm and a secret. The secret can be in the form of a password or a secret string, such as a 256-bit key.

Without knowledge of the secret, the payload of the JWT will remain encoded. 

`HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)`


## More Information: https://jwt.io/introduction



## Automating JWT Attacks with JWT_Tool

### WHat it can do?
With this, we will be able to analyze JWTs, scan for weaknesses, forge tokens, and brute-force signature secrets.

Manual: https://github.com/ticarpi/jwt_tool/wiki

 jwt_tool makes the header and payload values nice and clear. Additionally, jwt_tool has a “Playbook Scan” that can be used to target a web application and scan for common JWT vulnerabilities.

```
jwt_tool -t http://127.0.0.1:8888/identity/api/v2/user/dashboard -rh "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ1c2VyYWFhQGVtYWlsLmNvbSIsImlhdCI6MTY1ODUwNjQ0NiwiZXhwIjoxNjU4NTkyODQ2fQ.BLMqSjLZQ9P2cxcUP5UCAmFKVMjxhlB4uVeIu2__6zoJCJoFnDqTqKxGfrMcq1lMW97HxBVDnYNC7eC-pl0XYQ" -M pb
```

During this scan for common misconfiguration, JWT_Tool tested the various claims found within the JWT (sub, iat, exp)

## The None Attack


If you ever come across a JWT using "none" as its algorithm, you’ve found an easy win. After decoding the token, you should be able to clearly see the header, payload, and signature.
From here, you can alter the information contained in the payload to be whatever you’d like
In other words, you can remove everything following the third period in the JWT. Send the JWT to the provider in a request and check whether you’ve gained unauthorized access to the API.



```

./jwt_tool.py eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJyYWphdEBlbWFpbC50aG0iLCJyb2xlIjoidXNlciIsImlhdCI6MTY4MjkxNjcxMiwiZXhwIjoxNjgzNTIxNTEyfQ.h7olL4v1headjneje8E1IRKJSguTCwuRd-VgLFvdoGFNVauTPhtumtrH-yAFln6nh-mzkKG_da1767tCiz4XPHglNCryWLtV9Igrhy-1-QiczRskOyJsBdtV_bfSMan369Ky55v1ACiTc1-siaoRpnw4xBlGegWZK4U7MvrATLxEXmZBQz8zrSlJSIkYTbf0Wd-V9c8-eVA_hxQiao1-JjMKcQnFe9rFlDZmSq7WHj_gFij8c68qeiZ_2kPQljvCvgR7FjKkrEaImkyw0lG2XGkgWnZfKKcBrx5cg4cW2ymunEOTCL55NgBdDj04th_oNdEabbLY5i_ms15La7wJdg -X a
```


### The Algorithm Switch Attack

There is a chance the API provider isn’t checking the JWTs properly. If this is the case, we may be able to trick a provider into accepting a JWT with an altered algorithm.

One of the first things you should attempt is sending a JWT without including the signature. This can be done by erasing the signature altogether and leaving the last period in place, like this: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJoYWNrYXBpcy5pbyIsImV4cCI6IDE1ODM2Mzc0ODgsInVzZ XJuYW1lIjoiU2N1dHRsZXBoMXNoIiwic3VwZXJhZG1pbiI6dHJ1ZX0. If this isn’t successful, attempt to alter the algorithm header field to "none". Decode the JWT, update the "alg" value to "none", base64-encode the header, and send it to the provider.

```
jwt_tool eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ1c2VyYWFhQGVtYWlsLmNvbSIsImlhdCI6MTY1ODg1NTc0MCwiZXhwIjoxNjU4OTQyMTQwfQ._EcnSozcUnL5y9SFOgOVBMabx_UAr6Kg0Zym-LH_zyjReHrxU_ASrrR6OysLa6k7wpoBxN9vauhkYNHepOcrlA -X a
```
If successful, pivot back to the None attack. However, if we try this with crAPI, the attack is not successful. You can also use JWT_Tool to create a none token.


### Another attack

 if the provider uses RS256 but doesn’t limit the acceptable algorithm values, we could alter the algorithm to HS256.
 ***  This is useful, as RS256 is an asymmetric encryption scheme, meaning we need both the provider’s private key and a public key in order to accurately hash the JWT signature. *** 

Meanwhile, HS256 is symmetric encryption, so only one key is used for both the signature and verification of the token. If you can discover and obtain the provider’s RS256 public key then switch the algorithm from RS256 to HS256, there is a chance you may be able to leverage the RS256 public key as the HS256 key.

 It uses the format jwt_tool TOKEN -X k -pk public-key.pem, as shown below. You will need to save the captured public key as a file on your attacking machine (You can simulate this attack by taking any public key and saving it as public-key-pem).

```
 jwt_tool eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ1c2VyYWFhQGVtYWlsLmNvbSIsImlhdCI6MTY1ODg1NTc0MCwiZXhwIjoxNjU4OTQyMTQwfQ._EcnSozcUnL5y9SFOgOVBMabx_UAr6Kg0Zym-LH_zyjReHrxU_ASrrR6OysLa6k7wpoBxN9vauhkYNHepOcrlA -X k -pk public-key-pem
```


### JWT Crack Attack

#### What it is? The JWT Crack attack attempts to crack the secret used for the JWT signature hash, giving us full control over the process of creating our own valid JWTs.

#### How to Crack? Hash-cracking attacks like this take place offline and do not interact with the provider. Therefore, we do not need to worry about causing havoc by sending millions of requests to an API provider. You can use JWT_Tool or a tool like Hashcat to crack JWT secrets.

### STEPS

```
#I. Using crunch generate a list of passwords:
$crunch 5 5 -o crAPIpw.txt

# We can use this password file that contains all possible character combinations created for 5 character passwords against our crAPI token.

$ jwt_tool TOKEN -C -d /wordlist.txt The -C option indicates that you’ll be conducting a hash crack attack, and the -d option specifies the dictionary or wordlist you’ll be using against the hash. JWT_Tool will either return “CORRECT key!” for each value in the dictionary or indicate an unsuccessful attempt with “key not found in dictionary.”
```

Now that we have the correct secret key that is used to sign crAPI's tokens, we should be able can generate our own trusted tokens. To test out our new abilities, you can either create a second user account in crAPI or use an email that you have already discovered.

You can add your user token to the JWT debugger, add the newly discovered secret, and update the "sub" claim to any email that has registered to crAPI.

Use the token that you generated in the debugger and add it to your Postman Collection. Make a request to an endpoint that will prove your access such as GET /identity/api/v2/user/dashboard.

