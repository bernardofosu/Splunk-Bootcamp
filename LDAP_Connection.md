# Splunk LDAP Authentication Settings
#### NB: Before  we start the process below  we have to first set up our Active Directory on Windows Server under Server Manager and Create some groups and users. Its did not work for me on the windows server 2025 but work fine on windows server 2022.

## LDAP connection settings

### 1. LDAP strategy name *
Enter a unique name for this strategy.
Here you enter any name that you want to give to your LDAP connection

### 2. Host *
LDAP Host: You can use the private IP or DNS-resolved hostname or public IP address of your Active Directory or LDAP Server if its does not sit in the same local network.

#### You can test if the Splunk server can connect the LDAP server's private IP using the following commands:
```bash
ping <LDAP server's IP Address>
```
NB You have to open <All ICMP IPv4> at you AWS EC2 splunk server instance security groups

#### Other way to test the connection
```bash
telnet 172.31.69.293 389
```
##### output
```bash
[ec2-user@ip-172-31-23-185 ~]$ telnet 172.31.69.253 389
Trying 172.31.69.253...
Connected to 172.31.69.253.
```

#### Check DNS resolution for the LDAP server's hostname:
```bash
nslookup ldapserver.example.com
```

### Port
The LDAP server port defaults to 389 if you are not using SSL, or 636 if SSL is enabled.
Port: Use port 389 (for plain LDAP).

SSL enabled
You must also have SSL enabled on your LDAP server.


### Bind DN
This is the distinguished name used to bind to the LDAP server. This is typically the DN of an administrator with access to all LDAP users you wish to add to Splunk. 

Admin user DN: A common approach is to use an admin account (like a domain administrator or a service account) with sufficient permissions.

Any valid user DN: It’s not required that the Bind DN be an admin; it can be any user as long as the user has the necessary permissions to query or interact with the LDAP directory.
For example:
```bash
CN=admin,CN=Users,DC=bernardsplunk,DC=com #for an admin account. These is under the default Users
CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com
#for a regular user (if the user has permission to perform the required query). These under the SPLUNK OU which also under Splunk Users OU
```
NB: In our case we will a splunk service account to use as BIN DN and it can be any user with any username.


### Bind DN Password
Enter the password for your Bind DN user.
Confirm password

