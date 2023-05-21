What it is? 
#### "Active reconnaissance is the process of interacting directly with the target primarily through the use of scanning."


During this process you will be scanning systems, enumerating open ports, and finding ports that have services using HTTP. =>  Once you have found systems hosting HTTP, you can open a web browser and investigate the web application. You could find an API being advertised to end users or you may have to dig deeper. => Finally, you can scan the web app for API-related directories.

### TOOLS USED: Nmap, OWASP Amass, gobuster, kiterunner and devTools

## I. Nmap

Nmap is a powerful tool for scanning ports, searching for vulnerabilities, enumerating services, and discovering live hosts.
#### For API discovery, you should run two Nmap scans in particular: general detection and all port.

#### Nmap General Detection: 
scan uses default scripts (-sC) and service enumeration (-sV) against a target and then saves the output in three formats for later review (-oX for XML, -oN for Nmap, -oG for greppable, or -oA for all three):

```
nmap -sC -sV [target address or network range] -oA nameofoutput
```


#### Nmap All Port Scan

The Nmap all-port scan will quickly check all 65,535 TCP ports for running services, application versions, and host operating system in use:

```
nmap -p- [target address] -oA allportscan
```

 You’ll most likely discover APIs by looking at the results related to HTTP traffic and other indications of web servers. Typically, you’ll find these running on ports 80 and 443, but an API can be hosted on all sorts of different ports. Once you discover a web server, you can perform HTTP enumeration using a Nmap NSE script (use -p to specify which ports you'd like to test).

```
nmap -sV --script=http-enum <target> -p 80,443,8000,8080
```


## II. Amass

OWASP Amass is a command-line tool that can map a target’s external network by collecting OSINT from over 55 different sources.

You can set it to perform passive or active scans.

####  If you choose the active option, Amass will collect information directly from the target by requesting its certificate information.

#### If you choose passive option, it collects data from search engines (such as Google, Bing, and HackerOne), SSL certificate sources (such as GoogleCT, Censys, and FacebookCT), search APIs (such as Shodan, AlienVault, Cloudflare, and GitHub), and the web archive Wayback.


First, we can see which data sources are available for Amass (paid and free) by running:  
```

$ amass enum -list
```

Next, we will need to create a config file to add our API keys to.

`$ sudo curl https://raw.githubusercontent.com/OWASP/Amass/master/examples/config.ini >~/.config/amass/config.ini`

Now we can update the config.ini

Link to Censys:  [https://censys.io/register](https://censys.io/register)

Once you have obtained your API ID and Secret, edit the config.ini file and add the credentials to the file.

```
sudo nano ~/.config/amass/config.ini
```

Now use Amass for api recon.

```
amass enum -active -d target-name.com |grep api
```

This scan could reveal many unique API subdomains, including legacy-api.target-name.com. An API endpoint named legacy could be of particular interest because it seems to indicate an improper asset management vulnerability.

#### Amass command line options:

A.  Use the intel command to collect SSL certificates, search reverse Whois records, and find ASN IDs associated with your target.

```
amass intel -addr [target IP addresses]
```

If this scan is successful then it will provide you with domain names. 
These domains can then be passed to intel with the whois option to perform a reverse Whois lookup:

```
amass intel -d [target domain] –whois
```

This could give you a ton of results. Focus on the interesting results that relate to your target organization. Once you have a list of interesting domains, upgrade to the enum subcommand to begin enumerating subdomains. If you specify the **-**passive option, Amass will refrain from directly interacting with your target:

```
 amass enum -passive -d [target domain]
```

The active enum scan will perform much of the same scan as the passive one, but it will add domain name resolution, attempt DNS zone transfers, and grab SSL certificate information:

```
amass enum -active -d [target domain]
```

To up your game, add the -brute option to brute-force subdomains, -w to specify the API_superlist wordlist, and then the -dir option to send the output to the directory of your choice:

```
amass enum -active -brute -w /usr/share/wordlists/API_superlist -d [target domain] -dir [directory name]
```


### III. GoBuster

Gobuster can be used to brute-force URIs and DNS subdomains from the command line.

 In Gobuster, you can use wordlists for common directories and subdomains to automatically request every item in the wordlist and send them to a web server and filter the interesting server responses. The results generated from Gobuster will provide you with the URL path and the HTTP status response codes.

Once you find API directories like the /api directory shown in this output, either by crawling or brute force, you can use Burp to investigate them further. Gobuster has additional options, and you can list them using the -h option:

$ gobuster dir -h

If you would like to ignore certain response status codes, use the option -b. If you would like to see additional status codes, use -x. You could enhance a Gobuster search with the following:

```
gobuster dir -u

://targetaddress/ -w /usr/share/wordlists/api_list/common_apis_160 -x 200,202,301 -b 302
```


## IV Kiterunner

Kiterunner is an excellent tool that was developed and released by Assetnote.

#### While directory brute force tools like Gobuster/Dirbuster/ work to discover URL paths, it typically relies on standard HTTP GET requests. Kiterunner will not only use all HTTP request methods common with APIs (GET, POST, PUT, and DELETE) but also mimic common API path structures.

In other words, instead of requesting GET /api/v1/user/create, Kiterunner will try POST /api/v1/user/create, mimicking a more realistic request.

You can perform a quick scan of your target’s URL or IP address like this:

```
kr scan HTTP://127.0.0.1 -w ~/api/wordlists/data/kiterunner/routes-large.kite
```

If you want to use a text wordlist rather than a .kite file, use the brute option with the text file of your choice:

```
kr brute <target> -w ~/api/wordlists/data/automated/nameofwordlist.txt
```

If you have many targets, you can save a list of line-separated targets as a text file and use that file as the target.

#### One of the coolest Kiterunner features is the ability to replay requests.

#### Thus, not only will you have an interesting result to investigate, you will also be able to dissect exactly why that request is interesting. In order to replay a request, copy the entire line of content into Kiterunner, paste it using the kb replay option, and include the wordlist you used:

```
 kr kb replay "GET     414 [    183,    7,   8]

://192.168.50.35:8888/api/privatisations/count 0cf6841b1e7ac8badc6e237ab300a90ca873d571" -w

~/api/wordlists/data/kiterunner/routes-large.kite
```


#### Running this will replay the request and provide you with the HTTP response. You can then review the contents to see if there is anything worthy of investigation. I normally review interesting results and then pivot to testing them using Postman and Burp Suite.


## V. DevTools

DevTools contains some highly underrated web application hacking tools. 

#### (A) You can use the filter tool to search for any term you would like, such as "API", "v1", or "graphql" => This is a quick way to find API endpoints in use

#### (B) You can also leave the Devtools Network tab open while you perform actions on the web page.

#### (C) Edit and Resend : From network tab , right click on a request and select `Edit and Resend` option. This will allow you to check out the request in the browser, edit the headers/request body, and send it back to the API provider.

#### (D) You can use DevTools to migrate individual requests over to Postman using cURL.

