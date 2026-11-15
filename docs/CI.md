# Continuous integration

The build workflow checks the frontend and Rust code and packages the Tauri application for desktop platforms.

## Triggers and checks

`.github/workflows/build.yml` runs on pushes to `main` and `dev`, all pull requests, and manual dispatch. There are no path filters. New runs cancel obsolete runs for the same event and ref.

| Check | Pull requests and dev | Main |
| --- | --- | --- |
| Web (required) | Typecheck, production build, verified ZIP | Same |
| Windows x64 (required) | Rust checks and debug executable ZIP | Rust checks and release executable ZIP |
| Windows x64 (debug cache) | Not run | Rust checks and debug executable ZIP |
| Additional desktop targets | Manual `all_platforms` only, debug | Release builds |
| Required release checks | Require every selected job to succeed | Same |

The additional targets are Windows ARM64, Linux x64/ARM64, and macOS x64/ARM64. Linux produces DEB and AppImage packages; macOS produces and validates the app bundle and uploads a DMG. Windows ZIPs contain the executable and require WebView2 on the destination machine. Artifacts are unsigned CI outputs, not published releases.

The aggregate check fails if Web or Windows fails or is cancelled. Additional targets may be skipped only when they were not selected. The project currently has no frontend unit or UI tests, and the Rust starter has no application tests. Build checks validate compilation and packaging.

## Dependencies and caches

Node is selected by `.node-version`; Rust and its formatter/Clippy components are pinned by `rust-toolchain.toml`. `npm ci` and Cargo `--locked` use committed lockfiles. The composite action installs Linux build and packaging dependencies.

Native jobs build the frontend before Cargo checks, then reuse those assets when packaging. Debug jobs pass `--debug` explicitly to the Tauri CLI. Rust caches are separated by profile, OS and target. Only main writes Rust caches, allowing dev and PRs to reuse the default branch's caches. Artifacts are retained for 14 days.

## Local validation

Run from the repository root with the platform's Tauri prerequisites installed:

```sh
npm ci
npm run typecheck
npm run build
cargo fmt --manifest-path src-tauri/Cargo.toml --check
cargo test --manifest-path src-tauri/Cargo.toml --locked --all-features
cargo clippy --manifest-path src-tauri/Cargo.toml --locked --all-targets --all-features -- -D warnings
npm run tauri -- build --ci --debug --no-bundle -- --locked
```

## Branch protection

The repository ruleset targets `refs/heads/main` and `refs/heads/dev`. It requires a pull request, blocks force pushes and prevents deletion. Its bypass list is empty, so repository administrators are also subject to these rules. It does not require another person's approval, which permits a sole maintainer to merge a PR.

The ruleset configuration is stored in `.github/branch-ruleset.json`. GitHub settings must be applied through the API or repository settings; committing that file does not apply them. CI status checks are not enforced by this initial ruleset; `Required release checks` can be required after the workflow is installed and has run.

Reference: [GitHub repository rulesets API](https://docs.github.com/en/rest/repos/rules).
