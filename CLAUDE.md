# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Coding standards, correctness invariants, and the list of traps already fallen into live in [AGENTS.md](AGENTS.md). Read it first. What follows is only what it does not cover.

## Commands

```sh
cargo test --locked                                   # whole workspace, what CI runs
cargo test -p hatchzip --test extract                 # one integration test file (crates/cli/tests/*.rs)
cargo test -p hatchzip --test extract <name>          # one test by name substring
cargo test -p hatchzip-core                           # engine unit tests only
cargo test -p hatchzip-gui                            # GUI unit tests, shell flags, headless UI tests
cargo fmt --all --check && cargo clippy --all-targets --locked -- -D warnings
git config core.hooksPath .githooks                   # pre-commit runs fmt, pre-push mirrors the CI check and test jobs
```

`extract` deletes its input. Test on copies, and prefer the `check` and `list` subcommands when exploring.

## Workspace

Four crates:

- `crates/core` (`hatchzip-core`): the engine. Front ends call `session.rs`; `engine.rs`, `journal.rs`, `restore.rs` and `archive/*` sit behind it. Anything OS-specific goes in `platform/`.
- `crates/cli` (package `hatchzip`, binary also installed as `hz`): `cli.rs` is the clap definition, `tests/` holds the end to end suites (formats, codecs, crash/resume, passwords, restore, large sparse archives, tar.gz).
- `crates/gui`: the Slint desktop app. It drives the same `session`, `engine` and `restore` modules and also accepts the `--extract-here`, `--extract-to`, `--test` and `--register-shell` flags that file manager menus call. Its own `src/platform/` is per-OS too.
- `crates/shell`: registers and removes the right-click menu entries for the GUI, per user, only under names HatchZip owns.

## Gotchas

- The Rust version is pinned to 1.95 in `rust-toolchain.toml`, and the workspace `rust-version` is 1.93 (sevenz-rust2 needs it). AGENTS.md mentions an MSRV of 1.89, which is stale.
- On Linux the GUI crate needs `libfontconfig1-dev` to build. `hatchzip-core` also compiles the UnRAR C++ source (`unrar_sys`), so a C++ compiler is needed everywhere; on Windows `platform/windows.rs` links `advapi32` for it.
- `archive/rar.rs` frees RAR sets a whole volume at a time and decodes through UnRAR. WinRAR 7 can no longer write RAR 4, so `crates/cli/tests/rar.rs` writes its RAR 4 volumes itself and uses WinRAR-made fixtures for RAR 5 (how to rebuild them is in that file's header).
- `docs/how-it-works.md` and `docs/formats.md` are the design references for the journal, batch ordering, truncation and per-format unit rules. Changes to those mechanics start with an issue (see CONTRIBUTING.md).
- `TODO.md` is ticked only when a change is merged and tested.
