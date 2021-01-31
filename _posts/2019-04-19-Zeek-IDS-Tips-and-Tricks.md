---
title: Zeek Notes
classes: wide
categories:
  - Network Forensics
tags:
  - Zeek
  - Network Forensics
---

I have been using Zeek for my personal projects for sometime, I will write some Notes for using it and plan to update this list as I get into the advanced capabilities of Zeek.

<!-- more -->

### Note 1

When you write Zeek scripts, test them by executing in the command line. Use the command **bro -i interface scriptname.bro** You will be able to debug them on the fly, also you can see the errors if any in reporter.log file.

### Note 2

When you prepare your intel feeds to be used with Zeek IDS, you will have to make them tab separated (delimiter). It will be very difficult to know where you missed one tab, since nano, gedit and many editors have different tab distance between words you will go crazy trying to find out what is not right in your intel file. To be able to spot the missing tab easily use the command **cat -T feed.txt**.

### Note 3

Set expiry time for intel feeds if you are reading them from threat intelligence source, and set a cron job to periodical read the source looking for new entries.

### Note 4

In the feed file, if you want to log all intel hits, https URLs won’t work ( I had hard times on this, maybe only me) You should make the intel type **Intel::DOMAIN** and provide the indicator without http or https. 

Example:

> #fields indicator indicator_type meta.source meta.desc meta.url
>
> bad.com Intel::DOMAIN mysecret_source secretIntel.com 

Above URL with https and http will log the event in intel.log

### Note 5

If you have a bad URL in your feeds, you will notice intel.log will log many entries and that is normal because it logs each request and each response and some other things, see the meta.if_in. You can limit the number of entries in the log by specifying the meta.if_in in your feeds file header and limit it to one thing. Another option is to use the meta.do_notice which tells Zeek IDS if you get a match log this also in notice.log. You will get only one entry in notice.log.

### Note 6

you can use Zeek IDS as a parsing tool for offline pcaps. I use following command to parse pcaps and generate logs.
  
```sh 
bro -r file.pcap script.bro
```

### Note 7

In real production environment you may have to deploy your IDS properly and use network tap or span port to get the traffic to the IDS. Sensor location in the network is critical, have a look on the white paper [Network IDS & IPS Deployment Strategies](https://www.sans.org/reading-room/whitepapers/intrusion/paper/2143).

### Note 8

if you need to generate logs and you don’t have access to your system you could do that online [https://try.bro.org/](https://try.bro.org/) don’t do that with pcap that has sensitive data.

### Note 9

Now you can use [Brim tool](https://www.brimsecurity.com/) to generate and analyze Zeek logs. It supports Zeek and Suricata. The tool basically reads a pcap file and parse it to zeek log and scan it with Suricata.


![Brim tool](/imgs/brim_interface.jpeg)

![Log flow](/imgs/brimsecurity_log_flow.jpeg)
