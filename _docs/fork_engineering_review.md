# ArtCraft Fork Engineering Review

## Scope and safety rules

This repository is a fork of `storytold/artcraft`. This branch is
`improve/repo-review-and-quality`; the default branch and upstream repository
are not modified. The project instructions explicitly require keeping
`external/` and `secrets/` untouched and preserving untracked/ignored local
files. This review does not prune files, caches, assets, or generated data.

## Architecture observed

- Rust workspace with a Tauri desktop application and a separate pure-Rust
  `artcraft-native` egui application.
- Shared API client crates, SQLite task persistence, and SQLx offline metadata.
- A large React/TypeScript frontend managed with Nx and Vite.
- Provider integrations for ArtCraft services, fal, Midjourney, Grok, Sora,
  and World Labs.
- CI workflows include macOS and Windows publishing; the native app documents
  a dedicated `cargo test -p artcraft_native` test command.

## Findings and recommendations

### High priority

1. **Fork maintenance is the main architectural risk.**
   This is a large fork of an actively maintained upstream project. Before
   changing core behavior, establish an upstream sync policy: record the base
   upstream commit, keep custom changes isolated, and regularly compare the fork
   against `storytold/artcraft:main`. Avoid bulk merges without reviewing
   migration, API, and generated-file changes.

2. **Two desktop implementations increase maintenance cost.**
   The repository contains both Tauri/React and egui/Rust desktop clients.
   Keep shared API contracts in the existing client/schema crates. Avoid
   duplicating provider request validation, persistence rules, or authentication
   behavior in the two frontends.

3. **Authentication and provider credentials require strict handling.**
   Keep credentials in the existing OS/user-scoped credential stores. Never add
   tokens, cookies, certificates, or local environment files to source control
   or diagnostic output. Review logs and CI output before enabling verbose
   request tracing.

### Medium priority

4. **CI should distinguish fast checks from release publishing.**
   Add independent pull-request checks for formatting, Rust tests, frontend
   type-check/lint, and targeted native-app tests. Keep signing, notarization,
   and release publication in protected release workflows only.

5. **The frontend exposes several independent validation commands.**
   Standardize documented commands for Nx build, TypeScript type-check, lint,
   and tests. Confirm each command works from the workspace root before making
   it a required CI gate.

6. **Build workflows should minimize mutation and secret diagnostics.**
   Prefer temporary generated config files over editing tracked config in-place.
   Keep certificate checks to non-sensitive metadata and never print secret
   values or private-key material.

7. **Native and Tauri parity needs explicit contract tests.**
   Add tests around model capability parsing, provider submission validation,
   job resume behavior, authentication hand-off, and media persistence where
   the clients share behavior.

## Recommended implementation sequence

1. Establish and document upstream base commit and divergence policy.
2. Add PR-only CI checks that do not require signing secrets.
3. Add cross-client contract tests for shared API and provider behavior.
4. Review credential storage, redaction, and crash-recovery paths.
5. Reduce duplicate business logic by moving shared validation into existing
   Rust client/schema crates.
6. Only then consider larger UI or workspace restructuring.

## Changes made in this branch

- Added this repository-specific review and a staged improvement plan.
- No application source, credentials, external dependencies, generated assets,
  database migrations, or local data were changed in this first review pass.

## Validation status

No Rust or frontend build was run in this GitHub-only pass. The repository is
large and includes native toolchain and platform-specific requirements. Before
merging implementation changes, run `cargo fmt --check`, targeted
`cargo test`, frontend type-check/lint/build, and platform build smoke tests
in CI. Treat any release/signing workflow as separate from pull-request CI.
