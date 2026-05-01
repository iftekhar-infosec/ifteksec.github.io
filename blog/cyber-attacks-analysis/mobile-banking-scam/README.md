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

When the bank delivers the OTP as a push notification (common with banking apps that have their own notification channel), the malware with Notification Access permission intercepts the notification object before it is dismissed. The `NotificationListenerService` API provides access to the full notification text, from which the OTP is extracted programmatically. This method works regardless of the Android version and bypasses the SMS access restrictions introduced in newer OS versions.

#### 4.3.3 Accessibility-Based Screen Reading

As a fallback, even without SMS or notification access, if the OTP appears on screen, in a notification banner, in an SMS preview, or in any visible UI element, the Accessibility Service can read the text of any visible element on any screen. The malware continuously monitors accessibility events and uses pattern matching to detect and extract numeric strings matching OTP formats from any source.

#### TIMELINE ANALYSIS

From the moment the bank sends the OTP to the moment the transaction is authorized, the entire automated attack chain — interception, extraction, transmission, input, and confirmation — can complete in under 10 seconds. In many cases, the victim receives the debit SMS notification before they have even processed that an OTP was sent, as the notification arrives while the account has already been debited.

## 5. Why Conventional Defenses Fail

### 5.1 The Fundamental Architectural Problem

The attack exploits a systemic tension within the Android security model: the OS must simultaneously prevent malicious cross-app access while enabling legitimate accessibility tools to assist users with disabilities. The Accessibility Service API resolves this tension by creating a permissioned exception that grants broad cross-application access when explicitly granted by the user. There is no technical distinction at the OS level between a legitimate screen reader for a visually impaired user and a malicious RAT exploiting the same API.

