# Ditto Homebrew tap

Install and update **`dittosh`**, the [Ditto CLI](https://github.com/getditto/ditto-cli),
with Homebrew. The CLI provides offline DQL tools and installs Ditto agent skills
for supported AI coding tools.

This repository is a Homebrew **tap**: an additional source of packages. The tap
is named `getditto/tap`, its GitHub repository is `getditto/homebrew-tap`, and the
CLI formula is `dittosh`. The command is `dittosh`, to avoid colliding with the
macOS `ditto` utility.

The formula downloads a versioned CLI archive from
[GitHub Releases](https://github.com/getditto/ditto-cli/releases) and verifies its
SHA-256 checksum. Publishing the CLI to npm is not required. Homebrew uses npm
internally to install the archive and its public dependencies; users do not need
an npm account or publishing credentials.

## Supported systems

| System | Architecture |
| --- | --- |
| macOS | Apple Silicon / arm64 |
| Linux | x86-64 / x64 and arm64 |

Install [Homebrew](https://brew.sh/) first and follow its instructions for adding
`brew` to your shell's PATH. The formula manages its own Node.js 24 dependency;
you do not need a separate npm installation or an npm account.

Intel Macs are not supported by the CLI's current native Ditto SDK. This tap
does not support native Windows. The initial distribution is for Homebrew testing;
installation from the npm registry is deferred.

## Install

Add the tap, then install the CLI:

```sh
brew tap getditto/tap
brew install getditto/tap/dittosh
```

Or use a single command, which adds the tap automatically:

```sh
brew install getditto/tap/dittosh
```

Verify the installation:

```sh
dittosh --version
dittosh version
dittosh dql doctor
```

`dittosh version` should identify the installation as Homebrew. Use
`dittosh --help` for commands and the [CLI README](https://github.com/getditto/ditto-cli#readme)
for DQL and agent-skill examples. Installing this CLI does not automatically
install agent skills; choose the skills and AI tools through its `skills` commands.

## Update

```sh
brew update
brew upgrade getditto/tap/dittosh
```

`brew update` refreshes package definitions; `brew upgrade` installs a newer CLI
when available. The tap tracks stable releases. See the
[CLI releases](https://github.com/getditto/ditto-cli/releases) for release notes.

Check the installed and available versions with:

```sh
brew list --versions dittosh
brew info getditto/tap/dittosh
```

## Switching from npm

Use one package manager for the CLI to avoid competing `dittosh` commands on
PATH. If you previously installed it globally through npm:

```sh
npm uninstall -g @dittolive/cli
brew install getditto/tap/dittosh
```

Open a new terminal and run `dittosh version` to confirm Homebrew is selected.
This changes the CLI installation, not the agent skills it has already installed.

## Uninstall

```sh
brew uninstall dittosh
brew untap getditto/tap
```

Uninstalling the formula does not remove your CLI data or agent skills. Use the
CLI's documented commands to manage those separately.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| `brew: command not found` | Follow Homebrew's shell setup instructions and open a new terminal. |
| Formula unavailable | Run `brew update` and check that `Formula/dittosh.rb` is present on this repository's `main` branch. |
| Old version or wrong install channel | Run `command -v dittosh` and `dittosh version`; check for an npm installation earlier on PATH. |
| Unsupported platform | Check the architecture table above. The current native SDK does not support Intel Macs. |
| Offline-license expiry | Update the CLI. If the latest version also reports expiry, report it to the CLI maintainers; the license is embedded during release. |

For installation or formula problems, open an issue in
[this repository](https://github.com/getditto/homebrew-tap/issues). For CLI behavior,
open an issue in [ditto-cli](https://github.com/getditto/ditto-cli/issues).
Include your OS/architecture, `brew --version`, `brew info getditto/tap/dittosh`,
and the error. Remove credentials and private data before sharing output.

## Maintaining releases

CLI builds and release notes belong in **`getditto/ditto-cli`**. This tap contains
only the formula and its maintenance documentation. The initial `v0.1.0` Homebrew
release uses the tested `dittolive-cli-0.1.0.tgz` GitHub release asset, without
publishing `@dittolive/cli` to npm.

For this GitHub-only release path:

1. Run the CLI's Release workflow with `homebrew-test=true`. The bootstrap mode in
   [CLI PR #8](https://github.com/getditto/ditto-cli/pull/8) is pinned to the reviewed
   `v0.1.0` source commit. It builds using the CLI repository's dedicated
   offline-license and database-ID secrets, then runs coverage, end-to-end tests,
   and real Homebrew installation checks on all supported platforms.
2. After those checks pass, the workflow publishes the exact tested archive,
   `SHA256SUMS`, and a `dittosh.rb` that points at the GitHub release asset. It uses
   GitHub's automatically issued repository token; no npm package is published.
3. Copy that release's `dittosh.rb` into a PR here. Require the tap's macOS and
   Linux installation checks and normal review before merging. Users receive the
   new version through `brew update` and `brew upgrade` after it merges.

This path needs **no `NPM_TOKEN` or `HOMEBREW_TAP_TOKEN` secret**. The tap's CI is
read-only and uses public downloads. Normal GitHub review permissions still apply
when a maintainer merges a formula PR.

The bootstrap mode is deliberately limited to `v0.1.0`. It rejects a tag that
points at another commit and refuses to overwrite an already published release.
Future release automation should be agreed separately. The CLI's original tag-push
workflow still includes npm publishing and cross-repository tap updates; that
combined path is not used for this Homebrew test. Do not enable it or request
publishing tokens merely to update this tap.

Do not add placeholder versions, guessed checksums, or unpublished download URLs.
Never replace an existing release archive with different bytes. For a changed
build, publish a new CLI version and update the formula through another PR.

AI agents and human contributors should read [AGENTS.md](AGENTS.md) before making
maintenance changes.

References: [Homebrew taps](https://docs.brew.sh/How-to-Create-and-Maintain-a-Tap)
and [Node.js formula guidance](https://docs.brew.sh/Language-Specific-Formulae#nodejs).
