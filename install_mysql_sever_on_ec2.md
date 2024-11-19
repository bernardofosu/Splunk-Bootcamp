# Ports You have top open in your EC2 Security Groups
```bash
9998 and 9999 # for DB Connect App
3306 # for MySQL
5432 # for Postgress
```

# How to install MySQL Sever on EC2 instance
- Visit [MySQL Website](https://www.mysql.com)
- Go to downloads and click on MySQL Community (GPL) Downloads
- Click on MySQL Yum Repository but MySQL Community Server[](https://dev.mysql.com/downloads/mysql/) for Local Computer
- Right click on this link, No thanks, just start my download and Copy link
- You will get something like this 
```bash
https://dev.mysql.com/get/mysql84-community-release-el8-1.noarch.rpm
```
- SSH into your ec2 instance and paste this command and press enter
```bash
wget https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm
```
## Install mysql84-community-release
```bash
sudo dnf install mysql84-community-release-el9-1.noarch.rpm
```
**dnf:**
The Dandified YUM (DNF) package manager is the next-generation version of yum, used in Red Hat-based Linux distributions (e.g., Fedora, CentOS, RHEL, Rocky Linux, AlmaLinux).

Handles package management tasks like installation, updates, and removal of software.

**.rpm (Red Hat Package Manager)** file for the MySQL 8.4 Community Release repository.
This package contains repository configuration files for MySQL 8.4 on Enterprise Linux 9 (el9).

## Install mysql-community-server
```bash
sudo dnf install mysql-community-server
```

## Check mysql-community-server status
```bash
sudo systemctl start mysqld
sudo systemctl enable mysqld
```
## Enable mysql-community-server 
```bash
sudo systemctl enable mysqld
```
## Start mysql-community-server 
```bash
sudo systemctl start mysqld
```
## You can then secure your MySQL installation:
```bash
sudo mysql_secure_installation
```
## How to get the temporal password of mysql root user
```bash
sudo cat /var/log/mysqld.log | grep "password"
```
## Output of the above command 
```bash
2024-11-18T15:01:46.126807Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: ,ttNnM:+*2lY
```

## How Log in with root 
```bash
sudo mysql -u root -p
```
Enter your password

## How to Check for File Permissions or SELinux Issues (This method same allow the GRANT to work)
Ensure that the MySQL data directory and related files have the correct permissions, and check if SELinux is enforcing any additional restrictions on MySQL. If SELinux is enabled, try temporarily disabling it to see if it resolves the issue:
```1bash
sudo setenforce 0
```

## Try running the following command if you're logged in as the root user:
```bash
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
```
## After running the above command, make sure to flush the privileges:
```bash
FLUSH PRIVILEGES;
```
## How to check users and host
```bash
SELECT user, host FROM mysql.user;
```
The output you shared shows the users in the MySQL server:
| user             | host      |
| ---------------- | --------- |
| root             | %         |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
NB: The % sign shows that the user can accept traffic from everywhere

- root with access from any host (%).
- mysql.infoschema (a system user, typically for information schema).
- mysql.session (a system user related to session management).
- mysql.sys (a system user, typically for system operations).

If you need to modify privileges or delete users, make sure you're connected to MySQL as a privileged user (like root). You can then proceed with altering, granting, or dropping users based on your requirements.

## How to Reset the root password
Once logged into MySQL, execute the following command to reset the root password:
```bash
ALTER USER 'root'@'%' IDENTIFIED BY 'Root@544344';
```
The % symbol allows connections from any IP address. You can specify a specific IP address instead of % for additional security.

## Check the current password policy
You can check the current password policy by running the following query in MySQL:
```bash
SHOW VARIABLES LIKE 'validate_password%';
```
This will show the current password validation rules. You might see something like this:
+--------------------------+--------------------------------------+
| Variable_name            | Value                                |
+--------------------------+--------------------------------------+
| validate_password.check_user_name | ON                        |
| validate_password.length | 8                                    |
| validate_password.mixed_case_count | 1                             |
| validate_password.number_count | 1                                |
| validate_password.policy | MEDIUM                               |
| validate_password.special_char_count | 1                         |
+--------------------------+--------------------------------------+
- validate_password.policy indicates the policy level (LOW, MEDIUM, STRONG).
- validate_password.length shows the minimum password length.
- validate_password.mixed_case_count and other fields show how many - mixed case letters, numbers, and special characters are required.

## How to test the connection: 
Finally, you can test the remote connection with the ben user:
```bash
mysql -h 54.175.128.74 -u root -p
```

## How Create a new ben user with remote access and Grant him all previleges 
Now, create the ben user with access from any host (%), and set the password:
```bash
CREATE USER 'ben'@'%' IDENTIFIED BY 'Ben@20260918';
```
## Grant Privileges: After confirming the user exists, grant privileges with the following command:
```bash
GRANT ALL PRIVILEGES ON *.* TO 'ben'@'%' IDENTIFIED BY 'Ben@20260918';
```
```bash
FLUSH PRIVILEGES;
```
Verify Grants: Finally, verify that the privileges have been granted to the newuser by running:
```bash
SHOW GRANTS FOR 'ben'@'%';
```
The % symbol allows connections from any IP address. You can specify a specific IP address instead of % for additional security.

## How to check users and host
```bash
SELECT user, host FROM mysql.user;
```
The output you shared shows the users in the MySQL server:
| user             | host      |
| ---------------- | --------- |
| ben              | %         |
| root             | %         |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |

- ben with access from any host (%).
- root with access from any host (%).
- mysql.infoschema (a system user, typically for information schema).
- mysql.session (a system user related to session management).
- mysql.sys (a system user, typically for system operations).

## How to connect MySQL Sever to MySQL Workbench
```bash
click on the + > Add host (Public IP of EC2 Instance) > Add Username > Add Password > Test Connection
```
## How to Create some database in MySql Workbench
```bash
CREATE DATABASE crud_new1;
```
## Use Database
```bash
use crud_new1;
```

## Create some table in MySql Workbench
```bash
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);
```

## Set Up our Python Environment to Deploy our Streamlit CRUD
### Check Python Version
```bash
python --version
```

## Update the Packages
```bash
sudo apt update  # For Ubuntu/Debian-based distributions
sudo yum update -y # for Centos/AWS Linux
```
## How to Create a virtual environment: 
Run the following command to create a virtual environment:
```bash
python3 -m venv streamlit_crud
```
This creates a directory named ***streamlit_crud**
inside your project folder that contains the virtual environment.

## How to Activate the virtual environment: (Linux/macOS:)
```bash
source streamlit_crud/bin/activate
```
## How to Activate the virtual environment: (Windows)
```bash
.\streamlit_crud\Scripts\activate
```

## Installing pip (Python Package Installer)
```bash
sudo apt install python3-pip  # For Python 3 Ubuntu
sudo yum install python3-pip # for AWS linux
```

## Installing streamlit and install mysql-connector-python
```bash
pip install mysql-connector-python
pip install streamlit
```

## Clone our work we want to deploy from github
```bash
git clone https://github.com/bernardofosu/crud_mysl_streamlit.git
```
