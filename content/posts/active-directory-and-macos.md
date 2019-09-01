---
title: "Active Directory and MacOS"
date: 2019-06-20T22:00:16-07:00
draft: true
tags: [
        "pentesting",
        "red",
        "research",
        "tool"
    ]
---


## Interacting with Active Directory on the Mac

Did you ever have to interact with Active Directory on a MAC? 

If yes, this post might be interesting for you. I am pretty new to the Mac and basic things I know how to do on Windows need some research to figure out. This time around I explore Active Directory/LDAP Server interactions.

* First, there is the *Directory Utility* on MacOS which can be quite useful.
* Second, there is Apache's - *Directory Studio* - which is pretty amazing and feature rich.
* Third, you might want to write your own tools or scripts. 


There are ldap commands that allow you to do most tasks in automated fashion.

    ldapwhoami -x -Z -H ldaps://[your].[domain].[controller] -D wuzzi@domain.com -W

To perform a search the ldapsearch command is useful:

    ldapsearch -v -x -LLL -H ldaps://[your].[domain].[controller] -b OU=Users,OU=Managed,DC=[your],DC=[domain],DC=[controller] -D wuzzi@domain.com -Z -W -s sub "(objectClass=user)" cn givenName sn pwdLastSet

## Certificates
To trust the certificate of the LDAP server look into updating krb5.confi/kerb.conf files accordingly.

## Procesing Results
One can pipe this into a file, let's say users.ldif

There is a tool ldap2csv.sh (google for it) that can be used to convert the output to a csv file - I found this pretty useful.

    cat users.ldif | ./ldap2csv.sh cn givenName sn pwdLastSet samAccountName

## Search Filters

As part of the ldapsearch command replace 

    -s sub "(objectClass=user)"

with
    
    -s sub -f searchfilter.filter "(cn=%s)"
 
The *filter file* (in above case searchfilter.filter)  then just contains a list of names per line that will be substituted.
