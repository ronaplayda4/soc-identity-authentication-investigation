# SOC Identity Authentication Investigation

Microsoft Sentinel and Defender XDR case study examining multiple non-success sign-ins followed by successful authentication.

> **Final disposition:** Benign authentication activity  
> **Containment:** Not required  
> **Environment:** Authorized SOC training environment

## Executive Summary

A Microsoft Sentinel scheduled analytics rule generated a Medium-severity identity alert after detecting several non-success sign-in results followed by successful authentication. The alert suggested possible brute force or credential stuffing.

Review of the raw alert, original detection logic, Microsoft Entra `SigninLogs`, authentication result codes, device context, application activity, and a seven-day account baseline did not support account compromise. The non-success records primarily represented MFA requirements, an invalid session, and incomplete strong-authentication attempts. Activity came from one consistent US source with expected device and application characteristics, and no suspicious follow-on behavior was identified.

## Alert Context

| Field | Finding |
| --- | --- |
| Alert | Multiple Failed Logins Followed by a Successful Login |
| Source | Microsoft Sentinel / Defender XDR |
| Severity | Medium |
| Category | Credential Access |
| MITRE ATT&CK | T1110 - Brute Force (detection mapping reviewed) |
| Activity window | August 11, 2026, 01:24:45-01:43:06 UTC |
| Final disposition | Benign |

The analytic used a broad `ResultType != 0` condition. This treated several authentication interruptions as failed logins even though they were not all incorrect-password events. The rule summarized six qualifying non-success events and joined that count to 23 successful sign-in result rows. Therefore, “23 items” represented joined query results—not 23 alerts or 23 failed logins.

## Investigation Objectives

- Identify the affected account, source, location, applications, and device context.
- Determine whether the non-success records represented password failures, MFA events, session errors, or another authentication condition.
- Correlate the non-success activity with later successful sign-ins.
- Compare the source and device profile with the available account baseline.
- Check for suspicious applications, competing sources, lateral movement, or post-authentication abuse.
- Validate AI-generated triage claims against the raw telemetry.

## Investigation Process

1. Reviewed the alert title, severity, UTC activity window, provider, and complete analytic query.
2. Reproduced the alert conditions with a focused `SigninLogs` query.
3. Expanded and classified each qualifying non-success authentication record.
4. Correlated account, IP, location, application, browser, operating system, and device details.
5. Built a seven-day source and device baseline.
6. Searched for unfamiliar geography, competing sources, suspicious applications, and follow-on malicious activity.
7. Compared the AI triage draft with primary evidence and corrected unsupported claims.

## Evidence

### 1. Focused investigation query

The focused query reproduced the detection conditions and exposed the individual records behind the summarized alert count. Account and IP values are masked.

![Focused KQL investigation query](01-focused-kql-query.png)

### 2. Authentication results

Six qualifying non-success events were identified. Reading `ResultType` together with `ResultDescription` showed that the group was not six incorrect-password failures.

![Authentication results](02-authentication-results.png)

### 3. Device context

The activity showed Chrome on Windows, with an empty device ID and unmanaged/noncompliant status. This supported device-profile consistency but could not uniquely identify a physical computer.

![Device details](03-device-details.png)

### 4. Seven-day baseline

The available baseline showed 38 Windows/Chrome events (25 successful and 13 non-success) plus one Android Microsoft Authentication Broker event. Both platform profiles used the same sanitized public source and US location.

![Seven-day account baseline](04-seven-day-baseline.png)

## Authentication Timeline

| UTC time | Event | Analyst interpretation |
| --- | --- | --- |
| 00:38:38 | Android Authentication Broker non-success | Same source; consistent with a mobile authentication flow |
| 01:03:44 | First Windows/Chrome baseline event | Start of concentrated browser activity |
| 01:14:41 | Result 50074: strong authentication required | MFA challenge, not an incorrect password |
| 01:19:49 | Result 50074: strong authentication required | Repeated MFA challenge |
| 01:22:46 | Result 50133: session invalid | Password/session lifecycle condition |
| 01:25:51 | Result 500121: strong authentication failed | Incomplete or failed MFA attempt |
| 01:35:18 | Result 500121: strong authentication failed | Incomplete or failed MFA attempt |
| 01:40:50 | Result 500121: strong authentication failed | Incomplete or failed MFA attempt |
| 01:43:06 | Last alert-related successful sign-in | Success from the consistent source context |
| 01:48:58 | Alert generated | Scheduled analytic created one Medium alert |

## Key Findings

- The six qualifying records were not six password failures.
- Two records required MFA, one reflected an invalid session, and three represented incomplete or failed strong-authentication attempts.
- All relevant activity came from one US public source with consistent browser and device characteristics.
- Expected Microsoft and SOC training applications were accessed.
- No competing source, unfamiliar country, suspicious application, lateral movement, or data-exfiltration activity was identified.
- The available account baseline was limited to the investigation period; this limitation was documented rather than treated as proof of legitimacy.
- No confirmed malicious indicator of compromise was identified.

## Five-W Analysis

| Question | Evidence-based answer |
| --- | --- |
| Who | A sanitized SOC training account |
| What | MFA requirements, one invalid session, and three incomplete strong-authentication attempts followed by successful access |
| When | August 11, 2026; alert-related activity ended at 01:43:06 UTC |
| Where | One US public source using Windows/Chrome, with one Android authentication-broker interaction |
| Why | The analytic grouped multiple nonzero authentication results under a broad failed-login threshold |
| How | Browser and MFA authentication across expected Microsoft and SOC training applications |

## AI Triage Quality-Control Review

The AI-generated draft was treated as junior-analyst work and verified line by line. Several claims were unsupported by the telemetry, including a different compromised administrator account, SharePoint or Microsoft Graph abuse, and preparation for lateral movement or exfiltration. These claims were removed from the final analysis.

This review reinforced an important analyst principle: query text is not event evidence. For example, an IP address appearing in a `!=` exclusion condition is not automatically an observed source or indicator of compromise.

## Verdict and Resolution

**Resolution: Benign**

The alert detected real authentication activity, but the evidence was consistent with legitimate MFA and session-establishment behavior—not confirmed brute force or account compromise. One consistent source, expected applications, a coherent authentication sequence, numerous successful sign-ins, and the absence of malicious follow-on behavior supported the benign disposition.

No credential reset, device isolation, or IP blocking was recommended because the investigation did not establish malicious activity.

## Detection-Engineering Recommendations

- Separate incorrect-password results from MFA-required, invalid-session, KMSI interruption, and incomplete-authentication events.
- Require tighter temporal ordering between credential failures and a later successful sign-in.
- Correlate source IP, device, application, and correlation ID before raising the alert.
- Suppress known authentication-flow interruptions that inflate the brute-force threshold.
- Continue using AI triage as an aid, while requiring analyst verification against raw logs.

## Skills Demonstrated

- Microsoft Sentinel and Defender XDR alert investigation
- Identity threat hunting and Microsoft Entra sign-in analysis
- Authentication result-code interpretation
- Timeline construction and Five-W reporting
- Evidence-based alert disposition and containment decisions
- AI triage quality control
- MITRE ATT&CK mapping review
- Detection-engineering analysis and recommendations
- Evidence sanitization for public reporting

## Full Investigation Report

[Download the complete sanitized Word report](SOC_Identity_Authentication_Investigation_Portfolio_Rona_Playda.docx)

## Privacy Notice

Account names, public IP addresses, tenant identifiers, correlation IDs, and subscription details are masked. All activity occurred in an authorized training environment.

