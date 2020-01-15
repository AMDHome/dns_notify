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

## Usage
./dns_notify {-h} [-v] [-e | -E over_ride@email] [-d domain_name]

### Example
./dns_notify
./dns_notify -v -e
./dns_notify -E email@ucdavis.edu
./dns_notify -d subdomain.ucdavis.edu


## Options
**-d**: Specify domain: Only the specified domain will be checked. All other entries in CheckList will be skipped. The domain entered must already be in the CheckList file
**-e**: Email Override. This flag will override all email preferences stored In the checklist. All emails will be sent to the specified address. The sysadmins specified in the Checklist will not recieve an email. This flag will have no effect if the No Email flag (-n) is specified
**-h**: Help: Prints out this help screen
**-n**: No Email: Will not send any emails out. It will just print the email that would've been sent onto the screen
**-v**: Verbose mode: Prints out all command outputs"

## Updating DNS entries
If any legitamate DNS changes are made to the monitored domains, you will need to update the stored DNS entries. To do this just go into the domains folder that is in the same path as the script and delete the file that has the same name as the domain with the updated dns.

You can then run the dns_notify script to pull the new DNS records or just wait and let cron do it automatically.