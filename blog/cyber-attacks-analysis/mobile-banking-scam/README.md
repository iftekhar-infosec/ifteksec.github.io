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

### 2.3 Attack Attribution & Actor Profiles

The threat actors operating these campaigns range from organized criminal groups with technical infrastructure to individual low-skill operators using commercially available Remote Access Trojan (RAT) kits. The distinguishing characteristic of this threat class is that it does not require nation-state-level sophistication. Pre-packaged malware toolkits, including SpyNote, AhMyth, AndroRAT, and DroidJack, are commercially available or freely distributed on dark web forums, lowering the barrier to entry dramatically.

Some campaigns exhibit characteristics of organized operations: professionally crafted social media advertisements, localized Bengali-language lure content, dedicated command-and-control (C2) infrastructure, and automated OTP harvesting scripts, suggesting at least partial professionalization of the criminal operation.

## 3. Attack Vector Analysis

### 3.1 Stage 1 — Social Engineering & Initial Lure

The attack chain invariably begins on social media, most commonly Facebook, due to its dominant market penetration in Bangladesh. Threat actors deploy paid advertising campaigns or organic posts designed to appeal to specific socioeconomic motivations. Common lure archetypes include:

- Government subsidy or cash transfer applications (exploiting familiarity with programs like Tk 2,500 government allowances)
- Emergency microloan applications promising instant disbursement with no collateral
- Recharge offer applications claiming free mobile data or talktime
- Job application portals for government or NGO positions
- Utility bill management apps
- Fraudulent versions of legitimate services (fake bKash agent apps, fake Nagad tools)

The visual design of both the advertisements and the resulting landing pages is frequently sophisticated, incorporating logos, color schemes, and UI elements from legitimate government agencies or well-known financial brands. This represents a deliberate brand impersonation strategy designed to override the victim's skepticism.

#### TECHNICAL NOTE — AD TARGETING

Threat actors leverage Facebook's own targeting infrastructure to precisely identify high-probability victims: users in lower-income demographic segments, users who have recently interacted with MFS-related pages, users in districts with high remittance activity (Sylhet, Comilla, Chittagong), and mobile-only internet users — all indicators that the target is both financially active via mobile and potentially less technically sophisticated.

### 3.2 Stage 2 — APK Distribution & Sideloading

Upon engaging with the advertisement, the victim is redirected to a web page — never the official Google Play Store — that prompts the download of an Android Package file (.apk). This is the critical technical enabler of the entire attack: Android, by design, supports the installation of applications from sources other than the official Play Store. This feature, known as *sideloading*, is intended for legitimate developer use but is routinely exploited for malware distribution.

The installation process requires the victim to enable 'Install from Unknown Sources' or grant permission to 'Install Unknown Apps', a warning that Android displays explicitly. Threat actors circumvent this by providing in-page instructions that frame this permission as a necessary technical step for the application to function, neutralizing the security warning through social engineering.

#### 3.2.1 Disguise and Persistence Mechanisms

- The malicious application is frequently functional on the surface, performing some version of its advertised purpose to delay suspicion.
- The application icon may be changed to a system-looking icon or hidden entirely after installation.
- Some variants request Device Administrator privileges immediately upon installation, which prevents the user from uninstalling the app through normal means.
- The malware process is configured to auto-start on device boot, ensuring persistence across restarts.
- In more sophisticated variants, the malware periodically checks in with a C2 server to receive updated configuration, allowing the operator to modify its behavior or update evasion techniques remotely.

