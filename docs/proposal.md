# Proposal: agent-ready Concat with a separately distributable MCP integration

Date: 2026-10-17. Author: @chiarorosa. Status: proposed, not approved or implemented.

## Purpose

Allow a person and their chosen agent to work on the same Concat project: inspect the timeline, identify explicit targets, make authorized edits, observe results, undo changes, and export. Keep the user in control and preserve Concat as the source of editing semantics.

The first milestone is deliberately small: change a title and a clip's volume in the project open in the window, inspect the result, and undo from the GUI. A chat panel and broad production automation follow only after that foundation works.

This repository is an initial documentation home, not a competing editor, an official integration, or a claim that the maintainer has accepted the direction.

## Existing foundation and coordination

This assessment is based on Concat main `60ecb4a1fda62501f67afaf7e036143c95f196f8`.

- `concat-api` is a shared dispatcher built around existing project commands and host services. Its protocol already covers project lifecycle, edits, undo/redo, media import/probe, catalogue, templates, preview frames, exports, cancellation, and events.
- `concat-server` provides authenticated JSON-RPC connections and optional gRPC. The built-in server shares open-project ownership with the window and deliberately prevents an API session from editing the window's project.
- Therefore, an external adapter alone cannot provide live collaboration. It must not bypass the ownership lock or write project files behind the GUI's back.
- The canonical MCP documentation describes planned/prototyped functionality, not a shipped native transport.

