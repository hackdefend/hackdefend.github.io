---
title: A tour inside a SOC analyst mind
layout: single
toc_label: "Contents"
toc: true
categories: 
  - Analysis 
tags:
  - SOC
---

As a mentor in a SOC, I often receive questions from junior analysts about the best way to approach analysis. To help them develop their skills, I sometimes ask them to come up with their own analysis before I offer my support. It's always interesting to hear their different conclusions and ways of thinking. Some analysts have strong analytical abilities, while others need to be aware of human thinking limitations and biases and how to avoid them to some extent.

SOC jobs are focused on analysis and require the application of decision-making skills that are impacted by the maturity of an individual's cognitive abilities. In this blog post, I will attempt to list some of the analysis pitfalls that I have observed during my interactions with analysts in a SOC.

Typically, a SOC will have a dashboard that contains a list of security alerts. In the first 10 seconds after looking at the dashboard, a complex thought process occurs in the analyst's mind to select the right alert for investigation. Like any human, this thought process is likely to have pitfalls. In general, good analysts have better decision-making skills and analytical thinking capabilities than average people. They base their moves on the rules of logic. Applying good analytical techniques is crucial for analysis because not all information will be available. Analysts have to go through layers of abstraction to build context and make decisions.

<br>

[![The Big Bang Theory, going to the movies](/imgs/The Big Bang Theory, going to the movies.png)](https://www.youtube.com/watch?v=SPZRtlqBgYk){:target="_blank"}

*Video about decision making*.

---

The process of selecting the right alert for investigation is a decision-making skill that is influenced by a combination of gut feeling and cybersecurity experience. Knowing the potential pitfalls that an analyst may unknowingly fall into can help to improve their thinking and analysis. To make it easier to understand how SOC analysts work, a typical alert analysis can be divided into three main phases:

1- Alert selection 

2- Analysis 

3- Security writing

Each phase has its own potential pitfalls, and the quality of the analysis can vary depending on the analyst's experience and metacognitive awareness. Some examples of alerts that may appear in a SOC detection platform are:

- A sudden spike in network traffic from a specific IP address
- An unusual number of login attempts from a particular user
- A suspicious file detected on a client's endpoint


Each of these alerts requires a different approach and decision-making process, and the analyst must be aware of the potential pitfalls and biases that may affect their analysis. Following is a list of alerts in a typical SOC dashboard, analysts would make a guess which alert is worth investigating and what to be ignored and this is where many mistakes are done. 



| Alerts      | 
| ----------- | 
| T1059.001 - Powershell cmdline obfuscation  | 
| Internal Port Scan|
| T1003.001 - ProcDump64.exe execution  | 
| User agent Log4j signature   | 
| Log4j Scanning |
| Log4j Scanning |
| Log4j Scanning |
| Log4j Scanning |
| Log4j Scanning |
| Log4j Scanning |
| Log4j Scanning |
| Log4j Scanning |
| Repeated IPS hits for same IP Address   |
| Malicious file detected by AV| 
| T1053 - Scheduled Task/Job installation | 

Following is a simple graph showing the possible mistakes an analyst might do during alert analysis.
<br>

![what happends in a SOC analyst mind](/imgs/analyst mind theater.png)


---
# Alert Selection

Alert selection is crucial and I always ask analysts, How they select the alerts and why. I always try to improve the level of understanding of the rules based on the careful observation of how alerts are selected.

## Alert familiarity 

Alerts should be selected based on risk, but analysts often choose alerts based on what they think they know how to analyze. In most cases, alerts are selected based on detection rules or data that the analyst is comfortable with. To reduce this pitfall, the rule on-boarding process should include training on newly deployed rules, and the rule information section should provide clear explanations and resources to help analysts understand the rules. Analysts may avoid selecting alerts for PowerShell obfuscation because they fear that it may be difficult to decode and analyze. Instead, they may choose AV alerts because they are easier to understand and escalate. Analyzing patterns in how alerts are selected can help identify knowledge gaps in the team.

## Inductive Reasoning 

When multiple alerts coincidentally fire at the same time, analysts may quickly generalize their conclusion about all alerts by only checking one or two. This is due to the static nature of alert titles, which can lead to erroneous conclusions. By being aware of this mistake, analysts can avoid quickly jumping to conclusions and ignoring alerts. From a platform perspective, alert titles should be dynamically derived from the content of logs to help prevent this issue.

## Availability Bias

Analysts often use their previous knowledge about the infrastructure or detection rule to make decisions. For example, fixed scanning schedules can cause many rules to fire. Without properly tuning the SIEM to avoid such alerts, analysts may ignore a set of alerts or behaviors, assuming that it is a scan because they are used to seeing these alerts. Improving the tuning of the SIEM can help prevent this issue.

Another example is the regularl uncoordinated pentesting activity, if it's known that pentester is working on some network without proper communication, alerts could be loosly analyzed due to a previous knowledge of some sort of activity is going on in the network. Applying this to availability bias to above alert list, many log4j scanning alert may be skipped thinking that it's the scheduled scan. 


# Analysis 

The same though process during alert selection repeated again during the first 10 seconds of the drill down. More difficult time expensive choices an analyst has to select.

## Asking the right question 

Analyst has to actively think about right question to be asked, knowing why/how the answer will help to ask the next question, and how all questions may lead to answering a hypothesis in mind. Senior analyst will have more hypotheses than junior analysts. The more hypotheses are generated the more questions will be asked. Prioritizing the list of hypotheses and the related question is what is so called good analysis choices. 

Chris Sanders research [CREATIVE CHOICES: DEVELOPING A THEORY OF DIVERGENCE, CONVERGENCE, AND INTUITION IN SECURITY ANALYSTS](https://www.chrissanders.org/wp-content/uploads/2020/03/Creative-Choices-Developing-a-Theory-of-Divergence-Convergence-and-Intuition-in-Security-Analysts.pdf) tries to understand how SOC analyst generate hypotheses and ask the related investigative questions.


## Weak correlation

It's suffice to say the well known phrase in analysis [Correlation does not imply causation](https://en.wikipedia.org/wiki/Correlation_does_not_imply_causation). Each move and conclusion should be built on a strong foundation of evidence that are derived by applying rules of logic. Cause-and-effect relationships plays a big role in analysis, experience in IT and and the network being protected. 


## Trying to prove the impossible 

A fundamental question to consider during any investigation is whether we need to prove or disprove a hypothesis. Knowing which option to choose can save a lot of time, and in some cases, proving a hypothesis may be impossible. Security systems are made up of layers of abstraction that can be leveraged to facilitate investigation. Breaking down the situation into smaller areas with relationships and illuminating the relationships between those areas by asking the right questions can help disprove hypotheses or establish a relationship to a particular scenario. For example, in our dashboard, if the logs are not clear about why procdump64.exe is used, an analyst may think that they need to prove that credential dumping occurred in the system and find evidence to support this. However, it may be easier to find evidence about why procdump64.exe was used instead.
 
## Anchoring and confirmation bias

During analysis, it's very hard to avoid the temptation to go in a new investigation path as new information are available. Analyst For example may think communication over port 4444 in Repeated IPS hits for same IP Address alert is metasploit where it's not necessarily the case. Avoiding the new investigative paths is important, whenever a new path is to be considered for analysis, the cost of going into the analysis in terms of time, and whether the overall hypothesis put is contributing in solving the case.  


# Security Writing

Good analyst should have clear thinking that is reflected on writing. The Ability to accurately write about the events timeline crucial. Analysts sometimes make the mistake to write content to prove their sophistication and deep analysis where it should have been an easy to write an actionable instructions only. in addition recommendations must be reasonably selected. 
