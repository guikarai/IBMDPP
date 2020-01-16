# 1. Dynamic Enforcement with IBM Data Privacy Passports

IBM Data Privacy Passports and its Passport Controller gets clear data from source DBMS from then there it can perform Dynamic Enforcement. In this case the Passport Controller directly enforces the data (according to the policy) coming from the source DBMS. In this case the Passport Controller intercepts the queries that would regularly be going to the source DBMS. There is no copy of the data.

To test the Dynamic Enforcement capabilities of IBM Data Privacy Passports, from your Hands-on LAB machine, you will use Apache Beeline to to connect to multiple different DBMS and IBM Data Privacy Passports thanks to JDBC.

**Important note:** As of today, IBM DPP is still announced as a beta program, so the following may change in the future.

## 1.1 The statu quo - Connecting to the DBMS

You can find below a schema explaining how a JDBC application connects to a DBMS. In this schema, the DBMS is a PostgreSQL running on Linux on IBM Z (Private Cloud).
* **(1)** User connect to an URL pointing to the target DBMS. For such connection, it is mandatory to provide valid credentials, the name of the Database, driver name, ...
* **(2)** Once connected, user can SQL query the DBMS according to the need.
* **(3)** DBMS sent back SQL query output to the user.

<picture here>
  
:computer: Issue the command shown below to query the source DBMS.
```
beeline -u 'jdbc:postgresql://10.3.58.109/userdb' -n myuser -p XXXXX -d org.postgresql.Driver -e 'select * from customer limit 10;';
Connecting to jdbc:postgresql://10.3.58.109/userdb
Connected to: PostgreSQL (version 10.9 (Ubuntu 10.9-0ubuntu0.18.04.1))
Driver: PostgreSQL JDBC Driver (version 42.2.5)
Transaction isolation: TRANSACTION_REPEATABLE_READ
+------------------+---------+-------------+------------+------+------------+-----------------------------+----------------------+-------------------------------------------------+------------+--------------+--+
| orig_cif_number  | gender  | first_name  | last_name  | age  |    sin     |            email            |        phone         |                 mailing_address                 | prov_abbr  | postal_code  |
+------------------+---------+-------------+------------+------+------------+-----------------------------+----------------------+-------------------------------------------------+------------+--------------+--+
| 1000016268       | Female  | Madeline    | Campbell   | 39   | 346108285  | carrolljoseph@hill.com      | 1-394-776-2104       | 0649 Ann Greens, Edwardton                      | AB         | R9V7G8       |
| 1000012605       | Female  | Lynn        | Scott      | 27   | 978982456  | janet64@hotmail.com         | (749) 623-9781       | 9800 Tina Crescent, North Alexandraton          | SK         | R2A5R8       |
| 1000012979       | Female  | Lauren      | Arroyo     | 44   | 809852991  | scottwilson@foster.info     | 1-818-222-2486       | 065 Russell Dam Apt. 235, Newtonstad            | MB         | V6T 3Y3      |
| 1000001690       | Male    | Anthony     | Mendoza    | 65   | 210726574  | cgarcia@simpson-howard.com  | 589-767-7808         | 1803 Roberson Spur, New Krista                  | AB         | L4B1A4       |
| 1000011265       | Female  | Kimberly    | Quinn      | 62   | 262697551  | hendersonjames@price.biz    | 670 392 0575         | 638 Crystal Track, Port Stacy                   | AB         | P7B1C7       |
| 1000014413       | Male    | Michael     | Carter     | 57   | 467383604  | ryan84@hotmail.com          | (373) 822-1521       | 48321 Bailey Glens, Port Carlos                 | NS         | Y3M6L7       |
| 1000001944       | Male    | Allen       | Hernandez  | 30   | 908758262  | thomas70@ferguson.com       | (539) 376-1754 x502  | 467 Holland Forest Suite 332, Thompsonville     | NV         | Y3L 9E2      |
| 1000002939       | Female  | Alyssa      | Dunn       | 51   | 834736228  | cheryl06@hotmail.com        | (434) 546-5030 x237  | 0819 Mathew Inlet Suite 107, Lauraview          | YT         | K9J 4V6      |
| 1000004030       | Female  | Courtney    | Castro     | 53   | 320706455  | nathan12@gmail.com          | 553.894.8235         | 9403 Amanda Mission Suite 037, South Johnmouth  | NS         | T1M7N3       |
| 1000004906       | Male    | Derek       | White      | 34   | 306868178  | youngtracy@garrett.net      | (974) 406-4141 x724  | 1099 Melanie Village, West Meganshire           | NL         | Y6C5T1       |
+------------------+---------+-------------+------------+------+------------+-----------------------------+----------------------+-------------------------------------------------+------------+--------------+--+
```

Let's explain a litle bit what has just been done. 
You just SQL query:
* a **PostgreSQL** DBMS accessible on the url: 10.3.58.109
* the target database name is: **userdb**
* with the DBMS known user **myuser** and its password **XXXXX**
* used JDBC driver is: **org.postgresql.Driver**
* the issued SQL Query is: **select * from customer limit 10;**

:question: As a known an allowed user on DBMS, you are able from the source customer table to see content in clear.
For the same SQL query, but with different users, as long as users have the read rights on table, they will see the same data. There is no difference according the user.

:exclamation: Let's see how IBM Data Privacy Passports can help to change the experience at the consumption point

## 1.2 Understanding the Dynamic Enforcement

## 1.2 Conclusions regarding the Dynamic Enforcement
