# AI SDK Core Local Security Review

## Result

**No eligible core AI SDK vulnerability was validated.** No private report or
PoC ZIP was created because there is no validated candidate to report. The
review distinguished static suspicions from reproduced behavior and did not
manufacture a finding.

## Reviewed baseline

| Item | Value |
| --- | --- |
| Official repository | `https://github.com/vercel/ai` |
| Core package | `ai` |
| Stable npm version observed | `7.0.126` |
| Source commit | `45e1fbc3ea10d22f852e12b1c1851e289c85e835` |
| Retrieval date | `2026-10-01` |
| Package manager | `pnpm@11.23.0` |
| Runtime required by source | Node `>=22.13.0` (supported 22/24/26 ranges) |
| Lockfile | upstream `pnpm-lock.yaml` |
| Lockfile SHA-256 | `b25efe0584b19943abd5e5db7f5b6b6122c3d7068ccd3aff60e296fd9688d587` |

The tag checkout contains multiple package release tags, so the baseline was
confirmed from `packages/ai/package.json`, not from `git describe` output.

## Security guidance and policy gaps

The repository `.github/SECURITY.md` was available. It directs security reports
to Vercel's HackerOne program or the responsible-disclosure mailbox, but states
no detailed asset focus. The referenced HackerOne asset focus document could
not be accessed from this environment (the initial page request returned an
authorization error), so this review relied only on the policy evidence supplied
by the owner and the conservative exclusions in `scope.yaml`.

The requested saved policy JSON, methodology skill, and
`vercel_ai_local_hunt.sh` were absent. This is an unresolved evidence gap; work
that depended on their unknown contents was not performed.

## Focused coverage

1. **Tool approval integrity and execution boundary.** Traced reconstructed,
   client-controlled approval history through signature verification, schema
   revalidation, policy re-evaluation, own-property tool lookup, and the final
   execution call. Local upstream tests confirmed signatures bind approval ID,
   call ID, tool name, and canonical input; delimiter retupling, tampering, a
   different secret, invalid schema input, and inherited tool names are rejected.
2. **Provider-returned generated-file URLs.** Traced URL-backed generated files
   through `resolveGeneratedFileData`, the core downloader, and provider-utils'
   untrusted URL fetch path. Local tests confirmed DNS resolution to loopback is
   rejected before the replaced fetch function is called. This path also has a
   response-size limit. A generic URL-fetch concern was therefore rejected.
3. **Tool execution from model output.** Model-controlled tool name and input
   can reach an application's registered `execute` callback, but only because
   the application explicitly supplies that executable tool. Without a bypass
   of validation or configured approval, this is the documented tool-calling
   function rather than a core authorization vulnerability.

## Public duplicate and advisory check

The GitHub security-advisory API was queried for the official repository, the
latest 100 open/closed issues carrying the `security` label were queried, recent
patch history for the reviewed files was inspected, and an npm audit lockfile
was generated for `ai@7.0.126`.

The advisory query returned five 2026 advisories, all for AI SDK harness
packages rather than the core `ai` package: GHSA-4mqq-99j3-3hmv,
GHSA-222v-gj5h-ff73, GHSA-vmqp-7rwf-cq3w, GHSA-g48p-5rr5-8rgq, and
GHSA-qw9h-448j-6rph. The issue query returned no security-labeled issue in the
first page. `npm audit` reported zero known vulnerabilities for the generated
dependency tree. These searches are evidence of checks performed, **not proof
of novelty**.

## Candidate disposition

| Candidate | Status | Reason |
| --- | --- | --- |
| Forged or retupled tool approval executes a different tool/input | Rejected; locally tested | HMAC payload is injective and binds all operation fields; legacy fallback refuses newline-bearing fields; approval and schema are revalidated. |
| Prototype-chain tool selection bypasses validation | Rejected; locally tested | Tool lookup uses own properties, and regression coverage rejects inherited names. |
| URL-backed generated file permits SSRF to loopback/private DNS | Rejected; locally tested | Untrusted URL fetch validates DNS and blocks loopback before calling fetch. |
| Provider-selected tool call causes arbitrary application code execution | Rejected by boundary | Execution is the configured tool's documented purpose; no validation/approval bypass was established. |
| Malicious provider-returned content causes application-layer injection | Rejected by scope/boundary | Requires malicious upstream output and an unsafe consumer sink; no core SDK protection guarantee was identified. |

## Exact local commands

Retrieval and baseline:

```sh
npm view ai version dist.tarball gitHead --json
git clone --filter=blob:none --no-checkout https://github.com/vercel/ai.git /tmp/vercel-ai-review
git -C /tmp/vercel-ai-review checkout 'ai@7.0.126'
git -C /tmp/vercel-ai-review rev-parse HEAD
sha256sum /tmp/vercel-ai-review/pnpm-lock.yaml
```

Installation, build, and focused tests (public package retrieval only; tests
used synthetic data and local mocks):

```sh
cd /tmp/vercel-ai-review
npx -y -p node@22 -p pnpm@11.23.0 -c 'pnpm install --frozen-lockfile --ignore-scripts --filter ai... --network-concurrency=4 --fetch-retries=3'
npx -y -p node@22 -p pnpm@11.23.0 -c 'pnpm --filter ai... build'
cd packages/ai
npx -y -p node@22 -p pnpm@11.23.0 -c 'pnpm exec vitest --config vitest.node.config.js --run src/generate-text/tool-approval-signature.test.ts src/generate-text/validate-tool-approvals.test.ts src/generate-text/generated-file-download.node.test.ts'
```

Focused result: **3 test files passed; 37 tests passed; no type errors**.

The first test attempt mistakenly passed file names after an extra `--`, which
made Vitest run the broad suite before workspace dependencies were built. It
failed for missing local build artifacts and is not treated as a product test
failure. Dependencies were then built and the exact focused command above
passed.

## Strongest next direction

Review the browser realtime setup/token boundary using a loopback-only mock
relay: trace attacker-controlled setup responses, URL changes, token lifetime,
tool registration changes, and reconnect races through the browser transport.
The source already contains extensive regression coverage in this area, so the
next pass should target a concrete state-machine invariant and compare against
public patches before attempting a PoC.
