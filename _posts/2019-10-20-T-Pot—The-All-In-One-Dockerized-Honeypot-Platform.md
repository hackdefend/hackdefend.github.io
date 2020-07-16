---
layout: post
title: T-Pot — The All In One Dockerized Honeypot Platform

---


I am working on a project to gather online attack data for Intel analysis. The objective is to classify what is probing my network and what is probing everyone.

<br>

<!-- more -->


![T-Pot](/imgs/t-pot.png)

<br>

I will try to give a quick feedback for each honeypot I use or at least test. I searched online for open source honeypots projects to use them, I found ton of low interaction honeypots, mostly python scripts but they are not really useful. However, I use them to open the service port and provide the minimum interaction possible for each service. I rely most on Zeek or suricata logs running in the same honeypot server.

For this post, T-Pot ( The All In One Honeypot Platform ) which seemed very attractive to me with all services and NSM softwares. Project page on [github](https://github.com/dtag-dev-sec/tpotce).

T-Pot is a Debian server heavily dependent on docker and docker-compose. The project provides dockerized honeypots and bunch of pre-installed softwares such as ELK, Cockpit, Cyberchef, Spiderfoot, Suricata, p0f and others.

The server can be installed in several modes, each can customize the server to be suitable for different type of services. However, regardless what you select you can customize it later for the type of sensor you want to be running.

<br>

![t-pot installiation modes](/imgs/t-pot installiation modes.png)

<br>

## Memory usage

T-Pot needs at least 8 GB of memory. However, you can reduce the amount of memory by stopping some docker containers. Using the command docker stats I noticed that almost 3 GB of memory is consumed just by elasticsearch, logstash and Kibana. Suricat also eats some memory but it’s very essential for traffic logging, hence it can’t be disabled. These docker containers can be disabled when the server is up and running the could. When I visit the server I will start them to watch what is has been logged and disable them back.

Open the file ```/opt/tpot/etc/tpot.yml``` and change the settings **restart: always** to **restart: no** for the ones you don’t need to be running all the time.

In addition, there are other docker containers you don’t really need, in my case I disabled CyberChef, SpiderFoot, Head, heralding and adbhoney. As a result, I was able to reduce the memory from 8 GB to 3.5 GB. All honeypot will stay awake and log the attacks to log files at /data directory.

You can leave the system and come back to see what has been logged. You need to start elasticsearch, logstash and Kibana. Logstash will check the /data directory and push the new logs to elasticsearch.

## Logs Directory

T-Pot saves all the logs in /data directory. You can use Elk provided with the system or export outside the server. Log persistence configuration /opt/tpot/etc/logrotate/logrotate.conf.
Custom Honeypot

You can build your own honeypot. T-Pot provides a script to build an ISO image of your system.


## Stability

T-Pot needs a lot of work to maintain a degree of stability. I did use the system for a while but due to the problems I faced I will stop using it for now and keep watching the improvement of this great project.