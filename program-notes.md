# Hotcoin Review Notes

## Policy evidence and authority

The Hotcoin program text supplied by the workspace owner is treated as policy
evidence for this review. It was not independently verified as current. The
authorization is limited to the three targets recorded in `scope.yaml` and does
not extend to third-party services or unrelated infrastructure.

The operational constraints prohibit automated scanners, destructive testing,
denial of service, spam, access to other users' data, and public disclosure.
Any validated issue must be reported exclusively through HackenProof within 24
hours of discovery. A report must contain a runnable proof of concept; a static
observation alone is insufficient.

## Repository context

The Git root is `/workspace/OpenCore-Post-Install`, an OpenCore documentation
site. It contains neither Hotcoin web source nor the official Android or iOS
application binaries. Existing material under `evidence/ai-sdk-local/` concerns
a separate, completed review and was not used as evidence about Hotcoin.

## Review result

No Hotcoin vulnerability was identified or validated from this repository.
Connection attempts to the in-scope web root were rejected by the environment's
egress proxy during tunnel establishment, before an HTTP request reached the
application. The public Google Play listing was retrievable and confirmed the
package identity and store metadata, but it did not provide an official APK or
source code for static analysis. Apple's public lookup endpoint was blocked by
the same egress control. No circumvention, endpoint enumeration, scanner,
account action, or mobile binary retrieval was attempted.

Because there is no suspected vulnerability, reproducible state change, or
runnable proof of concept, creating a vulnerability report would be misleading
and contrary to the program's stated requirements. The evidence record in
`evidence/hotcoin/README.md` preserves the disposition and safe next steps.
