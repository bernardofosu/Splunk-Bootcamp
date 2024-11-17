# How create environment variables
```bash
export LDAP_PASSWD="your password here"
```
## How to reset the shell and save the changes
```bash
source ~/.bashrc
```
## You can add it directly to .bashrc file
```bash
nano ~/.bashrc
```
# How to check environment variables
In a Linux or Unix-like operating system (such as the one you're using on EC2), you can use various commands depending on what exactly you need to see. Here's how you can check environment variables:

## View All Environment Variables
To view all environment variables in your current session:
```bash
printenv
```
```bash
env
```
Both commands will list all environment variables and their values for the current session. This includes system-level variables (like PATH, HOME, etc.) and any user-defined ones.

## Check a Specific Environment Variable
If you want to check the value of a specific environment variable, you can use the echo command. For example, if you want to check the LDAP_PASS variable:
```bash
echo $LDAP_PASS
```
This will print the value of LDAP_PASS if it is set in your environment.

## View Environment Variables in a File
Environment variables are often set in configuration files like .bashrc, .bash_profile, or .profile. To check if an environment variable is set in these files, you can use grep to search for them.

For example, to find any mention of LDAP_PASS in your .bashrc:

```bash
grep LDAP_PASS ~/.bashrc
```
This will search the .bashrc file in your home directory for any lines containing LDAP_PASS.

## List Shell Variables
To list all shell variables (which may not necessarily be environment variables), use:
```bash
set
```
This will display both environment variables and shell variables, so it might show more than just the environment variables.

## Check for Exported Environment Variables
If you're trying to see if an environment variable has been exported to the environment (i.e., available to all child processes), you can check by using:

```bash
export
```
This command lists all exported environment variables, including those defined in your session.

## Check for Environment Variables in System-Wide Files
System-wide environment variables (those set for all users) are often found in /etc/profile, /etc/environment, or /etc/bash.bashrc. You can check these with:
```bash
cat /etc/environment
# Or for bash specifically:
```

```bash
cat /etc/profile
```
Let's say you set an environment variable LDAP_PASS like this in your ~/.bashrc:

```bash
export LDAP_PASS="your password"
```
```bash
echo $LDAP_PASS
```
This will return:

```css
your password
```
If you don’t see the variable or it’s empty, ensure that you've sourced your .bashrc file after making changes:

```bash
source ~/.bashrc
```
This reloads the .bashrc file to apply any changes.



