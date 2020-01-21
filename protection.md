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

## 1. Data Protection with IBM Data Privacy Passports

### 1.1 Protection of the data
:white_check_mark: **Objective:** Data is protected at the exfiltration point and off the platform.

You can find below how an IBM Data Privacy Passports administrator creates a TDO, from a particular DBMS, to a target DBMS (source may be the target). I took as example a source DBMS being a Linux on IBM Z PostgreSQL, and a target DBMS being AWS Oracle.
* **(1a)** DPP Admin connects to DPP and uses DPP SQL functionalities.
* **(1b)** DPP SQL query the source DBMS according to the DPP Admin need.
* **(1c)** Source DBMS sends back SQL query output to DPP as it is (in the clear).
* **(1d)** Sent back data is stored at first in a temporary local view.
* **(1e)** The Passport Controller apply then data protection policy to the data, and saved it as a table (this is the TDO) on target DBMS.

You can find below how a JDBC application experiences an SQL Query directly to a TDO.
* **(2a)** User connect to an URL pointing to a DBMS hosting a TDO, and SQL query it. For such connection, it is mandatory to provide valid credentials, the name of the TDO, driver name, ...
* **(2b)** DBMS checks provided credentials, and send back SQL query output to user (protected data). The data is still protected. The only way to experience the data in the clear is to send back the TDO to the Passport Controller.

**Note:** IBM DPP will must use a valid credentials on target DBMS in order to be able to create the TDO table.

