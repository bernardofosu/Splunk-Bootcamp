# Lightweight Directory Access Protocol (LDAP) Directory Structure
Let's assume your LDAP structure looks something like this:

You will click on tools and click on ADSI Edit

```bash
DC=bernardsplunk,DC=com
│
├── OU=SPLUNK
│   ├── OU=Splunk Groups
│   │   ├── CN=Splunk Admins
│   │   ├── CN=Splunk Power Users
│   │   └── CN=Splunk Users
│   │
│   └── OU=Splunk Users
│       ├── CN=splunk service
│       ├── CN=sir willy
│       └── CN=bernard ofosu
│
└── Other OUs or domains...
```
## Explanation of the Structure:
**OU=SPLUNK:** A top-level Organizational Unit (OU) that holds both **OU=Splunk Groups** and **OU=Splunk Users.**
**OU=Splunk Groups:** Contains groups such as **Splunk Admins**, **Splunk Power Users**, and **Splunk Users**.
**OU=Splunk Users:** Contains users such as **splunk service**, **sir willy**, and **bernard ofosu.**

## How to check the connection from the splunk server backend
```bash
ldapsearch -x -h 172.31.69.253 -b "dc=bernardsplunk,dc=com" "(objectClass=*)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -W
```
## What if i want to add my password to the ldapsearch command
```bash
ldapsearch -x -h 172.31.69.253 -b "dc=bernardsplunk,dc=com" "(objectClass=*)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -w "@gjhh677334"
```
## Ways to Hide the password
### Use an environment variable
Another option is to store the password in an environment variable and reference it in the command:
```bash
export LDAP_PASS="your_password_here"
```
```bash
nano ~/.bashrc
```
```bash
ldapsearch -x -h 172.31.69.253 -b "dc=bernardsplunk,dc=com" "(objectClass=*)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -w $LDAP_PASS
```
This approach can be safer than passing the password directly on the command line since the password is stored in an environment variable. However, environment variables can still be accessed in certain circumstances, so it's still not a foolproof method.
(CN=splunk service,...) before executing the search.
The **-W** flag causes ldapsearch to prompt for the password interactively. It doesn't accept any input directly, so that's why your command was still prompting you.

#### Explanation of the command and flags
**1. ldapsearch:**
This is the command-line tool used to perform an LDAP search. It is used to query the LDAP directory and retrieve data based on the search criteria provided.

**2. -x:**
This flag tells ldapsearch to use simple authentication instead of SASL (Simple Authentication and Security Layer). SASL is typically used for more secure binding, but with -x, you’re telling the tool to bypass that and use the simpler authentication method.

**3. -h 172.31.69.253:**
This specifies the host (LDAP server) to connect to. In this case, it’s an IP address 172.31.69.253. This is the LDAP server that ldapsearch will query. You can replace this with the domain name or IP address of your LDAP server.

**4. -b "dc=bernardsplunk,dc=com":**
This is the base DN (Distinguished Name) for the search. It specifies where to start the search in the directory.

dc=bernardsplunk,dc=com refers to the domain component bernardsplunk.com. This means that the search will start from this point in the LDAP directory, which corresponds to the root of the directory for the bernardsplunk.com domain.
In this case, you're searching the entire directory tree starting from the base domain bernardsplunk.com.

**5. "(objectClass=*)":**
This is the search filter. The filter specifies which entries in the directory should be returned.

(objectClass=*) means "return all objects" in the directory, regardless of their type. The * wildcard in the filter matches any object class, so this will return all entries in the directory under the given base DN dc=bernardsplunk,dc=com.

**6. -D** "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com":
This is the Bind DN (Distinguished Name) that will be used to authenticate to the LDAP server.

The -D flag specifies the user credentials for binding to the LDAP server. In this case, the bind DN is CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com, which refers to a service account named "splunk service" in the organizational unit (OU) Splunk Users, under the SPLUNK OU, in the bernardsplunk.com domain.
The account specified in the Bind DN must have the appropriate permissions to perform the query in the LDAP directory.

