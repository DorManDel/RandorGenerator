# RandorGenerator — Architecture, Security and Deployment Plan

Status: **design and recovery** (2026-11-25). This document describes intended architecture, not deployed functionality.

## What is already known

Earlier RandorGenerator prototypes existed outside this GitHub repository, including a Node/Express API, camera frame-difference sampling, a SHA-256-based entropy pool, OS randomness, an HMAC-based output path, lottery UI, and experimental entropy diagnostics. The linked GitHub repository was empty when this plan was drafted. **Recover, audit and test the latest original source package before replacing or rebuilding the core.**

## Product areas

1. **Random Lab:** dice, coins, roulette, unbiased ranges, lists and sampling.
2. **Entropy Lab:** webcam/aquarium camera, experimental external live sources, bit-level visualizations, frame differences, quality metrics and JSON exports.
3. **Key Studio:** secure browser-generated random bytes, passwords, API tokens, AES-GCM keys and HMAC keys with user-controlled formats.
4. **Developer API:** versioned endpoints for unbiased random data and source health, with documentation, authentication/rate limits for costly operations and security tests.

## Trust boundary — critical

- The security foundation for secrets is the OS/browser CSPRNG, using `crypto.getRandomValues`, `crypto.subtle.generateKey`, or Node `crypto.randomBytes` as appropriate.
- Images, decoded video frames, frame differences, camera timestamps, fish movements, and remote webcams are **experimental, possibly predictable or attacker-controlled inputs**. They may complement observations but are not assumed to provide validated cryptographic entropy.
- Conditioning with SHA-256/HMAC/HKDF does not create entropy from a predictable input. Do not claim that a camera added N secure bits without proper evidence and assessment.
- Public sources are `auxiliary-untrusted`, never the sole seed for cryptographic keys. Local cameras are `experimental-unvalidated` until assessed.
- The system must remain secure when any camera freezes, disconnects, repeats frames, or returns attacker-selected media.
- Separate deterministic seeded PRNGs (replay, demos and ASCean experiments) from secure randomness (secrets and tokens).
- Never log, cache, transmit to analytics, or expose secret key material unless the user explicitly chooses an export. Favor non-extractable `CryptoKey` objects for real encryption keys.

## Proposed data flow

```mermaid
flowchart TD
  A[OS / Browser CSPRNG] --> K[Secure random and Key Studio]
  B[Aquarium / local webcam] --> D[Frame sampler and change detector]
  C[Permitted public live feeds] --> D
  D --> H[Health checks and statistics]
  H --> Q[SHA-256 conditioning + provenance]
  Q --> X[Experimental entropy mixer / research view]
  A --> X
  X --> V[Research visuals and reproducible tests]
  K --> API[Versioned developer API]
```

This separates a **security-guaranteed route** (subject to the platform CSPRNG) from an **experimental route**. A future reviewed design may investigate safely mixing additional inputs into the secure route without introducing dependency on untrusted sources.

## Capture modes

- **Local device:** `getUserMedia` requires explicit browser permission and usually HTTPS. Local camera selection is per device/browser context.
- **Remote aquarium/PC:** a consented local PC collector reads the connected camera and sends *only needed samples/metrics* over an authenticated outbound connection. A phone dashboard can request start/stop through a separate authenticated control channel. Never expose a private aquarium feed publicly by default.
- **External camera:** source catalog with `provider`, `landingPage`, `accessMethod`, `rightsStatus`, `corsStatus`, `health`, `lastSampleAt`. Only ingest feeds when terms/permission and technical read-back permit it. A third-party embedded viewer alone does not authorize canvas pixel extraction.

## Entropy experiment contract

For each sample track: source ID; timestamp; dimensions; frame interval; pixel-change ratio; sample digest; repeated-frame and source-health flags; estimator/test version; explicit `secureEntropyBitsCredited: 0` for unvalidated camera sources. Avoid storing raw camera frames by default. Add frame-freeze, repetitive-pattern and rate checks; compare with synthetic deterministic controls. Report *observed variability*, not inferred cryptographic strength.

## Key Studio controls

- Output kind: password/passphrase, hex/base64 token, random bytes, UUID, WebCrypto AES-GCM or HMAC key.
- Length / bit size / charset (upper/lowercase/digits/symbols), optional minimum category counts, field-based templates and delimiters.
- Character selection must avoid modulo bias (rejection sampling or equivalent); account for reduced search space under strict templates.
- Secrets are generated locally in the browser by default; show once on explicit user action, clearable; avoid URL query strings, logs, clipboard by default, and third-party scripts on secret screens.
- `generateKey({ name: "AES-GCM", length: 256 }, false, ["encrypt","decrypt"])` is the preferred non-exportable encryption-key path. Offer export only in an explicit separate mode.
- Never assert that camera visuals make a key stronger than an already secure CSPRNG key.

## Suggested API (design, not deployed)

- `GET /api/v1/health`
- `GET /api/v1/entropy/sources` (status, never secret state)
- `POST /api/v1/random/integers` with bounds and count, uniform sampling
- `POST /api/v1/random/bytes` with byte count and encoding
- `POST /api/v1/experiments` for non-secret diagnostics (authenticated)
- Keys should usually be created with browser WebCrypto rather than remotely generated or delivered by a public server.

Validate input limits, request volumes and origins. Use rate limiting and authentication on private/expensive endpoints. Ensure no caching of secret responses.

## Deployment strategy

1. **Recover original source** and tests; place in the RandorGenerator repository after audit, preserving history and previous modules where possible.
2. **Static frontend** on GitHub Pages or another HTTPS static host. Keep Random Lab and local Key Studio usable even when backend is offline.
3. **Backend** on a Node-compatible HTTPS host for API and remote collector control. GitHub Pages cannot run Node/Express server code.
4. **Private camera collector** runs only on the trusted Windows PC; outbound authenticated connection with allowlisted operations. No direct camera-port exposure and no raw frames by default.
5. **External source adapters** enabled only once rights and read-back are verified. Keep Times Square as a source research candidate, not a promised integrated feed.
6. **Portfolio card** points to a real tested public URL. Link back to GitHub, show honest `Research / Live Demo` status.
7. **QA gate**: secure-context camera test, source freeze, permission denied, disconnect, mobile viewport, replayable diagnostics, unbiased output tests, API authorization, budgets and privacy.

## Portfolio card copy (English)

**RandorGenerator**
*Exploring randomness — from camera noise and live signals to secure key generation.*

Tags: `Web Crypto` `Entropy Lab` `Node.js` `Camera Signals`

Primary CTA: **Try Live Demo** (only when deployment verified)
Secondary CTA: **View Source**

## Milestones

- M0: Recover original package; inventory modules, defects, and deployment blockers.
- M1: Stabilize secure random/key-generation frontend with no required camera/server dependency.
- M2: Restore aquarium camera analysis + privacy-preserving PC collector and remote mobile controls.
- M3: Experimental camera source catalog with rights/CORS checks and statistical diagnostics.
- M4: Secure and documented developer API with rate limiting and test suite.
- M5: Add portfolio card, verify live HTTPS endpoint, publish release notes and PCM checkpoint.

### References

- MDN MediaDevices.getUserMedia: https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia
- MDN Crypto.getRandomValues: https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues
- MDN SubtleCrypto.generateKey: https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/generateKey
- MDN CORS-enabled canvas: https://developer.mozilla.org/en-US/docs/Web/HTML/How_to/CORS_enabled_image
- NIST SP 800-90B: https://csrc.nist.gov/pubs/sp/800/90/b/final
- NYC DOT data feeds: https://www.nyc.gov/html/dot/html/about/datafeeds.shtml
