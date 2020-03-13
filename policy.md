# Policy

With IBM Data Privacy Passports, the security and the privacy of the data is dicted by a policy.
This is very important to understand what is discribed in the policy, in order to protect accuratetly data according to users, and their needs to know.

Let's see first what a policy is, and second what is the policy in action in this hands-on lab environment.

# Policy content, structure and blokcs

IBM Data Privacy Passports focuses on data structures and data elements as the “carriers” of Trust, which means it “embeds” the security and privacy policies of an enterprise into the data in a way that enables the data to form a “data layer” agnostic of the processes that consume it. To do that the enterprise policies regarding the usage of data must be captured in a data-centric manner. The policy is the rules book describing what data need to be protected, and how, and according which enterprise business functions and their users. The policy looks likes and xml files made of sections.


You can find below an high level orverview of how a policy is organized:
![alt-text](https://github.com/guikarai/IBMDPP/blob/master/policy-block-structure.png?raw=true)

There are several important sections/blocks in a policy:

* **"Definition structure block"** It helps to define the name of the organization, the validity date of the policy, and its status (Prod, Test, Debug...).

* **"Need to know" data element collection"** These are a set of sections that defined what data need to be protected/enforced by a defaut policy, and what expectional rules need to be applied for selected users and business functions.

* **"Enterprise Functions"** This is a data element collection, to define consistant view of Business Functions, and the relationship with the users.

* **"dbViews"** This is data element collection to define how the policy can access to data from external sources (eg. z/OS Db2, AWS Oracle, AWS PostgreSQL, LoZ PostgreSQL, Db2 on Power). This data element collection is key, because it will be use to Enforced data in flight, and to read Trusted Data Objects from the source. Source data remain in the clear and clients connect to IBM Data Privacy Passports as a proxy that will enforce data for them.
* **"Configuration block"** It is about a set of default settings, key stores, ciphers to be used in the policy.


# Policy status and content

Now it is time for some hands-on. The whole hands-on lab architecture is based on a docker container infrastructure. Each lab practitioner have his own docker container. The docker container was designed to be used to query and to access IBM Data Privacy Passports services.

Please ask to the hands-on lab instructor about **your port number** and the **password** to be used.

:computer: On Putty, please issue the command below to connect to your lab environment running in a docker container:

```
ssh -p <your port here> root@10.3.58.109
```
    
:computer: Now, let's position in /. Please issue the following command:
```
cd /
```

IBM Data Privacy Passports exposes services via APIs. One of the API can display the content of the policy in use.

:computer: Issue the command shown below to query as an IBM Data Privacy Passports Administrator the policy description:
```
beeline -u "jdbc:hive2://10.3.58.108:10010" -n DAUser -p XXXXX -e "@dp describe policy";
```
Expected output is:

```
Connecting to jdbc:hive2://10.3.58.108:10010
Connected to: Spark SQL (version 2.2.2)
Driver: Hive JDBC (version 1.2.1)
Transaction isolation: TRANSACTION_REPEATABLE_READ
+----------------------------------------------------------------------------------------+--+
|                                         value                                          |
+----------------------------------------------------------------------------------------+--+
| <DPPolicy>                                                                             |
|   <org>SC1</org>                                                                       |
|   <startDate>2019-12-01T00:00:00.000</startDate>                                       |
|   <endDate>2022-12-01T00:00:00.000</endDate>                                           |
|   <status>Valid</status>                                                               |
|   <environment>Demo</environment>                                                      |
|   <mode>Debug</mode>                                                                   |
|   <needToKnow>                                                                         |
|     <dataElements>                                                                     |
|       <dataElement>                                                                    |
|         <match>orig_cif_number</match>                                                 |
|         <defaultScanner>Naive</defaultScanner>                                         |
|         <defaultProtection>                                                            |
|           <method>Local</method>                                                       |
|           <result>Replace</result>                                                     |
|           <newDataElement>$_DP</newDataElement>                                        |
|           <metadata></metadata>                                                        |
|           <parameters>                                                                 |
|             <keysetname>'demompl'</keysetname>                                         |
|             <keyname>'mykeyforapp1'</keyname>                                          |
|           </parameters>                                                                |
|         </defaultProtection>                                                           |
|         <defaultEnforcement>                                                           |
|           <method>Mask</method>                                                        |
|           <degrade>AcceptPassport</degrade>                                            |
|           <regex>                                                                      |
|             <pattern>.</pattern>                                                       |
|             <replace>X</replace>                                                       |
|           </regex>                                                                     |
|         </defaultEnforcement>                                                          |
|         <consent>false</consent>                                                       |
|         <enterpriseFunctions>                                                          |
|           <enterpriseFunction>                                                         |
|             <targetID>DO</targetID>                                                    |
|             <enforcement>                                                              |
|               <method>ClearValue</method>                                              |
|               <degrade>AcceptPassport</degrade>                                        |
|             </enforcement>                                                             |
|           </enterpriseFunction>                                                        |
|         </enterpriseFunctions>                                                         |
|       </dataElement>                                                                   |
|       <dataElement>                                                                    |
|         <match>gender</match>                                                          |
|         <defaultScanner>Naive</defaultScanner>                                         |
|         <defaultProtection>                                                            |
|           <method>Local</method>                                                       |
|           <result>Replace</result>                                                     |
|           <newDataElement>$_DP</newDataElement>                                        |
|           <metadata></metadata>                                                        |
|           <parameters>                                                                 |
|             <keysetname>'demompl'</keysetname>                                         |
|             <keyname>'mykeyforapp1'</keyname>                                          |
|           </parameters>                                                                |
|         </defaultProtection>                                                           |
|         <defaultEnforcement>                                                           |
|           <method>ClearValue</method>                                                  |
|           <degrade>AcceptPassport</degrade>                                            |
|         </defaultEnforcement>                                                          |
|         <consent>false</consent>                                                       |
|         <enterpriseFunctions>                                                          |
|           <enterpriseFunction>                                                         |
|             <targetID>DA</targetID>                                                    |
|             <enforcement>                                                              |
|               <method>Mask</method>                                                    |
|               <degrade>AcceptPassport</degrade>                                        |
|               <regex>                                                                  |
|                 <pattern>Male</pattern>                                                |
|                 <replace>XXXXX</replace>                                               |
|               </regex>                                                                 |
|               <regex>                                                                  |
|                 <pattern>Female</pattern>                                              |
|                 <replace>XXXXX</replace>                                               |
|               </regex>                                                                 |
|             </enforcement>                                                             |
|           </enterpriseFunction>                                                        |
|         </enterpriseFunctions>                                                         |
|       </dataElement>                                                                   |
|       <dataElement>                                                                    |
|         <match>first_name</match>                                                      |
|         <defaultScanner>Naive</defaultScanner>                                         |
|         <defaultProtection>                                                            |
|           <method>Local</method>                                                       |
|           <result>Replace</result>                                                     |
|           <newDataElement>$_DP</newDataElement>                                        |
|           <metadata></metadata>                                                        |
|           <parameters>                                                                 |
|             <keysetname>'demompl'</keysetname>                                         |
|             <keyname>'mykeyforapp1'</keyname>                                          |
|           </parameters>                                                                |
|         </defaultProtection>                                                           |
|         <defaultEnforcement>                                                           |
|           <method>Mask</method>                                                        |
|           <degrade>AcceptPassport</degrade>                                            |
|           <regex>                                                                      |
|             <pattern>^.*$</pattern>                                                    |
|             <replace>XXXXX</replace>                                                   |
|           </regex>                                                                     |
|         </defaultEnforcement>                                                          |
|         <consent>false</consent>                                                       |
|         <enterpriseFunctions>                                                          |
+----------------------------------------------------------------------------------------+--+
|                                       TRUNCATED ON PURPOSE                             |
+----------------------------------------------------------------------------------------+--+

629 rows selected (0.246 seconds)
Beeline version 1.2.1 by Apache Hive
Closing: 0: jdbc:hive2://10.3.58.108:10010
```

# Next steps

This close the Policy chapter. Next chapter in this hands-on labs is [Enforcement of the data](https://github.com/guikarai/IBMDPP/blob/master/enforcement.md).