**7. -W:**
This flag tells ldapsearch to prompt for the password for the user specified in the Bind DN. When you run this command, ldapsearch will ask you for the password associated with the Bind DN 

## Searching for Organizational Units (OU):
If you want to search for OUs (e.g., OU=Splunk Users, OU=Splunk Groups), you would use this:
To search for all OUs, use the filter (objectClass=organizationalUnit).

Searching for Organizational Units (OU):
If you want to search for OUs (e.g., OU=Splunk Users, OU=Splunk Groups), you would use this:
```bash
ldapsearch -x -h 172.31.69.253 -b "DC=bernardsplunk,DC=com" "(objectClass=organizationalUnit)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -W
```

## Search for All Groups in the Directory:
### Meaning of (objectClass=group):
**objectClass=group:** This filter specifies that the entries you are searching for should be of the group object class.

**Groups** are used to organize users or other entities within an LDAP directory, such as security groups, distribution groups, or other logical groupings.

The group object class can represent a variety of group types, including security groups (for permissions and access control) or distribution groups (for email or communication purposes).

To find all groups in the LDAP directory, you can use the filter (objectClass=group).
```bash
ldapsearch -x -h 172.31.69.253 -b "DC=bernardsplunk,DC=com" "(objectClass=group)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -W
```
The filter **(objectClass=group)** is used to search for LDAP entries where the object class is group. This means it will find entries that represent groups within the LDAP directory, which are typically collections of users or other objects for organizational purposes.

### LDAP Group Types:
#### There are generally two types of groups in LDAP:
**1. Security Groups:** Used for managing permissions and access control within the LDAP directory (e.g., access to specific resources).

**2. Distribution Groups:** Primarily used for email lists or organizing users for communication purposes.

### Searching for All Groups in OU=Splunk Groups: in a specific OU
```bash
ldapsearch -x -h 172.31.69.253 -b "OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com" "(objectClass=group)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -W
```
This command will return all groups under OU=Splunk Groups,OU=SPLUNK.

### Searching for a Specific Group (e.g., Splunk Admins):
```bash
ldapsearch -x -h 172.31.69.253 -b "OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com" "(CN=Splunk Admins)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -W
```
This command will return the group Splunk Admins in the OU=Splunk Groups,OU=SPLUNK container.

Step 2: Find Users in a Specific Group
If you want to find users that are members of a particular group (e.g., Splunk Admins), you can use the memberOf filter.



## Example 1: Find Users in Splunk Admins Group:
```bash
ldapsearch -x -h 172.31.69.253 -b "DC=bernardsplunk,DC=com" "(&(objectClass=user)(memberOf=CN=Splunk Admins,OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com))" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -W
```
This command will return all users who are members of the Splunk Admins group.

## Example 2: Find Users in Splunk Power Users Group:
```bash
ldapsearch -x -h 172.31.69.253 -b "DC=bernardsplunk,DC=com" "(&(objectClass=user)(memberOf=CN=Splunk Power Users,OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com))" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -W
```
## How to list all user accounts in a specific base DN (e.g., OU=Splunk Users)
You would use the filter (objectClass=user) as shown in the following example:

```bash
ldapsearch -x -h 172.31.69.253 -b "OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" "(objectClass=user)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -W
```
## LDAP Object Classes:
**objectClass=user:** This refers to an entry of type "user" in the LDAP directory. This can represent a user object (such as a person or service account), and it indicates that the entry is a user object in the directory schema.

So, when you use the filter (objectClass=user), you're searching for user entries (not an OU) that match the user object class.

This will return all user objects within the OU=Splunk Users organizational unit.

### Step 3: Search for Multiple Groups
If you want to search for multiple groups (e.g., Splunk Admins, Splunk Power Users, and Splunk Users), you can adjust the filter to match any of these groups using | (OR) logic.

