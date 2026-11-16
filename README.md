# Security Write-ups

A collection of personal notes and permitted write-ups from hands-on cybersecurity training and lab exercises.

## Overview

I'm developing my offensive security skills through Hack The Box Academy's **Certified Penetration Testing Specialist (CPTS) learning path**, along with other practical security labs.

This repository will document my approach to enumeration, vulnerability analysis, exploitation in authorised environments, and remediation. It is a work in progress; write-ups will be added as exercises are completed and publication is permitted.

> **Certification status:** CPTS learning path in progress. I do not currently claim to hold the CPTS certification.

## Write-ups

### Hack The Box Write-ups

| Machine | Difficulty | OS | Status | Write-up |
|---|---|---|---|---|
| Nibbles | Easy | Linux | Completed | [View](hackthebox/nibbles.md) |
| Footprinting | Easy | Linux | Completed | [View](hackthebox/footprinting-easy.md) |
| Footprinting | Medium | Windows | Incomplete | [View](hackthebox/footprinting-medium.md) |
| TwoMillion | Easy | Linux | Incomplete | [View](hackthebox/twomillion.md) |

## Write-up format
1. Overview: Machine name, difficulty, operating system, completion status, and a short attack-path summary.
2. Reconnaissance and enumeration: Actual commands executed, relevant outputs, discovered services and their significance.
3. Vulnerability identification: How you identified a weakness, why you believed it was exploitable and what evidence confirmed it.
4. Initial access: The steps that resulted in access, including the relevant commands and successful output.
5. Privilege escalation: The enumeration, misconfiguration or vulnerability that enabled elevated privileges, with evidence of the resulting access.
6. Remediation: Specific changes that would prevent the vulnerabilities you identified.
7. Lessons learned: What worked, what failed, and what you would approach differently.

## Areas of focus

- Network and service reconnaissance
- Web application security testing
- Vulnerability assessment and exploitation
- Linux and Windows security fundamentals
- Privilege escalation in controlled lab environments
- Remediation and lessons learned

## Tools I'm learning

- Nmap
- Burp Suite
- Metasploit
- Linux command-line utilities

## Publication policy

I will only publish material that the relevant platform permits me to share. This includes respecting Hack The Box and other platforms' rules concerning active machines, Academy exercises, proprietary training material, solutions, and flags. Any findings involving real systems will be shared only with appropriate authorisation.

The content is for educational purposes and authorised security testing only.
