# Password-Audit-Active-Directory-Hardening-Lab-Hydra-John-the-Ripper-Domain-Password-Policy-
This lab involved hands on credential auditing by attacking a target over SMB, dumping domain password hashes, and cracking them using different John the Ripper attack modes. I then used PowerShell to strengthen the domain password and account lockout policies on the domain controller.

Password Audit & Active Directory Hardening Lab

Kali Linux · Hydra · John the Ripper · SMB/CIFS · Windows Server / Active Directory · PowerShell

Overview

This lab was hands on practice with credential auditing. I attacked a target over SMB to gain access, retrieved a dump of the domain password hashes, and cracked as many as possible using different John the Ripper attack modes. I then switched to the defender side and used PowerShell to strengthen the domain password and account lockout policy on the same domain controller.

I have documented the real process here, including the incorrect flags, file path mistakes and permission issues I came across, rather than only showing the final successful results. I also left the actual password hashes out of this write up because the focus is on demonstrating the workflow rather than publishing sensitive hash material.

Objective

The aim was to simulate a realistic password security audit. I used a discovered credential to access the network, retrieved the domain password hashes, tested them against both dictionary and brute force attacks, and then used the results to make an actual security improvement to the domain password policy.

Environment

Attacker: Kali Linux VM with a root shell

Target: DC10 (DC10.ad.structureality.com), Windows Server domain controller

SMB Share: HR on 10.1.16.2

Wordlist: SecLists, specifically xato-net-10-million-passwords.txt

Cracking Tool: John the Ripper using an NTLM hash dump

What I Did

Building the Attack Wordlists

I created a username list containing the domain accounts I wanted to test against SMB. I also created an initial password list using common passwords before moving on to a much larger SecLists dictionary.

While setting everything up, I made a file path mistake by using /usr/share/seclist/ instead of /usr/share/seclists/. Reading the error and correcting the path was a useful reminder to pay attention to what the terminal is actually telling you.

Credential Attack Using Hydra

I used Hydra against the SMB service on 10.1.16.2 with my username and password lists.

Hydra identified two valid credential pairs using the same password. This showed how password reuse can quickly become a security issue.

I first tried accessing the HR share with an account that did not have permission and received an access denied error. I then used the valid credentials discovered by Hydra and successfully mounted the share. This gave me access to files including ACCEPTABLE USE POLICY.pdf, EMPLOYEES.csv and TERMS.docx.

Cracking the Domain Password Hashes

I then used John the Ripper against the NTLM password hash dump with the large SecLists wordlist.

My first attempt failed because I accidentally typed --fromat=NT instead of --format=NT. After correcting the flag, the dictionary attack successfully cracked 11 of the 17 hashes very quickly.

I used john --show=left --format=NT to identify which accounts were still uncracked rather than assuming the first attack had finished the job.

Incremental Brute Force Attack

For the remaining accounts, I switched John to incremental mode. This allowed John to try different character combinations instead of relying on a wordlist.

I first limited the attack to six characters to keep the runtime manageable. This cracked another account with a short password.

I then cleared John’s saved results and ran a full incremental attack to get a clean result. This brought the total to 12 out of 18 hashes cracked, leaving six accounts that survived both attacks.

Active Directory Hardening

After completing the attacks, I switched to the defender side and checked the existing domain password policy using PowerShell and Get-ADDefaultDomainPasswordPolicy.

I then used Set-ADDefaultDomainPasswordPolicy to strengthen the policy.

Setting	New Value
Minimum password length	12 characters
Minimum password age	3 days
Maximum password age	365 days
Lockout threshold	3 failed attempts
Lockout duration	15 minutes
Lockout observation window	15 minutes

After making the changes, I ran Get-ADDefaultDomainPasswordPolicy again to confirm that the new settings had been applied successfully.

Skills I Picked Up

I learned how to track John the Ripper’s results across multiple attacks and understand the difference between dictionary and incremental attacks.

I also learned how password cracking results can be used to justify actual security improvements rather than simply reporting which passwords were weak.

Another important lesson was troubleshooting. I made mistakes with command flags and file paths, but reading the errors carefully helped me understand and fix the problems.

I also made sure not to include the actual password hashes in the public write up. This reinforced the importance of handling sensitive security information responsibly.

How This Applies in the Real World

This lab demonstrates part of what can happen during an internal security assessment. A security professional may test whether weak credentials can be used to gain access, assess password strength and then recommend changes based on the findings.

The biggest lesson from this lab was how quickly a dictionary attack could crack most of the passwords. It showed that weak passwords do not necessarily require sophisticated techniques to compromise.

The password policy changes also demonstrated the connection between offensive and defensive security. The attack showed the weakness, while the policy changes provided a practical way to reduce the risk.

What I Want to Learn Next

I want to learn rule based cracking with John the Ripper and Hashcat, explore GPU accelerated cracking and test the new account lockout policy against another controlled brute force attempt.

I also want to learn how to extract credentials using tools such as secretsdump and eventually turn the results from labs like this into proper security assessment reports.

Limitations

The hash dump was provided rather than extracted by me from the beginning, so I did not complete the entire credential extraction process myself.

The first incremental attack was limited to six characters, meaning the results were limited by the amount of time I was willing to let the attack run.

I also only tested one main wordlist. A real security assessment would normally use multiple wordlists and potentially organisation specific information.

Finally, I confirmed that the new lockout policy was applied, but I did not test it against a live brute force attempt afterward. That would be something I would improve in a future lab.

Where I’m Coming From

I’m making the move into cybersecurity from a healthcare background. I’m currently studying for CompTIA Security+ and building hands on labs to develop practical experience alongside my studies.

I’m still early in my cybersecurity journey, so I have deliberately kept the mistakes in this write up. The incorrect flag, wrong file path and troubleshooting are all part of the learning process. I want my projects to show not only the final result but also how I worked through problems along the way.

References

John the Ripper Documentation

Hydra GitHub

SecLists GitHub

Microsoft Active Directory PowerShell Documentation

Microsoft SMB Documentation

NIST Digital Identity Guidelines

CompTIA Security+ Exam Objectives

Kali Linux
