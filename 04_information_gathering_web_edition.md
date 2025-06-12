# [Information Gathering - Web Edition](https://academy.hackthebox.com/module/details/144)

## Skills Assessment

To complete the skills assessment, answer the questions below. You will need to apply a variety of skills learned in this module, including:

- Using `whois`
- Analysing `robots.txt`
- Performing subdomain bruteforcing
- Crawling and analysing results

Demonstrate your proficiency by effectively utilizing these techniques. Remember to add subdomains to your `hosts` file as you discover them.

### Questions

#### Question #01

**Question**

What is the IANA ID of the registrar of the `inlanefreight.com` domain?

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ whois inlanefreight.com

[SNIP]

Domain Name: inlanefreight.com
Registry Domain ID: 2420436757_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.registrar.amazon
Registrar URL: https://registrar.amazon.com
Updated Date: 2024-07-02T22:07:11Z
Creation Date: 2019-08-05T22:43:09Z
Registrar Registration Expiration Date: 2025-08-05T22:43:09Z
Registrar: Amazon Registrar, Inc.
Registrar IANA ID: 468
Registrar Abuse Contact Email: trustandsafety@support.aws.com
Registrar Abuse Contact Phone: +1.2024422253

[SNIP]
```

**Answer**

```
468
```

#### Question #02

What http server software is powering the `inlanefreight.htb` site on the target system? Respond with the name of the software, not the version, e.g., Apache.

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ echo -e '94.237.123.50\tinlanefreight.htb' | sudo tee -a /etc/hosts

94.237.123.50	inlanefreight.htb
```

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ whatweb http://inlanefreight.htb:40825/

http://inlanefreight.htb:40825/ [200 OK] Country[FINLAND][FI], HTML5, HTTPServer[nginx/1.26.1], IP[94.237.123.50], Title[inlanefreight], nginx[1.26.1]
```

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ curl -I http://inlanefreight.htb:40825/

HTTP/1.1 200 OK
Server: nginx/1.26.1
Date: Fri, 23 May 2025 07:07:29 GMT
Content-Type: text/html
Content-Length: 120
Last-Modified: Thu, 01 Aug 2024 09:35:23 GMT
Connection: keep-alive
ETag: "66ab56db-78"
Accept-Ranges: bytes
```

**Answer**

```
nginx
```

#### Question #03

 What is the API key in the hidden admin directory that you have discovered on the target system?

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ gobuster vhost -u http://inlanefreight.htb:40825 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain -t 10

[SNIP]

Found: web1337.inlanefreight.htb:40825 Status: 200 [Size: 104]

[SNIP]
```

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ echo -e '94.237.123.50\tweb1337.inlanefreight.htb' | sudo tee -a /etc/hosts

94.237.123.50	web1337.inlanefreight.htb
```

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ curl http://web1337.inlanefreight.htb:40825/robots.txt

User-agent: *
Allow: /index.html
Allow: /index-2.html
Allow: /index-3.html
Disallow: /admin_h1dd3n
```

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://web1337.inlanefreight.htb:40825/indexFUZZ

[SNIP]

.html                   [Status: 200, Size: 104, Words: 4, Lines: 2, Duration: 16ms]

[SNIP]
```

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://web1337.inlanefreight.htb:40825/admin_h1dd3n/FUZZ.html -ic

[SNIP]

index                   [Status: 200, Size: 255, Words: 23, Lines: 2, Duration: 16ms]

[SNIP]
```

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ curl http://web1337.inlanefreight.htb:40825/admin_h1dd3n/index.html

<!DOCTYPE html><html><head><title>web1337 admin</title></head><body><h1>Welcome to web1337 admin site</h1><h2>The admin panel is currently under maintenance, but the API is still accessible with the key e963d863ee0e82ba7080fbf558ca0d3f</h2></body></html>
```

**Answer**

```
e963d863ee0e82ba7080fbf558ca0d3f
```

#### Question #04

After crawling the `inlanefreight.htb` domain on the target system, what is the email address you have found? Respond with the full email, e.g., `mail@inlanefreight.htb`.

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ gobuster vhost -u http://web1337.inlanefreight.htb:40825 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain

[SNIP]

Found: dev.web1337.inlanefreight.htb:40825 Status: 200 [Size: 123]

[SNIP]
```

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ echo -e '94.237.123.50\tdev.web1337.inlanefreight.htb' | sudo tee -a /etc/hosts

94.237.123.50	dev.web1337.inlanefreight.htb
```

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:40825/

[SNIP]

2025-05-23 02:23:08 [scrapy.core.engine] INFO: Spider closed (finished)
```

```
┌─[eu-academy-1]─[10.10.14.99]─[htb-ac-1461567@htb-zdik7x4vou]─[~]
└──╼ [★]$ cat results.json | jq

{
  "emails": [
    "1337testing@inlanefreight.htb"
  ],
  "links": [

[SNIP]

  ],
  "external_files": [],
  "js_files": [],
  "form_fields": [],
  "images": [],
  "videos": [],
  "audio": [],
  "comments": [
    "<!-- Remember to change the API key to ba988b835be4aa97d068941dc852ff33 -->"
  ]
}
```

**Answer**

```
1337testing@inlanefreight.htb
```

#### Question #05

What is the API key the `inlanefreight.htb` developers will be changing too?

**Answer**

```
ba988b835be4aa97d068941dc852ff33
```

---
---
