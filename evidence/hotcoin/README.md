# Hotcoin Security Review

## Result

**No qualifying vulnerability was validated.** The checked-out repository is
not a Hotcoin codebase, no application package was supplied, and the permitted
passive web retrieval did not return application content. The continued review
retrieved only public Google Play listing metadata. No report or proof of
concept was fabricated.

## Coverage and disposition

| Requested activity | Result | Reason |
| --- | --- | --- |
| Static repository review | Not applicable to Hotcoin | The repository is the Dortania OpenCore Post-Install documentation site. |
| Assessment of a suspected issue | Not performed | No suspected vulnerability, request/response pair, affected account state, or PoC was provided. |
| HackenProof report drafting | Not warranted | A clear impact and runnable PoC are required, but no finding was validated. |
| Passive web-root retrieval | Blocked | Envoy rejected the CONNECT tunnel; no request reached Hotcoin. |
| Android metadata review | Completed | Google Play confirmed `com.pro.hot`, the listing title, and its September 21, 2026 update date. |
| Android binary review | Not performed | No official APK/AAB was supplied or available from the public listing. |
| iOS metadata review | Blocked | Envoy rejected the connection to Apple's public lookup endpoint. |
| iOS binary review | Not performed | No official IPA was supplied or downloaded. |

The proxy's HTTP 403 is not treated as a response from Hotcoin, a vulnerability,
an availability finding, or evidence about Hotcoin's controls. Curl reported a
final status of `000`, confirming that the TLS tunnel was not established.

## Exact safe check

The web-root connection check was:

```sh
curl -sS -L --max-time 20 -A 'Mozilla/5.0 security-research' \
  -o /dev/null -D /tmp/hotcoin-review/www-recheck.headers \
  -w 'final=%{url_effective} code=%{http_code}\n' \
  https://www.hotcoin.com/
```

Observed proxy output:

```text
HTTP/1.1 403 Forbidden
content-length: 16
content-type: text/plain
server: envoy
curl: (56) CONNECT tunnel failed, response 403
final=https://www.hotcoin.com/ code=000
```

The continued review also retrieved the public Google Play listing:

```sh
curl -L --max-time 30 -sS -A 'Mozilla/5.0' \
  -o /tmp/hotcoin-review/play.body \
  -D /tmp/hotcoin-review/play.headers \
  'https://play.google.com/store/apps/details?id=com.pro.hot&hl=en&gl=US'
```

The listing returned HTTP 200 and identified:

* package: `com.pro.hot`;
* title: `Hotcoin: Buy BTC, ETH & Crypto`;
* description: `Fast trading of BTC, ETH, SOL, DOGE, XRP, BCH and 3,500+
  Cryptos`;
* updated date: September 21, 2026; and
* developer website and privacy-policy links on the in-scope Hotcoin domain.

This metadata confirms target identity only. It does not establish application
behavior or provide a binary suitable for security analysis. A connection to
Apple's public lookup endpoint was attempted once and failed at the Envoy
CONNECT tunnel in the same way as the Hotcoin web-root check.

No credentials, cookies supplied by the researcher, personal data, user
accounts, exploit payloads, or state-changing requests were used.

## Safe next steps

1. Supply an official application package or an exact source snapshot if a
   static mobile or source review is desired.
2. For a suspected web issue, supply a redacted request/response pair, the
   affected first-party endpoint, prerequisites, and the state change observed
   using researcher-owned accounts.
3. Reproduce any candidate narrowly against owned test data, record expected
   versus actual behavior, and stop before accessing another user's data.
4. Draft a private report only after validating impact and a runnable PoC.

## Report template for a validated finding

```markdown
# [Vulnerability class] in [first-party component]

## Summary
[One paragraph describing the broken security invariant and concrete impact.]

## Affected target
[Exact in-scope URL, Android package, or iOS app ID and tested version.]

## Preconditions
[Researcher-owned accounts, roles, and setup. Do not include live secrets.]

## Steps to reproduce
1. [Deterministic step.]
2. [Exact request or application action.]
3. [Observed unauthorized state change or sensitive response.]

## Runnable PoC
[Minimal code or commands, sanitized and rate-limited.]

## Actual result
[Evidence of the security impact using only researcher-owned data.]

## Expected result
[The authorization, validation, or isolation behavior that should occur.]

## Impact
[Who can exploit it, what they gain, and the maximum demonstrated scope.]

## Remediation
[Specific server-side or application control, plus a regression-test idea.]
```

The template is intentionally unpopulated: inferred endpoints, invented
responses, and non-runnable AI-generated PoCs do not meet the supplied rules.
