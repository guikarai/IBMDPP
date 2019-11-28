# IBM Data Privacy Passports Applied - Hands-on LABs

Seen and used in IBM Systems Technical University
* Upcoming May 25-29	TechU in Amsterdam, Netherlands

# About this Hands-on LAB
When you will complete this hands-on exploration of IBM Data Privacy Passports, you will understand:
* How is organized an IBM Data Privacy Passports Policy
* How data is enforced “on the fly”
* How data is protected then enforced
* How a TDO protects the data

# Architecture
This journey requires an existing Linux on IBM Z environment provided to you by the hands-on LAB team. From there you will be able to connect to several DBMS and IBM Data Privacy Passports appliance.
![alt-text](https://github.com/guikarai/IBMDPP/blob/master/IBM-DPP-Landscape.png?raw=true)

1. User to experience direct connection to a data source.
2. User to experience Dynamic Enforcement of the data thanks to IBM Data Privacy Passports.
3. User to experience Data Protection with Trusted Data Object thanks to IBM Data Privacy Passports.

## What is IBM Data Privacy Passports?
IBM Data Privacy Passports is a data centric audit and protection (DCAP) solution that protects and enforces appropriate use of data after it leaves the system of record, minimizing the risk of security breach, potential noncompliance and financial liability.

Data Privacy Passports is a data centric security solution that enables data to play an active role in its own protection. It lets you implement field level data protection to protect that data throughout its lifecycle.

The data protection policy is enforced from a central point of authority that allows you to have full control over your data, no matter where it goes. As a result, only authorized applications or users can obtain a view of the data, where that view can be enforced through policy. This creates data protection that spans hybrid and multi-party computing environments, including data stored in public cloud deployments.

## What is the relationship between Protected data and Enforced data?

**Protected Data:** Protected data has been encrypted to prevent unauthorized access by users who are not approved to view a given data element. A Passport Controller encrypts raw data into protected data via Trusted Data Objects before leaving the platform. This protected data can be shown in different views based on the policy rules and the user’s need to know.

**Enforced Data:**
Once a Trusted Data Object reaches an authorized user, data elements are transformed from protected data into enforced data. Enforced data has been masked or redacted to reveal only data that is authorized for a given user based on policy controls determined by the central Trust Authority.

# Included components

## Featured technologies
* [LinuxONE Crypto](https://www.ibm.com/it-infrastructure/linuxone/capabilities/secure-cloud)
* [OpenSSL](https://www.openssl.org/)
* [IBM LinuxONE](https://www.ibm.com/it-infrastructure/linuxone)
* [AWS](https://aws.amazon.com/)
* [z/OS Db2](https://www.ibm.com/analytics/db2/zos)
* [Oracle](https://docs.oracle.com/en/database/index.html)

## IBM Data Passports components and roles
There are 4 key components of Data Privacy Passports - Policy, Trust Authority, Passport Controller and Trusted Data Object. Let’s look at each one.

**Policy:** IBM Data privacy Passports focuses on data structures and data elements as the “carriers” of Trust, which means it “embeds” the security and privacy policies of an enterprise into the data in a way that enables the data to form a “data layer” agnostic of the processes that consume it. To do that the enterprise policies regarding the usage of data must be captured in a data-centric manner. The policy is the rules book describing what data need to be protected, and how, and according which enterprise business functions and their users. The policy looks likes and xml files made of sections.

**Trusted Data Object:** A Trusted Data Object contains data that is bundled and portable between multiple environments. Data consumers can freely use data from various sources while access and control is enforced through centrally controlled policy in real time. A TDO is the encrypted data element plus metadata. The data element is encrypted using a specific key (or set of keys) and all required instructions on how to process the TDO are included in the metadata.

**Trust Authority:** The Trust Authority is where the policy governing the protection and usage of the data is maintained. The Trust Authority also serves as the main key store for the Data Privacy Passports solution. The Passport Controller communicates with the Trust Authority to obtain policy information and keys. An enterprise Trust Authority requires performant, scalable, and robust cryptographic and key management services making it an ideal match for IBM Z. For the initial delivery the Trust Authority and Passport Controller will be deployed together into a single Secure Service Container running on either IBM z15™ or IBM LinuxONE™ III. Note – The sources and target DBMS are not required to run on IBM Z or LinuxONE servers. The initial delivery supports SQL data sources accessed via JDBC. 

**Passport Controller:** The Passport Controller is a data broker that provides an intercept point to work in cooperation with the Trust Authority to transform raw data into Trusted Data Objects. It also serves to enforce data protection policies. The Passport Controller gets clear data from source DBMS from then there a few options:
1. Dynamic Enforcement – In this case the Passport Controller directly enforces the data (according to the policy) coming from the source DBMS. In this case the Passport Controller intercepts the queries that would regularly be going to the source DBMS. There is no copy of the data.
2. Persisted Enforcement – In this case the Passport Controller is used to enforce data from a source DBMS and save the contents into a target DBMS. The enforcement is done entirely based on the policy. Here there will potentially be several copies of data depending on the different enforcement that needs to be applied for different applications.
3. Protection – In this case the Passport Controller protects the data (according to the policy) and stores the protected data (TDOs) into the target DBMS. Here there is a single copy of data saved as TDOs.
4. Protect and then enforce –In this case, the Passport Controller will be established as a proxy for accessing the protected table and will intercept the SQL requests and apply enforcement to the data before it is returned to the consumer. This is using a single copy of the data to provide multiple views.

# Steps

## **Step 0** Hands-on LAB Environment overview

You will find on Hands-on LAB laptop a sticker with your number id.

Each time in the Hands-on LAB you see <yourID>, please replace <yourID> with the number as writen on the sticker.

## **Step 1** Policy Exploration
    1. What is a policy?
    2. Active policy description
    3. Understanding the policy

## **Step 2** Connecting to a Source DBMS
**The statu quo - Connecting to the DBMS**
You can find below how a JDBC application use to connect to a DBMS. I took as example a connection to a Linux on IBM Z PostgreSQL.
1. User connect to an URL pointing to the target DBMS. For such connection, it is mandatory to provide valid credentials, the name of the Database, driver name, ...
2. Once connected, user can SQL query the DBMS according to the need.
3. DBMS sent back SQL query output to the user.
![alt-text](https://github.com/guikarai/IBMDPP/blob/master/statuquo.png)

**Using Apache Beeline to start a JDBC connection**

Apache Beeline a Hive client that uses JDBC to connect to HiveServer2 and many DBMS (z/OS Db2, PostgreSQL, Oracle...). You can also use Beeline to access IBM Data Privacy Passports remotely. For following, we will use essentially Beeline to connect to both DBMS and IBM Data Privacy Passports.

As any JDBC client, Beeline needs some parameters to be able to connect, and to execute commands on the connected DBMS. Commands may be SQL query. What is most of the time required is the following:
```
   -u <database url>               the JDBC URL to connect to
   -n <username>                   the username to connect as
   -p <password>                   the password to connect as
   -d <driver class>               the driver class to use
   -i <init file>                  script file for initialization
   -e <query>                      query that should be executed
```
To connect to the source DBMS, you need to know that:
* Target DBMS is postgreSQL
* Url of the DBMS: **jdbc:postgresql://10.3.58.109/userdb**
* Driver name is: **org.postgresql.Driver**
* Authorized username: **myuser**
* Password of the authorized username: **myuser**

Now it is time to use beeline to connect to the source DBMS and to display the first 10 lines of the Customer table.

**Querying the source DBMS**
Please issue the following command:
```
[root@rhl76dpp scripts]# beeline -u 'jdbc:postgresql://10.3.58.109/userdb' -n myuser -p myuser -d org.postgresql.Driver -e 'select * from customer limit 10';
```
Result will be something similar as follow:
```
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
10 rows selected (0.037 seconds)
Beeline version 1.2.1.spark2 by Apache Hive
Closing: 0: jdbc:postgresql://10.3.58.109/userdb
```

* **Step 3** Dynamic Enforcement
    4. Querying IBM Data Privacy Passports as a Data Owner (DO)
    4. Querying IBM Data Privacy Passports as a Data Administrator (DA)
    4. Querying IBM Data Privacy Passports as Application1 (App1)    

* **Step 3** Persisted Enforcement
    1. Creating a protected table for Data Administrator (DA) from the source to a PostgreSQL on Linux on IBM Z
    4. Querying the protected table of Data Administrator (DA)
    2. Creating a protected table from the source to Oracle on AWS
    4. Querying the source DBMS as a Data Owner (DO)

    4. Querying the source DBMS as Application1 (App1)   

* **Step 4** Protection, Protection the Enforcement



