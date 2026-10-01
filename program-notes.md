# Vercel AI SDK Local Review Notes

## Policy evidence and authority

The Vercel program policy referenced by the workspace owner is treated as
**supplied policy evidence only**. This review does not claim that the policy
was independently verified as current. No saved policy JSON was present in the
workspace or elsewhere on the local filesystem when searched on 2026-10-01.
Likewise, no installed bug-bounty methodology skill or
`vercel_ai_local_hunt.sh` was found. Consequently, no hunt script was executed,
and no script network destinations or commands could be inspected.

The workspace owner authorized local source review and tests against isolated
instances they control. That acknowledgment is limited to this local lane and
does not authorize platform testing.

## Operational interpretation

* Public retrieval of official source, stable package releases, documentation,
  policies, advisories, issues, and patches is research, not active testing.
* Active tests are limited to local processes, synthetic inputs, mock providers,
  and loopback-bound services.
* Live Vercel services, production APIs, customer systems, and third parties are
  out of scope.
* Provider-package behavior, examples, application mistakes, and malicious
  upstream data without a documented protection boundary are not eligible core
  SDK findings.
* Nothing may be published or submitted automatically. Any future report must
  be reproduced and reviewed manually by the workspace owner first.

## Repository context

The actual Git root is `/workspace/OpenCore-Post-Install`. This repository is
not the AI SDK repository; the official AI SDK source was cloned into the
ephemeral `/tmp/vercel-ai-review` directory for review. The baseline and results
are recorded under `evidence/ai-sdk-local/` without vendoring the upstream tree.
