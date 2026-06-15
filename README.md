# 🎣 Phishing Campaign Analysis — Credential Harvesting via Office 365 Impersonation

> **Analyst:** Kgalalelo Clive Moledi
> **Date: 15 June 2026  
> **Classification:** Threat Intelligence / Phishing Analysis  
> **Severity:** 🔴 High  

---

## Executive Summary

This report documents a hands-on phishing campaign investigation conducted in a sandboxed environment. The analysis covers the full attack chain — from initial email triage through phishing kit extraction, hash verification, VirusTotal correlation, and adversary infrastructure enumeration. The campaign was designed to harvest Microsoft 365 credentials by impersonating a legitimate corporate login portal, targeting multiple employees across the organisation.

Key findings include a confirmed malicious payload (`Update365.zip`), a threat actor–controlled credential exfiltration address (`m3npat@yandex.com`), and a spoofed sender domain (`groupmarketingonline.icu`) consistent with business email compromise (BEC) tactics.

---

## Table of Contents

1. [Phase 1 — Email Triage](#phase-1--email-triage)
2. [Phase 2 — HTML Attachment Analysis](#phase-2--html-attachment-analysis)
3. [Phase 3 — Phishing Infrastructure Investigation](#phase-3--phishing-infrastructure-investigation)
4. [Phase 4 — Phishing Kit Retrieval & Hash Verification](#phase-4--phishing-kit-retrieval--hash-verification)
5. [Phase 5 — VirusTotal Intelligence](#phase-5--virustotal-intelligence)
6. [Phase 6 — Phishing Kit Deep Dive](#phase-6--phishing-kit-deep-dive)
7. [Indicators of Compromise (IOCs)](#indicators-of-compromise-iocs)
8. [Conclusion](#conclusion)

---

## Phase 1 — Email Triage

The investigation began with a queue of **5 suspicious emails** flagged for review. The first priority was identifying the most actionable sample.

![Email Queue Overview](https://github.com/user-attachments/assets/cd87e6ad-3e98-4e03-bf48-31f033b84bc4)

### Target Email: *"Quote for Services Rendered"*

The subject line **"Quote for Services Rendered"** was selected as the primary sample due to its social engineering characteristics — invoking urgency around a financial transaction to lower the recipient's guard.

![Email Detail View](https://github.com/user-attachments/assets/04a80509-0009-4305-9577-5cb8405fe5a9)

**Key Artifacts Extracted:**

| Artifact | Value |
|---|---|
| **Recipient** | William McClean |
| **Sender Display Name** | Accounts Payable |
| **Sender Email** | `Accounts.Payable@groupmarketingonline.icu` |
| **Tactic** | BEC / Sender Spoofing |

> ⚠️ **Analyst Note:** The `.icu` TLD is a well-known indicator associated with low-cost, disposable domains commonly registered for phishing campaigns. Legitimate accounts payable correspondence would originate from the organisation's own domain.

---

## Phase 2 — HTML Attachment Analysis

A second email targeting **Zoe Duncan** contained an HTML attachment — a classic phishing delivery mechanism used to bypass email gateway filters, since the malicious content lives in the attachment rather than a link.

![Zoe Duncan Email with HTML Attachment](https://github.com/user-attachments/assets/a4b8b2ae-60a0-461b-ad98-014a85246fbb)

The attachment was opened inside an **isolated sandbox environment** to safely observe its behaviour.

![HTML Attachment Rendered](https://github.com/user-attachments/assets/a2aa4971-7a0f-4a6c-b101-c1dc3a28db75)

### Findings

The rendered page presented a convincing **Microsoft 365 login portal**, branded with the target organisation's logo and styling. However, a critical red flag was immediately visible:

- **The root domain did not belong to Microsoft.**
- The URL pointed to an adversary-controlled host, designed to capture submitted credentials in real time.

This technique — known as a **"clone phishing" login page** — is highly effective against non-technical users who focus on the visual appearance of a page rather than the address bar.

![Phishing URL Domain Inspection](https://github.com/user-attachments/assets/3f09a25b-fe00-4cbc-8535-4e2c82b2ad7c)

---

## Phase 3 — Phishing Infrastructure Investigation

With the phishing domain identified, the next step was to probe the server's directory structure for exposed files — a common misconfiguration on adversary-controlled infrastructure.

Navigating to the **`/data/` directory** on the phishing host revealed an open directory listing.

![Open Directory Listing on Phishing Server](https://github.com/user-attachments/assets/f70636dc-33b1-41cc-b94d-4447132aac0e)

> 💡 **Why this matters:** Threat actors frequently deploy phishing kits quickly and carelessly, leaving directory indexing enabled. This operational security failure allows analysts to retrieve the full kit for forensic analysis — turning the attacker's mistake into intelligence.

---

## Phase 4 — Phishing Kit Retrieval & Hash Verification

The phishing kit archive was downloaded from the exposed directory for offline analysis.

![Phishing Kit Archive](https://github.com/user-attachments/assets/fc4470a3-dedc-42da-b050-756e60a84ebb)

A **SHA-256 hash** was immediately computed to establish a cryptographic fingerprint of the file, enabling correlation with threat intelligence platforms and ensuring file integrity.

```bash
sha256sum Update365.zip
# ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686
```

![SHA256 Hash Computation](https://github.com/user-attachments/assets/9b041cd1-9731-4935-adc1-b7d092587249)

| Property | Value |
|---|---|
| **Filename** | `Update365.zip` |
| **SHA-256** | `ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686` |

---

## Phase 5 — VirusTotal Intelligence

The hash was submitted to **VirusTotal** for multi-engine scanning and threat intelligence correlation.

![VirusTotal Results](https://github.com/user-attachments/assets/3bbb9a9e-0750-4f4e-a83e-d1b69f9c502d)

### Results: ✅ Confirmed Malicious

![VirusTotal Detection Detail](https://github.com/user-attachments/assets/5f08f570-6fcf-439d-af4a-69842b63d6f1)

| Metric | Result |
|---|---|
| **Detection Ratio** | 30 / 62 vendors |
| **Classification** | Trojan / Phishing |
| **Masquerade** | Office 365 Update |
| **Verdict** | 🔴 High Risk — True Positive |

> **Analysis:** A 30/62 detection rate is significant. The file masquerades as a legitimate Office 365 update (`Update365`) — a social engineering lure designed to make targets believe they are installing a required software update, when in reality they are executing a credential harvester.

---

## Phase 6 — Phishing Kit Deep Dive

With the kit extracted and verified, the internal file structure was examined to understand the full credential capture mechanism.

**Directory of interest:** `Update365/office365/Validation/`

The file `submit.php` was identified as the **credential exfiltration handler** — the backend script triggered when a victim clicks "Sign In."

![submit.php Exfiltration Script](https://github.com/user-attachments/assets/e2258035-65df-4fca-a3bc-823b22abd3ee)

### Critical Finding: Threat Actor Email Address

Hardcoded within `submit.php` was the adversary's collection address:

```
m3npat@yandex.com
```

This Yandex address is where **all harvested credentials are emailed in real time** the moment a victim submits their username and password. Yandex is frequently abused by threat actors due to its anonymity and lack of cooperative response to abuse reports from Western law enforcement.

---

## Indicators of Compromise (IOCs)

| Type | Indicator | Context |
|---|---|---|
| **Email Domain** | `groupmarketingonline.icu` | Spoofed sender domain |
| **Sender Address** | `Accounts.Payable@groupmarketingonline.icu` | Phishing lure sender |
| **File Name** | `Update365.zip` | Phishing kit archive |
| **SHA-256** | `ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686` | Phishing kit hash |
| **Exfil Address** | `m3npat@yandex.com` | Credential collection email |
| **Attack Path** | `/Update365/office365/Validation/submit.php` | Credential handler script |
| **Tactic** | HTML attachment + clone login page | Delivery & capture mechanism |

---

## Conclusion

This investigation successfully traced a credential harvesting campaign from initial email delivery through to full phishing kit analysis. The attack chain demonstrated several hallmark techniques of a moderately sophisticated threat actor:

- **Domain spoofing** using a disposable `.icu` TLD to impersonate an internal accounts payable department
- **HTML attachment delivery** to evade URL-based email gateway filtering
- **Microsoft 365 login page cloning** to deceive victims with familiar branding
- **Open directory exposure** on the phishing server, revealing the full kit
- **Yandex-based exfiltration** for anonymised, real-time credential collection

The phishing kit (`Update365.zip`) was confirmed malicious by 30 of 62 VirusTotal vendors, and the hardcoded exfiltration address (`m3npat@yandex.com`) provides a high-confidence pivot point for further threat actor attribution and campaign tracking.

**Defensive Recommendations:**
- Block `.icu` TLD at the email gateway
- Alert on HTML-only attachments from external senders
- Implement MFA across all Microsoft 365 accounts to limit credential theft impact
- Add `m3npat@yandex.com` and the identified domain to threat intelligence blocklists
- Report the phishing infrastructure for takedown

---

*Analysis conducted in an isolated sandbox environment. No live systems were exposed during this investigation.*
