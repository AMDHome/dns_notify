# dns_notify
## Summary
Script to monitor and alert of DNS changes (Specifically for UC Davis DNS records)

## About
./dns_notify is a script that is meant to be run by a cron job with no arguements to monitor the dns for important UC Davis domains. The first time it runs on a specific domain, it will save a copy of the DNS records as reported by dig. On consecutive runs it will run the current DNS records against the one it has on file. If there are any differents in the DNS records it will email the specified sysadmins. 

All domains that will be checked along with their sysadmin's email will be stored in the file CheckList.

## Setup

### Prerequisites
Before running this program, make sure the following bash programs/commands are installed and set up.
- dig
- ssmtp (Setup with SMTP server)
- sendmail
- sed/awk/diff

### Setup
To use this program just clone it into a directory and then open up crontab as root (`cron -e` or `crontab -e`). Once there add an entry to run the script however often you wish. The example below will run it once every 30 minutes at the 00 and 30 minute of every hour:

```
0,30 * * * * /root/dns_notify/dns_notify
```

If you use the cron entry above make sure you change the path to the location where you cloned dns_notify to.

Modify the Bash variable `CURRENT_DIR` (on line 21) to reflect the location of the cloned folder. 

### Updating CheckList
The CheckList file must follow the following rules
- Each domain and its corosponding sysadmin list must be incased with three hypens `---`.
- The domain must follow immediately after the three hyphens.
- The sysadmin list for any given domain must be in this format `Name, Email`.
	- The sysadmin name and email cannot contain commas (`,`) or colons (`:`).
	- The sysadmin name can have spaces. You can put First/Last or just an ID here. It will not be used in the script. It is just a placeholder to help you identify who they are.
- If you wish to temporarily remove a sysadmin or domain, you can comment it out bash style (`#` at the beginning of the line). Disabled sysadmins will not recieve emails.
- If you wish to temporarily remove a domain you will need to comment out the domain as well as it's accompanying sysadmins. Disabled domains will not be checked.
- You must have at least one active sysadmin per active domain.

should be in the following format.
```
---
domain1.ucdavis.edu
Sysadmin Name, sysadmin1@email.com
#Disabled Name, sysadmin2@email.com
Another, sysadmin3@email.com
---
domain2.ucdavis.edu
Sysadmin, sysadmin3@email.com
---
#disableddomain.ucdavis.edu
#First Last, sysadmin@email.com
#Second Sysadmin, sysadmin2@email.com
#---
domain.3.ucdavis.edu
Sysadmin, sysadmin1@email.com
---
```


## Usage
./dns_notify {-h} [-v] [-e | -E over_ride@email] [-d domain_name]

### Example
./dns_notify
./dns_notify -v -e
./dns_notify -E email@ucdavis.edu
./dns_notify -d subdomain.ucdavis.edu


## Options
**-d**: Specify domain: Only the specified domain will be checked. All other entries in CheckList will be skipped. The domain entered must already be in the CheckList file
**-e**: Email Override. This flag will override all email preferences stored In the checklist. All emails will be sent to the specified address. The sysadmins specified in the CheckList will not recieve an email. This flag will have no effect if the No Email flag (-n) is specified
**-h**: Help: Prints out this help screen
**-n**: No Email: Will not send any emails out. It will just print the email that would've been sent onto the screen
**-v**: Verbose mode: Prints out all command outputs"

## Updating DNS entries
If any legitamate DNS changes are made to the monitored domains, you will need to update the stored DNS entries. To do this just go into the domains folder that is in the same path as the script and delete the file that has the same name as the domain with the updated dns.

You can then run the dns_notify script to pull the new DNS records or just wait and let cron do it automatically.