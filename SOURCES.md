# Sources

Primary sources relied on for the research note in [NOTES.md](./NOTES.md).

## Tracking issue (primary)

- **dominictarr/event-stream#116 — "I don't know what to say."** (opened 2018-11-21 by @FallingSnow; maintainer response and community analysis in-thread)
  https://github.com/dominictarr/event-stream/issues/116
- **dominictarr/event-stream#115 — "Backdoored dependency? flatmap-stream-0.1.1 and flatmap-stream-0.1.2"** (first public report, opened 2018-11-19 by @NewEraCracker)
  https://github.com/dominictarr/event-stream/issues/115

## Commits

- "add flat map" — commit by `right9ctrl` adding `flatmap-stream` as a dependency (2018-09-09):
  https://github.com/dominictarr/event-stream/commit/e3163361fed01384c986b9b4c18feb1fc42b8285
- "3.3.6" — the version-bump/publish commit (2018-09-09):
  https://github.com/dominictarr/event-stream/commit/5999958dfc1b0a80e6caeac4cdc76b3b828bdfe2

## Package repositories

- event-stream: https://github.com/dominictarr/event-stream
- flatmap-stream: https://github.com/hugeglass/flatmap-stream

## Security advisories (GitHub advisory database)

- GHSA-mh6f-8j2x-4483 — "Critical severity vulnerability that affects event-stream and flatmap-stream" (reviewed; critical):
  https://github.com/advisories/GHSA-mh6f-8j2x-4483
- GHSA-9x64-5r7x-2q53 — "Malicious Package in flatmap-stream" (reviewed; critical):
  https://github.com/advisories/GHSA-9x64-5r7x-2q53
- npm advisory 737 (referenced from GHSA-9x64-5r7x-2q53):
  https://www.npmjs.com/advisories/737

> Note: the **malware-type** advisory database lookups for both `event-stream` and `flatmap-stream` returned **no records** at the time of this research; all records above came from the **reviewed** advisory type.

## Additional references from the issue discussion

- Minified source of the malicious package (shared in-thread for analysis):
  https://unpkg.com/flatmap-stream@0.1.1/index.min.js
- In-thread payload analysis comments referenced by the issue body:
  - https://github.com/dominictarr/event-stream/issues/116#issuecomment-441759047
  - https://github.com/dominictarr/event-stream/issues/116#issuecomment-441746370
  - https://github.com/dominictarr/event-stream/issues/116#issuecomment-441749105
