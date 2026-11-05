# Maintaining the Ditto Homebrew tap

## Purpose and ownership

This public repository distributes `dittosh`, the Ditto CLI, through the
`getditto/tap` Homebrew tap. Keep content suitable for customers. The CLI source,
version, npm publication, release notes, and formula generator belong in
`getditto/ditto-cli`. Agent-skill content belongs in `getditto/agent-skills`.

Read `README.md` and inspect the current CLI release workflow before changing
release behavior. Do not assume a release exists because a version is mentioned
in documentation. Confirm its published npm metadata and GitHub release assets.

## Repository layout

- `Formula/dittosh.rb`: generated stable formula, added by the first CLI release.
- `Formula/.gitkeep`: preserves the directory before the first release.
- `.github/workflows/validate.yml`: formula syntax, installation, functional test,
  and install-channel checks on macOS arm64 and Linux x64/arm64.
- `README.md`: customer installation/update instructions and maintainer setup.
- `AGENTS.md`: these maintenance instructions.

Before the first formula exists, CI explicitly reports bootstrap status and skips
installation jobs. A green bootstrap check is not evidence that the CLI installs.

## Working agreements

- Work on a branch and open a PR; do not commit maintenance changes directly to
  `main`. The established CLI release publisher is the exception for generated
  formula updates.
- Keep diffs focused. Do not change GitHub permissions, create credentials, or
  publish a CLI release as part of a documentation or formula-review task.
- Keep secrets out of this public repository, logs, issues, and PR descriptions.
  The tap's CI needs no secrets; the publishing token lives in `ditto-cli`.
- Use read-only workflow permissions and pin new actions to reviewed commit SHAs.
  Use `pull_request`, not privileged execution of PR code via `pull_request_target`.
- Never invent an artifact URL, version, checksum, supported architecture, or test
  result. Report missing upstream publication as a blocker for installation.

## Formula contract

The CLI publisher generates the formula in `scripts/release.ts` and verifies it
with `scripts/release-smoke.ts`. Fix recurring generation issues in the CLI
repository first; a tap-only edit will otherwise be overwritten next release.

Preserve these properties unless a reviewed upstream release changes them:

- The class is `Dittosh` and the executable is `dittosh`, never `ditto`.
- The source is an immutable, version-specific `@dittolive/cli` npm tarball with
  its verified SHA-256 checksum. Do not use `latest` URLs or branch archives.
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

1. Confirm the npm version, GitHub tag, and reviewed release notes agree. The
   notes live at `releases/vX.Y.Z.md` in `ditto-cli`.
2. Check the formula URL against npm's `dist.tarball`. Download the tarball to a
   temporary directory and independently calculate SHA-256; compare it to the
   formula and the GitHub release's `SHA256SUMS`. Never source or execute metadata.
3. Compare `Formula/dittosh.rb` to the release's generated `dittosh.rb` asset. If
   a difference is intentional, document why and fix the generator where needed.
4. Review the dependency and platform changes. Preserve the existing version's
   checksum when no new package bytes are being released.
5. Require the formula jobs to pass on all supported platforms. On the first
   release, remove the bootstrap notice in `README.md` after installation succeeds.

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

- **No formula:** confirm the upstream npm release exists and the CLI tap-update
  job ran. This repository does not publish packages or mint release versions.
- **Tap update denied:** ask maintainers to inspect `HOMEBREW_TAP_TOKEN`, its
  repository scope, organization authorization, and branch rules in `ditto-cli`.
- **Checksum mismatch:** stop installation work and compare the immutable release
  assets. Never change the checksum just to make an unexpected download pass.
- **Expired offline license:** coordinate a new CLI patch release with a renewed
  license. Do not embed a replacement token into this repository.
- **Failed install or wrong update channel:** reproduce on the failing platform,
  then fix the CLI's formula generator and add suitable upstream coverage.

Use the upstream publishing runbook for retrying a partial release. Do not move
tags, overwrite a published npm version, or force-push this tap to hide a failure.
Report what was verified, what remains blocked, and the relevant run/PR links.
