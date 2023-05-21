Passive API Racon is the act of obtaining information about a target without directly interacting with the target's system. *** Your goal is to find and document public information about your target's attack surface. ***

passive reconnaissance leverages open-source intelligence (OSINT), which is data collected from publicly available sources.

Hunt for API endpoints,  exposed credentials, version information, API documentation, and information about the API’s business purpose.

Any discovered API endpoints will become your targets later, during active reconnaissance.

Credential-related information will help you test as an authenticated user or, better, as an administrator.

Version information will help inform you about potential improper assets and other past vulnerabilities.

API documentation will tell you exactly how to test the target API.

Discovering the API’s business purpose can provide you with insight into potential business logic flaws

Critical Data Exposure => API keys, credentials, JSON Web Tokens (JWT)

Other high-risk findings would include leaked ** PII ** or sensitive user data such as SSN, full names, email addresses, and credit card information.

These sorts of findings should be documented and reported immediately because they present a valid ** critical weakness. **

## PASSIVE RECON. TECHNIQUES

### GOOGLE DORKING

Google Dorking Query:

(i) inurl:"/wp-json/wp/v2/users" => Finds all publicly available WordPress API user directories

(ii) intitle:"index.of" intext:"api.txt" => Finds publicly available API key files.

(iii) inurl:"/api/v1" intext:"index of /" => Finds potentially interesting API directories.

(iv) ext:php inurl:"api.php?action=" => Finds all sites with a XenAPI SQL injection vulnerability. (This query was posted in 2016; four years later, there are currently 141,000 results.)

(v) intitle:"index of" api_key OR "api key" OR apiKey -pool => This is one of my favorite queries. It lists potentially exposed API keys.



### GITDORKING

(i) filename:swagger.json

(ii) extension:.json

Methods:

Begin by searching GitHub for your target organization’s name paired with potentially sensitive types of information, such as “api key,” "api keys", "apikey", "authorization: Bearer", "access_token", "secret", or “token.”

** Then investigate the various GitHub repository tabs to discover API endpoints and potential weaknesses. **

Analyze the source code in the Code tab, find bugs in the Issues tab, and review proposed changes in the Pull requests tab. => Code contains the current source code, readme files, and other files. This tab will provide you with the name of the last developer who committed to the given file, when that commit happened, contributors, and the actual source code.  => Using the Code tab, you can review the code in its current form or use ctrl-F to search for terms that may interest you (such as API, key, and secret).


####  view historical commits to the code by using the History button found near the top-right corner of the above image.  If you came across an issue or comment that led you to believe there were once vulnerabilities associated with the code, you can look for historical commits to see if the vulnerabilities are still viewable.

### The Pull requests tab is a place that allows developers to collaborate on changes to the code. If you review these proposed changes, you might sometimes get lucky and find an API exposure that is in the process of being resolved.

### As this change has not yet been merged with the code, we can easily see that the API key is still exposed under the Files changed tab. The Files changed tab reveals the section of code the developer is attempting to change.

## If you donot find any weakness in a GITHUB repository => use it instead to develop the profile of your target. => Take notes of programming languages in use => API endpoint information & usages information 


### TruffleHog



It is a tool for automatically discovering exposed secrets.

Using truffleHog docker we can run trufflehog scan for our target's github:

```
$ sudo docker run -it -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --org=target-name
```

### Explanation:  the org that was targeted was Venmo and the results of the scan indicate URLs that should be investigated for potentially leaked secrets. In addition to searching Github, TruffleHog can also be used to search for secrets in other sources like Git, Gitlab, Amazon S3, filesystem, and Syslog.


Link: https://github.com/trufflesecurity/trufflehog

## API DIRECTORIES

Programmableweb.com is a go-to source for API-related information.
Link: https://www.programmableweb.com/apis/directory

(i) To learn about APIs, you can use their API University.
(ii) To gather information about your target, use the API Directory, a searchable database of over 23,000 APIs.
(iii) Expect to find API endpoints, version information, business logic information, the status of the API, source code, SDKs, articles, API documentation, and a changelog.


## SHODAN

Shodan is the go-to search engine for devices accessible from the internet. 

What it does? 
### Shodan regularly scans the entire IPv4 address space for systems with open ports and makes their collected information public on https://shodan.io.

(i) You can use Shodan to discover external-facing APIs and get information about your target’s open ports, making it useful if you have only an IP address or organization’s name to work from.

(ii) you can search Shodan casually by entering your target’s domain name or IP addresses; alternatively, you can use search parameters like you would when writing Google queries.

### SHODAN QUERIES:

(i) hostname:"targetname.com" => Using hostname will perform a basic Shodan search for your target’s domain name. This should be combined with the following queries to get results specific to your target.



(ii) "content-type: application/json" => APIs should have their content-type set to JSON or XML. This query will filter results that respond with JSON.

(iii) "content-type: application/xml" => This query will filter results that respond with XML.

(iv) "200 OK" => You can add "200 OK" to your search queries to get results that have had successful requests.  However, if an API does not accept the format of Shodan’s request, it will likely issue a 300 or 400 response.

(v) "wp-json" => This will search for web applications using the WordPress API.


### WAYBACK MACHINE

The Wayback Machine is an archive of various web pages over time.
This is great for passive API reconnaissance because this allows you to check out historical changes to your target.

#### How it could be useful?
(a) To see changes to existing API documentation.
(b) Possibilities of finding *** Zombie APIs *** . 
ZOMBIE APIs : "If the API has not been managed well over time, then there is a chance that you could find retired endpoints that still exist even though the API provider believes them to be retired."
###  Zombie APIs fall under the Improper Assets Management vulnerability on the OWASP API Security Top 10 list.

### Finding and comparing historical snapshots of API documentation can simplify testing for Improper Assets Management.








