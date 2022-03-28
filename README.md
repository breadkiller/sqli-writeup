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
The Employee Record Management System aimed to reduce the paper work of the company by managing all data online. The project provides two modules: User Module used to contain every users information, and Admin Module used to manage users' information. It is compiled from PHP and uses MySQL database. In this project,a vulnerability in the code of system is used by attackers to log in as an admin, modifying and managing sensitive data which should have been restricted to general users.

#### Vulnerable Code Piece
The query is constructed dynamically by concatenating a constant base query string and a user input string, the query only behaves correctly if itemName does not contain a single-quote character.
Use username "name' or '1'='1" will causes the where clause to always evaluate to true, and it returns all entries stored in the admin table which including admin name and password.
<img width="1001" alt="code" src="https://user-images.githubusercontent.com/47571034/160301404-82ec9f66-db55-4839-9872-47cb21ff70cc.png">

## How to Fix
mysqli_real_escape_string
The string to be escaped.[^2]
Characters encoded are NUL (ASCII 0), \n, \r, \, ', ", and Control-Z.

name' or '1'='1  ---- name or 1=1

Escape special characters
```
$uname = mysqli_real_escape_string($con, $_POST['username']);
$Password = mysqli_real_escape_string($con, $_POST['Password']);
```

## Prevention
### 1. Escaping All-User Input
Use the scheme provided by the DBMS to escape all-user input, which will prevent the database from parsing inputs from users as commands or conditional statements. Instead of that, the database will treat them as literal inputs. For example, a “*” from user will not be understood as “any” by the database.

### 2. Prepared Statement (Parameterized Queries)
When a prepared statement is used, the SQL statement will be pre-compiled and then treat the input as a variable, instead of waiting for the input to be a part of the statement content.[^3]
<img width="646" alt="code" src="https://user-images.githubusercontent.com/47571034/160352478-2f6a32dc-2564-419d-8de9-2502c54dbf64.png">

## Read More: SQLi Types
### 1. In-band SQLi
The concept of in-band injection is to send carefully arranged queries to the database server, which can unexpectedly access the database and retrieve needed information. This type of injection can be  further broken down into error-based SQLi and union-based SQLi.

- #### Error-based
Error-based SQLi determines the structure of the entire database by analyzing error messages from the response of system. To prevent error-based SQLi, developers can disable or apply restricted access on these error messages after the developing stage.[^4]
```
// Example Reference: https://medium.com/@hninja049/example-of-a-error-based-sql-injection-dce72530271c (by ninja hatori)
// In this case, the correct url target to send the query is http://testphp.vulnweb.com/artists.php?artist=1
// However, we can change artist number from 1 to -1 to execute an error message
// Malicious url: http://testphp.vulnweb.com/artists.php?artist=-1
```
![Error Message1](https://miro.medium.com/max/1400/0*HFIz560RYRMX9E_E)

```
// In the error message, it exposes the name of the vulnerable function.
// Continue with the query. By adding an "order by" suffix, we can examine the number of columns.
```

![Error Message2](https://miro.medium.com/max/1302/0*hSwPhIApUKRolxwF)

```
// The error message is not eliminated, which means the target table does not have 4 or more columns.
// Thus, we can try 3 this time.
```

![Error Message3](https://miro.medium.com/max/1288/0*K9dezZpwdUN5_R0R)

```
// The error message just disappears, which indicates the number of columns in this table is three.
// We can then use operation like UNION SELECT to view content from the table.
```

![Error Message4](https://miro.medium.com/max/1274/0*KjVOqNK7Dkmxe_uU)

- #### Union-based
Union-based SQLi uses the UNION operator to SELECT from multiple tables to extent the result from the original single SELECT query. These multiple results will fuse as a single HTTP response, where attackers can get their wanted information.[^4]
```
SELECT BookName, BookType
FROM Books
WHERE BookID = '999' UNION SELECT Username, Password FROM Accounts;

// In this example, the attacker attempts to launch a union-based injection to extend results from SELECT and retrieve the password.
```

### 2. Blind (inferential) SQLi
Blind SQLi is named “blind” as it usually won’t let the attackers see the content of a database in system responses directly, instead of which attackers can observe the behavior of the target system by sending data payloads.

- #### Boolean-based
The Boolean blind SQLi will let the server to attempt return a different result from the current status. If the Boolean result in the query is TRUE, the database will change the content of response (return a different result), or the response will remain unchanged (when the Boolean result in the query is FALSE. This process is slower than in-band injection while it can still help attackers understand the structure of the whole database and detect vulnerable entry points.[^5]
```
// In the first SQL query, the attacker uses a AND operator to include a statement that will always return FALSE (1=2).
SELECT name, description, date FROM books WHERE ID = 5 and 1=2

// Then in the second SQL query, the attacker changes the statement to the one that will always return TRUE (1=1).
SELECT name, description, date FROM books WHERE ID = 5 and 1=1

// If there is a vulnerability existing in the database, it is expected not to return content for the first query but to display something for the second command. 
// Thus, attackers can know the database reacts differently between their injected statements. 
// Attackers usually use this type of injection to locate vulnerability among a database, after which they will inject some true malicious operations.
```

- #### Time-based
Similarly, the time-based SQLi also focuses on how the database reacts on the received payload. However, the vulnerability detection now depends on the responding time instead of the content changing of response. After a delay, wait, or sleep is injected, if the HTTP response is returned at once, then the response is not affected by time-related operations; if it takes some time to send back the response, then we know the injected command works and there might be a vulnerability existing on the table/database[^5]

```
SELECT name FROM students WHERE id=1-SLEEP(20)
```

### 3. Out-of-band SQLi
This injection method is used when it’s not able to launch an attack in the same channel as the database or when the responding time for the time-based blind method is not stable. An example of this type of injection is starting a DNS request based on the database server’s own command.[^4]

## Useful Links
- SQL Injection Examples on w3schools: https://www.w3schools.com/sql/sql_injection.asp
- Readings on CVE-44966: https://github.com/nu11secur1ty/CVE-nu11secur1ty/tree/main/vendors/PHPGURUKUL/ANUJ%20KUMAR/Employee-Record-Management-System-SQL-Injection-Bypass-Authentication
- Readings on SQL Injection: https://www.imperva.com/learn/application-security/sql-injection-sqli/
- SQLi Play and Demo: https://www.codingame.com/playgrounds/154/sql-injection-demo/sql-injection
- More Options for SQLi Prevention: https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html

## References
[^1]: imperva, [https://www.imperva.com/learn/application-security/sql-injection-sqli/](https://www.imperva.com/learn/application-security/sql-injection-sqli/).
[^2]: CWE, [https://cwe.mitre.org/data/definitions/89.html](https://cwe.mitre.org/data/definitions/89.html)
[^3]: OWASP Cheat Sheet Series, [https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
[^4]: Acunetix, [https://www.acunetix.com/websitesecurity/sql-injection2/](https://www.acunetix.com/websitesecurity/sql-injection2/).
[^5]: Beagle Security, [https://beaglesecurity.com/blog/vulnerability/boolean-based-blind-sql-injection.html](https://beaglesecurity.com/blog/vulnerability/boolean-based-blind-sql-injection.html).