### Example: Find All Users in Any of the 3 Groups (Splunk Admins, Splunk Power Users, Splunk Users):
```bash
ldapsearch -x -h 172.31.69.253 -b "DC=bernardsplunk,DC=com" "(&(objectClass=user)(|(memberOf=CN=Splunk Admins,OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com)(memberOf=CN=Splunk Power Users,OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com)(memberOf=CN=Splunk Users,OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com)))" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -W
```
This command will return all users who are members of any of the three groups Splunk Admins, Splunk Power Users, or Splunk Users.


### Breakdown of the Command:
**1. -x:** Use simple LDAP (no SASL).

**2. -h** 172.31.69.253: LDAP server address.

**3.-b** "OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com": Base DN for searching within OU=Splunk Groups in the SPLUNK OU.

**4. "(objectClass=group)":** Filter to return entries of type group.

**5. "(&(objectClass=user)(memberOf=CN=Splunk Admins,OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com))":** Filter to find users who are members of the Splunk Admins group.

**6. -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK DC=bernardsplunk,DC=com":**
The DN of the service account to authenticate the search.

**7. -W:** Prompt for the password of the service account.

### Explanation Key Points:
**Base DN (-b):**
The base DN determines the starting point for your search. For example, you search for groups under OU=Splunk Groups,OU=SPLUNK, and for users under the root DN DC=bernardsplunk,DC=com.

**Filters:** The filter (&(objectClass=user)(memberOf=...)) is used to find users who are members of a specific group. Modify the memberOf DN to point to your exact group location.

Search for multiple groups: Use **logical OR (|)** in the filter to search for multiple groups at once.
This approach should help you query specific groups, users, and members based on the nested structure you described.

**LDAP Object Classes:**
**objectClass=user:** This refers to an entry of type "user" in the LDAP directory. This can represent a user object (such as a person or service account), and it indicates that the entry is a user object in the directory schema.
So, when you use the filter (objectClass=user), you're searching for user entries (not an OU) that match the user object class.

**Common Name (CN)** is a standard LDAP attribute used to identify an object, often used for naming users, groups, computers, and other objects within a directory structure.
In the context of a group, CN would represent the name of the group (e.g., Splunk Admins, Marketing Team).
For a user, the CN might represent their full name (e.g., John Doe).

**CN** is commonly used in Distinguished Names (DNs) to uniquely identify entries within the LDAP hierarchy.
Distinguished Name (DN):

The **DN** is the unique identifier for an object in the directory. It includes all the attributes that make the object unique within the LDAP structure.
The CN is typically a part of the DN. For example:
A user DN: CN=John Doe,OU=Users,DC=bernardsplunk,DC=com
A group DN: CN=Splunk Admins,OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com
Example:

**CN=Splunk Admins:** This represents the Common Name of a group called "Splunk Admins".

In a **DN** like CN=Splunk Admins,OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com, 

The **CN=Splunk Admins** part identifies the group itself, while the rest of the **DN (OU=Splunk Groups, OU=SPLUNK, DC=bernardsplunk, DC=com)** describes its path or location within the LDAP directory structure.

## LDAP Naming Structure Example:
Consider the following example LDAP path for a group:
```bash
CN=Splunk Admins,OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com
```
**CN=Splunk Admins:** Common Name of the group (Splunk Admins).

**OU=Splunk Groups:** Organizational Unit where the group resides (Splunk Groups).
**OU=SPLUNK:** Higher-level Organizational Unit that contains the Splunk Groups OU.

**DC=bernardsplunk,DC=com:** Domain Components (bernardsplunk.com) that represent the domain.

CN is an LDAP attribute that is used to uniquely identify objects (such as users, groups, or other entities) within a directory.
It is typically used in Distinguished Names (DNs) to form the full unique path of an object in the LDAP directory.

