# Dynamic Enforcement with IBM Data Privacy Passports

IBM Data Privacy Passports and its Passport Controller gets clear data from source DBMS from then there it can perform Dynamic Enforcement. In this case the Passport Controller directly enforces the data (according to the policy) coming from the source DBMS. In this case the Passport Controller intercepts the queries that would regularly be going to the source DBMS. There is no copy of the data.

To test the Dynamic Enforcement capabilities of IBM Data Privacy Passports, from your Hands-on LAB machine, you will use Apache Beeline to to connect to multiple different DBMS and IBM Data Privacy Passports thanks to JDBC.

**Important note:** As of today, IBM DPP is still announced as a beta program, so the following may change in the future.

## Step 1 - The statu quo - Connecting to the DBMS

You can find below a schema explaining how a JDBC application connects to a DBMS. In this schema, the DBMS is a PostgreSQL running on Linux on IBM Z (Private Cloud).
* (1) User connect to an URL pointing to the target DBMS. For such connection, it is mandatory to provide valid credentials, the name of the Database, driver name, ...
* (2) Once connected, user can SQL query the DBMS according to the need.
* (3) DBMS sent back SQL query output to the user.

## Step 2 - Understanding the Dynamic Enforcement

## Conclusions regarding the Dynamic Enforcement
