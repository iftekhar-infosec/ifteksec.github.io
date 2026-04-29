# A Technical Breakdown of Mobile Banking Scams in Bangladesh

## 1. Executive Summary

A sophisticated and rapidly escalating cybersecurity threat is currently targeting mobile banking users across Bangladesh. The attack vector exploits the convergence of social media advertising platforms, permissive Android application architecture, and human behavioral vulnerabilities to compromise mobile banking accounts, including those protected by biometric authentication, without the victim's knowledge.

This report presents a comprehensive technical analysis of the attack mechanism, covering the full infection lifecycle from initial social engineering through to fraudulent financial transfer. It further examines why conventional security measures, including fingerprint locks, bank-issued OTPs, and platform-level protections, fail against this class of attack, and provides a stratified mitigation framework for individuals, financial institutions, and regulatory bodies.

## 2. Threat Landscape in Bangladesh

### 2.1 Socioeconomic Context

Bangladesh has experienced one of the fastest trajectories of mobile financial service (MFS) adoption in South Asia. Platforms such as bKash, Nagad, Rocket, and conventional mobile banking apps from institutions including Dutch-Bangla Bank, BRAC Bank, and Islami Bank command tens of millions of active users. As of 2025, bKash alone reported over 65 million registered accounts, the vast majority of which are accessed exclusively through smartphones.

This explosive growth has occurred in a population where digital literacy has not kept pace with adoption. A large segment of the user base consists of individuals who are accessing internet-connected financial services for the first time, often through inexpensive Android devices running outdated operating system versions that lack critical security patches.

### 2.2 Why Bangladesh Is a High-Value Target

| Risk Factor	| Analysis |
|-------------|----------|
| Large MFS User Base | Tens of millions of accounts with real monetary value, high transaction volumes, and low average technical literacy among users. |
| Outdated Android Versions	| A significant proportion of devices run Android 8 or 9, lacking modern background process restrictions introduced in Android 10–14. |
| Facebook Penetration	| Facebook is the dominant internet platform in Bangladesh, with over 50 million users, making it the most effective malware distribution channel. |
| Limited Regulatory Enforcement | Compared to developed nations, there is limited capacity for rapid takedown of malicious APK distribution infrastructure. |
| Single-Device Dependency	| Most users operate a single smartphone that serves simultaneously as their banking terminal and OTP receiving device (a critical architectural vulnerability). |
| Low Incident Reporting	| Victims frequently do not report incidents due to shame, distrust of institutions, or lack of awareness that a crime occurred. |
