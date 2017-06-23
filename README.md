# pihole-scheduler
Change PiHole upstream DNS server on schedule

## Why?
I like the theory of blocking Wifi during certain times. Google Wifi has this feature, as do many routers. It enables the user to have "family time" or "homework time," stuff like that.

But it has downsides. During no-wifi time, iPhones can send/receive iMessages but not with photos. Apple notifications are enabled, but email is blocked. And Google is important during homework time, but Facebook is not.

What I want:

* Certain times "all Wifi" is blocked to a device (use case: tech sabatical, iPhones/iPads/computers are blocked; music devices are not)
* Other times, block time-wasting websites to all devices (use case: family time, homework)
* Else, enable all internet to all devices

## Theory

* Use Google's "Family Time" (or similar router technology) to enforce first option
* Use a very restrictive OpenDNS to enforce second option
* Use an unfiltered DNS provider for the third option

## Setup
First, record your current Pihole settings:
```
cat /etc/pihole/setupVars.conf
```

Translating that file to the below command lines:

* DNS_FQDN_REQUIRED -> domain-needed (false: domain-not-needed)
* DNS_BOGUS_PRIV -> bogus-priv (false: no-bogus-priv)
* DNSSEC -> dnssec (false: no-dnssec)

Using the above, create a command line to apply your current settings:
```
pihole -a setdns 208.67.222.222,208.67.220.220 domain-needed bogus-priv no-dnssec
```

Then, add those settings to a cron job:
```
crontab -e
## Locked down internet
0 1 * * * PATH="$PATH:/usr/local/bin/" pihole  -a setdns 208.67.222.222,208.67.220.220 domain-needed bogus-priv no-dnssec

## Open the internet
0 4 * * * PATH="$PATH:/usr/local/bin/" pihole  -a setdns 8.8.8.8,8.8.4.4 domain-needed bogus-priv no-dnssec

```

The above example runs at 6pm PDT, sets the upstream DNS to OpenDNS with "never forward non-FQDNs" and "never forward reverse lookups for private IP ranges" enabled. At 9pm PDT, it sets upsteam DNS to Google.


## Troubleshooting
Verify crontab settings:
```
## Get current date
date

## Edit crontab for a minute or two in the future
crontab -e
54 16 * * * date >> /home/pi/test.txt

## Verify crontab worked
cat /home/pi/test.txt
```
