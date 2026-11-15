# Repository instructions

## Project scope

- This repository is `piano-practice`, a Tauri 2 starter application with Vue 3, TypeScript and Rust.
- Put UI code in `src/`, static assets in `public/`, native commands in `src-tauri/src/`, and Tauri configuration and capabilities in `src-tauri/`.
- Validate data at IPC and file boundaries. Grant only the Tauri capabilities required by a feature. Do not add broad shell, filesystem or remote access as a shortcut.

## Implementation and validation

- Keep changes focused and preserve unrelated work. Follow existing module boundaries; add abstractions only when they simplify a concrete responsibility.
- Use Vue Composition API with `<script setup lang="ts">`. Keep state ownership explicit and clean up event listeners and timers with their owning component or composable.
- Keep TypeScript strict checks enabled. Avoid `any`, unchecked assertions and compiler suppressions. Use Rust `Result` for recoverable errors.
- Use Node.js 26.x and npm. Run `npm ci` for clean installs. Commit `package-lock.json` and `src-tauri/Cargo.lock` when dependencies change.
- Use the Rust version and components in `rust-toolchain.toml`. Default to debug for local native builds, tests and Clippy.
- For frontend changes, run `npm run typecheck` and `npm run build`. For native changes, build the frontend first, then run Cargo format checks, tests and Clippy as described in [CI](docs/CI.md).
- Test observable behavior when adding features or fixing bugs. The starter has no frontend test suite; report typechecking, compilation and behavior tests separately.
- Report the checks actually run, their platform, failures and skipped coverage. A browser build does not validate Tauri IPC or a native window.

## CI and artifacts

- CI runs on pushes to `main` and `dev`, on pull requests and on manual dispatch.
- Preserve the check names `Web (required)`, `Windows x64 (required)` and `Required release checks`.
- Build production frontend assets on every branch. Use debug native builds outside `main`. Main builds release packages for all configured desktop targets and also warms the Windows x64 debug cache.
- Keep external Actions pinned to full commit hashes. Use read-only workflow permissions and disable persisted checkout credentials.
- Run `npm run test:tooling` when changing release scripts. See [CI](docs/CI.md) for build checks and [Releasing](docs/RELEASING.md) for publication.
- Tags publish the verified Web and desktop release artifacts from a successful main build at the exact tag commit. Do not rebuild packages during publication. Cloudflare Pages deployment is disabled.

## Branch strategy

- Never push directly to, force-push or delete `main` or `dev`. Update them through pull requests, including repository maintenance changes.
- Create feature branches from `dev` and target `dev` with their PRs. Merge `dev` into `main` through a release PR.
- Create urgent fixes from `main`; merge them into `main` through a PR, then synchronize `main` into `dev` through another PR.
- Use `codex/<topic>` for agent branches unless the user specifies a name. Branch routes do not grant permission to merge, tag or publish.
- Do not weaken branch protection or add bypass actors to get a change merged.

## Commits and documentation

- Use one concise Conventional Commit line: `type(scope): imperative description`. Include a scope; omit bodies and footers unless requested.
- Keep documentation accurate for the current project. Use English for repository instructions and technical documents by default. Do not create translations or changelogs until the project needs them.
- Keep the application version consistent in `package.json`, `package-lock.json`, `src-tauri/Cargo.toml`, `src-tauri/Cargo.lock` and `src-tauri/tauri.conf.json` when preparing a version change. There is no automatic version synchronization yet.
- Load relevant available skills for the task. Reference only skills that are available in the environment or repository.
