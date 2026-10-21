# The event-stream / flatmap-stream Incident (2018)

*Research note prepared for our npm supply-chain security-review session. Compiled from the original tracking issue in `dominictarr/event-stream`, the package repositories' commit history, and GitHub's security advisory database.*

## Background

[`event-stream`](https://github.com/dominictarr/event-stream) was a hugely popular npm package (streaming helpers for Node.js) maintained by Dominic Tarr, pulling in millions of weekly installs and appearing in dependency trees of countless projects — often several levels deep (e.g. `npm-run-all` → `ps-tree` → `event-stream`). By 2018 the maintainer had effectively stopped using or maintaining the package.

An unknown GitHub user (`right9ctrl`) emailed the maintainer offering to take over maintenance of the module. Dominic Tarr handed over **npm publish rights** to this stranger — a purely social-engineering attack requiring no exploit at all. The attacker used that access to add a new dependency, `flatmap-stream`, which they also controlled on npm, and shipped it inside `event-stream@3.3.6` on **9 September 2018**. `flatmap-stream` was a brand-new, zero-download package whose repo contained a single commit — nothing about it would have survived even light due diligence.

The payload was **not** in the readable source: `flatmap-stream@0.1.1`'s published tarball contained extra code *only* in the minified `index.min.js`, including an AES-encrypted blob disguised as a test fixture and code that reads the `process`/environment of whatever imported it. The attack was therefore invisible to anyone diffing the repo against the published package.

## Timeline

Reconstructed from the discussion in [issue #116](https://github.com/dominictarr/event-stream/issues/116) ("I don't know what to say."), [issue #115](https://github.com/dominictarr/event-stream/issues/115), and the commit histories of the two packages.

| Date | Event |
|---|---|
| Before Sept 2018 (exact date not public) | `right9ctrl` emails Dominic Tarr offering to maintain `event-stream`; Tarr grants them **npm publish rights**. ("he emailed me and said he wanted to maintain the module, so I gave it to him" — maintainer's response in the tracking issue.) |
| 2018-09-05 | `hugeglass/flatmap-stream` GitHub repo created (single "First commit"). `right9ctrl` begins committing to `event-stream` (v3.3.5). |
| 2018-09-09 08:07 UTC | Commit [`e3163361`](https://github.com/dominictarr/event-stream/commit/e3163361fed01384c986b9b4c18feb1fc42b8285) ("add flat map") by `right9ctrl` adds `flatmap-stream` as a dependency of `event-stream`. |
| 2018-09-09 08:17 UTC | Ten minutes later, `right9ctrl` bumps the version and publishes **`event-stream@3.3.6`**, which depends on the malicious **`flatmap-stream@0.1.1`**. |
| Mid-Sept 2018 | `right9ctrl` rewrites flatmap natively (commits of 2018-09-16/20), drops the `flatmap-stream` dependency, and moves the repo to 4.x. `flatmap-stream@0.1.2`, with the injection removed, follows on npm (the repo's "latest version removes the injection", per the tracking issue). Crucially, **`3.3.6` is never yanked**, so everyone pinned to the 3.x line keeps pulling the backdoored dependency. |
| Sept–Nov 2018 | `event-stream@3.3.6` spreads transitively across the ecosystem. Documented example from the discussion: `npm-run-all@4.1.3` → `ps-tree@1.1.0` → `event-stream@3.3.6` → `flatmap-stream`. |
| 2018-11-19 | [**Issue #115**](https://github.com/dominictarr/event-stream/issues/115) opened by NewEraCracker: "Backdoored dependency? flatmap-stream-0.1.1 and flatmap-stream-0.1.2" — first public flag of the suspicious dependency. |
| 2018-11-21 | [**Issue #116**](https://github.com/dominictarr/event-stream/issues/116) opened by FallingSnow ("I don't know what to say.") with a cleaned-up breakdown of the code found only in the minified `flatmap-stream@0.1.1`: hex-encoded `require` calls, an AES-encrypted payload disguised as `./test/data`, and `process` access. Analysis continues in-thread. |
| 2018-11-22 | Maintainer Dominic Tarr responds — the key comment of the thread: he had been emailed by `right9ctrl`, gave him the module, no longer uses it, and *no longer has publish rights on npm*. `npm owner ls event-stream` confirmed `right9ctrl` was the sole owner. |
| 2018-11-23 | Community triage: reachability checks (`npm ls event-stream flatmap-stream`), credential-rotation advice, and contact with npm support. |
| 2018-11-26 | **Target identified** by @maths22: the payload activates only for Copay-related packages. Issue #116 body updated with "Am I affected? / What does it do / What can I do" guidance (downgrade to `event-stream@3.3.4` or upgrade to 4.x). By this point npm had removed (yanked) the malicious versions from the registry, and GitHub advisory `GHSA-mh6f-8j2x-4483` is published this same day. |

## The payload (what it actually did)

Per the advisory record for `flatmap-stream` (`GHSA-9x64-5r7x-2q53`):

- Read AES-encrypted data from a file **disguised as a test fixture** (`test/data`).
- Grabbed the npm package description of the *importing* module via an automatically set environment variable, and used that description as the **decryption key**.
- Decrypted data formed a module that was **compiled in memory and executed**; it wrote the processed block of JS out to a file in a dependency packaged by build scripts, with minor decoding transforms (believed to be for evading dynamic analysis tools).
- The resulting code: detected the environment (Mobile / Cordova / Electron), checked the victim's **Bitcoin and Bitcoin Cash balances**, and if the balance exceeded 100 BTC / 1000 BCH, harvested the full account data and **private keys** and sent them to the attacker's collection endpoint.

The victim application was **Copay** (the BitPay-built Bitcoin wallet) — the package-description key matched Copay's own npm description, so the payload armed itself only inside Copay builds. It would likely have worked for `copay-dash` too, since the two shared a package description.

## Advisories

Lookups were performed against GitHub's global security advisory database for both packages, covering both **reviewed** advisories and **malware** listings.

| Package | Lookup type | Result |
|---|---|---|
| `event-stream` | reviewed | **1 advisory**: [`GHSA-mh6f-8j2x-4483`](https://github.com/advisories/GHSA-mh6f-8j2x-4483) — "Critical severity vulnerability that affects event-stream and flatmap-stream". Severity **critical** (CVSS 9.8, CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H, CWE-506 Embedded Malicious Code). Vulnerable: `event-stream = 3.3.6` (first patched: 4.0.0); also lists `flatmap-stream >= 0`. Published 2018-11-26. |
| `event-stream` | malware | **No results** — the malware-type lookup for `event-stream` returned nothing. |
| `flatmap-stream` | reviewed | **2 advisories**: (1) [`GHSA-9x64-5r7x-2q53`](https://github.com/advisories/GHSA-9x64-5r7x-2q53) — "Malicious Package in flatmap-stream", severity **critical** (CVSS 9.8, CWE-506), vulnerable: `flatmap-stream = 0.1.1`, references npm advisory 737; (2) `GHSA-mh6f-8j2x-4483` — the same combined event-stream/flatmap-stream advisory as above. |
| `flatmap-stream` | malware | **No results** — the malware-type lookup for `flatmap-stream` returned nothing. |

Both advisory lookups that did return records classified the packages as **critical** severity with CWE-506 (Embedded Malicious Code).

## Takeaways for our review

- **Publish-rights transfers are the softest target.** No technical vulnerability was needed — one email got a stranger publish rights to a package with millions of installs.
- **The registry tarball is not the repo.** The malicious code existed only in the published `index.min.js` + test fixture, invisible to repo-diffing.
- **New, zero-download dependencies added by a new maintainer should be an alert signal** — especially when popular alternatives (e.g. `Array.prototype.flatMap`, `lodash`) already exist.
- **Version pinning and lockfiles limit blast radius** — but only for fresh installs; cached `3.3.6` tarballs kept working, which is why the downgrade recommendation (to `3.3.4`) was issued for cached installs.
- **Audit your transitive dependencies**: `npm ls event-stream flatmap-stream` was the community's key triage command.

## Sources

See [SOURCES.md](./SOURCES.md).
