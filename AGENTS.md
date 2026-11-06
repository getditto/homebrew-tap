# Maintaining the Ditto Homebrew tap

## Purpose and ownership

This public repository distributes `dittosh`, the Ditto CLI, through the
`getditto/tap` Homebrew tap. Keep content suitable for customers. The CLI source,
version, release archives, release notes, and formula generator belong in
`getditto/ditto-cli`. Agent-skill content belongs in `getditto/agent-skills`.

Read `README.md` and inspect the current CLI release workflow before changing
release behavior. Do not assume a release exists because a version is mentioned
in documentation. Confirm the actual GitHub release assets and their checksums. npm publication is
not required for the current Homebrew test release.

## Repository layout

- `Formula/dittosh.rb`: formula for the tested CLI archive hosted on GitHub Releases.
- `.github/workflows/validate.yml`: formula syntax, installation, functional test,
  and install-channel checks on macOS arm64 and Linux x64/arm64.
- `README.md`: customer installation/update instructions and maintainer setup.
- `AGENTS.md`: these maintenance instructions.

Require actual installation jobs for formula changes. A skipped installation
job is not evidence that the CLI installs.

## Working agreements

- Work on a branch and open a PR; do not commit maintenance changes directly to
  `main`. Formula updates follow the same review process.
- Keep diffs focused. Do not change GitHub permissions, create credentials, or
  publish a CLI release as part of a documentation or formula-review task.
- Keep secrets out of this public repository, logs, issues, and PR descriptions.
  The tap's CI needs no secrets. Do not request a personal access token or npm
  publishing token for the GitHub-only release path.
- Use read-only workflow permissions and pin new actions to reviewed commit SHAs.
  Use `pull_request`, not privileged execution of PR code via `pull_request_target`.
- Never invent an artifact URL, version, checksum, supported architecture, or test
  result. Report missing upstream publication as a blocker for installation.

## Formula contract

The CLI generates the formula in `scripts/release.ts` and verifies it with
`scripts/release-smoke.ts`. For the initial Homebrew test, its npm-registry URL
is explicitly replaced with the matching GitHub release asset URL, preserving
the tested tarball and checksum. No cross-repository publisher is enabled for
this path. Fix recurring generation issues upstream when changing automation.

Preserve these properties unless a reviewed upstream release changes them:

- The class is `Dittosh` and the executable is `dittosh`, never `ditto`.
- The source is the version-specific GitHub release archive at
  `getditto/ditto-cli/releases/download/vX.Y.Z/dittolive-cli-X.Y.Z.tgz`, with its
  verified SHA-256 checksum. Do not use `latest` URLs or branch archives.
- The formula uses the same package bytes tested and published by the CLI release.
  Do not rebuild a release or replace its existing npm version or Git tag.
- Install with Homebrew's `std_npm_args` under `libexec`; do not install into the
  user's global npm prefix or run `sudo npm install`.
- Declare `node@24` and preserve the launcher PATH that includes Homebrew and
  that Node runtime. `dittosh version` must detect Homebrew, so updates use brew.
- macOS support is arm64 only with the current SDK. Linux supports x64/arm64.
  Do not claim Intel Mac or native Windows Homebrew support without upstream
  native SDK support and platform validation.
- The functional test checks the installed version and offline DQL INSERT/SELECT
  with a temporary store. Never enable synchronization or introduce credentials
  into the formula. The release embeds the dedicated CLI offline license.
- The tap tracks stable releases only. Never downgrade the stable formula to an
  older version or promote a prerelease into it.

## Reviewing a release update

1. Confirm the packaged version, GitHub tag, and reviewed release notes agree. The
   notes live at `releases/vX.Y.Z.md` in `ditto-cli`.
2. Check the formula URL against the GitHub release's archive asset. Download it
   to a temporary directory and independently calculate SHA-256; compare it to the
   formula and the GitHub release's `SHA256SUMS`. Never source or execute metadata.
3. Compare `Formula/dittosh.rb` to the release's generated `dittosh.rb` asset. If
   a difference is intentional, document why and fix the generator where needed.
4. Review the dependency and platform changes. Preserve the existing version's
   checksum when no new package bytes are being released.
5. Require the formula jobs to pass on all supported platforms before merging.

## Validation

For an existing installed formula:

```sh
ruby -c Formula/dittosh.rb
brew update
brew install getditto/tap/dittosh
brew test getditto/tap/dittosh
dittosh version --format json
```

The JSON `channel` must start with `homebrew `. `brew test` must exercise offline
DQL successfully. These commands install the published main-branch formula;
they do not by themselves validate a local PR's formula. For PR changes, use
the checked-out tap in CI (see `validate.yml`), or an isolated test environment
with `brew tap getditto/tap /absolute/path/to/checkout`. Do not replace a user's
existing tap or CLI installation merely to run a test.

Run `actionlint .github/workflows/validate.yml` after workflow changes when
available, and `git diff --check` for every change. Do not run an installation
test before there is a real published artifact; state that limitation explicitly.

## Release failures and recovery

- **No formula:** confirm the GitHub release exists and its formula PR has merged.
  This repository does not publish packages or mint release versions.
- **Tap update denied:** check the contributor's normal GitHub permissions and
  required PR reviews. Do not bypass branch rules or create publishing tokens.
- **Checksum mismatch:** stop installation work and compare the immutable release
  assets. Never change the checksum just to make an unexpected download pass.
- **Expired offline license:** coordinate a new CLI patch release with a renewed
  license. Do not embed a replacement token into this repository.
- **Failed install or wrong update channel:** reproduce on the failing platform,
  then fix the CLI's formula generator and add suitable upstream coverage.

Use README.md for the current GitHub-only publication path. Do not move
tags, overwrite a published npm version, or force-push this tap to hide a failure.
Report what was verified, what remains blocked, and the relevant run/PR links.
