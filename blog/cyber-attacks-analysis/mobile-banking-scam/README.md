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

### 3.3 Stage 3 — Permission Exploitation

The malware's operational capability is entirely dependent on the permissions it acquires from the victim. The following table details the critical permissions requested and their specific offensive applications:

| Permission | Risk Level	| Offensive Application |
|------------|------------|-----------------------|
| `Accessibility Service`	| CRITICAL	| Grants programmatic control over all UI elements across all applications. Enables reading screen content, simulating user inputs, intercepting text from any app, and detecting when specific applications are launched. This single permission is the cornerstone of the entire attack. |
| `Device Administrator`	| CRITICAL	| Prevents uninstallation through normal means. Survives factory reset attempts on some device configurations. |
| `Notification Access`	| HIGH	| Reads all incoming notifications in real time, including OTP delivery notifications from banking apps and SMS notifications, before the user sees them. |
| `READ_SMS`	| HIGH	| Directly reads SMS messages from the device's inbox, providing access to OTP messages even if notifications are dismissed. |
| `SYSTEM_ALERT_WINDOW` (Draw Over Apps)	| HIGH	| Renders invisible overlay windows on top of other applications. Used to display fake login screens over legitimate banking apps to harvest credentials. |
| `READ_CONTACTS`	| MEDIUM	| Harvests the victim's contact list for use in secondary attacks, spreading malware to contacts or enabling targeted social engineering. |
| `CAMERA / RECORD_AUDIO`	| MEDIUM	| Enables covert audio/video recording. Used for surveillance, verification bypass research, or extortion. |
| `INTERNET`	| MEDIUM	| Required for all C2 communication, exfiltrating harvested data, receiving commands, and streaming screen data to the attacker. |
| `RECEIVE_BOOT_COMPLETED` | LOW	| Ensures the malware process restarts automatically when the device is rebooted, maintaining persistence. |

## 4. Technical Breakdown of the Attack Chain

### 4.1 The Accessibility Service — The Master Key

The *Android Accessibility Service* was designed to assist users with disabilities by enabling screen readers and alternative input methods. However, its architecture provides an application with capabilities that are functionally equivalent to having complete, persistent access to the device's screen and input system. From a security perspective, granting Accessibility Service permission to an untrusted application is equivalent to handing the attacker a keyboard and a live camera feed of your screen.

Specifically, through the Accessibility Service API, the malicious application can:

- Register an `AccessibilityServiceInfo` listener that triggers on events across any application
-	Invoke `performGlobalAction()` to simulate the Back, Home, and Recents buttons
-	Call `performAction()` on specific UI nodes, equivalent to tapping any button on screen
-	Use  `findAccessibilityNodeInfosByText()` to locate and read text fields, including password fields in some implementations
-	Set text in input fields using `ACTION_SET_TEXT`, allowing automated form filling
-	Read the full UI tree of any open application, extracting account numbers, balances, and transaction details

#### CRITICAL TECHNICAL INSIGHT

Android's security model is fundamentally app-sandboxed: apps cannot normally read the memory or UI of other apps. The Accessibility Service is an officially supported exception to this rule, creating a legitimate API that provides exactly the cross-application access that an attacker needs. This is not a vulnerability in the traditional sense but a feature being weaponized.

### 4.2 Biometric Authentication Bypass — Three Techniques

The most counterintuitive aspect of this attack for laypeople is how biometric security — fingerprint and face recognition — fails to prevent the fraud. The answer is that the attacker never attempts to defeat the biometric lock. Instead, three distinct techniques are employed to work entirely around it:

#### 4.2.1 Technique A — Session Hijacking (Authenticated Window Exploitation)

Most mobile banking applications maintain an authenticated session for several minutes after the user closes the app (without logging out). This is by design, as requiring re-authentication on every screen transition would be unusable. The malware exploits this authenticated window:

-	The Accessibility Service listener detects when the user opens and uses the banking application
-	When the user closes the app, the malware notes the timestamp
-	Within the authenticated window (typically 3-10 minutes), the malware programmatically re-opens the banking application in the background
-	Because the session is still active, no re-authentication (and therefore no biometric) is required
-	The malware then uses Accessibility Service actions to navigate to the transfer function and initiate a transaction

#### 4.2.2 Technique B — Real-Time Remote Access (RAT-Based)

In more sophisticated deployments, the malicious application functions as a Remote Access Trojan, establishing a persistent connection to the attacker's C2 server and streaming the device screen in real time:

-	The attacker monitors the live screen stream, waiting for the victim to unlock the banking application using their fingerprint
-	Once the victim completes biometric authentication — legitimately — the attacker observes the active session
-	The attacker then takes direct remote control of the device, using the already-authenticated session to perform transfers
-	From the banking application's perspective, all interactions originate from the same device, through a legitimate session, initiated by legitimate user input — the fraud is architecturally indistinguishable from legitimate use

#### 4.2.3 Technique C — Overlay Attack (Credential Harvesting)

Using the `SYSTEM_ALERT_WINDOW` permission, the malware renders a pixel-perfect replica of the banking application's login screen over the real application. When the user attempts to open their banking app:

-	The overlay is displayed instead of the real app
-	The victim enters their PIN, MPIN, or password into the fake overlay
-	These credentials are transmitted to the attacker's server
-	The overlay dismisses itself, revealing the real app (or displaying a generic error)
-	The attacker now possesses the banking credentials directly, enabling independent access without needing the victim's device at all

#### 4.3 OTP Interception — The Final Lock Bypassed

The One-Time Password is the last defensive mechanism between an attacker and a completed fraudulent transfer. Its interception is, technically, the simplest part of the entire operation, because the device that receives the OTP is the same device that has been compromised.

4.3.1 SMS-Based OTP Interception
ATTACK FLOW:   Bank initiates transfer → Sends OTP via SMS to victim's number   SMS arrives on compromised device   Malware's BroadcastReceiver intercepts android.provider.Telephony.SMS_RECEIVED intent   OTP extracted via regex pattern matching (e.g., [0-9]{4,8})   OTP transmitted to C2 server via HTTPS in < 500ms   Attacker (or automated script) inputs OTP into banking session   Transfer authorized — funds moved

On Android 9 and below, applications could directly register a BroadcastReceiver for incoming SMS messages with the READ_SMS permission. Google partially restricted this in Android 10+ by limiting which apps could be the default SMS handler, but the Accessibility Service remains capable of reading SMS notification content as it appears on the notification shade.

#### 4.3.2 Notification-Based OTP Interception

When the bank delivers the OTP as a push notification (common with banking apps that have their own notification channel), the malware with Notification Access permission intercepts the notification object before it is dismissed. The NotificationListenerService API provides access to the full notification text, from which the OTP is extracted programmatically. This method works regardless of the Android version and bypasses the SMS access restrictions introduced in newer OS versions.

#### 4.3.3 Accessibility-Based Screen Reading

As a fallback, even without SMS or notification access, if the OTP appears on screen, in a notification banner, in an SMS preview, or in any visible UI element, the Accessibility Service can read the text of any visible element on any screen. The malware continuously monitors accessibility events and uses pattern matching to detect and extract numeric strings matching OTP formats from any source.

#### TIMELINE ANALYSIS

From the moment the bank sends the OTP to the moment the transaction is authorized, the entire automated attack chain — interception, extraction, transmission, input, and confirmation — can complete in under 10 seconds. In many cases, the victim receives the debit SMS notification before they have even processed that an OTP was sent, as the notification arrives while the account has already been debited.
