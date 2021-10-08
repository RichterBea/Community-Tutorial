---
SPDX-License-Identifier: MIT
path: "/tutorials/tutorial-template"
slug: "tutorial-template"
date: "2021-01-01"
title: "How to install Fail2ban on Ubuntu 20.04"
short_description: "This is a tutorial explains how to install and configure Fail2ban on Ubuntu 20.04"
tags: ["Linux", "Install", "Ubuntu"]
author: "Beatrice Richter"
author_link: "https://github.com/RichterBea"
author_img: "https://avatars3.githubusercontent.com/u/....."
author_description: ""
language: "en"
available_languages: ["en"]
header_img: "header-x"
cta: "dedicated"
---

## Introduction

Fail2ban is used to detect unnormal login activity by monitoring log files or ports. 
In these tutorial we will configure Fail2ban to work with the system log files.

You can scan log files and count all entries that matching the regular expression patterns.  When their number reaches a predefined threshold, Fail2ban bans the IP for a specific timespan.
For this it uses the configured firewall, for this tutorial we will use iptables.


Example terminology that you can use in the tutorial:
* a root user
* a configured firewall
* 
* Username: `holu` (short for Hetzner OnLine User)
* Hostname: `<your_host>`
* Domain: `<example.com>`
* Subdomain: `<sub.example.com>`
* IP addresses (IPv4 and IPv6):
   * Server: `<10.0.0.1>` and `<2001:db8:1234::1>`
   * Gateway `<192.0.2.254>` and `<2001:db8:1234::ffff>`
   * Client private: `<198.51.100.1>` and `<2001:db8:9abc::1>`
   * Client public: `<203.0.113.1>` and `<2001:db8:5678::1>`

## Step 1 - Installing Fail2ban

You are able to install Fail2ban via apt. 

```bash
sudo apt update
sudo apt install fail2ban
```

Now you have Fail2ban installed and the Fail2ban service will start automatically. YOu can check this with

```bash
systemctl status fail2ban
```

## Step 2 - Configure Fail2ban

Now we need the config files of Fail2ban:
* /etc/fail2ban/jail.conf
* /etc/fail2ban/jail.d/*.conf
* 
You should copy the config files to `.local` files and modify these files. 
Because the `.conf` files will be overwritten when the package is updated. 

### Step 2.1 - jail.local

In these file you can add every IP address you want to white list. For example:

```bash
ignoreip = 8.8.8.8 ; google
           127.0.0.1 ; localhost
```

Also the Fail2ban jails will be defined here. 

For every service you want to observe you need to define one jail. A jail includes name of service, specially defined filters and actions.

We want to observe the auth.log for ssh login attempts. The jail is implemented as follows:  

```bash
[sshd]    # name of jail
enabled = true
logpath = /var/log/auth.log   # you can add more than one logfile to observe
filter = sshd-filter    # the filter works with regex
bantime = 1800    # in seconds
findtime = 10m
maxretry = 5
action = jail-action
```

Without suffix the ban time default is by seconds. You can use suffix like 1d or 10m (which is the fail2ban default). 

The files where the actions are defined under `actions.d/`

### Step 2.2 - jail-action.conf

The jail action contains the instructions what to do with IP addresses which are matched by the filters.
These actions can be used for all jails.

We want to implement the action as follows:

```bash
[Definition]
actionban = iptables -A INPUT -s <ip> -j DROP
actionunban = iptables -D INPUT -s <ip> -j DROP
```


### Step 2.3 - sshd.conf

The filter are defined under `filter.d/`
For this tutorial we will define a simple filter. Within the `auth.log` we can find many entries like as following:

```bash
Failed password for root from x.x.x. port 58380 ssh2
Failed password for root from x.x.x.x port 64608 ssh2
Failed password for root from x.x.x.x port 62142 ssh2
```

So our filter for ssh could look like this: 

```bash
[Definition]
failregex = Failed password for root from <ip> 
```




## Conclusion

A short conclusion summarizing what the user has done, and maybe suggesting different courses of action they can now take.

##### License: MIT

<!--

Contributor's Certificate of Origin

By making a contribution to this project, I certify that:

(a) The contribution was created in whole or in part by me and I have
    the right to submit it under the license indicated in the file; or

(b) The contribution is based upon previous work that, to the best of my
    knowledge, is covered under an appropriate license and I have the
    right under that license to submit that work with modifications,
    whether created in whole or in part by me, under the same license
    (unless I am permitted to submit under a different license), as
    indicated in the file; or

(c) The contribution was provided directly to me by some other person
    who certified (a), (b) or (c) and I have not modified it.

(d) I understand and agree that this project and the contribution are
    public and that a record of the contribution (including all personal
    information I submit with it, including my sign-off) is maintained
    indefinitely and may be redistributed consistent with this project
    or the license(s) involved.

Signed-off-by: [submitter's name and email address here]

-->
