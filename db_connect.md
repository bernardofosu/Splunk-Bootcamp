# How to Install DB Connect

## Installation and setup overview
Follow this overview to install and set up Splunk DB Connect.

### Prerequisites
To deploy Splunk DB Connect on either a single instance of Splunk Enterprise or on a search head in a distributed deployment, you must meet the system requirements.

After meeting the [system requirements](https://docs.splunk.com/Documentation/DBX/3.18.1/DeployDBX/Prerequisites), start the DB Connect installation process:

- Download and install the DB Connect app.
- Install a JDBC driver for your database. 
- Install [database drivers](https://docs.splunk.com/Documentation/DBX/3.18.1/DeployDBX/Installdatabasedrivers).
- After installing DB Connect and restarting Splunk Enterprise, launch DB Connect.
Create a database identity and set up a database connection.
- [Create a new database input](https://docs.splunk.com/Documentation/DBX/3.18.1/DeployDBX/Createandmanagedatabaseinputs) and use the data input in a search.

# Configure Java Runtime Environment (JRE) for Splunk DB Connect
[Download and install](https://www.oracle.com/java/technologies/downloads/?er=221886#java21) a compatible version of the Java Runtime Environment (JRE).

## Install the JRE or JDK on the Splunk Enterprise server
NB: You acn upload to the or secure copy to upload to the Splunk Enterprise server or use **wget** to download on Splunk Enterprise server
```bash
tar xfvz /opt/openjdk-21.0.5_linux-x64_bin.tar.gz -C /opt
```
```bash
chown -R splunk:splunk /opt/jdk-21.0.5
```
```bash
chmod -R 700 /opt/jdk-21.0.5
```
## Creates a symbolic link (soft link), which is like a shortcut that points to another file or directory.
To make upgrades of Java easy, use the following command to make a link (alias) to the Java installation, so that JAVA_HOME can be defined once in Splunk DB Connect and then used there, even after an upgrade:
```bash
ln -s /opt/jdk-21.0.1 /opt/java
```
The above command creates a symbolic link named /opt/java that points to the directory /opt/jdk-21.0.1. Here's an explanation of what it does and why it might be useful:
### What the Command Does
**1. ln:**
The command to create links (hard links by default).

**2. -s:**
Creates a symbolic link (soft link), which is like a shortcut that points to another file or directory.
**3. /opt/jdk-21.0.1:**
The target directory that you want to link to.

**/opt/java:**. The name of the symbolic link.

**4.** After running this command:
**/opt/java** will act as a shortcut to **/opt/jdk-21.0.1.**


## Set up the JAVA_HOME in the OS environment and add it and the bin folder to the PATH so that the Splunk Enterprise user can use it.
This is critical, since most issues that are seen with installing Splunk DB Connect are because users can't find the Java installation and the Java bin directory. You can use the following commands in the **~/.bashrc** or **/etc/profile** on the Splunk user:
```bash
export JAVA_HOME=/opt/java
```
```bash
export PATH="$JAVA_HOME/bin:$PATH"
```
```bash
source ~/.bashrc or source /etc/profile
```
## Warning 
If you are installing DB Connect on Windows Splunk, then install it as normal with an MSI. However, you should verify that the JAVA_HOME and PATH environment variables are set up in the system environment similar to the above, as the installer doesn’t always get it right.

## Install the Splunk Enterprise heavy forwarder. 
Make sure that it is functional and it is at least sending its internal logs to the indexers with a search such as 
```bash
index=_* host=<SplunkHFName>.
```

## Make sure that the Splunk user environment can get to Java, with this command:
```bash
/opt/splunk/bin/splunk cmd java -version
```
## The output should look some like these
```bash
openjdk version "21.0.1" 2023-10-17
OpenJDK Runtime Environment (build 21.0.1+12-29)
OpenJDK 64-Bit Server VM (build 21.0.1+12-29, mixed mode, sharing)
```

[Read More](https://lantern.splunk.com/Splunk_Platform/Product_Tips/Extending_the_Platform/Configuring_Splunk_DB_Connect)