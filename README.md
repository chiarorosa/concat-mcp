# concat-mcp

Planning-stage MCP integration for [Concat](https://github.com/jub0t/Concat), the video editor created by **Jareer ([@jub0t](https://github.com/jub0t)) and Concat contributors**.

**Status: documentation-only proposal. No MCP server, installer, package, or working integration is available yet.** This is an independent community initiative by [@chiarorosa](https://github.com/chiarorosa), not an official Concat project or an endorsed integration.

The goal is to let people use their preferred agent to inspect, edit, preview, and export Concat projects through the editor's own API. Editing semantics, validation, project ownership, and undo must remain in Concat; this project would provide a thin MCP adapter and installation/discovery experience.

## Proposal

Read the [implementation proposal](docs/proposal.md) for architecture, scope, milestones, acceptance criteria, installation paths, and decisions requiring upstream agreement.

This complements [Concat issue #337](https://github.com/jub0t/Concat/issues/337), opened by [@1-Bart-1](https://github.com/1-Bart-1). It does not claim ownership of that proposal or replace its offered implementation. We want to coordinate before writing overlapping code. Upstream may prefer an in-tree MCP crate with independent packaging; this repository can remain documentation and distribution coordination if so.

## Planned entry points

- From an agent client: documented installation and configuration of a versioned MCP package.
- From Concat: a future **Connect an agent** flow that configures a supported client and checks connectivity.

Both are plans, not currently usable installation instructions. Giving a client this repository URL is not a guarantee of automatic installation. No automatic downloads or client configuration changes are implemented here.

## Scope

Start with local MCP, explicit project authorization, capability discovery, and a small inspect–edit–preview–undo workflow. An integrated chat panel and provider/runtime connections are later work, not requirements for using an external agent. There is no general shell tool, cloud service, or paid API requirement in the proposed MCP server.

## Credits and license

Concat is the upstream editor, created by Jareer (@jub0t) and Concat contributors. All credit for its editing engine, application, API, and existing transports belongs to their respective authors. This repository currently contains documentation and the license text only; no Concat implementation code has been copied.

Licensed under **GNU Affero General Public License v3.0 or later (`AGPL-3.0-or-later`)**, matching [Concat's workspace license declaration](https://github.com/jub0t/Concat/blob/60ecb4a1fda62501f67afaf7e036143c95f196f8/src/Cargo.toml). [LICENSE](LICENSE) is copied verbatim from that upstream revision. Any future reused source must retain its original copyright and license notices.

Hyperframes is acknowledged as a design reference in the proposal; no Hyperframes code is included.
