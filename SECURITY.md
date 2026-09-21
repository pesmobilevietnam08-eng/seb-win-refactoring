**AI Usage Disclosure**
Yes, Artificial Intelligence (LLM) was utilized during the preparation of this report. Specifically, AI was used to: (1) Translate my manual technical analysis from Vietnamese to English, (2) format the markdown structure of this report, and (3) assist with summarizing the CVSS 3.1 scoring metrics. The core security research, reverse-engineering of the threat vector, and manual testing were conducted by me personally. I have fully read, verified, and self-authored the core findings of this security advisory.

**Describe the Vulnerability**
This is a chained vulnerability report demonstrating a complete bypass of SEB's Kiosk mode on Windows. By chaining 6 specific weaknesses, an attacker with Local Administrator privileges can run an automated AI-solver tool (such as the actively distributed `Kaito_2.6_Pro`) that intercepts exam questions and inputs answers without SEB detecting any anomalies. The chain involves:
1. Screen capture bypass via DirectX Desktop Duplication API (DDA).
2. HTTPS MITM via local proxy and injected Root CA.
3. Stealth overlay window (using `WS_EX_TOOLWINDOW | WS_EX_TRANSPARENT`).
4. Synthetic input injection via `SendInput` API without hardware validation.
5. Process evasion due to lack of Authenticode signature checks.
6. Lack of pre-exam Root CA store integrity auditing.

**Steps to Reproduce**
1. Log into a Windows machine with Local Administrator privileges.
2. Inject a custom Root CA into `Cert:\LocalMachine\Root` and set up a local MITM proxy.
3. Execute a malicious background tool configured with `WS_EX_TOOLWINDOW` and renamed to a benign process name (e.g., `svchost.exe`).
4. Launch Safe Exam Browser and connect to an exam session.
5. The background tool uses `IDXGIOutputDuplication` to read the screen framebuffers, bypassing `SetWindowDisplayAffinity`.
6. The tool forwards the captured questions to an LLM API.
7. The tool displays the answer in a transparent topmost overlay and uses `SendInput` to automatically click the correct answers.

**Expected Behavior**
SEB should proactively prevent this attack chain by:
- Polling for active `IDXGIOutputDuplication` sessions and terminating if found.
- Implementing TLS Certificate Pinning to prevent MITM even if a Root CA is injected.
- Validating the `LLKHF_INJECTED` flag in low-level mouse/keyboard hooks to block `SendInput`.
- Detecting topmost layered windows (even with `WS_EX_TOOLWINDOW` flags).

**Screenshots**
*Not available.* The proof-of-concept testing was conducted in an isolated lab environment which has since been securely wiped according to our internal security protocols. However, the exact technical vectors are fully detailed in the "Steps to Reproduce" section for your engineering team to replicate.

**Version Information**
- OS: Windows 11 (Build 26100) / Windows 10
- SEB-Version: SEB 3.10.2

**Additional Context**
*Regarding Log Files:* Log files are not attached. Due to the strict data privacy and security policies of the university's IT research group, we are not authorized to export or disclose internal exam-session log files. 

More importantly from a technical standpoint: **The SEB log files do not contain any relevant data regarding this exploit.** Because this vulnerability chain operates at the DWM/GPU layer (DDA) and injects input at the Win32 API level via overlay, SEB's internal monitoring mechanisms do not detect any focus loss, unauthorized windows, or process anomalies. Therefore, standard SEB logs simply show a normal, clean exam session, which is exactly why this bypass chain is critical. 

This attack vector is actively being packaged into automated cheating tools sold to students. I am reporting this responsibly so the SEB team can implement structural defenses before this methodology becomes more widespread.
