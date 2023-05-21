
#### One of the best ways to fine-tune your brute-force attack is to generate passwords specific to your target. 

To do this, you could leverage the information revealed in an excessive data exposure vulnerability . 

### How to Create an Username & Password List ?

For more information about creating targeted password lists, check out the Mentalist app (https://github.com/sc0tfree/mentalist) or the Common User Passwords Profiler (https://github.com/Mebus/cupp).


### Some useful notes on WFuzz

When fuzzing API followings are useful.

Important items to note for API testing include the headers option (-H),
hide responses (--hc, --hl, --hw, --hh), and POST body requests (-d).

 We will need to specify the content-type headers for APIs which will be 'Content-Type: application/json'


*** Example *** 
wfuzz -d '{"email":"a@email.com","password":"FUZZ"}' -H 'Content-Type: application/json' -z file,/usr/share/wordlists/rockyou.txt -u http://127.0.0.1:8888/identity/api/auth/login --hc 405

** The --hc option hides responses with certain response codes.  This is useful if you often receive the same status code, word length, and character count in many requests **

** the -H option lets you add a header to the request. Some API providers may respond with an HTTP 415 Unsupported Media Type error code if you don’t include the Content -Type:application/json header when sending JSON data in the request body. **

## PASSWORD SPRAYING

combining a long list of users with a short list of targeted passwords.

** The real key to password spraying is to maximize your user list. The more usernames you include, the higher your odds of compromising a user account with a bad password. **

**  Build a user list during your reconnaissance efforts or by discovering excessive data exposure vulnerabilities **

```
grep -oe "[a-zA-Z0-9._]\+@[a-zA-Z]\+.[a-zA-Z]\+" crAPI_creds.txt
```


## BASE 64 ENCODING

 If you test an authentication attempt and notice that an API is encoding to base64, it is likely making a comparison to base64-encoded credentials on the backend. This means you should adjust your fuzzing attacks to include base64 payloads using Burp Suite Intruder, which can both encode and decode base64 values.
