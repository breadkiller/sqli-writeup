## Case Introduction
The vulnerability we would like to analyze is from the SQL Injection attack on PHPGURUKUL Employee Record Management System 1.2, which occurred on the 13th of December in 2021. In this attack, a vulnerability in the code of system is used by attackers to log in as an admin, modifying and managing sensitive data which should have been restricted to general users. 

This note will first provide detailed explanation on how to realize this attack based on the found vulnerability, how to fix it, and how to prevent similar potential problems, then the team will introduce more types of SQL injection with examples.

## SQL Injection (SQLi)
SQL injection is an attacking behavior which takes advantage of potential vulnerabilities of database and manipulates backend database using malicious SQL code to access information that should not have been displayed. SQL injection has three main types. They are in-band injection, Blind injection, and Out-of-band injection.[^1]

```
SELECT BookName, BookType
FROM Books
WHERE BookID = '999' UNION SELECT Username, Password FROM Accounts;

// In this example, the attacker attempts to launch a union-based injection to extend results from SELECT and retrieve the password.
```

## Our Case: SQLi Attack on Employee Record Management System

## How to Fix

## Prevention

## Read More: SQLi Types
### 1. In-band SQLi
The concept of in-band injection is to send carefully arranged queries to the database server, which can unexpectedly access the database and retrieve needed information. This type of injection can be  further broken down into error-based SQLi and union-based SQLi.

- #### Error-based
Error-based SQLi determines the structure of the entire database by analyzing error messages from the response of system. To prevent error-based SQLi, developers can disable or apply restricted access on these error messages after the developing stage.[^2]
```
// Example Reference: https://medium.com/@hninja049/example-of-a-error-based-sql-injection-dce72530271c
// In this case, the correct url target to send the query is http://testphp.vulnweb.com/artists.php?artist=1
// However, we can change artist number from 1 to -1 to execute an error message
// Malicious url: http://testphp.vulnweb.com/artists.php?artist=-1
```
![Error Message](https://miro.medium.com/max/1400/0*HFIz560RYRMX9E_E)

```
// In the error message, it exposes the number of columns (62).
```

- #### Union-based
Union-based SQLi uses the UNION operator to SELECT from multiple tables to extent the result from the original single SELECT query. These multiple results will fuse as a single HTTP response, where attackers can get their wanted information.[^2]
```
SELECT BookName, BookType
FROM Books
WHERE BookID = '999' UNION SELECT Username, Password FROM Accounts;

// In this example, the attacker attempts to launch a union-based injection to extend results from SELECT and retrieve the password.
```

### 2. Blind (inferential) SQLi
Blind SQLi is named “blind” as it usually won’t let the attackers see the content of a database in system responses directly, instead of which attackers can modify the structure of database by sending data payloads, observing the behavior of the target system.

- #### Boolean-based
The Boolean blind SQLi will let the server to attempt return a different result from the current status. If the Boolean result in the query is TRUE, the database will change the content of response (return a different result), or the response will remain unchanged (when the Boolean result in the query is FALSE. This process is slower than in-band injection while it can still help attackers understand the structure of the whole database.[^3]

- #### Time-based
Similarly, the time-based SQLi also focus on whether the database returns TRUE or FALSE after receiving a payload. However, the TRUE or FALSE result now depends on the responding time instead of the content changing of response. If the HTTP response is returned at once, then the result is FALSE; if it takes some time to send back the response, then the result in query is TRUE.[^3]

### 3. Out-of-band SQLi
This injection method is used when it’s not able to launch an attack in the same channel as the database or when the responding time for the time-based blind method is not stable. An example of this type of injection is starting a DNS request based on the database server’s own command.[^2]

## Useful Links
* More Info on SQL Injection: https://www.imperva.com/learn/application-security/sql-injection-sqli/

## References
[^1]: imperva, [https://www.imperva.com/learn/application-security/sql-injection-sqli/](https://www.imperva.com/learn/application-security/sql-injection-sqli/).
[^2]: Acunetix, [https://www.acunetix.com/websitesecurity/sql-injection2/](https://www.acunetix.com/websitesecurity/sql-injection2/).
[^3]: Beagle Security, [https://beaglesecurity.com/blog/vulnerability/boolean-based-blind-sql-injection.html](https://beaglesecurity.com/blog/vulnerability/boolean-based-blind-sql-injection.html).
