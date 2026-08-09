# Terminal3 ADK — Sandbox Testing & Feedback

Submission for the Terminal3 ADK bounty (Superteam Earn): testing the T3N SDK quickstart flow, documenting a bug encountered during authentication, and suggesting an initial use case.

## Environment

- Device: Mobile (Android), developed entirely via Termux + Acode
- Node.js: v24.17.0
- npm: 11.17.0
- SDK: `@terminal3/t3n-sdk` (latest, installed via `npm install @terminal3/t3n-sdk tsx`)
- Network: `testnet` (via `setEnvironment("testnet")`)

## What was completed

- [x] Signed up via SSO on the claim page, obtained API key + DID
- [x] Set up project (`npm init`, installed SDK + tsx)
- [x] Wrote `quickstart.ts` exactly as provided in the "Connect and authenticate" step of the docs
- [x] Ran `npx tsx quickstart.ts`
- [ ] Blocked at `t3n.handshake()` — see bug report below

## Bug encountered

### `TypeError: Cannot read properties of undefined (reading 'unsafe_trust_server')`

**Where:** Thrown inside `t3n.handshake()`, deep in the SDK's internal call chain (`isUnsafeTrustServer` → `assertNodeTrusted` → `handleGuestToHost` → `handleWasmRequest` → `runFlow` → `handshake`).

**Reproduction steps:**
1. Follow the Quickstart docs exactly (`docs.terminal3.io/developers/adk/get-started/quickstart`)
2. Claim API key + DID from the sandbox claim page
3. `npm install @terminal3/t3n-sdk tsx`
4. Create `quickstart.ts` with the exact code block provided in step 3 of the Quickstart ("Connect and authenticate")
5. Export a valid `T3N_API_KEY` (verified 66-char format, `0x` + 64 hex chars, no whitespace)
6. Run `npx tsx quickstart.ts`

**Result:** Script crashes inside `t3n.handshake()` before printing the `Connected as:` DID line.

**What we checked before concluding it's a platform bug:**
- Confirmed the API key format was valid (`echo -n "$T3N_API_KEY" | wc -c` → 66)
- Confirmed no whitespace/hidden characters in the exported key
- Cross-referenced the ["Common errors"](https://docs.terminal3.io/developers/adk/tips/common-errors) page — this exact error (`unsafe_trust_server`) is **not listed** anywhere in the documented error tables (tenant operations, generic HTTP 500s, integration gotchas, or auth/wallet linking)
- The error is a raw `TypeError` from an `undefined` property read, not a structured `bad_request` JSON-RPC error as described in the docs' error-handling convention — suggesting a config object the SDK expects (likely something related to trusted-node config for the WASM component) is not populated when following the Quickstart steps as written

**Full stack trace:** see `screenshots/handshake-error.png`

**Suggested fix direction:** The `T3nClient` constructor or `setEnvironment("testnet")` may need to populate a trust-server config value that isn't set by the documented Quickstart flow. Worth checking whether `loadWasmComponent()` or the `T3nClient({ wasmComponent, handlers })` config needs an additional field (e.g. an explicit trusted-node allowlist) not mentioned in the current docs.

## Screenshots

See `/screenshots` folder:
- `claim-page.png` — API key + DID claimed
- `quickstart-code.png` — quickstart.ts content
- `handshake-error.png` — full error stack trace

## Initial use case (bonus)

*[To be filled in — see notes below]*

## Notes

Development done entirely from an Android phone using Termux (Node.js/npm) and Acode as editor — no desktop environment used.
