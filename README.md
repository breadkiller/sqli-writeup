## Case Introduction

## SQL Injection (SQLi)
SQL injection is an attacking behavior which takes advantage of potential vulnerabilities of database and manipulates backend database using malicious SQL code to access information that should not have been displayed. SQL injection has three main types. They are in-band injection, Blind injection, and Out-of-band injection.[^1]

## SQLi Types
### In-band SQLi
The concept of in-band injection is to send carefully arranged queries to the database server, which can unexpectedly access the database and retrieve needed information. This type of injection can be  further broken down into error-based SQLi and union-based SQLi.

#### Error-based
Error-based SQLi determines the structure of the entire database by analyzing error messages from the response of system. To prevent error-based SQLi, developers can disable or apply restricted access on these error messages after the developing stage.[^2]

#### Union-based
Union-based SQLi uses the UNION operator to SELECT from multiple tables to extent the result from the original single SELECT query. These multiple results will fuse as a single HTTP response, where attackers can get their wanted information.[^2]

### Blind (inferential) SQLi
Blind SQLi is named “blind” as it usually won’t let the attackers see the content of a database in system responses directly, instead of which attackers can modify the structure of database by sending data payloads, observing the behavior of the target system.

#### Boolean-based
The Boolean blind SQLi will let the server to attempt return a different result from the current status. If the Boolean result in the query is TRUE, the database will change the content of response (return a different result), or the response will remain unchanged (when the Boolean result in the query is FALSE. This process is slower than in-band injection while it can still help attackers understand the structure of the whole database.[^3]

#### Time-based
Similarly, the time-based SQLi also focus on whether the database returns TRUE or FALSE after receiving a payload. However, the TRUE or FALSE result now depends on the responding time instead of the content changing of response. If the HTTP response is returned at once, then the result is FALSE; if it takes some time to send back the response, then the result in query is TRUE.[^3]

### Out-of-band SQLi
This injection method is used when it’s not able to launch an attack in the same channel as the database or when the responding time for the time-based blind method is not stable. An example of this type of injection is starting a DNS request based on the database server’s own command.[^2]

## Useful Links
* More Info on SQL Injection: https://www.imperva.com/learn/application-security/sql-injection-sqli/

## References
[^1]: imperva, [https://www.imperva.com/learn/application-security/sql-injection-sqli/](https://www.imperva.com/learn/application-security/sql-injection-sqli/).
[^2]: Acunetix, [https://www.acunetix.com/websitesecurity/sql-injection2/](https://www.acunetix.com/websitesecurity/sql-injection2/).
[^3]: Beagle Security, [https://beaglesecurity.com/blog/vulnerability/boolean-based-blind-sql-injection.html](https://beaglesecurity.com/blog/vulnerability/boolean-based-blind-sql-injection.html).
