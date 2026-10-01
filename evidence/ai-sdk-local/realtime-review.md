# Realtime Boundary Review — Continuation

## Result

**No eligible core AI SDK vulnerability was validated in this continuation.**
The work stayed within the pinned `ai@7.0.126` source tree, synthetic browser
objects, and local mocks. It did not contact a Vercel runtime, provider API, or
customer deployment.

## Reviewed security boundaries

### Setup response to browser WebSocket

**Attacker considered:** an attacker able to influence bytes returned by an
application's realtime setup endpoint, but without control of the application
code or the configured model implementation.

**Expected guarantee:** malformed setup data must be rejected before a browser
socket is allocated; stale setup work must not create a socket after disconnect
or reconnect.

**Trace:**

1. `AbstractRealtimeSession.connect()` posts only `sessionConfig` to the
   application-configured token endpoint.
2. `validateRealtimeSetup()` requires a non-empty token, an absolute `ws:` or
   `wss:` URL with a hostname, and structurally valid optional tools.
3. `BrowserRealtimeTransport.connect()` asks the configured model to derive its
   final URL/protocols, validates the final URL again, then allocates the socket.
4. Attempt identity and transport epochs fence responses and callbacks belonging
   to disconnected or replaced connections.

Pinned source:

* [`realtime-session.ts`](https://github.com/vercel/ai/blob/45e1fbc3ea10d22f852e12b1c1851e289c85e835/packages/ai/src/realtime/realtime-session.ts)
* [`validate-realtime-setup.ts`](https://github.com/vercel/ai/blob/45e1fbc3ea10d22f852e12b1c1851e289c85e835/packages/ai/src/realtime/validate-realtime-setup.ts)
* [`browser-realtime-transport.ts`](https://github.com/vercel/ai/blob/45e1fbc3ea10d22f852e12b1c1851e289c85e835/packages/ai/src/realtime/browser-realtime-transport.ts)

**Local result:** malformed token/URL payloads, retired setup responses, late
socket callbacks, and reconnect races were rejected or fenced in the focused
suite. No boundary bypass was reproduced.

**Disposition:** rejected. If an attacker controls the configured setup endpoint
itself, that attacker is already inside an application-owned trust boundary and
can return its own short-lived session material. That prerequisite does not
establish a core SDK authorization bypass.

### Realtime tool calls to application execution

**Attacker considered:** a remote model influenced by untrusted conversation
content that can emit an arbitrary tool name and JSON arguments.

**Expected guarantee:** the core runtime parses the event and hands it to the
application callback; it does not claim to authorize or execute server tools by
name. The documented design requires app-specific endpoints with application
authentication, authorization, validation, and rate limiting.

**Trace:**

1. `RealtimeEventReducer.reduceServerEvent()` parses a completed function-call
   argument string and emits a `tool-call` effect only after successful JSON
   parsing.
2. `AbstractRealtimeSession.handleReducerEffect()` records the call and invokes
   the application-provided `onToolCall` callback.
3. `executeTool()` sends a returned value back only while the same attempt is
   active and connected. Disconnect/reconnect fencing prevents a delayed old
   callback from publishing into the replacement session.

Pinned source:

* [`realtime-event-reducer.ts`](https://github.com/vercel/ai/blob/45e1fbc3ea10d22f852e12b1c1851e289c85e835/packages/ai/src/realtime/realtime-event-reducer.ts)
* [`realtime-session.ts`](https://github.com/vercel/ai/blob/45e1fbc3ea10d22f852e12b1c1851e289c85e835/packages/ai/src/realtime/realtime-session.ts)
* [Realtime tool-calling documentation](https://github.com/vercel/ai/blob/45e1fbc3ea10d22f852e12b1c1851e289c85e835/content/docs/03-ai-sdk-core/36-realtime.mdx)

**Local result:** delayed tool callbacks were fenced after remote closure and
after reconnect; malformed arguments did not invoke the handler; provider
delegation was rejected where unsupported. No authorization bypass was
reproduced.

**Disposition:** rejected by documented boundary. A reproduction that dispatches
an arbitrary `toolName` to a privileged generic endpoint would depend on the
intentionally unsafe application pattern that the documentation explicitly
warns against. It is not an eligible core SDK finding.

### Tool definitions returned by setup

**Attacker considered:** an attacker who can alter a setup response's optional
tool-definition array.

**Expected guarantee:** setup tool definitions configure what the browser sends
to the provider; they do not grant server-side authority or automatically bind
an executable SDK `ToolSet`.

**Trace:** validated setup tools are merged into the session update. Subsequent
calls still reach only the application-owned `onToolCall` callback. The helper
documentation states that `experimental_getRealtimeToolDefinitions()` does not
execute tools.

**Local result:** the setup schema rejects malformed definitions. A valid but
attacker-selected definition can influence the provider-facing advertisement,
but it creates no execution capability without unsafe application dispatch.

**Disposition:** rejected. Control of the application setup response is already
privileged, and the hypothesized impact additionally requires unsafe application
code.

### Session token expiry metadata

**Attacker considered:** a replaying browser client holding setup JSON after its
optional `expiresAt` value.

**Observation:** the core validates the shape of `expiresAt` but does not enforce
the timestamp locally.

**Disposition:** rejected. The actual client secret is provider- or
gateway-issued and enforced at the connection boundary; `expiresAt` is response
metadata, not a documented local authorization control. Demonstrating impact
would require testing an excluded live provider or inventing a mock provider
that intentionally accepts expired credentials.

## Public overlap check

GitHub issue and pull-request searches were run for realtime security, tool
validation, token URLs, arbitrary `onToolCall` dispatch, and WebSocket changes.
Two results materially clarified the boundary:

* [PR #13893](https://github.com/vercel/ai/pull/13893) records the design decision
  not to provide a generic server-side tool RPC route because applications must
  authenticate users, bind sessions, allowlist names, validate call IDs, rate
  limit, and authorize invocations.
* [PR #14034](https://github.com/vercel/ai/pull/14034) proposed HMAC tool tokens
  for such a generic route but was closed without merge after the generic route
  was removed from the design.

The open [Gateway realtime client-secret hardening PR #16284](https://github.com/vercel/ai/pull/16284) was also reviewed. It concerns
the Gateway provider package and live Gateway token policy, both excluded from
this core-only local lane. Search results are not proof that an issue is novel.

## Local validation

The first focused group covered setup validation, final WebSocket URL validation,
stale-attempt fencing, tool callback lifecycle, command correlation, close
behavior, and regression cases:

```text
Test Files  5 passed (5)
Tests       149 passed (149)
Type Errors no errors
```

The second group covered continuous WebSocket sessions, WebRTC, live-session
lifecycle, bounded event channels, and command tracking:

```text
Test Files  5 passed (5)
Tests       109 passed (109)
Type Errors no errors
```

Exact commands are stored in `test-results.txt`. In total, **258 focused tests
passed across 10 files**.

## Remaining gaps

* The saved policy JSON, methodology skill, and `vercel_ai_local_hunt.sh` remain
  unavailable, so their schemas or instructions could not be applied.
* The detailed HackerOne asset focus remains unavailable. This review therefore
  makes no claim that experimental realtime functionality is bounty-eligible.
* Provider adapters and Gateway token minting were not tested because they are
  outside the authorized core-only lane.

## Strongest next direction

Review UI message reconstruction and data-stream parsing at the core boundary,
with emphasis on prototype-safe part lookup, tool-approval association,
cross-message identifier collisions, and whether malformed client-supplied
history can cross a documented validation boundary. Use only synthetic streams
and local mock models, and compare each candidate with the existing approval
hardening patches before assigning eligibility.
