---
title: Sysinternals case writeup 
classes: wide
categories:
  - Network Forensics
tags:
  - Forensics
  - Malware Analysis
---

This is a small writeup for the [sysinternals case](https://www.ashemery.com/dfir.html#Challenge7) recently published by Ali Hadi.  

<!-- more -->

The case starts after donwloading a sysinternals tool that is suspected to be a malware since it didn't behave as expected and the user noticed a slowness in the system. To start the case analysis, file system and evidence of execution artifacts are the first two traces to start analyzing. MFT is a good candidate for file creation and amcache or prefetch for evidence of execution. 

MFT showed that two files with the name sysinternals.exe were created, first file was clean and is likely a currpted since it doesn't contain MZ header. And second file was malicious and resulted in 32 hits on [VirusTotal](https://www.virustotal.com/gui/file/72e6d1728a546c2f3ee32c063ed09fa6ba8c46ac33b0dd2e354087c1ad26ef48/behavior
). 

|   File Path | Hash  |
| ------------- | ------------- |
| Users\Public\Downloads\sysinternals.exe  | EE18B3A542E2C27AB8E7506BC4B39379  |
| Users\IEUser\AppData\Local\Packages\Microsoft.MicrosoftEdge_8wekyb3d8bbwe\AC\#!001\MicrosoftEdge\Cache\WMFWC1O7\sysinternals[1].exe  | D1A27B871A86C5371215F71885862CFF  |


 ![Virustotal result](/imgs/sysinternals[1].exe_result.png)
 
 
Looking at the sysinternals[1].exe using PEStudio shows the use of windows functions such as URLDownloadToFileA, InternetOpenUrlA and ShellExecuteA, which is a good indcation to look for another sample possibiley downloaded form internet. 

Following are Intersting strings inside sysinternals[1].exe 

- /C c:\Windows\vmtoolsIO.exe -install && net start VMwareIOHelperService && sc config VMwareIOHelperService start= auto
- c:\Windows\Temp\Hex2Dec.zip (there is no evidenec that the file was created on disk)
- c:\Windows\vmtoolsIO.exe


Strings and code indicate the existence of other files, as well as installing a malicious service called VMwareIOHelperService. 

 ![c code of sysinternals[1].exe](/imgs/sysinternals_code.png)

Next step is to look for network communication to know from where those files were downloaded. Using a sandboxing solutions is quick win to study the malware behaviour as well as to get a copy of PCAP. 
Fortunately the sample was available in Virustotal with the malware behaviour report. Virusttotal shows many maliciuos TTPs. Hosts file modification was interesting. Disk image hosts file was indeed modified. 

 ![Hosts file](/imgs/sysinternals_hosts_file.png)


 ![sysinternals[1].exe pcap](/imgs/sysinternals[1].pcap.png)

Since the domain was locally hosted, it won't be possible to setup the same server. Luckily the file vmtoolsIO.exe is still avaiable at c:\Windows.

From the strings of sysinternals[1].exe we can run the vmtoolsio.exe with commandline -install to cause the service installation. Once

Vmtoolsio.exe installs a service that deletes all prefetch file and keep deleting any created prefetch.   

 ![vmtooslio deleting prefetch files](/imgs/vmtoolsio.png)





