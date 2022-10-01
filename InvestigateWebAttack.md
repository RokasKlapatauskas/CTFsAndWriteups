# Investigate Web Attack

Write up for "Investigate Web Attack" challenge on letsdefend.io

Challenge can be [found here](https://app.letsdefend.io/challenge/investigate-web-attack/)
if you want to have a go yourself.

Before starting this challenge there is a [prerequisite log file (pass = infected)](https://app.letsdefend.io/download/downloadfile/WebLog.zip) that must be downloaded. 

This file only contains HTTP logs so it is safe to download onto your main machine, unlike some other challenges on letsdefend, which contain live malware and should only be downloaded onto an isolated enviroment.

Open up the logs in any text editor / software you wish and we can begin with the first question.

## Question 1
```
Which automated scan tool did attacker use for web reconnansiance?
```
<img src="https://user-images.githubusercontent.com/106356256/193424515-5e2677d5-c6f5-4e9f-8302-836b5b0486dd.png" style="height: auto; width:auto;"/>

Initally, we see some fairly routine GET requests. The time spacing between each requests is fairly fast, but not unreasonable, furthermore, they are most likely linked to a single page e.g. facebook, twitter, linkedin logos so the time shouldn't really be considered. Lastly, the UserAgent section of the request indicates a normal system `rgb(9, 105, 218)` `"Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:89.0) Gecko/20100101 Firefox/89.0"`  we should also take note of the ip address for threat intelligence purposes, as it is likely we will have many other connections in these logs and should therefore link malicious activity to specific actors to  filter their traffic in the future.

<img src="[Screenshot_2.png](https://user-images.githubusercontent.com/106356256/193424581-60c0a18d-5287-4160-a184-f6b9b41c51d2.png)" style="height: auto; width:auto;"/>

For the majority of the rest of the log file, we see some extremely suspicious activity. We see hundreds of similar requests per second amounting to around 8000 requests in 30 seconds. This is clearly impossible to do by hand and therefore must be the result of some automated tool, as suggested in the question. The UserAgent field gives us the answer to our question ``` "Mozilla/5.00 (Nikto/2.1.6) (Evasions:None) (Test:006922)"``` 

```
Nikto is an Open Source (GPL) web server scanner which performs comprehensive tests against web servers for multiple items, 
including over 6700 potentially dangerous files/programs, checks for outdated versions of over 1250 servers,
and version specific problems on over 270 servers. 
It also checks for server configuration items such as the presence of multiple index files, HTTP server options, 
and will attempt to identify installed web servers and software. 
Scan items and plugins are frequently updated and can be automatically updated.

Nikto is not designed as a stealthy tool. It will test a web server in the quickest time possible,
and is obvious in log files or to an IPS/IDS. 
However, there is support for LibWhisker's anti-IDS methods in case you want to give it a try.
```
## Question 2
```

After web reconnansiance activity, which technique did attacker use for directory listing discovery?
```
<img src="https://user-images.githubusercontent.com/106356256/193424590-40a33d6c-a78f-44aa-915f-58cb963fec0a.png" style="height: auto; width:auto;"/>

After sifting through all of the Nikto tests, we see more of the same. Automated sequential searching, with a ridiculous amount of requests per second. This kind of trial and error tactic is known as brute forcing, and can be used to attack many things, such as passwords, if the correct measures aren't in place to stop it. In this case, the attacker is brute forcing the directories of the web server, which gives us the answer to question 2.

## Question 3
```
What is the third attack type after directory listing discovery?
```
<img src="https://user-images.githubusercontent.com/106356256/193424596-bf38ee0c-8d88-4a56-be06-bd77f83ab90b.png" style="height: auto; width:auto;"/>

Again we have to sift through all of the directory brute forcing requests, before we arrive at the third attack. Again, we see more of the same. Repetitive POST requests to login.php every few seconds. As previously mentioned, this is another form of Brute forcing. Given that login.php is being attacked it is safe to assume that the attacker is attempting to find a weak username/password that he can log in to the website with.

## Question 4
```
Is the third attack success?
```
<img src="https://user-images.githubusercontent.com/106356256/193424609-96163187-38ce-4442-84fc-2650938e415b.png" style="height: auto; width:auto;"/>

Upon viewing the end of the login.php logs, we unfortunately see evidence the attack was successful. We see a POST request with a 302 response code, this indicates that a redirect will occur, following this we see a GET request to portal.php. This is enough evidence to assume that the attacker successfully logged in and was then redirected to the home page, as is common on many websites.

## Question 5 & 6
```
What is the name of fourth atttack?
```
<img src="https://user-images.githubusercontent.com/106356256/193424629-9f570409-d30e-44db-a30a-e9bdad54e325.png" style="height: auto; width:auto;"/>

Nearing the end of the log file, we find evidence of the fourth attack and the answers our questions 5 & 6. 

The attacker has discovered a vulnerability in the php of the website. This vulnerability allows him to inject code that is then executed. There are many varieties of code injection and all are potentially devestating. SQL injection and XSS are both famous attacks and are classified as types of code injection. 

His first payload ```whoami``` is used to check if the exploit is succesful and also to get additional information on how he can procede. 

For example, if whoami returned information that the user was a privilaged admin account it would be much easier for him to achieve his objectives and wouldn't require him to do any privilage escalation. 

## Question 7
```
Is there any persistency clue for the victim machine in the log file ? If yes, what is the related payload?
```
<img src="https://user-images.githubusercontent.com/106356256/193424669-c595704b-6c89-468b-a4a5-ed2873b5dc77.png" style="height: auto; width:auto;"/>

For the final question, we are asked if there is any evidence for persistence. The final two logs show us just that. The payload is URL encoded so we will decode it for easier reading.
The payload is as follows:

```'net user hacker Asd123!! /add'```

The attacker has made an account on the system with the username: hacker and password: Asd123!

This is clearly evidence of persistence, as the attacker can now simply SSH to the account at any time and wreak havoc.

