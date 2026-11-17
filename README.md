# Phishing Email Investigation

# Overview
  A hands-on analysis of a suspicious email pulled from a live Gmail spam folder. The email was examined safely (without ever clicking it), its authentication verified, the sender and link reputation checked with free analyst tools, the indicators of compromise extracted, and a documented verdict reached.
Outcome: the email authenticated correctly and no tool flagged it as malicious. It is an unsolicited bulk spam, not a credential phishing attack. Distinguishing them from one another, and proving it with evidence, is the purpose of the exercise.

# Tools use
- Gmail "Show original" - to read the raw message source and SPF/DKIM/DMARC results
- VirusTotal - domain and IP reputation
- urlscan.io - safely sandboxed the link (scanned unlisted to avoid exposing the recipient identifier)
- IP geolocation lookup - country of the sending mail server

## The email under investigation

| Field | Value |
| --- | --- |
| From | `jobaidcenter <insider@jobaidcenter[.]com>` |
| To | *(redacted - recipient inbox)* |
| Subject | "Take the next step toward your goal" |
| Date | Thu, 5 Nov 2026 |
| Found in | Gmail **Spam** folder |

## Step 1 — Authentication (SPF / DKIM / DMARC)

From the raw headers ("Show original"):

| Check | Result | Meaning |
| --- | --- | --- |
| SPF | **PASS** (IP `193.37.47[.]61`) | Sent from a server authorized by the domain |
| DKIM | **PASS** (`jobaidcenter.com`) | Genuinely signed by the domain, not forged |
| DMARC | **PASS** | Visible "From" matches the authenticated domain |

**Key insight:** All three passing means the email is authentic to its sending domain and was not spoofed but it does not mean the sender is trustworthy. A spammer can register a throwaway domain and configure SPF/DKIM/DMARC correctly so everything "passes." Therefore, this not brand impersonation; it is authenticated mail from a real but low-reputation sender.

## Step 2 — Sender and domain reputation
- Domain jobaidcenter[.]com - VirusTotal: 0 / 92 vendors flagged it malicious; domain created ~9 months ago. A clean record, but young and unestablished ("not reported" does not mean "safe").
- Sending IP 193.37.47[.]61 — reported as hosted in Bulgaria. An unknown "job aid" sender emailing a US inbox from foreign mail infrastructure is a soft suspicion signal, not proof of anything on its own.

## Step 3 — Link analysis
The email body contained a click-tracking link. The URL was defanged for this writeup and the real URL scanned in urlscan.io (unlisted):
- Link: hxxps://rk.jobaidcenter[.]com/p/line/clk?id=...
- The /clk path + rk. subdomain = a click-tracking redirect, typical of marketing/affiliate email.
- urlscan verdict: No classification (not flagged malicious).
- Link host: 96.44.184[.]245 — United States, HostPapa (a cheap US host commonly used for bulk mail).
- Only 2 HTTP transactions, 1 domain — a simple redirect, not a credential-capture kit. No fake login page.

## Indicators of Compromise (IOCs)

| IOC type | Value (defanged) | Source |
| --- | --- | --- |
| Sender address | `insider@jobaidcenter[.]com` | Raw headers |
| Sender domain | `jobaidcenter[.]com` | Raw headers / VirusTotal |
| Sending (mail) IP | `193.37.47[.]61` (Bulgaria) | SPF header |
| Link | `hxxps://rk.jobaidcenter[.]com/p/line/clk` | Email body |
| Link host IP | `96.44.184[.]245` (US, ColoCrossing/HostPapa) | urlscan.io |

## Analyst note — verdict
Verdict: Spam / unsolicited bulk email · Severity: Low. The message authenticated correctly (SPF, DKIM, and DMARC all passed) from jobaidcenter[.]com, a 9 month old domain that no security vendor flags as malicious (0/92 on VirusTotal). The link in the body is a click-tracking redirect hosted in the US with no malicious classification on urlscan.io, and there is no credential-harvesting page or brand spoofing anywhere in the message. This is unsolicited marketing / lead-generation mail, not a targeted phishing attack. Recommended action: report as spam and block the sender domain; no escalation required.

## What this project demonstrates
- SPF / DKIM / DMARC prove an email is authentic to its domain, not that the domain is safe.
- "Clean" / "No classification" on reputation tools means not reported, not confirmed safe.
- Mail-sending infrastructure and web-hosting infrastructure are often separate IPs and must be recorded separately.
- A suspicious email can be analyzed without ever clicking it — raw source, defanged URLs, sandboxed link analysis.
- Reaching and documenting a measured verdict matters more than assuming every suspicious email as "phishing."





