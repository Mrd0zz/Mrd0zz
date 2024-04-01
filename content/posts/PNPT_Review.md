---
title: "PNPT Review 2024"
date: 2024-03-29
draft: false
categories:
  - Review
tags:
  - Active Directory Exploitation
  - Penetration Testing
  - OSINT  
---

I recently passed the PNPT exam and in this article I will share my views on the exam, course material and if you should take it or not.

<!--more-->

## Background

- I am an InfoSec Engineer and my day-to-day activities doesn’t include anything related to pentesting. I have played CTFs in the past and solved machines on HTB & THM. So, I have some knowledge about penetration testing limited to mentioned platforms. I can say this was my first time dealing with a penetration test followed by a professionally crafted report and 15 min debrief session.

## The Course Material

- The learning material provided by TCM Security includes five extensive courses - Practical Ethical Hacking, Windows Privilege Escalation, Linux Privilege Escalation, Open-Source Intelligence, External Pentest Playbook. I will describe each course in detail below. PEH, OSINT and EPP are sufficient to pass the exam, but in my opinion if you're new to pentesting field you should complete all these 5 courses and take detailed notes as WPE and LPE will give you more understanding of how things work and what other exploitation techniques are there. As I had knowledge about linux and windows priv esc I left out these courses.

  #### 1. Practical Ethical Hacking

  - This is the heart of this course covering most of the basics of penetration testing and ethical hacking. It also included basics of networking, linux and python scripting. It explains step-by-step the ethical hacker's methodology by going through each phases like reconnaissance, scanning & enumeration, exploitation, post exploitation, maintaining access and most important part - Report writing. To try your methodology there are 5 capstone challenges available with walkthoughs if you stuck on something. The best part I liked about this course was that we have to setup our own vulnerable AD environment giving an insight on how AD functions and how we can leverage these misconfigurations to our benefit. Heath also showcases 3 case studies related to AD which were very good and gives real word example how vulnerable organizations actually are. The course also contains Web application enumeration and exploitation which is not required for PNPT but helpful if you're preparing for another certifications (like eWPT, OSCP).      

  #### 2. OSINT

  - As I had previous experience playing CTFs and that includes OSINT section, so this was easy for me to follow along. It was a comprehensive course on OSINT and covered many tools and techniques to gather lot of data about someone just only by what is available online. 

  #### 3. External Pentest Playbook

  - This course focused on external pentest tactics and methodology giving an insight on how actual penetration tests are carried out in real world. It covers all the documentation (like RoE), and communication with the interested parties. It also gives tips on a professional report and checklists to follow during a pentest.   

## Preparation before the exam

- After completing the course I did two machines from Tryhackme - [Reset](https://tryhackme.com/r/room/resetui) and [Badbyte](https://tryhackme.com/r/room/badbyte). Badbyte covers port forwarding which was crucial for this exam. 

- Also I completed all the attacks mentioned in PEH 2-3 times to get familiar with AD attacks and made a sample pentest report just like the one provided in the material.

## The Exam

- You can directly start the exam by going to TCM Security exams page which will start your timer for 5 days and provide ovpn file with Rules of Engagement. 

- They also provide different wordlists to try in exam. I have seen many people failing in the OSINT plus External network part and not getting the required password. I also got the password after end of 1st day and let me tell you - the password is in one of these files but when I got it I banged my head against the wall as it was so realistic and there was no need of bruteforce. I used my bash skills to create a sorted and unique password list from the provided wordlist and finally got the password. It brought feeling of both happiness (for receiving the password) and sadness (for losing the precious first day over something simple).

- After getting initial access, paved my way through the external network and then to internal network. It took me another 3.5 days to compromise the Domain Admin. So in total 4.5 days I have completed my exam. Lateral movement in the internal network was easy if you focus on what's in front of you and try everything with that information. As I came from a CTF background and having that mindset I tried not to overcomplicate things and that helped me to easily pass the exam. **I can only say that everything needed to pass the exam is in the course material. Listen to what Heath says in every video carefully and keep that in mind during the exam. Also, if any reference to article or blog post is provided in the course material there is a reason for that.**      

## Reporting and Debrief

- As there was some time left after compromising DA, I started my report and ensured that I have relevant and enough screenshots along with proper findings as a proof of gainig DA. I made the report in the same format provided in the course material and this took me another day to finalize and submit my report.

- After my report was accepted, I needed to choose a time slot for the debrief session. I made a presentation of all my findings and remediation steps (kind of summary of report) and gave debrief, following which I received my confirmation that I have passed the exam. 

## Tips for the exam

- 5 days is more than enough time to complete the exam. Take frequent breaks, short walks and most stay hydrated.

- I cannot stress this enough, but don't overcomplicate things. **Everything required to clear the exam is covered in the course material.**

- Don't cram the course, understand it and try the attacks taught in the course. The exam doesn't test your exploitation skills but how you can leverage basic information and misconfigurations (that sysadmins generally do).

- Everything is easy. Don't make it complex.

## Should you do it

- In this price section, I think this is the best exam in the market. The course material is realistic and does cover everything from a professional pentest POV. While it may not yet have the same level of recognition as other certifications in the market, it should be on the list for aspiring penetration testers.

If you have questions please reach out to me. I will be happy to answer them.