Unauthenticated Fund Transfer Page Exposed via Content Discovery”.
# Unauthenticated Fund Transfer Page Exposed via Content Discovery

**Environment:** TryHackMe, Offensive Security Intro (training lab, not a real client)
**Date:** October 2026
**Author:** [Datboizay09]
**Overall risk:** High

## 1. Executive Summary
A training banking application exposed an unlinked page, `/bank-transfer`,
that required no authentication. A tester found it using content discovery
and could [describe in one phrase what the page let you do, e.g. "submit
a fund transfer between accounts"]. No credentials were needed. The top
fixes are to enforce server-side authentication and authorization on every
sensitive route, derive the source account from the user's session rather
than the form, and alert on enumeration activity.

## 2. Scope and Methodology
- **Target:** fakebank.thm (training lab)
- **Approach:** [dirb] was run against the web
  root with [wordlist name] and responses were reviewed by HTTP status code.
- **Frameworks:** OWASP Top 10 (A01: Broken Access Control),
  MITRE ATT&CK T1595.003 (Wordlist Scanning), T1190 (Exploit Public-Facing Application)

## 3. Attack Path
Content discovery → unlinked page found (HTTP 200) → unauthenticated
access to transfer form → funds can be moved

## 4. Findings

### Finding 1: Sensitive page discoverable and accessible without authentication

| Field | Detail |
|---|---|
| Severity | High |
| Affected asset | fakebank.thm/bank-transfer |
| Description | The page is not linked from the site but is also not protected. Content discovery found it, and it returned 200 OK to an unauthenticated request. |
| Evidence | Figure 1 |
| Business impact | A direct path to financial fraud, with no credentials required. |
| Root cause | Reliance on obscurity instead of access control. |
| Remediation | Enforce server-side authentication on all sensitive routes; add role-based authorization; restrict administrative paths by VPN or IP allow-list. |
| Detection | A burst of 404 responses from one source followed by a 200 on a sensitive path. |

![Figure 1](01-content-discovery-bank-transfer.png)
*Figure 1: Content discovery reveals an unlinked /bank-transfer page returning HTTP 200.*

### Finding 2: Transfer function does not verify account ownership

| Field | Detail |
|---|---|
| Severity | High |
| Affected asset | fakebank.thm/bank-transfer |
| Description | The form accepts an account number with no check that the requester is authorized to use it. |
| Evidence | Figure 2 |
| Business impact | Funds can be moved from accounts whose numbers are known or guessed. |
| Root cause | Missing server-side authorization; user-supplied input is trusted (IDOR-style weakness). |
| Remediation | Bind the source account to the authenticated session; require MFA or transaction signing; apply transfer limits; validate all parameters server-side. |
| Detection | Alert when the source account does not match the logged-in user; alert on unusual transfer velocity. |

![Figure 2](02-transfer-form-redacted.png)
*Figure 2: Transfer form with account numbers masked.*

## 5. Remediation Roadmap

| Priority | Action | Control mapping |
|---|---|---|
| Immediate | Require authentication on /bank-transfer | NIST 800-53 AC-3 |
| Immediate | Enforce server-side authorization on the source account | NIST 800-53 AC-6 |
| Short-term | MFA for transfers; rate limiting and WAF rules | CIS Control 6 |
| Long-term | Centralized logging and enumeration alerting | CIS Control 8 |

## 6. Cloud Translation
If this application were hosted on AWS, the same issues map to: AWS WAF
rate-based rules to throttle enumeration, ALB or CloudFront access logs
sent to a SIEM to detect the 404 burst, and Cognito or API Gateway
authorizers so the route cannot be reached unauthenticated. The same flaw
appears in cloud environments as an unauthenticated API endpoint.

## 7. Detection Ideas
- Many 404s from a single IP in under a minute
- Any request to /bank-transfer without a valid session
- Transfers where the session user does not own the source account

## 8. Lessons Learned
[2-3 sentences in your own words: what surprised you, and what you would
check first on a real application.]
