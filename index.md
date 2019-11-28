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

* **Step 0** Hands-on LAB Environment overview
You will find on Hands-on LAB laptop a sticker with your number id.
Each time in the Hands-on LAB you see <yourID>, please replace <yourID> with the number as writen on the sticker.

* **Step 1** Policy Exploration
    1. What is a policy?
    2. Active policy description
    3. Understanding the policy

* **Step 2** Connecting to a Source DBMS
    1. Connecting to a source DBMS
    2. Querying the source DBMS

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



