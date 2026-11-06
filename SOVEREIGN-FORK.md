# Sovereign Fork — OpenClicky → Hermes Visual Operator (M3ta-OS)

**Fork of:** [jasonkneen/openclicky](https://github.com/jasonkneen/openclicky) (MIT) at commit `e9eb06a`
**Fork home:** `m3ta-a3y3-inT3LLiG3nc3/openclicky` (M3ta's agent-org GitHub account; chosen because the
ecosystem's prototype/agent repos live there — builder-brokers, bridge work — while KINDdom-m3TA holds
product repos)
**Authorized:** M3ta, 2026-09-18 (fork approval) + 2026-11-06 ("Ready. Go. You got my word.")
**Role:** Hermes visual-operator prototype for M3ta-OS — a menu-bar companion that sees the screen,
answers by voice, and *points*: a click-through, non-focus-stealing per-display overlay driven by
provider-neutral grounding tags.

## What is kept (the core assets)

- The overlay recipe: one borderless, transparent, click-through `NSWindow` per display at
  `.screenSaver` level, `canBecomeKey/canBecomeMain = false`, joins all Spaces — it guides the user's
  eye and never takes control away from the user.
- The grounding-tag protocol, parsed from model output and stripped from spoken text (partial tags
  suppressed mid-stream so they never leak into speech):
  `[POINT:x,y:label:screenN]`, `[RECT:x,y,w,h:label:screenN]`, `[SCRIBBLE:x1,y1;...:label:screenN]`
- The local external-control bridge (`127.0.0.1:32123`): `/health`, `/mcp/tools`, `/mcp` (JSON-RPC),
  `/cursor`, `/caption`, `/screenshot`, `/speak`, SSE `/events` — token-gated, loopback only.
  This is the Hermes integration surface (see "Hermes wiring" in the build report,
  `~/workspace/your_files/openclicky-fork-build-2026-11-06.md`).

## Sovereign changes vs upstream

1. **Telemetry: none.** Upstream had already removed PostHog (`ClickyAnalytics` is a no-op stub whose
   static API remains only so call sites compile). Verified in this fork: no PostHog package in
   `Package.resolved`, no analytics SDK anywhere in the source, `ClickyAnalytics.isEnabled == false`.
   No third-party telemetry may be reintroduced.
2. **Update feed severed.** `SUFeedURL` and `SUPublicEDKey` removed from `cursor-buddy/Info.plist`;
   Sparkle automatic checks/update flags set to `false`. The fork never polls upstream's appcast.
   Updates are pulled deliberately through git.
3. **Signing team is NOT inherited.** The project still carries upstream's `DEVELOPMENT_TEAM`
   (`SW75ZJJ5R6`) in the pbxproj as a historical default. Builds for M3ta's machines override signing
   at build time (ad-hoc / local development signing) until the fork is re-signed under M3ta's own
   team in Xcode. TCC identity note: macOS ties permission grants to the signing identity, so the
   production TCC grants must be made against the final signed build, in Xcode, by M3ta.
4. **Bridge token resolution reordered** (`AppBundleConfiguration.externalControlBridgeToken()`):
   the operator-managed local secrets file (`~/.config/openclicky/secrets.env`, mode 600) is now
   authoritative; the Keychain/settings value is fallback. Upstream's Keychain-first order let a
   stale Keychain item silently shadow operator-set tokens and made rotation impossible from the
   terminal — found live during the first fork deployment (2026-11-06).

## Provider routing — local-first (M3ta's model rule)

Upstream's "money rule" already implements the required posture and is preserved unchanged:

- **Existing paid/local auths first:** Claude Agent SDK (uses the local Claude Code sign-in) is the
  primary Claude path; direct Claude REST is fallback-only (bills per token). Same shape for the
  Codex app-server → OpenAI key fallback.
- **On-device lane:** Apple Foundation Models run locally and free.
- **Metered APIs are escalation only** — never the default short-circuit.
- **Screen/vision data** (the most sensitive flow in the stack) goes only to the provider the user
  has configured; there is no fork-operated proxy and no third-party relay.

## Permissions (M3ta's taps only — never granted programmatically)

The app needs, on the Mac, in System Settings → Privacy & Security:
**Screen Recording**, **Microphone**, **Accessibility** (and it will prompt for them on first run).
Per upstream's own warning, terminal-built (unsigned/ad-hoc) builds can invalidate TCC grants —
the TCC-validating build is the Xcode build on the Mac, signed with the final identity.

## License

MIT (upstream). Upstream attribution retained in `LICENSE` and `THIRD_PARTY_NOTICES.md`.
