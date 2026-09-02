# SOC Identity Authentication Investigation

## Overview

This project documents a Microsoft Sentinel and Defender XDR identity investigation involving multiple non-success sign-in events followed by successful authentication.

The alert initially suggested possible brute-force or credential-stuffing activity. Examination of the underlying Microsoft Entra sign-in records showed that the events consisted primarily of MFA requirements, an invalid session, and incomplete strong-authentication attempts.

## Investigation Outcome

- **Alert severity:** Medium
- **Data source:** Microsoft Sentinel / Defender XDR
- **Category:** Credential Access
- **MITRE ATT&CK mapping:** T1110 — Brute Force
- **Final disposition:** Benign
- **Containment required:** No

## Skills Demonstrated

- Microsoft Sentinel and Defender XDR investigation
- KQL querying and event correlation
- Microsoft Entra sign-in analysis
- Authentication result-code interpretation
- Timeline and Five-W analysis
- Detection-engineering recommendations
- AI triage quality-control review
- Evidence sanitization for public reporting

## Report

[Download the complete investigation report](SOC_Identity_Authentication_Investigation_Portfolio_Rona_Playda.docx)

## Privacy Notice

Account names, public IP addresses, tenant identifiers, correlation IDs, and subscription details have been masked. The investigation was conducted in an authorized SOC training environment.