![alt-text](https://github.com/guikarai/IBMDPP/blob/master/Protection.png?raw=true)

### 1.2 Direct SQL query to an existing TDO

:exclamation: Data being protected for exfiltration, the data is secured. Instructions to decrypt the data is provided in the metadata. It will help Passport Controller to find-out the appropriate and required keys from key-stores.

:exclamation: By the policy it is possible to let some column into the clear.

:computer: Let's experience what data looks like in a TDO:

```
beeline -u 'jdbc:postgresql://10.3.58.109/userdb' -n myuser -p XXXXX -d org.postgresql.Driver -e 'select * from protected_app1_customer limit 10;';
```
Expected output is:
```
Connecting to jdbc:postgresql://10.3.58.109/userdb
Connected to: PostgreSQL (version 10.10 (Ubuntu 10.10-0ubuntu0.18.04.1))
Driver: PostgreSQL JDBC Driver (version 42.2.5)
Transaction isolation: TRANSACTION_REPEATABLE_READ
+-----------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------+---------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------+--+
|                                           orig_cif_number                                           |                                      gender                                       |                                      first_name                                       |                                      last_name                                       |                                          age                                           |                                           sin                                           |                                                email                                                 |                                                phone                                                 |                                                                      mailing_address                                                                       |                                      prov_abbr                                       |                                      postal_code                                       |
+-----------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------+---------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------+--+
| ##P1L{"#K":"demompl.mykeyforapp1","#D":{"gwKoiYGOQj7O6ts0j95S+g==":"Long"},"#A":"orig_cif_number"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"aZMTWQaxn37v0EI6iyJQqA==","#A":"gender"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"Gth30l2YH9rCbo5Pz4aEfA==","#A":"first_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"7us8Bj7yFF++hlCpDAmUWA==","#A":"last_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"QM9OxQeF6t29smYc6lhPTQ==":"Int"},"#A":"age"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"98XsP522qqamC5Cc12I59A==":"Long"},"#A":"sin"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"wxxzWMLWVGyQI4s7W4lQUq0J/UnY0tIdgsafMvbrI7w=","#A":"email"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"4u8LM7cUOT8+W3135tzpgHcl0Gg+QpKSRlcqsoUfyXQ=","#A":"phone"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"HzxxZv/n9YJ27jc7WFgCeXbUysyceIFrV+bx4hH07k0=","#A":"mailing_address"}                                              | ##P1L{"#K":"demompl.mykeyforapp1","#D":"B+agxwnzKdO1BB6FzawNfQ==","#A":"prov_abbr"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"7fRclMdfnSYBNY70NFPb7g==","#A":"postal_code"}  |
| ##P1L{"#K":"demompl.mykeyforapp1","#D":{"DwdOY7Y2Xl4LtE6WoNyfbA==":"Long"},"#A":"orig_cif_number"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"aZMTWQaxn37v0EI6iyJQqA==","#A":"gender"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"lqBpqWnWlkq/Xhqb8zjyjQ==","#A":"first_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"YbrWf2TsdHadWnlgDkQqxA==","#A":"last_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"erTjNeq1252ZQ8vvxgUQ9A==":"Int"},"#A":"age"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"lRw365+4y2ajbQocJ8Wouw==":"Long"},"#A":"sin"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"nqLY/JKXh0xd/gMY9mAaBIneZS4HGVArt+sTcrd0tCU=","#A":"email"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"a30g2lFvQdy1rWrgmP/f4xeeslLGFd0AB7R1CPXqPTE=","#A":"phone"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"WYe/S8mPlHx4Wni1q3zt9x8dg2jLaZNuzoF+Zgn7HAxQxPrPXAhtAlXOPk6Z4JKl","#A":"mailing_address"}                          | ##P1L{"#K":"demompl.mykeyforapp1","#D":"+yCq2vXLK/4zgWQ0OH5QnA==","#A":"prov_abbr"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"a1FpPOBPpiEjE0/ZB139bg==","#A":"postal_code"}  |
| ##P1L{"#K":"demompl.mykeyforapp1","#D":{"5V3ZtsMNnNFmZZy9UlZQ1w==":"Long"},"#A":"orig_cif_number"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"aZMTWQaxn37v0EI6iyJQqA==","#A":"gender"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"FVOiR4eJcXa1IHG6HEqV/A==","#A":"first_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"YuDFeWiBL0aghT0KBECudg==","#A":"last_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"syo6Cr1tFSPhcwl6ZrKIOA==":"Int"},"#A":"age"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"Q9vCa8ItqSlv6jW6XQ7phg==":"Long"},"#A":"sin"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"jF5CzA6tY08SK0wqlcJK9ZBFw6tkH5ZJ/gVRtwvQlrI=","#A":"email"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"1c/aEiSCvspnmGoRSknHbUAurYE/hl45RcnNmtuGIhI=","#A":"phone"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"f6bHp99jTjMAQenMb/Mvy9Ms/PTUZlu2ez/84C/aXz4Dt49Ow9dtkdQYICdCvuLs","#A":"mailing_address"}                          | ##P1L{"#K":"demompl.mykeyforapp1","#D":"m3fgvRRzFoiN1EBs+1y+Kw==","#A":"prov_abbr"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"hn5GD4DnihJ+rzGE6flx4A==","#A":"postal_code"}  |
| ##P1L{"#K":"demompl.mykeyforapp1","#D":{"nrgh7c0kkhI5zQBoA904GQ==":"Long"},"#A":"orig_cif_number"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"Vrk2zoEITE+p0kK7PBSn1Q==","#A":"gender"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"BODUuDbELk8eFDYV9GUTHQ==","#A":"first_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"KgwpE3SXhLEa44FotaqvzQ==","#A":"last_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"34IeR+8LxUEqBT9xnTrCwQ==":"Int"},"#A":"age"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"qTpcXDMNFvyQ8HSZWlY/ZA==":"Long"},"#A":"sin"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"jDk249xzrjU0LKxAuDl/D0tbY37opIdog6I797kUT9k=","#A":"email"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"zsmaENDYlECbuzE/J6hovw==","#A":"phone"}                      | ##P1L{"#K":"demompl.mykeyforapp1","#D":"nvcofQ6zZ+kACG3WR28NC7KHwIhj196MpHmdEatl8GHRMch+9HssKQ2o05Kl5rMq","#A":"mailing_address"}                          | ##P1L{"#K":"demompl.mykeyforapp1","#D":"B+agxwnzKdO1BB6FzawNfQ==","#A":"prov_abbr"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"ZTB85uRYkC4Egh7Kwq3pxA==","#A":"postal_code"}  |
| ##P1L{"#K":"demompl.mykeyforapp1","#D":{"ZEM61eIqkJMhHmGODTk26A==":"Long"},"#A":"orig_cif_number"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"aZMTWQaxn37v0EI6iyJQqA==","#A":"gender"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"fyPePr3k1PWzYKUBzhnvRA==","#A":"first_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"XpmsNn5he+0D0PofkatcFA==","#A":"last_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"QdyJVgMBS2J063NFSRPYjg==":"Int"},"#A":"age"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"Q59/kwFaRKI5TRwoqx7mtw==":"Long"},"#A":"sin"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"UpOP9KV0jybs4csa8xg68HtvvSFCxGozGSt9+OVgpMc=","#A":"email"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"XWzWwk76ey5AB+a/fXmp9g==","#A":"phone"}                      | ##P1L{"#K":"demompl.mykeyforapp1","#D":"UJ1cNZ5pyfHRVuJXiXbvbc2wSUuaX/RKCznOf2WsMSQ=","#A":"mailing_address"}                                              | ##P1L{"#K":"demompl.mykeyforapp1","#D":"B+agxwnzKdO1BB6FzawNfQ==","#A":"prov_abbr"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"ZSPnRevCVTS+YikSZWgKSA==","#A":"postal_code"}  |
| ##P1L{"#K":"demompl.mykeyforapp1","#D":{"+d1hCj2xrmOiNA30OEQO6A==":"Long"},"#A":"orig_cif_number"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"Vrk2zoEITE+p0kK7PBSn1Q==","#A":"gender"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"aeEdwicKoqi5I75PudkH8Q==","#A":"first_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"/0ytTE5DZr4btg4xQG5Gfw==","#A":"last_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"dzQA+SeUqKPhuFs9vvm3Dw==":"Int"},"#A":"age"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"h6hsxNS5nxH19ILLdkZx5A==":"Long"},"#A":"sin"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"1chidem6+z/Uch7cMZvsqMEjOANUYJyvveEVA/nQwrY=","#A":"email"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"j+kmTu5E9jukeP+MwMrcFWjxHuPgchx+3kylWOcYIuE=","#A":"phone"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"SJsIzrYRi6iX6QTA3wrah6U/NNeLIf5toZKx+VFjnk2PZxY5LO+bfNTv8dhhhRT7","#A":"mailing_address"}                          | ##P1L{"#K":"demompl.mykeyforapp1","#D":"/G+DpfOfzRXMBLDBur8+FA==","#A":"prov_abbr"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"GFIFrrt6KYVCkoYmXowZmQ==","#A":"postal_code"}  |
| ##P1L{"#K":"demompl.mykeyforapp1","#D":{"NXyqagm6dO29kVLMCQJ2rA==":"Long"},"#A":"orig_cif_number"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"Vrk2zoEITE+p0kK7PBSn1Q==","#A":"gender"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"aB+l8wQ2oYdrt9Nrgfmxxw==","#A":"first_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"ipxBcNKHciGgUf6p8Zo8zA==","#A":"last_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"b83MzPvT01L3Y4PfseGpnw==":"Int"},"#A":"age"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"ypnYlS2KJnG+jbBWfESbOQ==":"Long"},"#A":"sin"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"LETI6KK9wSXaFu6sTcdxmhJ5gkA/iwTsTmKLDl4U0Ls=","#A":"email"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"hsxLVewaHAXYJygl24dRQghVr3IVO2p0OMLOMnmbs7s=","#A":"phone"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"qHXBsptZxdwkiqwIDiST3D3FpaU1zUg9WiEvcKGq9CLQNFmo0pAC22Z8O4wZRtzI","#A":"mailing_address"}                          | ##P1L{"#K":"demompl.mykeyforapp1","#D":"NIxO29grIq92v2upXZGT9g==","#A":"prov_abbr"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"90u+gpb1nfkApJCA3Pd/eA==","#A":"postal_code"}  |
| ##P1L{"#K":"demompl.mykeyforapp1","#D":{"DQ6DlOg8yrm5W+LpR4uEmQ==":"Long"},"#A":"orig_cif_number"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"aZMTWQaxn37v0EI6iyJQqA==","#A":"gender"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"9ZsLx0ayfmfoU28IYhUJdg==","#A":"first_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"iogerpbu2m5AeVylCBiU0A==","#A":"last_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"iTJoDvF62kQbACrCIM1l/Q==":"Int"},"#A":"age"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"ZJR4AfqE1kvEAB0/tJdtnw==":"Long"},"#A":"sin"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"YMS7TRP5wwPSMxq45fch3gO7fsYqJ9ANcT7/0YfbqpY=","#A":"email"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"wm1qq8NeVV7yqsNKlwN6X+i5xSoLvLwSeY61Uorh+Ag=","#A":"phone"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"HLy9F3ldXHoxc0XjNrb3PAdKVTnxak0+ygZuLRRs80xaq9tQAARiobnDnFPHN3BV","#A":"mailing_address"}                          | ##P1L{"#K":"demompl.mykeyforapp1","#D":"m/kR1v3q9NcrlozCyybQGQ==","#A":"prov_abbr"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"+zxbEY4U/YUNxCNK2vjg/w==","#A":"postal_code"}  |
| ##P1L{"#K":"demompl.mykeyforapp1","#D":{"cqrUF5IMvQZ3IEMYoqrXDQ==":"Long"},"#A":"orig_cif_number"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"aZMTWQaxn37v0EI6iyJQqA==","#A":"gender"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"fQq3teeXJHZf3zgVqj3UJg==","#A":"first_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"TnIhmNTjTJ0GA3PsjAev4Q==","#A":"last_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"KMWor7Q7LJvi12l0ttb+zA==":"Int"},"#A":"age"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"0zEBoKmqh9t4RcbDItZ+Sg==":"Long"},"#A":"sin"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"L6vCwdWirVjyi/XtiJUM4CyR0qgSoEe676EIrLLY1CM=","#A":"email"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"cOoH1rRzJ63F4OZNI+VyBQ==","#A":"phone"}                      | ##P1L{"#K":"demompl.mykeyforapp1","#D":"J3efK9tbYsiuUyfHl23Q+ya3qEtOrJea6QlMrGB+/kNZszPmbpWr5/eOEnvZKcc2gdN05Rc/XzSaLR/0Gq/7Rg==","#A":"mailing_address"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"/G+DpfOfzRXMBLDBur8+FA==","#A":"prov_abbr"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"zrUZIqXAVjcbKd3iKqMw/Q==","#A":"postal_code"}  |
| ##P1L{"#K":"demompl.mykeyforapp1","#D":{"mDuPPnvsse+3mzMblAIi/Q==":"Long"},"#A":"orig_cif_number"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"Vrk2zoEITE+p0kK7PBSn1Q==","#A":"gender"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"9bkxzP50VexFgoAzC4eRuw==","#A":"first_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"xU3s/l+D6HNWTYA5A08ecA==","#A":"last_name"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"Z2WpxcCoanwaPzXUwaOkdQ==":"Int"},"#A":"age"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":{"tTwiMDBJy1TP79jeZ1b/eg==":"Long"},"#A":"sin"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"gCMiW8H4PEDrOmWO6tchkV/BiiOcqZzaJUwgjDuVpIU=","#A":"email"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"xf0qPj39r93AuoG+UUF+LkLsIcvBZCdrVv1GO/nA9dc=","#A":"phone"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"lXp27MzjIoK3DEx66BnAAWWvbrRGEYmPWzAXySKE11p9r2VP60ToWPWtRPnQU+/C","#A":"mailing_address"}                          | ##P1L{"#K":"demompl.mykeyforapp1","#D":"AtwXZWcLb+Mot/gWC9qPmw==","#A":"prov_abbr"}  | ##P1L{"#K":"demompl.mykeyforapp1","#D":"Z5EaEsNxd6ChF5Mx+KvHMw==","#A":"postal_code"}  |
+-----------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------+---------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------+--+
10 rows selected (0.142 seconds)
Beeline version 1.2.1.spark2 by Apache Hive
Closing: 0: jdbc:postgresql://10.3.58.109/userdb
```
:exclamation: Off-course we can't understand anything, we just query a TDO directly. What we see is security mecanism and the data together. As a TDO, data is secured no matter where it is, and no matter the security of environment the data is crossing.

There is only one way to get access to the data of the TDO in the clear, this is via IBM Data Privacy Passports.

## 2. Protection and then enforcement of the data with IBM Data Privacy Passports
:white_check_mark: **Objective:** Data is protected at the exfiltration point and off the platform, and data is enforced at the consumption point.

You can find below how a JDBC application experiences an SQL Query to a TDO via IBM Data Privacy Passports.
* **(1)** User connect to an URL pointing to IBM Data Privacy Passports. For such connection, it is mandatory to provide valid credentials, the name of the TDO, driver name, ... 
* **(2)** IBM Data Privacy Passports checks in the policy the DbViews section to find the appropriate JDBC connection profile to SQL query the TDO.
* **(3a)** DBMS sends back SQL query output to IBM Data Privacy Passports as it is (protected data).
* **(3b)** The Passport Controller directly enforces the data (according to the policy) coming from the source DBMS.
* **(3c)** IBM Data Privacy Passports via the Passport Controller send back the data (enforced data) to the user.

![alt-text](https://github.com/guikarai/IBMDPP/blob/master/Protect-then-enforce.png?raw=true)

Let's SQL query a TDO via IBM Data Privacy Passports with different users.

### 2.1 SQL query as a Data Owner (DO) of a TDO via IBM Data Privacy Passports:

:computer: Let's experience what data looks like in a TDO:

```
beeline -u "jdbc:hive2://10.3.58.108:10010" -n DO -p XXXXX -e "select * from AWSpostgresql.protected_app1_customer LIMIT 10;"
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

# 3. Conclusions and next steps

You can find below a summary of the different Protection use cases.
![alt-text](https://github.com/guikarai/IBMDPP/blob/master/Protection-matrix.png?raw=true)

This close the Data Protection chapter. Next chapter in this hands-on labs is [Audit trail](https://github.com/guikarai/IBMDPP/blob/master/audit-trail.md).

