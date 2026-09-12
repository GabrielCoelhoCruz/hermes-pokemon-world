# Pokémon World: product and technical handoff

Status: **draft consolidation; canonical source recovery required**.
Tracking: [planning map #1](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/1).
Source review date: 2026-09-12. Available runtime baseline:
`bdb3948bcf4b1135847ba66feeac18237a82805e`.

This document consolidates the accepted decisions recorded in issues #2–#12.
It is an implementation handoff draft, not a replacement for the accepted
ADRs or the exact acceptance contract that remain in unpublished local commits.
Closing a decision ticket does not establish implementation readiness.

## 1. Product outcome and scope

Pokémon World is a local-first operational surface for Hermes Desktop Profiles.
The user can scan the fleet, identify work needing attention, select a Profile,
submit a mission to its owner-routed persistent Bot Chat, and inspect results.
Pokémon characters, sprites, scenery, and terminology provide the local visual
experience. Presentation never supplies operational authority.

V1 requires Connected districts, compact Microregions, character/scenery
composition, a World Summary, a persistent Mission Composer, and a selected
Profile Inspector. The user's private Local Asset Pack may contain Pokémon
sprites and content. The distributable Core supplies original or neutral
fallback visuals and remains operational without the pack.

Games, competitive counters, employee-of-the-month ranking, trophies, rewards,
and tokens are excluded from v1. Their return requires a separate future ADR.
Dynamic ambience is deferred: clocks, day/night transitions, switchable skins,
expression-only effects, roaming/greetings, particles, and sounds. The existing
Office feature set does not define the target v1 scope.
Asset selection and final art direction remain implementation decisions.

Sources: [interaction scope #9](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/9#issuecomment-5511672903),
[local pack decision #8](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/8#issuecomment-5644570549).

## 2. User interaction and spatial hierarchy

Connected districts preserve a world-level scan and expose functional grouping.
A Fallback Edge accommodates Profiles without valid world semantics. District
membership is presentation and does not imply shared operational ownership.

| Surface | Required information and behavior |
| --- | --- |
| World Summary | Attention order: failed/unknown, unread result, working, external chat busy, idle. Selecting an item centers its Microregion and selects its Profile without unexpectedly opening chat. |
| Mission Composer | Remains globally available. Selecting an actionable Profile prefills the target; an explicit selector may change it. Show stable identity and owner. |
| Microregion | Compact Profile identity, responsibility, and operational signal; no full history or duplicated complete control set. |
| Profile Inspector | Identity/responsibility, owner, current state, latest-output summary, and only Kernel-authorized actions. |
| Bot Chat | Explicit open action or double-click opens the same persistent conversation. |
| External Chat Activity | Separate context, never represented as a World-submitted mission or eligible for World completion effects. |

Failed, unknown, and unread-result signals require text, icons, and accessible
semantics; color and animation alone are insufficient. Playful visuals cannot
obscure operational signals. Synthetic demonstration fixtures cannot become
operational targets; selecting one preserves the real Composer target and
keeps its operational controls disabled.

The topology and state decisions report prototype checks at desktop sizes and
390×844. These are historical reports about an unintegrated prototype, not
evidence for the current runtime. Real Desktop pane dimensions, narrow layouts,
touch, zoom/reflow, screen readers, and human visual judgment still require
production validation. Recover the prototype before using it as a layout
reference; exact measurements cannot be reconstructed from issue summaries.

Sources: [hierarchy #4](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/4#issuecomment-5496864496),
[topology #5](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/5#issuecomment-5501105215),
[state/prototype #7](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/7#issuecomment-5506443861).

## 3. Fleet scope and canonical identity

The World roster is the union of Profiles exposed by configured Desktop
connections. The operational roster includes only Profiles with one exact
current owner route. Canonical identity is the structured
`RoutedProfileRef { connectionId, profileId }`.

Identical Profile IDs on different connections are distinct. Display names,
aliases, trainers, partners, habitats, models, and providers are presentation
attributes. Cache and persistence keys must retain source identity; do not
substitute a display name or a collision-prone concatenation for the reference.
Owner ambiguity fails closed.

Source loss retains the last-known projection as **Source Unavailable**. It
does not authorize actions, confirm removal, or turn a Profile into a Fallback
Habitat. Only a successful complete authoritative roster response may confirm
removal. Renames, connection replacements, and ownership migration require
explicit migration; do not infer identity continuity from a matching label.

Source: [fleet identity #12](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/12#issuecomment-5504019684).
The precise ADR migration rules remain pending source recovery.

## 4. Operational truth and the headless Kernel

Project state through four independent axes: **Availability Gate**, **Mission
State**, **Attention Markers**, and **Interaction Context**. Derive the global
attention ordering without inventing authority, availability, completion, or
read evidence. The full axis variants, interleavings, and transition contract
belong to ADR 0003; this draft does not invent missing enum values or rules.

The reusable Kernel comprises owner resolution and leasing, persistent Bot Chat
lookup/create/open, versioned job state, event correlation, recovery polling,
and explicit `completed`, `failed`, and `unknown` outcomes. Preserve these
boundaries when extracting it:

- Put exact owner routing and route leases behind a host adapter.
- Keep persistent conversation resolution in one conversation registry.
- Move eligibility, persistence, events, polling, and reload recovery out of
  scene components and into the lifecycle coordinator.
- Put the guard against submitting over unknown work in command policy, not
  only in the Composer UI.
- Persist operational completion independently from presentation effects.
- Expose theme-neutral Profile snapshots and authorized commands to the World.

The baseline is still a single `plugin.js`: `sendTask` enters `startRound`,
`finishJob` enters `celebrate`, and the Office scene attaches recovery polling.
Its 45-test suite is a regression baseline, not proof of the new headless
architecture or Desktop/provider E2E behavior.

Sources: [Kernel research #2](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/2#issuecomment-5495667998),
[operational axes #7](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/7#issuecomment-5506443861).

## 5. Data ownership and the two manifests

| Data | Owner / location | Boundary |
| --- | --- | --- |
| Portable world semantics | Small versioned `ui_meta["hermes-pokemon-world"]` | Profile World Manifest; no asset bytes, local paths, or operational authority. |
| Desktop-local preferences/cache | Plugin storage | Local state; not a binary asset store or source of Profile identity. |
| Character/scenery bytes | User's private Local Asset Pack | Presentation files selected on that Desktop. |
| World mission truth | Operational Kernel | Owner/session-correlated state and recovery. |
| Shared Hermes avatar | Existing Profile `avatar` capability | Shared identity image, not a generic World asset store. |

Profile World Manifest Schema 1 is atomic and closed at root and nested levels,
with closed identity variants, bounded semantic IDs and plain-text labels,
and distinct `malformed` and `unsupported` outcomes. Reject unknown fields,
bytes/encoded payloads, paths, URIs, CSS, routes, permissions, sessions, and
operational state. Exact field sets and bounds must come from ADR 0002;
the summary does not supply enough information to implement its parser.

Metadata writes must be owner-routed, merge-preserving, revision-aware, and
verified by application readback. Missing assets preserve valid world semantics.
An absent, malformed, or unsupported **Profile World Manifest** instead creates
a deterministic Fallback Habitat in the Fallback Edge using Core seed version 1,
without automatic metadata writes. Recover the exact seed algorithm from the
canonical ADR before implementation.

The **Local Asset Pack manifest** is a different document: it maps presentation
slots to relative local files and declares schema version, pack version, and
compatible Core range. It may record media type, byte length, dimensions, and
SHA-256. Relative paths belong here, never in portable Profile metadata.
Exact limits, format allowlists, and pack schema encoding need an implementation
specification; Schema 1 of the Profile World Manifest does not define them.

Sources: [metadata boundary #3](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/3#issuecomment-5495668582),
[Profile World Manifest #6](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/6#issuecomment-5504023472),
[pack policy #8](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/8#issuecomment-5644570549).

## 6. Private Pokémon pack and failure behavior

V1 uses an explicitly selected local folder, not an executable bundle or ZIP.
The Core does not download, upload, synchronize, or collect telemetry about
private pack content, nor link to that content. Repository examples, screenshots,
CI artifacts, diagnostics, crash reports, issues, PRs, and releases must not
carry the private files. This permits the user's desired local Pokémon sprites
and content while keeping the distributable artifact independent of the pack.

Reject unsafe paths, traversal, links escaping the root, active/executable
content, unsupported formats, unsafe sizes/dimensions, duplicate slots, and
incompatible manifest versions. A structurally malformed, unsafe, or
incompatible pack manifest rejects the pack atomically. Keep the previous
valid configuration where possible and give an actionable local diagnostic.
An isolated missing/corrupt/unsupported asset in an otherwise accepted manifest
falls back for that slot only, preserving valid Profile world semantics.

Provide revalidation, disable-pack, and open-location controls. Never search
online for a replacement. No asset outcome may hide status, change mission
truth, or break operational controls.

The local folder requirement is a product contract, not evidence that the
current host can read it. `plugin.js` is blob-loaded; sibling-file references
are not served. Generic local-folder selection/read access needs a verified
host mechanism or a separately specified host-side import/protocol. Do not
persist blob URLs as durable asset references or put file bytes in `ui_meta`
or plugin storage. The existing unchecked metadata image string and image
element without load-error fallback do not meet the future pack contract.

Sources: [pack policy #8](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/8#issuecomment-5644570549),
[SDK research #3](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/3#issuecomment-5495668582),
[runtime baseline](../../plugin.js).

## 7. Architecture, distribution, and migration

Accepted ADR 0005 requires modular source behind deep interfaces and adapters,
with a deterministic build emitting exactly one plain-ESM `plugin.js`. Only
mapped SDK/React runtime imports are allowed. End users do not run an
installation-time build. The Kernel remains inside the standalone Desktop
plugin. Build tooling is future work; the current repository has no production
build command. The generated artifact must retain the existing installation
contract while source ownership becomes modular.

Keep installed plugin ID/folder `hermes-office`. The canonical World route will
be `/world`; `/office` is only a bounded compatibility alias where supported
and collision-free. During strangler migration keep one active plugin,
Coordinator, event/effect owner, envelope writer, and renderer. Shadow
comparison is read-only and cannot duplicate effects. Recover ADR 0005 before
specifying exact envelopes, cutover operations, or rollback procedures.

The immediate implementation dependency is a generic typed PluginStorage
read-outcome capability: `missing | value | malformed | read-failure`, plus a
minimum compatible Desktop gate. Both must be implemented and independently
verified before the dependent storage/migration work proceeds. Existing
best-effort storage handling is not evidence of this capability. This handoff
does not authorize or perform an external Hermes SDK change.

Source: [architecture #10](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/10#issuecomment-5516601641).

The [pinned upstream prerequisite audit](host-prerequisite-audit.md) confirms
that the inspected storage API still collapses absent, malformed, and failed
reads. It also distinguishes existing file-picker/reveal conveniences from
the complete local-folder pack capability still to be established.

## 8. Acceptance and readiness

The accepted contract defines **54 rows**, partitioned as **6 / 40 / 8** across
`IMPLEMENTATION_READY`, `LOCAL_E2E_READY`, and `RELEASE_READY`. The #11 closure
reports the following historical state, not results from this consolidation:

| Rows | Reported status / evidence |
| --- | --- |
| DEP-01 through DEP-04 | PASS / MIXED |
| DEP-05, DEP-10, DEP-12, DEP-13, DEP-14 | BLOCKED / NONE |
| Remaining 45 rows | NOT RUN / NONE |

DEP-05 and DEP-10 concern the typed storage capability and compatible Desktop
pin. The summaries do not define DEP-12 through DEP-14 or enumerate the other
rows. Do not guess their meanings, renumber them, or replace the 54-row contract
with a new checklist. No readiness gate is claimed to have passed here.

The contract's reported evidence dimensions include exact-SHA
browser/Desktop/gateway/backend/session/reload behavior; storage migration and
corruption; one writer under hot reload; fallback Profiles versus synthetic
fixtures; static Pokémon v1 presentation without games/rewards; automated
checks versus VoiceOver/human review; and provider cost, cost-stop, retention,
cleanup, and readback. These must be bound to their exact canonical rows after
the contract is recovered. The pack privacy boundary from #8 also applies when
collecting evidence; use original/neutral assets in published captures.

Source: [handoff evidence #11](https://github.com/GabrielCoelhoCruz/hermes-pokemon-world/issues/11#issuecomment-5517050903).

## 9. Canonical source recovery

The issue closures explicitly report unpublished local commits. On 2026-09-12,
the current checkout lacks `CONTEXT.md`, the ADRs, the topology prototype, and
the 54-row handoff contract. GitHub's commit API returned HTTP 422 for
`eca9f574ff72bbd3c5942eae18f7190132c6c3de` (no commit found).

| Source artifact | Reported commit | Resolution |
| --- | --- | --- |
| Office Kernel research report | `329ac6bcd8f4866d6181c6edfd08d0c8e90f90fc` | #2 |
| Asset/metadata research report | `cb99f42f5ea05083683b5c47debd80ac26251034` | #3 |
| Final topology prototype | `b0277d40081d543a92c087802de149e51a6f0f4a` | #5 |
| ADR 0001: routed Profile identity | `7fac1fa89e6e2b103029431e274e7fec58b5ec75` | #12 |
| ADR 0002: world manifest/fallback | `7fac1fa89e6e2b103029431e274e7fec58b5ec75` | #6 |
| ADR 0003: operational axes | `f1fbb2c9297cb2e73c89eafe475795cb269f228d` | #7 |
| Later topology/state prototype | `c2370da8c3082688ea6032b9f1d894cd1e5d893d` | #7 |
| ADR 0004: interaction scope | `815565e2fa784b93bee79989c0735352cc5da214` | #9 |
| ADR 0005: architecture/migration | `0d4099fd23d1e9152389cf56c3aaefdfcc440aee` | #10 |
| `docs/handoff/implementation-readiness-and-e2e-evidence.md` | `eca9f574ff72bbd3c5942eae18f7190132c6c3de` | #11 |

The reported #11 commit descends from ADRs 0001–0005; recovering that branch
should recover those documents and `CONTEXT.md`, subject to tree inspection.
Research and prototype worktrees may require separate exports. Verify the
recovered file bytes against published hashes where supplied, including the
handoff contract SHA-256
`d818110cd7df9ba5ae28d1979364b57db8b557165a69d3e604e82c16650fce8c`.
Do not create replacement ADRs under the accepted filenames from these summaries.

See [recovering the canonical sources](recover-canonical-sources.md) for exact
Mac-side checks and a separate recovery branch that leaves `master` untouched.

## 10. Handoff sequence and completion rule

1. Recover the canonical documents, reconcile this draft with their exact
   contracts, and add durable links to the actual source files.
2. Incorporate #8's accepted private-pack policy alongside those earlier
   decisions. Preserve exact Profile manifest/identity rules and the 54
   acceptance rows; identify any unresolved conflicts explicitly.
3. Have an independent Codex reviewer check decision coverage, dependency order,
   and acceptance traceability. Make the consolidated handoff reviewable in a
   PR. Close #1 only once the source-complete planning deliverable is accepted;
   documented implementation blockers may remain without being declared passed.
4. Specify the typed storage capability and compatible Desktop pin as the
   prerequisite work. Separately resolve local-folder asset access. Verify
   these external mechanisms before depending on them in production work.
5. Once the applicable implementation gate passes, create an executable brief
   for the modular build and headless Kernel extraction, preserving Office
   behavior through the controlled single-owner cutover.
6. Add routed world projection, metadata/asset resolution, then the World
   renderer and local Pokémon presentation. Use the recovered evidence contract
   to determine when Desktop E2E and release claims become justified.

This ordering refines the earlier suggestion to start immediately with Kernel
extraction: issues #10 and #11 establish a storage/host prerequisite first.
The planning PR itself changes documentation only. No SDK capability, new
runtime, provider execution, asset pack, or release is produced by it.
