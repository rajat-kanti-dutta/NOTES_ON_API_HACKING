
WAFs, for example, can be triggered by a wide variety of things, like:

(i) -   Too many requests for resources that do not exist.
(ii) -   Too many requests within a certain amount of time
(iii) -   Common attack attempts, like SQL injection and XSS attacks
(iv) -   Abnormal behavior, like tests for authorization vulnerabilities

### Evading security controls is a process of trial and error

## String Terminators

Null bytes and other combinations of symbols are often interpreted as string terminators used to end a string.

 If these symbols are not filtered out, they could terminate the API security control filters that may be in place.  When you are able to successfully send a null byte it is interpreted by many back-end programming languages as a signifier to stop processing.


 If the null byte is processed by a back-end program that validates user input, then the security control could be bypassed because it stops processing the following input.

```
%00

0x00

//

;

%

!

?

[]

%5B%5D

%09

%0a

%0b

%0c

%0e
```

### Where to place?
String terminators can be placed in different parts of the request, like the path or POST body, to attempt to bypass any restrictions in place.

For example, in the following injection attack to the user profile endpoint, the null bytes entered into the payload could bypass filtering

POST /api/v1/user/profile/update

[…]

{

“uname”: “hapihacker”

“pass”: "%00'OR 1=1"

}

*** In this example, input validation measures might be bypassed due to the null byte placed before the SQL injection attempt. ***


## Case Switching

Case switching is the literal switching of the case of the letters within the URL path or payload.

For example, take the following POST requests. Let’s say you were targeting a social media site by attempting an IDOR attack against a uid parameter in the following POST request:  

POST /api/myprofile 

[…] 

{uid=§0001§}

An API may leverage rate-limiting to only allow 100 requests per minute. Based on the length of the uid value, you know that to brute force it you’ll need to send 10,000 requests to exhaust all possibilities. To bypass rate-limiting or other WAF controls you may be able to simply alter the URL path by switching upper- and lower-case letters in the path: 

POST /api/myProfile 

POST /api/MyProfile 

POST /aPi/MypRoFiLe

*** Each of these path iterations could cause the API provider to handle the request differently, potentially bypassing the rate limit. ***

#### In some cases, a control like rate-limiting may disappear completely with a different spelling or the new spelling may have its own renewed rate limit. In the case where rate-limiting is no longer applied, you can just send over as many requests as you need to the endpoint with the casing switched. In the case where rate-limiting is renewed, you could use the BurpSuite Pitchfork attack to pair a certain number of attacks to a certain number of brute-force attempts.

For example:

POST /api/myprofile paired with uid 001 - 100

POST /api/Myprofile paired with uid 101 - 200

POST /api/mYprofile paired with uid 201 - 300

If you were using Burp Suite’s Intruder for this attack, you could set the Attack Type to Pitchfork and use the same value for both payload positions. This tactic allows you to use the smallest number of requests required to brute force the uid.

## Encoding Payloads

 Encoded payloads can often trick WAFs while still being processed by the target application or database
Even if the WAF or an input-validation measure blocks certain characters or strings, they might miss encoded versions of those characters. 

### Alternatively, you could also try double encoding your attacks.


## Payload Processing using Burp Suite

####  Under the Intruder Payloads option is a section called Payload Processing that allows you to add rules that Burp will apply to each payload before it is sent.
#### The Add button will let you add various rules to each payload, like a prefix, a suffix, encoding, hashing, and custom input. It can also match and replace various characters.

Check the payload column of your attack to make sure the payloads have been processed properly.


## Evasion with Wfuzz

Wfuzz also has some great capabilities for payload processing.

Documentation Link ref: https://wfuzz.readthedocs.io/

(i) If you need to encode a payload, you’ll need to know the name of the encoder you want to use. To see a list of all Wfuzz encoders, use the following:
```
wfuzz -e encoders
```

(ii) to use an encoder, add a comma to the payload and specify its name:
```
wfuzz -z file,wordlist/api/common.txt,base64 http://hapihacker.com/FUZZ
```

In this example, every payload would be base64-encoded before being sent in a request.