[Issue #337](https://github.com/jub0t/Concat/issues/337), opened by @1-Bart-1, already proposes live GUI editing and MCP and offers implementation work. We should coordinate with that author and @jub0t before writing overlapping code. Existing comments from @d-enk provide useful experience with the JSON-RPC workflow. There is no maintainer acceptance recorded in the discussion reviewed for this proposal.

Sources: [API](https://github.com/jub0t/Concat/blob/60ecb4a1fda62501f67afaf7e036143c95f196f8/src/crates/concat-api/src/lib.rs), [protocol](https://github.com/jub0t/Concat/blob/60ecb4a1fda62501f67afaf7e036143c95f196f8/src/crates/concat-api/src/message.rs), [server](https://github.com/jub0t/Concat/blob/60ecb4a1fda62501f67afaf7e036143c95f196f8/src/crates/concat-server/src/lib.rs), [window ownership](https://github.com/jub0t/Concat/blob/60ecb4a1fda62501f67afaf7e036143c95f196f8/src/crates/concat/src/panes/settings.rs), [MCP status](https://github.com/ConcatApp/website/blob/9653fe754dffd23b8708fa2037bb591eb51ebe9b/src/content/docs/transports/mcp.mdx).

## Recommended division of responsibilities

| Layer | Responsibility | Proposed home |
| --- | --- | --- |
| Editor/domain API | Commands, validation, project ownership, revisions, undo, rendering and jobs | Concat upstream |
| MCP adapter | Typed tools mapped to the API, discovery, connection and result translation | Separately distributable; repository location subject to upstream preference |
| Client setup | Versioned installation, supported client configuration, diagnostics and removal | MCP distribution, with optional Concat UI entry point |
| Agent runtime | Model interaction, conversation, tool loop and usage | Existing user-selected client first; an optional dedicated runtime later |
| Integrated chat | Context attachments and controls in the editor | Later Concat contribution, if wanted upstream |

Separate distribution does not require duplicating code or independent development. If upstream prefers an in-tree crate, we can package that component independently and use this repository for documentation/distribution coordination. A standalone adapter, if selected, should communicate through the public protocol rather than vendor editing logic.

Do not commit to a language, SDK, package registry, or executable name until deployment constraints and upstream preferences are agreed. Evaluate maintained MCP SDKs and license compatibility rather than inventing a protocol variant.

## API as the durable contract

The API should describe meaningful editing operations, not mirror every menu item. New editor functionality should be assessed for automation exposure alongside its implementation.

A capability contract should report protocol/API version, build/platform features, operation schemas, parameters, limits, side effects, target types, undo behavior, and relevant events. Prefer deriving schemas from authoritative types. A curated mapping determines which operations are appropriate for MCP exposure; not every internal function becomes a tool automatically.

Maintain a coverage table: operation → GUI → API → MCP → tested versions. A new capability needs documentation, a contract test, and either an MCP mapping or an explicit unsupported status. Missing fields must be represented as unknown, not invented.

The adapter negotiates compatibility at connection time. Unsupported versions fail with an actionable diagnostic or expose an explicitly smaller tested capability set. A package release documents supported Concat versions; CI tests supported releases plus upstream main where practical. A scheduled or dependency-triggered check can identify drift, but must not automatically publish an unreviewed adapter release.

New menu entries alone do not guarantee safe automatic support. Schema compatibility, target semantics, permissions, and tests remain necessary. Use bounded tool families and targeted discovery to avoid overwhelming the model with an unstructured catalogue.

## Live session semantics

The GUI remains the sole owner of its active session. Authorized requests for that project are forwarded through a bounded queue to the appropriate event loop and receive a reply through a channel. Headless projects remain exclusively owned by a headless session. Do not open a second writable session for the same project.

Reads include unsaved state and a revision token. Writes explicitly identify project, timeline and target using native IDs where adequate. A changed user selection must not redirect an already addressed operation. The initial conflict policy is simple: reject a stale revision, reread and replan. Do not implement speculative automatic merging.

Apply edits through existing commands. One call is one undo unit; a batch can group a gesture if its atomicity is verified. A failure midway through a batch must either leave no change or report exactly which changes occurred. Do not promise rollback unless tested.

A multi-call agent run may span multiple undo entries. A later “undo this run” action must not remove interleaved human edits. Initially expose ordinary undo/history and reserve run-level undo for histories where ownership and contiguity are proven.

Project switch/close invalidates queued requests tied to the old session. Expensive previews and exports should use immutable snapshots identified by revision and the existing shared job/export resources, without blocking the GUI.

## MCP MVP and execution evidence

Start with local stdio. Reuse existing token-based server access for a GUI connection. Keep stdout reserved for MCP and logs on stderr. Remote HTTP/TLS/OAuth deployment is a separate design, not an MVP requirement. Browser WebMCP is not a prerequisite for the native application.

Initial tool families: version/capabilities, bounded project inspection, media probe/import within scope, catalogue lookup, a small typed edit subset, preview frame, undo/redo, export and job progress/cancellation. Expand only with tested coverage. Preserve structured errors rather than flattening everything into text.

Proposed operation receipts distinguish rejected, queued, applied in memory, persisted, verified by independent readback, and failed. Include request/run identifiers, explicit targets, revisions, changed/no-op status and available evidence. Names are provisional and must fit API compatibility decisions.

Cancellation removes queued work and requests cooperative job interruption. It does not reverse an already applied edit. A timeout or lost reply does not prove that nothing happened. Writes need an idempotency strategy and result lookup within a declared retention scope; after restart, do not promise durable deduplication unless implemented. Verify state before retrying an uncertain mutation.

A successful tool call is not proof of a correct video. Read back changed properties and inspect frames at relevant moments; use audio and duration checks where applicable. Do not claim saved or verified when only dispatch has completed.

## Installation from either direction

### From Claude Code, Codex, or another MCP client

Provide a dedicated repository with a clear status, supported clients and versions, versioned package/release artifacts, integrity information, installation/removal instructions and a diagnostic command. The desired experience is that a user can give a capable client this repository URL and ask it to configure the MCP connection.

This is a usability goal, not a promise that every client auto-installs arbitrary repositories. Client-specific configuration must follow the official documented mechanism at implementation time. Do not publish commands for packages that do not exist.

### From Concat

Propose a **Connect an agent** entry point: detect supported installed clients; show what will be installed or configured; obtain the user's action; configure a scoped connection; test it; and offer repair/removal. Preserve unrelated configuration, handle malformed files safely, create recoverable backups and avoid duplicating entries. Never overwrite client configuration wholesale.

Both entry points use the same adapter and compatibility contract. They should diagnose an absent/closed editor, missing permission, wrong token, incompatible version and unavailable capability. Installation should not silently download a runtime or change project permissions.

## Optional integrated chat and model connections

External agents are sufficient for the first useful release. A later native chat panel can reuse the same tools and authorization rather than introduce a second automation stack.

The panel would support messages with explicit media/clip/time-range attachments; current selection/playhead context; visible tool activity and results; streaming; stop; a bounded message queue; and safe undo/history links. Creative preview and manual editing must remain usable throughout a run. Drawn annotations and advanced steering are later additions.

Start with one runtime offering an official execution interface, then optionally one API provider. Login through an officially supported agent account and API-key billing are distinct integrations. A chat subscription does not automatically grant generic API access. Do not extract private browser cookies or credentials to imitate supported access.

Runtime compatibility, authentication, cancellation, sandboxing and billing presentation need validation on each supported platform. API keys belong in an OS credential store, never in projects, logs or exported diagnostics. Display usage when provided and unknown when unavailable; enforce run limits without pretending estimates are exact.

Concat acting as a client of third-party MCP servers—for transcription, generation or other services—is a separate opt-in extension with independent data-sharing permissions and costs.

## Security and resource requirements

Live editing is off by default and authorized per project. Apply authorization at the API boundary too, so a direct JSON-RPC client cannot bypass MCP restrictions. Separate reading, editing, media access, export and sending content to a model/provider.

Validate authorized filesystem roots, including symlinks/junctions, traversal and platform-specific paths. Project content, subtitles, metadata and external tool output are untrusted data, not instructions granting more authority. Provide no arbitrary shell tool as part of the editor API. If a chosen agent runtime has its own shell, validate its isolation before claiming that it is confined.

Keep bounded queues, output sizes, image resolution, tool rounds and deadlines. Reuse existing GPU/export contention controls. Filter events by authorized project/session. Disconnect/revoke must stop further unauthorized calls. Process cleanup must only terminate processes started for this integration.

Do not upload media or frames without the selected data-sharing policy. Keep telemetry separate from project content and public feedback opt-in. Crash recovery follows real native persistence guarantees; do not imply in-memory undo survives a restart.

## Proposed milestones and work I am offering

I am proposing to contribute this incrementally, coordinating ownership with #337 rather than assuming its work is unclaimed:

| Stage | Contribution | Acceptance evidence |
| --- | --- | --- |
| 0 | Agree scope, ownership, protocol shape and repository/package location | Maintainer feedback and a small RFC; no overlapping implementation |
| 1 | Inventory current API coverage and define tested discovery/compatibility | Capability fixtures and documented gaps; no invented tools |
| 2 | Contribute live read/write bridge upstream where agreed | Unsaved reads, visible title/volume changes, shared undo, safe project switching |
| 3 | Add conflict, batch, receipt and retry semantics in small upstream changes | Stale writes rejected; batch behavior proven; uncertain results recoverable within stated guarantees |
| 4 | Implement thin local MCP adapter and client documentation | Two real MCP clients complete inspect–edit–preview–undo; direct API parity |
| 5 | Publish a versioned distribution with diagnostics and configuration/removal support | Install, upgrade, connect and uninstall verified without damaging existing client settings |
| 6 | Contribute optional Concat connection UI | Both installation directions reach the same adapter and authorization |
| 7 | Expand tested production coverage and skills/examples | Clips, titles, audio, available captions/effects/templates and export verified |
| 8 | Discuss native chat/runtime/API integration separately | Explicit upstream interest and tested runtime contract before implementation |

Stages 1–3 may be split or reordered with the existing #337 contributor. No schedule or maintainer endorsement is implied. I can take adapter/distribution/documentation work while coordinating core changes with whoever owns the live-session bridge. An in-tree implementation remains an acceptable outcome.

Each implementation PR should be small, follow upstream contribution rules and DCO, show the behavior it changes, and include meaningful regression/contract coverage. No broad refactor is required to begin.

## Test and release gates

Use deterministic fixtures and a simulated client/provider in ordinary CI; no paid model calls or user credentials. Separately opt into real client/model testing and record versions.

Required regression cases: changed selection cannot redirect writes; human edits cause stale-revision rejection; failed batches do not hide partial application; retries do not duplicate committed edits within the promised scope; cancellation reports already-applied work; project switching cannot affect another project; unauthorized paths and malicious media instructions cannot widen access; disconnects and crashes have truthful recovery behavior.

An end-to-end test must enter through the public MCP boundary, authorization and dispatcher, not just invoke private handlers. Test title and volume changes in a real GUI session, compare readback/frames and manual undo/redo, and verify an exported fixture's duration and audiovisual output. Run Windows and Linux checks before advertising them; other platforms only after validation. Compare rendered output with documented tolerances rather than claiming bit-identical encodes across GPUs/codecs.

Release metadata should name the Concat/API/MCP/client versions tested. Default CI must not publish automatically on schema drift. The pilot gate is no wrong-target writes, no silently lost changes and passing deterministic contract tests; creative quality is assessed separately.

## Design references and attribution

[Hyperframes](https://github.com/heygen-com/hyperframes) is a useful reference for agent-oriented authoring. Its documentation describes a Studio-connected agent path, contextual editing, and a distinct hosted MCP path. We use those ideas as product requirements, not as evidence that Concat must adopt its HTML/browser architecture or hosting model.

References: [agent connection](https://github.com/heygen-com/hyperframes/blob/main/docs/studio-app/connect-your-agent.mdx), [contextual editing](https://github.com/heygen-com/hyperframes/blob/main/docs/studio-app/edit-with-the-agent.mdx), [inspection and editing tools](https://github.com/heygen-com/hyperframes/blob/main/docs/guides/webmcp.mdx), [hosted MCP](https://github.com/heygen-com/hyperframes/blob/main/docs/guides/mcp.mdx). These are documentation/code observations, not an independent benchmark or a claim that every reference feature is production-proven. No Hyperframes code is included.

Concat was created by **Jareer (@jub0t) and Concat contributors**. Its existing engine, application, API and commands are the foundation. The proposal is offered as a community contribution, with no claim of official status. This repository uses `AGPL-3.0-or-later`, matching the upstream workspace declaration, with the upstream LICENSE text preserved verbatim. Reused code must retain its original notices.

## Decisions requested before implementation

- Does an opt-in live-session bridge fit Concat's intended safety model?
- How should work be divided with @1-Bart-1 and other interested contributors?
- Prefer an independently maintained thin adapter, or an in-tree crate with separate packaging?
- Which capability/schema and compatibility contract is acceptable upstream?
- Is an optional connection UI desirable after an external MCP client works reliably?

The immediate proposal is stages 0–4, with distribution and UI described as follow-ups. This keeps the first review focused while preserving a clear route toward the broader experience.
