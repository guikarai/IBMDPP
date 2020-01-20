# Audit trail

An audit trail (also called audit log) is a security-relevant chronological record, set of records, and/or destination and source of records that provide documentary evidence of the sequence of activities that have affected at any time a specific operation, procedure, or event.

IBM Data Privacy Passports plays a critical roles enforcing/protecting the data. 
Moreover, IBM Data Privacy Passports is admnistrated by Admin users in charge of:
* creating the policy
* creating routes to DBMS using DMBS's valid users and credentials
* co-signing a policy
* deploying a policy

Such activities needs to be logged acurrately for both security and compliance reasons. Let's access to some audit logs.

## 1. Query audit

IBM Data Privacy Passports is based on a engine accepting some command lines. One of this command line **"@DP show metrics"** dislayed the latest queries addressed and executed by IBM Data Privacy Passports.

:computer: Issue the command shown below to query as an IBM Data Privacy Passports Administrator the query log:
```
beeline -u 'jdbc:postgresql://10.3.58.109/userdb' -n myuser -p XXXXX -d org.postgresql.Driver -e 'select * from customer limit 10;';
```
Expected output is:

```
```

As you can see, we can get usefull information including the following:
* Why:
* What:
* How:
* When:
* By who:

## 2. Other audits
IBM Data Privacy Passports plays a critical roles enforcing/protecting the data. IBM Data Privacy Passports generates multiple fine logs in order to capture:
* executed queries
* emiteed events and incidents
* abbends
* error processes

There is a hierarchy of logs than can be accessed and used as evidence in case of a fine investigation.

## 3. Thank you
:clap: :metal: Congratulations! You did it, you just finished the IBM Data Privacy Applied Hands-on lab.

If you need to go deeper, do not hesitate to contact me: :email: guillaume_hoareau@fr.ibm.com