(iii)  The encoder feature can also be used with multiple encoders. To have a payload processed multiple encoders specify them with a hyphen.

```
wfuzz -z list,TEST,base64-md5-none
```

#### Then you would receive one payload encoded to base64, another payload encoded by MD5, and a final payload in its original form (none=not encoded). This would result in three different payloads.

```
wfuzz -z list,a-b-c,base64-md5-none -u http://hapihacker.com/api/v2/FUZZ
```

#### In this case, the list of payloads was individually processed three different times. Each letter (a, b, and c)  was base64 encoded, hashed with md5, and sent in the original form.

### If, instead, you want each payload to be processed by multiple encoders, separate the encoders with an @ sign:


## To dive deeper into the topic of WAF bypassing, I recommend checking out the incredible Awesome-WAF GitHub repo ([https://github.com/0xInfection/Awesome-WAF](https://github.com/0xInfection/Awesome-WAF)), where you’ll find a ton of great information.



## COMBINING TECHNIQUES

-   Combined an excessive data exposure finding with a password-spraying attack to compromise accounts.
-  Combined a Broken Function Level Authorization finding with a Mass Assignment attack to add some funds back to your crAPI account.
-  Combined an Improper Assets Management vulnerability to exploit a successful brute force attack against a multi-factor authentication system.


## SUGGESTIONS FOR COMBINING VULNERABILITIES AND TECHNIQUES

There are many different possible ways to combine API attack techniques. One thing that you may not realize is that once you have found a small crack in the security of an API then you may have found a launchpad for the rest of your attack.

### Excessive Data Exposure and Improper Assets Management are both API weaknesses that stand out above the others to combine in order to escalate the impact of your attacks.

-   Excessive data exposure is a vulnerability that can be leveraged in many other attacks. Excessive Data Exposure can be a great source of information about users and administrators. Make sure that you take note of all parameters leaked in exposure, such as username, email, privilege level, and unique user identifiers. All of this information could be challenging to come across otherwise but can be leveraged in authentication and authorization attacks.
- ** Improper Assets Management vulnerability is a weakness that should be combined with all other attack techniques. Improper Assets Management implies that a version of the API may not be as supported as the current version and thus it may not be as protected. Perform all of the other attacks against the unmanaged endpoint. You may find that the security of a api/test/login, api/mobile/login, api/v1/login, api/v1/internal/users may have fewer protections than the supported counterparts. => With Improper Assets Management findings there could be lax rate-limiting, missing protections for brute force attempts, missing input validation, unlimited multifactor authentication requests, authorization vulnerabilities, etc. With APIs affected by Improper Assets Management vulnerabilities you never know what you’re going to get. So, make sure to perform the full gambit of attack techniques against unsupported versions of the API. **
-   If you are testing a file upload function, attempt to use the directory traversal fuzzing attack along with the file upload to manipulate where the file is stored. Upload files that lead to a shell and attempt to execute the uploaded file with the corresponding web application.
-  Perhaps you have discovered a Broken Function Level Authorization (BFLA) or Mass Assignment weakness that has given you access to administrative functionality. There is a chance that an API provider trusts its administrators and does not have as many technical protections in place. You could then use this new level of access to search for Improper Assets Management, injection weaknesses, or Broken Object Level Authorization weaknesses.
-  If you are able to perform a mass assignment, then there is a chance that the parameter or parameters that you can alter may not be protected from injection attacks and/or Server-side Request Forgery. When you have discovered mass assignment fuzz the parameter for other weaknesses.
-    If you are able to update to perform a BFLA attack and update another user’s profile information perhaps you could combine this with an XSS attack.
-   Combine Injection attacks with most other vulnerabilities. For example, if you are able to manipulate a JSON Web Token. Consider including various different types of injection attacks into the JSON Web Token.
-  if you find a BOLA vulnerability that provides you with other users’ bank account information check to see if you are also able to exploit a BFLA vulnerability to transfer funds.
-   If you are up against API security controls like a WAF or rate-limiting, make sure to apply evasion techniques to your attacks. Encode your attacks and attempt to find ways to bypass these security measures. If you have exhausted your testing efforts, return to fuzzing wide with encoded and obfuscated attacks.









 