Bind DN: The user that Splunk should bind as (e.g., 
```bash
CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com
```
LDAP Password: The password for the Bind DN.
In LDAP, the Bind DN and password represent the credentials used to authenticate to the directory server. For a successful bind, the user account associated with the Bind DN must have appropriate access rights to perform the requested LDAP operations.

NB: Base DN: Set the appropriate Base DN based on your Active Directory setup. 


### How to check the connection from the splunk server backend
```bash
ldapsearch -x -h 172.31.69.253 -b "dc=bernardsplunk,dc=com" "(objectClass=*)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -W
```
Enter LDAP Password:

#### Explanation

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
## What if i want to add my password to the ldapsearch command
```bash
ldapsearch -x -h 172.31.69.253 -b "dc=bernardsplunk,dc=com" "(objectClass=*)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -w "@gjhh677334"
```
### Key points: 
The **-w** option expects the password as a direct argument.

Special characters like &&, $, !, etc., should be wrapped in quotes (" or ') to prevent the shell from interpreting them.

Incorrect usage: -w &&hehgeft99937 is problematic because && is used as a logical operator in the shell (to run multiple commands), and the -w flag expects a password argument immediately after it.

Using quotes ensures the entire string (including special characters) is passed correctly as the password to ldapsearch.

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

## You can Create a text file for the password (it did not work for me)
```bash
nano password.txt
```
### Error
``` 
Warning: Password file password.txt is publicly readable/writeable
ldap_bind: Invalid credentials (49)
        additional info: 80090308: LdapErr: DSID-0C09050E, comment: AcceptSecurityContext error, data 52e, v4f7c
[ec2-user@ip-172-31-23-185 ~]$
```
### Solution
The two issues you're encountering are:

Warning about the password file being publicly readable/writable.
LDAP bind error (invalid credentials) with the code 80090308 and error data 52e, which indicates an authentication failure.

Warning: Password file password.txt is publicly readable/writeable
This warning occurs because the file containing the password (password.txt) is not secured with the proper file permissions. It’s important to ensure that only authorized users can read/write the password file to protect sensitive information.

To fix this, you should set the file permissions to be more restrictive:

```bash
sudo chmod 600 password.txt
```

### Debugging with ldapsearch
Try running ldapsearch with increased verbosity using the **-d** flag to get more information about what's happening:

```bash
ldapsearch -x -d 255 -h 172.31.69.253 -b "OU=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com" "(objectClass=group)" -D "CN=splunk service,OU=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com" -y password.txt
```

## User Settings
### User base DN *
The location of your LDAP users, specified by the DN of your user subtree. If necessary, you can specify several DNs separated by semicolons.
```bash
"CN=Splunk Users,OU=SPLUNK,DC=bernardsplunk,DC=com"
```
```bash
"CN=SPLUNK,DC=bernardsplunk,DC=com"
```
If you are creating all the users and groups under one OU then you can use that OU DN for both User base DN or Group base DN 

### User base filter
The LDAP search filter used to filter users. Highly recommended if you have a large amount of user entries under your user base DN. For example, '(department=IT)'

### User name attribute *
The user attribute that contains the username. Note that this attribute's value should be case insensitive.
Set to 'uid' for most configurations. In Active Directory (AD), this should be set to 'sAMAccountName'.
```bash
sAMAccountName
```
### Real name attribute *
The user attribute that contains a human readable name. This is typically 'cn' (common name) or 'displayName'.
```bash
cn
```

### Email attribute
The user attribute that contains the user's email address. This is typically 'mail'.

### Group mapping attribute
The user attribute that group entries use to define their members. If your LDAP groups use distinguished names for membership you can leave this field blank.

## Group settings
### Group base DN *
The location of your LDAP groups, specified by the DN of your group subtree. If necessary, you can specify several DNs separated by semicolons.
```bash
"CN=Splunk Groups,OU=SPLUNK,DC=bernardsplunk,DC=com"
```
```bash
"CN=SPLUNK,DC=bernardsplunk,DC=com"
```
NB: If you are creating all the users and groups under one OU then you can use that OU DN for both User base DN or Group base DN 

### Static group search filter
The LDAP search filter used to retrieve static groups. Highly recommended if you have a large amount of group entries under your group base DN. For example, '(department=IT)'

### Group name attribute *
The group attribute that contains the group name. A typical value for this is 'cn'.
```bash
cn
```
### Static member attribute *
The group attribute whose values are the group's members. Typical values are 'member' or 'memberUid'. Groups list user members with values of groupMappingAttribute, as specified above.
```bash
member
```
### Nested groups
Controls whether Splunk will expand nested groups using the 'memberof' extension. Only check this if you have nested groups and the 'memberof' extension on your LDAP server.

### Dynamic group settings
Dynamic member attribute
The dynamic group attribute that contains the LDAP URL used to find members. This setting is required to configure dynamic groups. A typical value is 'memberURL'.

### Dynamic group search filter
The LDAP search filter used to retrieve dynamic groups (optional). For example, '(objectclass=groupOfURLs)'

## Advanced settings
Enable referrals with anonymous bind only
Splunk can chase referrals with anonymous bind only. You must also have anonymous search enabled on your LDAP server. Turn this off if you have no need for referrals.

### Search request size limit
<1000>
Sets the maximum number of entries requested by LDAP searches. The number actually returned is subject to the limit imposed by the LDAP server.

### Search request time limit
<15>
The maximum time limit in seconds to wait for LDAP searches to complete. This should be less than the UI timeout of 30s.

### Network socket timeout
<20>
The maximum amount of seconds to wait on a connection to the LDAP server without activity. As a connection could be a search, this must be greater than the search time limit. Enter -1 for an infinite timeout. This should be less than the UI timeout of 30s.

```bash
scp -i C:\Users\Administrator\Desktop\Splunk_Key.pem C:\Users\Administrator\Desktop\certificate.cer ec2-user@172.31.23.185:/tmp/
```

```bash
telnet 34.20.17.197 636
```
