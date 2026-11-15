# Web and desktop releases

`.github/workflows/release.yml` runs when a `v*` tag is pushed. It publishes a GitHub prerelease containing the Web ZIP, Windows x64/ARM64 portable ZIPs, Linux x64/ARM64 DEB and AppImage packages, and macOS x64/ARM64 DMGs. The release also includes each platform's manifest and `SHA256SUMS`.

Cloudflare Pages deployment is explicitly disabled with `if: ${{ false }}`. That job appears as skipped and does not require credentials. The Web ZIP is still published as a release asset. Website hosting is not enabled by this workflow.

## Prepare a release

1. Keep the version consistent in the npm and Cargo manifests and lockfiles and in `src-tauri/tauri.conf.json`.
2. Merge the changes through dev and then main using PRs.
3. Wait for every job in the main build to succeed. The build must include manifests produced by the release tooling.
4. Create and push `v<package.json version>` at that exact main commit when publication is authorized.
5. Confirm the release workflow succeeds and that all expected assets are present.

The workflow generates release notes from GitHub's release-notes API. Releases are marked as prereleases and are not marked as the latest stable release. Rerunning publication replaces assets with the same names and updates the notes for the same tag.

## Artifact verification

Publication downloads artifacts; it does not build the application again. The resolver selects the latest main build for the exact tag commit and requires all its jobs to succeed. A newer failed or incomplete run blocks fallback to an older run. Missing, expired, duplicate or debug-only artifacts block publication.

Each package manifest records the application name and version, commit, clean tracked-source status, profile, target, workflow run ID and attempt, and SHA-256 checksums. Downloaded manifests must match the selected run and release version. Verification rejects changed bytes, unexpected files, duplicate names and invalid paths. Mixed run attempts are rejected; rerun the entire main workflow if a partial retry produces mixed artifacts.

Build artifacts expire after 14 days. If they are unavailable, run Build and test manually on main at the intended commit before publishing. Earlier builds without manifests cannot be published with this workflow.

## Tooling checks

Run `npm run test:tooling` for artifact selection and integrity tests. Check the shell scripts with `bash -n`. For workflow validation, ignore only actionlint's constant-false diagnostic because the Pages job is intentionally disabled:

```sh
actionlint -ignore 'constant expression "false" in condition' .github/workflows/build.yml .github/workflows/release.yml
```

These checks do not publish a release or deploy a website.
