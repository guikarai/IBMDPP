# Policy

With IBM Data Privacy Passports, the security and the privacy of the data is dicted by a policy.
This is very important to understand what is discribed in the policy, in order to protect accuratetly data according to users, and their needs to know.

Let's see first what a policy is, and second what is the policy in action in this hands-on lab environment.

# Policy content, structure and blokcs

IBM Data Privacy Passports focuses on data structures and data elements as the “carriers” of Trust, which means it “embeds” the security and privacy policies of an enterprise into the data in a way that enables the data to form a “data layer” agnostic of the processes that consume it. To do that the enterprise policies regarding the usage of data must be captured in a data-centric manner. The policy is the rules book describing what data need to be protected, and how, and according which enterprise business functions and their users. The policy looks likes and xml files made of sections.


You can find below an high level orverview of how a policy is organized:

There are several important sections/blocks in a policy:

* **"Definition structure block"** It helps to define the name of the organization, the validity date of the policy, and its status (Prod, Test, Debug...).

* **"Need to know" data element collection"** These are a set of sections that defined what data need to be protected/enforced by a defaut policy, and what expectional rules need to be applied for selected users and business functions.

* **"Enterprise Functions"** This is a data element collection, to define consistant view of Business Functions, and the relationship with the users.

* **"dbViews"** This is data element collection to define how the policy can access to data from external sources (eg. z/OS Db2, AWS Oracle, AWS PostgreSQL, LoZ PostgreSQL, Db2 on Power). This data element collection is key, because it will be use to Enforced data in flight, and to read Trusted Data Objects from the source. Source data remain in the clear and clients connect to IBM Data Privacy Passports as a proxy that will enforce data for them.
* **"Configuration block"** It is about a set of default settings, key stores, ciphers to be used in the policy.


# Policy status
