# Protection of the data

The Passport Controller is a data broker that provides an intercept point to work in cooperation with the Trust Authority to transform raw data into Trusted Data Objects. This is the data protection activity.

A **Trusted Data Object** (or simply **TDO**) protects the data at the point of exfiltration. That way, data outside the security perimeter is still secured. To do so, IBM DPP creates a copy of a data source, and save it as a TDO. Data and the security mechanism makes the TDOs. The security of the data stays with the data. 

Can be named a TDOs:
* the whole protected table
* a protected table extracts(1-n rows)
* or protected table elements (1-n cells)

As a reminder, and to prove that a TDO secures data, you can find below a TDO sample.
```
##P1L{
"#K":"demompl.mykeyforapp1",
"#D":{"Mnnx1QXxJFCKnUt8TVhY9g==":"Long"},
"#A":"orig_cif_number"
}
```

There are two main data protection use cases:
* **Protection** – In this case the Passport Controller protects the data (according to the policy) and stores the protected data (TDOs) into the target DBMS. Here there is a single copy of data saved as TDOs.
* **Protect and then enforce** – In this case, the Passport Controller will be established as a proxy for accessing the protected table and will intercept the SQL requests and apply enforcement to the data before it is returned to the consumer. This is using a single copy of the data to provide multiple views.

Let's see in action both use cases.

## 1. Protection

### 1.1 Protection of the data
:white_check_mark: Data is protected at the exfiltration point and off the platform.

You can find below how an IBM Data Privacy Passports administrator creates a TDO, from a particular DBMS, to a target DBMS (source may be the target). I took as example a source DBMS being a Linux on IBM Z PostgreSQL, and a target DBMS being AWS Oracle.
* **(1a)** DPP Admin connects to DPP and uses DPP SQL functionalities.
* **(1b)** DPP SQL query the source DBMS according to the DPP Admin need.
* **(1c)** Source DBMS sends back SQL query output to DPP as it is (in the clear).
* **(1d)** Sent back data is stored at first in a temporary local view.
* **(1e)** The Passport Controller apply then data protection policy to the data, and saved it as a table (this is the TDO) on target DBMS.

You can find below how a JDBC application experiences an SQL Query directly to a TDO.
* **(3a)** User connect to an URL pointing to a DBMS hosting a TDO, and SQL query it. For such connection, it is mandatory to provide valid credentials, the name of the TDO, driver name, ...
* **(3b)** DBMS checks provided credentials, and send back SQL query output to user (protected data). The data is still protected. The only way to experience the data in the clear is to send back the TDO to the Passport Controller.

**Note:** IBM DPP will must use a valid credentials on target DBMS in order to be able to create the TDO table.

<picture here>

### 1.2 Direct SQL query to an existing TDO

:exclamation: Data being protected for exfiltration, the data is secured. Instructions to decrypt the data is provided in the metadata. It will help Passport Controller to find-out the appropriate and required keys from key-stores.

:exclamation: By the policy it is possible to let some column into the clear.

:computer: Let's experience what data looks like in a TDO:

```
beeline -u 'jdbc:postgresql://10.3.58.109/userdb' -n myuser -p XXXXX -d org.postgresql.Driver -e 'select * from enforced_customer_app1 limit 10;';
```
Expected output is:
```
```
:exclamation: Off-course we can't understand anything, we just query a TDO directly. What we see is security mecanism and the data together. As a TDO, data is secured no matter where it is, and no matter the security of environment the data is crossing.

There is only one way to get access to the data of the TDO in the clear, this is via IBM Data Privacy Passports.

## 2. Protect and then enforce
:white_check_mark: Data is protected at the exfiltration point and off the platform, and data is enforced at the consumption point.

You can find below how a JDBC application experiences an SQL Query to a TDO via IBM Data Privacy Passports.
(2a) User connect to an URL pointing to IBM Data Privacy Passports. For such connection, it is mandatory to provide valid credentials, the name of the TDO, driver name, ...
(2b) IBM Data Privacy Passports checks in the policy the DbViews section to find the appropriate JDBC connection profile to SQL query the TDO.
(2c) DBMS sends back SQL query output to IBM Data Privacy Passports as it is (protected data).
(2d) The Passport Controller directly enforces the data (according to the policy) coming from the source DBMS.
(2e) IBM Data Privacy Passports via the Passport Controller send back the data (enforced data) to the user.

<picture here>

Let's SQL query a TDO via IBM Data Privacy Passports with different users.

### 2.1 SQL query as a Data Owner (DO) of a TDO via IBM Data Privacy Passports:

:computer: Let's experience what data looks like in a TDO:

```
beeline -u "jdbc:hive2://10.3.58.108:10010" -n DO -p XXXXX -e "select * from AWSpostgresql.protected_customer_App1 LIMIT 10;"
```
Expected output is:
```
```

### 2.2 SQL query as a Data Administrator (DA) of a TDO via IBM Data Privacy Passports:

:computer: Let's experience what data looks like in a TDO:

```
beeline -u "jdbc:hive2://10.3.58.108:10010" -n DA -p XXXXX -e "select * from AWSpostgresql.protected_customer_App1 LIMIT 10;"
```
Expected output is:
```
```

### 2.3 SQL query as a Data Consummer (App1) of a TDO via IBM Data Privacy Passports:

:computer: Let's experience what data looks like in a TDO:

```
beeline -u "jdbc:hive2://10.3.58.108:10010" -n App1 -p XXXXX -e "select * from AWSpostgresql.protected_customer_App1 LIMIT 10;"
```
Expected output is:
```
```

### 2.4 SQL query as an unknow user (Test) of a TDO via IBM Data Privacy Passports:

:computer: Let's experience what data looks like in a TDO:

```
beeline -u "jdbc:hive2://10.3.58.108:10010" -n Test -p XXXXX -e "select * from AWSpostgresql.protected_customer_App1 LIMIT 10;"
```
Expected output is:
```
```

# 3. Conclusions

You can find below a summary of the different Protection use cases.
![alt-text](https://github.com/guikarai/IBMDPP/blob/master/Protection-matrix.png?raw=true)

This close the Data Enforcement chapter. Next chapter in this hands-on labs is [Audit trail](https://github.com/guikarai/IBMDPP/blob/master/Audit-trail.md).

