# dns_notify
## Summary
Script to monitor and alert of DNS changes (Specifically for UC Davis DNS records)

## Files
dns_notify: Main Script
CheckList: Configuration file

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
If deploying for non UC Davis DNS records, change the Bash variable `DNS_SERVER` to your primary DNS server.

If implemented elsewhere please change the email that is sent by editing the `MESSAGE` bash variable in the `send` function

### Updating CheckList
The CheckList file is formatted in blocks of domains/sysadmins that will be checked/notified. Each block of domains/sysadmins must be wrapped in three hyphens `---`. Subsequent blocks may share the hyphens. An example of a block is below:

```
---
www.ucdavis.edu
Sysadmin Name, sysadmin1@email.com
#Disabled Name, sysadmin2@email.com
Another, sysadmin3@email.com
---
```

Each block is composed of two sections: a Domain, and a list of sysadmins.

#### Domain section
The domain section is the first line right after the three hyphens `---`. This will be the domain that the dns_notify script checks.

If there are special arguements you wish to add you can put them right after it with a space seperating the domain from the arguements.

The two supported arguements are as follows:
(`T`): This option will omit checking the TTL for just this domain. Use this if you dont know the primary DNS server that holds the records
(`$ip`): This option will change the DNS server for this specific domain. It will not use the hardcoded server in the script.

These two arguements are mutually exclusive. If you have one you cannot have the other

Here are the examples of how the domain section should be formatted:
```
ucdavis.edu
ucdavis.edu T
ucdavis.edu 8.8.8.8
```

#### Sysadmin Section
The Sysadmin section will follow right after the Domain Section. This is a list of sysadmins that will be notified of any changes.

The Sysadmin section must follow the following rules
- Each sysadmin must be on his/her own line
- The sysadmin list for any given domain must be in this format `Name, Email`.
	- The sysadmin name and email cannot contain commas (`,`) or colons (`:`).
	- The sysadmin name can have spaces. You can put First/Last or just an ID here. It will not be used in the script. It is just a placeholder to help you identify who they are.
	- You must have something in the name section even if it is meaningless

Examples of valid lines of the Sysadmin List:
```
First Last, sysadmin1@email.com
AnotherSysadmin, sysadmin2@email.com
A Third Sysadmin, sysadmin3@email.com
This System Admin Has A Really Long Name, sysadmin4@email.com
```

#### Disabling checks/sysadmins
If you wish to permanently disable any checks/sysadmins simply delete them from the CheckList file. If you wish to temporarly delete them you can comment them out bash style. Some rules are below:

- If you wish to temporarily remove a sysadmin or domain, you can comment it out bash style (`#` at the beginning of the line). Disabled sysadmins will not recieve emails.
- If you wish to temporarily remove a domain you will need to comment out the domain as well as it's accompanying sysadmins. Disabled domains will not be checked.
- You must have at least one active sysadmin per active domain.



#### Example Checklist
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
**-t**: No TTL: Enabling this flag will omit the TTL from any DNS checks. Use this if checking DNSs that arent on the specified DNS server.
**-v**: Verbose mode: Prints out all command outputs"

## Updating DNS entries
If any legitamate DNS changes are made to the monitored domains, you will need to update the stored DNS entries. To do this just go into the domains folder that is in the same path as the script and delete the file that has the same name as the domain with the updated dns.

You can then run the dns_notify script to pull the new DNS records or just wait and let cron do it automatically.

## Other Notes
This script will ignore RRSIG and SOA records as they are automatically generated by infoblox and change often.
