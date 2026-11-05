# Ditto Homebrew tap

Install and update **`dittosh`**, the [Ditto CLI](https://github.com/getditto/ditto-cli),
with Homebrew. The CLI provides offline DQL tools and installs Ditto agent skills
for supported AI coding tools.

This repository is a Homebrew **tap**: an additional source of packages. The tap
is named `getditto/tap`, its GitHub repository is `getditto/homebrew-tap`, and the
CLI formula is `dittosh`. The command is `dittosh`, to avoid colliding with the
macOS `ditto` utility.

> **First-release availability:** adding the tap works once this setup is merged.
> Installing `dittosh` also requires the first stable CLI release to publish
> `@dittolive/cli` to npm and add `Formula/dittosh.rb` to this repository's `main`
> branch. Until then, Homebrew will report that the formula is unavailable.
> See [maintainer setup](#maintainer-setup) for the remaining steps.

## Supported systems

| System | Architecture |
| --- | --- |
| macOS | Apple Silicon / arm64 |
| Linux | x86-64 / x64 and arm64 |

Install [Homebrew](https://brew.sh/) first and follow its instructions for adding
`brew` to your shell's PATH. The formula manages its own Node.js 24 dependency;
you do not need a separate npm installation or an npm account.

Intel Macs are not supported by the CLI's current native Ditto SDK. Native
Windows installations use npm; see the [CLI installation instructions](https://github.com/getditto/ditto-cli#readme).

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
| Formula unavailable | Run `brew update` and check whether `Formula/dittosh.rb` exists on this repository's `main` branch. Before the first release, adding the tap alone does not make the CLI installable. |
| Old version or wrong install channel | Run `command -v dittosh` and `dittosh version`; check for an npm installation earlier on PATH. |
| Unsupported platform | Check the architecture table above. The current native SDK does not support Intel Macs. |
| Offline-license expiry | Update the CLI. If the latest version also reports expiry, report it to the CLI maintainers; the license is embedded during release. |

For installation or formula problems, open an issue in
[this repository](https://github.com/getditto/homebrew-tap/issues). For CLI behavior,
open an issue in [ditto-cli](https://github.com/getditto/ditto-cli/issues).
Include your OS/architecture, `brew --version`, `brew info getditto/tap/dittosh`,
and the error. Remove credentials and private data before sharing output.

## Maintainer setup

The release publisher is maintained in **`getditto/ditto-cli`**, not in this tap.
The proposed [release workflow](https://github.com/getditto/ditto-cli/pull/7)
tests and publishes the npm package and GitHub release, then writes a generated
`Formula/dittosh.rb` here using the same npm tarball and SHA-256 checksum.

Before the first release:

1. Merge this PR to establish the tap's documentation and validation workflow.
2. Merge the CLI release workflow and its required CLI changes. Follow its
   [publishing runbook](https://github.com/getditto/ditto-cli/blob/3345ec0e58e473bfd1822733aa861cfa215f7eea/docs/publishing.md)
   for npm publishing and the dedicated offline-license secrets.
3. Give the CLI publisher a fine-grained GitHub token with **Contents: read and
   write** for **this repository only**. Store it as the `HOMEBREW_TAP_TOKEN`
   repository secret in **`getditto/ditto-cli`**. Configure any required organization
   authorization and branch-rule permissions. No npm token belongs in this tap.
4. Use the CLI publisher's default `getditto/homebrew-tap`, or set its
   `HOMEBREW_TAP_REPOSITORY` variable to that value explicitly. Its current
   publisher commits directly to the tap's default branch; rules must allow the
   publishing identity to do this, or the publisher must be changed to open PRs.
5. Release the CLI from a reviewed version tag. Confirm the generated formula is
   present on `main`, this tap's formula checks pass, and installation works on
   a supported system. Then remove the first-release notice above.

Do not add a placeholder formula, guessed checksum, or an unpublished download
URL to make the tap appear ready. The first real formula comes from the release
publisher after npm publication. The tap does not build or publish the CLI itself.

For subsequent releases, the CLI publisher updates the stable formula; users
receive it through `brew update` and `brew upgrade`. Prereleases do not replace
the stable formula. AI agents and human contributors should read
[AGENTS.md](AGENTS.md) before making maintenance changes.

References: [Homebrew taps](https://docs.brew.sh/How-to-Create-and-Maintain-a-Tap)
and [Node.js formula guidance](https://docs.brew.sh/Language-Specific-Formulae#nodejs).
