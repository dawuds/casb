# Encrypted-upload bypass

> CASB DLP inspects what it can read. A user who encrypts content client-side before upload — 7zip with password, RMS-protected, PGP, zero-knowledge SaaS — defeats content inspection by class.

## The failure

A malicious insider (or compromised account) wants to exfiltrate sensitive data through a sanctioned SaaS. They:

1. Take the sensitive file
2. Encrypt it with a password they retain — `7z a -p<password> exfil.7z sensitive.xlsx`, or zip-with-password, or third-party PGP, or save as a password-protected Office document
3. Upload the encrypted file to OneDrive / Box / Dropbox / Google Drive

The CASB's DLP scanner reads the file, cannot decrypt it, and tags it `Password encrypted` / `Azure RMS encrypted` / `Corrupt file`. The DLP signature does not fire (the content is not readable). The file uploads successfully. The attacker downloads from the SaaS (on their own device, in their own time) and decrypts.

## What enables it

| Mechanism | Why CASB cannot inspect |
|---|---|
| **Password-protected archives** | DLP cannot brute-force the password; content is unreadable |
| **Azure RMS / Microsoft Purview labels applied by the user** | The encryption is legitimate; the content is unreadable to the DLP scanner regardless |
| **Third-party encryption (PGP, age, etc.)** | Same — unreadable |
| **Zero-knowledge SaaS storage** (e.g. some end-to-end-encrypted services) | The CASB can see the upload metadata; cannot see the content |
| **Self-hosted-key SaaS** | Where the SaaS holds the data encrypted with the customer's key, even the SaaS cannot read it; CASB can read no more |
| **OneNote / proprietary file formats** | Some formats are structured in a way that DLP scanners parse imperfectly |
| **Steganography** (image-embedded data) | DLP doesn't analyse images for embedded payloads |

## Which vendors exhibit it

All CASB vendors. This is structural — the DLP engine cannot read what is encrypted. Vendor differences are around:

| Vendor | Behaviour on unscannable content |
|---|---|
| **MDA** | Tags with `Password encrypted` / `Azure RMS encrypted` / `Corrupt file`. The `Always apply if data cannot be scanned` flag determines fail-open (allow) vs fail-closed (block on unscannable). Practitioner choice |
| **Netskope / Zscaler / Palo Alto** | Similar — flag unscannable; per-policy fail-open or fail-closed decision |
| **Skyhigh** | Similar |

The fail-open / fail-closed decision is the practitioner control. Both options have costs.

## Why the failure exists (technical)

DLP is a content-classification problem. The classifier needs readable content. Encryption is, by design, the absence of readable content for anyone without the key. There is no technical solution that lets a DLP engine inspect content the engine cannot decrypt.

The semi-solutions:
- Inspect content **at the endpoint before encryption** (Endpoint DLP — the answer that works)
- Inspect content **at upload to a managed SaaS where the encryption key is held by the firm** (e.g. Microsoft Purview-managed encryption — partial)
- **Block all uploads of unscannable content** — fail-closed posture, high false-positive rate (legitimate password-protected files exist)
- **Apply Purview labels at the source** so that encryption is firm-controlled and the firm retains visibility — partial

None of these is a CASB-side solution. The fix is at adjacent layers.

## What compensates

| Layer | Control |
|---|---|
| **Endpoint DLP** | Inspect content on the device before any encryption is applied. The dominant compensating control |
| **Purview Endpoint DLP** | Microsoft's endpoint-side DLP; pairs with MDA |
| **Acceptable Use Policy + HR consequence** | Explicit policy against unauthorised encryption + exfil-by-encryption is itself a control — establishes deterrent + grounds for action |
| **UEBA / Insider Risk Management** | Look for the *behaviour* pattern (a user suddenly encrypting + uploading many files) even if the content is unreadable. Microsoft Purview IRM, equivalent UEBA |
| **Anomaly on file extension** | Many encrypted-archive extensions (.7z, .zip-password, .gpg) trigger anomaly on a user who has never uploaded such files |
| **Bandwidth / file-size anomaly** | A user uploading a single large encrypted blob is unusual; correlate with departure patterns |
| **Block specific extensions for upload** | Fail-closed on .7z, .gpg, .pgp etc. as a session policy — high friction, high coverage |

## What practitioners should NOT promise

| Don't say | Reason |
|---|---|
| "Our DLP catches sensitive data leaving via SaaS" | It catches data the DLP can read. Encrypted content is invisible |
| "Insider exfil is mitigated by our CASB" | Only the unencrypted-by-user path; the encrypted-by-user path is wide open |
| "We block 7zip-with-password uploads" | Only if you have a fail-closed policy or specific extension block — and the FP rate is high |

## What an auditor will probe

| Question | Honest answer |
|---|---|
| Demonstrate that an encrypted-by-user exfil is detected | The CASB doesn't read it; the IRM / UEBA / endpoint DLP layer is where the detection lives |
| What is the policy on unscannable content (fail-open vs fail-closed)? | Document the decision; document why |
| If an insider used 7zip to exfil 100 files, would you know? | Honestly, maybe — via behavioural anomaly, file-size pattern, or extension trigger. Maybe not |

## Detection KQL (behavioural pattern, since content is unreadable)

```kql
// Users uploading unusual-format files at unusual rates
CloudAppEvents
  | where TimeGenerated > ago(7d)
  | where ActionType == "FileUploaded"
  | where FileName has_any (".7z", ".zip", ".rar", ".gpg", ".pgp", ".age", ".enc")
  | summarize FileCount = count(), TotalSizeMB = sum(FileSize) / 1024 / 1024 by AccountObjectId
  | where FileCount > 10 or TotalSizeMB > 100
  | order by FileCount desc
```

This catches the volume pattern, not the content. It's the only CASB-side signal available for the encrypted-upload class.

## Promotion candidate

Public-wedge candidate; currently a residual-risk reference.