| Defense Mechanism	| Why It Fails |
|-------------------|--------------|
| Biometric Lock (Fingerprint/Face ID)	| Protects against unauthorized access to the device. Does not protect against malware that operates after the victim has already authenticated. The biometric is defeated by the victim themselves performing a legitimate unlock, after which the attacker exploits the active session. |
| OTP / Two-Factor Authentication	| Designed to prevent unauthorized access from a separate device using stolen credentials. Completely ineffective when the OTP delivery channel (the victim's phone) is the same compromised device executing the attack. The OTP is intercepted on the device before the victim can read it. |
| Bank Transaction Anomaly Detection | Uses heuristics like device fingerprint, IP address, behavioral patterns, and geolocation to flag suspicious transactions. Fails because: the device is legitimate, the IP is the victim's real IP, the session is authenticated via real biometrics, and the behavioral pattern of using the app matches the victim's history, because the attack uses the same device. |
| Google Play Protect	| Scans applications installed from the Play Store and known APK repositories. Completely bypassed by sideloaded APKs that have not been submitted to Google's scanning infrastructure. Signature-based detection also fails against newly compiled or obfuscated malware variants. |
| App-Level PIN	| A secondary PIN within the banking application is bypassed through either the session hijacking technique (the PIN was already entered by the victim) or the overlay attack (the PIN is harvested via the fake UI). Accessibility Service can also read PIN inputs in some implementations. |
| SSL/TLS Encryption | Protects data in transit between the banking app and the bank's servers. Entirely irrelevant to this attack, the attacker never sits between the app and the server. The attack occurs entirely on the device, before encryption is applied. |

## 6. Illustrative Attack Scenarios

### Scenario A — The Automated Silent Transfer

A user in Dhaka sees a Facebook advertisement offering a free 50 GB data package from their mobile operator. They tap the ad, are redirected to a convincing webpage, and download and install the offered application, granting all requested permissions including Accessibility Service. The app shows a message saying 'offer applied' and appears to work.

Later that evening, the user opens their bKash app, authenticates with their fingerprint, checks their balance, and closes the app. Forty-five seconds later, the malware detects the active session is still valid, programmatically reopens bKash in the background using Accessibility Service, navigates to 'Send Money,' enters a number controlled by a money mule, inputs the maximum allowed amount, and confirms the transaction. The bank sends an OTP — the malware intercepts it from the notification in 200 milliseconds, inputs it, and the transaction completes. The user receives a debit notification and is confused — they are not even looking at their phone.

### Scenario B — The Remote-Operated RAT Attack

A user installs a fake 'Nagad agent registration' application they saw shared in a Facebook group. The app is a functional SpyNote RAT. The attacker, monitoring connected devices on their C2 dashboard, sees the device come online. They observe via live screen stream that the user is about to open their bank's mobile app. When the user authenticates with their fingerprint, the attacker immediately takes remote control, initiates a transfer, and waits for the OTP. They read the OTP directly from the screen stream as it appears in the notification banner, input it manually, and confirm the transfer — all within 8 seconds of the OTP arriving.

### Scenario C — The Credential Harvest + Delayed Attack

A malware app with overlay capability detects when the victim attempts to open their Dutch-Bangla Bank mobile app. It immediately renders a pixel-perfect overlay of the app's login screen. The victim enters their account number, username, and password into the fake screen. The credentials are transmitted to the attacker. The overlay displays a 'server error' message and dismisses. The attacker, now possessing full login credentials, accesses the account from their own device at a later time — potentially days later — bypassing the need to access the victim's phone at all.

## 7. Detection — How to Know If You Are Compromised

Malware of this type is designed to operate silently and is often difficult to detect through routine use. However, several indicators may suggest compromise:

### 7.1 Device Behavioral Indicators

- Unexpected rapid battery drain: background processes including screen streaming, C2 communication, and continuous monitoring consume significant power
- Elevated data consumption: particularly for mobile data, as screen streams and exfiltrated data are transmitted continuously
- Device heating without active use: background CPU-intensive processes such as screen recording generate heat
- Unusual delays when opening banking applications: an overlay being rendered creates a brief but detectable lag
- Screen briefly lighting up when the device should be idle: a sign that a background process is activating the display
- Banking app behaving unexpectedly: buttons appearing to be tapped, screens navigating without user input

### 7.2 Account-Level Indicators

-	Transaction notifications for operations you did not initiate: even small test transactions (threat actors frequently test with small amounts before large transfers)
-	OTP messages arriving when you have not initiated any transaction
-	Failed login attempts to your banking account from unfamiliar sessions (if your bank provides login history)
-	Account balance discrepancies

### 7.3 How to Audit Your Device

1. Go to Settings → Accessibility → Installed Services: any application listed here that is not a known, trusted accessibility tool (e.g., a screen reader you deliberately installed) should be treated as suspicious and investigated immediately.
2.	Go to Settings → Apps → [select app] → Permissions: review what permissions each unfamiliar application has been granted. `READ_SMS` and Notification Access are particularly high-risk.
3.	Go to Settings → Security → Device Admin Apps — any application listed here that is not Google's own services or a known MDM solution is a serious red flag.
4.	Install a reputable mobile antivirus application (Bitdefender Mobile Security, Kaspersky for Android) from the official Play Store and run a full scan.
5.	Check Settings → Battery → Battery Usage — examine which apps are consuming battery in the background. Unfamiliar applications with high background usage are suspicious.

## 8. Mitigation Framework

### 8.1 Individual User Mitigation

#### 8.1.1 The Single Most Effective Defense — Physical OTP Isolation

The most technically robust individual defense against this attack class is the physical separation of the OTP delivery channel from the compromised attack surface, namely, using a separate basic keypad mobile phone (e.g., a Nokia 105, Symphony B26, or similar feature phone) as the registered banking SIM device.

#### WHY THIS WORKS

The attacker's control is limited to the compromised Android smartphone. If the OTP is delivered to a separate physical device — a basic phone that cannot install apps, has no internet connection, and runs no exploitable OS — the malware has no attack surface on that device. The OTP arrives in a completely isolated environment. Even a fully compromised smartphone with a persistent RAT cannot intercept an SMS delivered to a different physical SIM card in a different physical phone.

Implementation: Register the banking SIM number with every bank account and MFS account. Keep the basic phone permanently separate from the smartphone. Never insert the banking SIM into the smartphone even temporarily. Cost: approximately 800–1,500 BDT for a basic handset.

### 8.1.2 Application Hygiene

- Install applications exclusively from the Google Play Store and never from links received via Facebook, WhatsApp, SMS, or any website.
-	Treat any advertisement on social media offering a downloadable application as suspicious by default, regardless of how legitimate it appears.
-	Before installing any application from the Play Store, examine: the developer name (is it the official company?), the number of reviews (fake apps often have very few), the date published (newly published apps from unknown developers are higher risk), and the permissions requested during installation.
-	Regularly audit installed applications and uninstall anything that is no longer used or that you do not recognize.

### 8.1.3 Permission Discipline

-	Treat a request for Accessibility Service permission from any non-system application as an immediate red flag. Legitimate utility apps (flashlights, calculators, loan apps, photo editors) have absolutely no valid reason to request this permission.
-	Treat a request for Device Administrator permission from any non-MDM application as an immediate red flag.
-	Deny SMS read permission to any application that is not your native messaging app.
-	Review and revoke notification access for all non-essential applications in Settings → Apps → Special App Access → Notification Access.

### 8.1.4 Operating System & Security

-	Keep the device's Android OS fully updated: security patches address newly discovered privilege escalation vulnerabilities.
- Enable Google Play Protect and ensure it is actively scanning (Settings → Security → Play Protect). While it will not catch all threats, it provides a baseline defense against known malware signatures.
-	Enable full-disk encryption if not already enabled by default (modern Android versions enable this by default).
-	Avoid using rooted devices for banking: root access removes the OS-level sandboxing that provides baseline security isolation between apps.

8.2 Financial Institution Mitigation

8.2.1 Behavioral & Device Analytics
Banks and MFS providers should implement real-time behavioral analytics that extend beyond simple device fingerprinting to include:
•	Detection of Accessibility Service being active on the device at the time of transaction — this is an accessible signal via the Android APIs when the banking app is in the foreground, and its presence during a transaction should trigger elevated scrutiny.
•	Transaction velocity analysis — flagging multiple transaction attempts initiated within short windows following initial authentication.
•	UI interaction pattern analysis — bot-driven Accessibility Service interactions exhibit different timing and navigation patterns than genuine human input; machine learning models can flag anomalous interaction signatures.
•	Out-of-pattern transaction flagging — transfers to newly registered payees for large amounts, especially from accounts with no prior history of such transfers, should trigger mandatory human verification.

8.2.2 OTP Delivery Enhancement
•	Implement time-limited OTPs with windows of 60 seconds or less, reducing the exploitation window for automated interception.
•	Consider deploying authenticator app-based TOTP (Time-based One-Time Passwords via apps like Google Authenticator) as an alternative to SMS OTPs for high-value transactions, as these are significantly harder to intercept remotely.
•	For transactions above a defined threshold (e.g., 5,000 BDT), implement a mandatory cooling-off delay of 60–120 seconds with a cancellation notification, giving victims time to detect and cancel fraudulent transactions before they complete.
•	Implement OTP binding to the transaction details — the OTP message should explicitly state the recipient number and amount ('Your OTP to send 10,000 BDT to 01XXXXXXXXX is 847291'). This forces the victim to consciously see the transaction details, making unauthorized transactions detectable even when the OTP message appears.

8.2.3 Customer Education
•	Prominent in-app warnings — banking apps should display a persistent alert if Accessibility Services are detected as active on the device.
•	Mandatory onboarding security education — new account registration should include a security briefing that covers sideloading risks and permission abuse.
•	Proactive SMS campaigns educating existing customers about the specific threat and the physical OTP isolation solution.

8.3 Regulatory & Government Mitigation
•	Bangladesh Telecommunication Regulatory Commission (BTRC) and Bangladesh Financial Intelligence Unit (BFIU) should establish a coordinated rapid response mechanism for reporting and taking down malware distribution infrastructure, including fraudulent Facebook advertisements.
•	Mandate that all licensed MFS operators and scheduled banks implement minimum security standards for their mobile applications, including Accessibility Service detection, screenshot prevention (FLAG_SECURE), and transaction behavioral analytics.
•	Establish a national cybercrime reporting portal with a simplified interface accessible to non-technical users, with guaranteed response SLAs.
•	Require social media platforms operating in Bangladesh to implement enhanced verification for financial application advertisements and provide automated mechanisms for reporting suspected malware distribution campaigns.
•	Expand the national curriculum for digital literacy programs to include practical cybersecurity education — specifically covering permission awareness, sideloading risks, and safe banking practices.

## 9. Prioritized Recommendations

| ID	| Recommendation	| Rationale / Expected Outcome |
|-----|-----------------|------------------------------|
| R-01	| Physical OTP Isolation via Separate Feature Phone	| Register banking SIM on a basic keypad phone. This single action neutralizes the OTP interception attack vector entirely regardless of smartphone compromise status. |
| R-02	| Never Sideload Applications	| Install exclusively from Google Play Store. Treat all social media advertisement-driven app downloads as malicious by default. |
| R-03	| Deny Accessibility Service to Unknown Apps	| Immediately revoke Accessibility Service access from any application that is not a known, trusted screen reader. |
| R-04	| Bank: Implement Accessibility Detection in Apps	| Banking apps should check for active Accessibility Services and warn users or block high-value transactions when detected. |
| R-05	| Bank: Transaction Cooling-Off Period | Implement a mandatory 60-second cancellation window for high-value transactions with explicit recipient/amount display in the OTP message. |
| R-06	| OS: Keep Android Updated	| Security patches in Android 11–14 significantly restrict background app behavior. Upgrade from Android 8/9 as a priority. |
| R-07	| Regulator: Mandatory App Security Standards	BFIU/Bangladesh Bank should mandate minimum technical security standards for all licensed MFS mobile applications. |
| R-08	| Regulator: Social Media Ad Verification	Require BTRC to implement protocols with Meta/Facebook for rapid takedown of fraudulent application advertisements. |


