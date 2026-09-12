# Hermes Desktop prerequisite audit

Audited on 2026-09-12 against upstream `main` resolved to
`c6f87deb2c38d75518c793790f1cc9afa37f0695`. This is a source inspection pin,
not a verified installed Desktop version or minimum compatible release.

## Storage prerequisite remains open

The inspected `PluginStorage` interface exposes `get`, `set`, and `remove`.
Its getter returns the caller's fallback when the raw read is null or JSON
parsing fails. The lower-level `readKey` also returns null on storage-access
exceptions. The public getter therefore cannot distinguish the three cases
required by the accepted architecture: missing, malformed, and read failure.
It has no typed outcome method in this inspected contract.

Sources: [PluginStorage interface and implementation](https://github.com/NousResearch/hermes-agent/blob/c6f87deb2c38d75518c793790f1cc9afa37f0695/apps/desktop/src/contrib/plugin.ts#L33),
[raw read implementation](https://github.com/NousResearch/hermes-agent/blob/c6f87deb2c38d75518c793790f1cc9afa37f0695/apps/desktop/src/lib/storage.ts#L29).

A disposable Node VM probe extracted the actual getter and `readKey` bodies
from these pinned files, removed only the TypeScript annotations on `readKey`,
and injected a localStorage stub and no-op persistence observer. It used a
distinct string sentinel as fallback. Observed results:

| Input | Result |
| --- | --- |
| Missing key | Fallback sentinel |
| Invalid JSON | Same fallback sentinel |
| Exception from localStorage.getItem | Same fallback sentinel |
| Valid JSON object | Parsed object |
| Valid JSON null | Null, not the sentinel |

All five observations matched the inspected source. This probe establishes
information loss in this API path; it does not test real Electron storage,
migrations, hot reload, persistence durability, or the future typed API.
It does not turn any row of the canonical acceptance contract into PASS.

The next storage work specification needs to preserve those distinct outcomes
before the information is discarded. The plugin cannot reconstruct them from
the existing fallback return value. Independently verify the sanctioned host
capability and pin a compatible Desktop before dependent implementation.
The exact API naming and migration behavior still require the recovered ADR
0005 and the acceptance contract; this audit does not choose them.

## Local pack access is a separate capability

The inspected curated `PluginOs` API includes a single-file open dialog and a
path-reveal operation. Its public dialog options have no directory-selection
option, and that interface exposes no file-content reader. These conveniences
do not, by themselves, establish the complete local-folder pack mechanism.
This finding is scoped to the inspected public interface, not a claim that
Electron or all possible Hermes APIs lack filesystem capabilities.

Source: [PluginOs and dialog options](https://github.com/NousResearch/hermes-agent/blob/c6f87deb2c38d75518c793790f1cc9afa37f0695/apps/desktop/src/contrib/plugin.ts#L44).

The pack implementation must establish selection, reading, bounded path
resolution, and renderer asset delivery on the Desktop machine. A selected
path alone is insufficient, especially with remote Profile backends. Keep this
dependency separate from the storage outcome capability and retain #8's local
Pokémon content policy.

## Impact on the handoff

The [consolidated specification](pokemon-world-specification.md) remains a
draft. Upstream inspection confirms that the storage prerequisite cannot be
marked satisfied from this pin. It does not remove the need to recover the
unpublished ADRs and exact 54-row evidence contract. No Hermes source or plugin
runtime was modified during the audit.
