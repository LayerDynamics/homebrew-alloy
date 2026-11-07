# homebrew-alloy

The primary installation for [Alloy](https://github.com/LayerDynamics/alloy):
a local agent harness with unified CPU/Metal inference on Apple Silicon.

## Install

Requires **Apple Silicon, macOS 14 or newer, and Xcode 15 or newer** with
command-line tools and the Metal compiler. This first formula builds from an
immutable source commit. The native build can take several minutes. Homebrew
shows build stages; add `--verbose` to the install command for detailed
compiler output.

**Availability:** the working formula is prepared in this change. The command
below becomes available after [Alloy PR #26](https://github.com/LayerDynamics/alloy/pull/26)
and this tap formula merge. This is a development preview, without a claimed
prebuilt bottle or tagged binary release.

```bash
brew install LayerDynamics/alloy/alloy
alloy --version
alloyd --check-native
alloy start
```

Use the full `LayerDynamics/alloy/alloy` name to select this tap. The formula
installs the `alloy` CLI, matching `alloyd` daemon, Metal kernels and native
monitor together. Its private **Node 26.13.0** runtime is managed with the
package; your shell's Node version does not need to change. **Rust 1.99.1**
is used only during the source build.

Installation does not download model weights, change your Alloy settings,
or start a daemon or background service. The first `alloy start` asks for a
local starter-model artifact or an explicit download choice. To provision
the pinned starter separately, run `alloy setup --download`.

Wait for **`READY — Alloy API 1`** and keep that terminal open. In another
terminal:

```bash
alloy monitor
# Or open a chat client:
alloy chat
```

The monitor includes system-wide CPU, unified memory and supported GPU/ANE
measurements alongside Alloy's live runtime metrics. It uses an unprivileged
native helper: no `sudo`, authorization prompt or privileged monitoring
service. Unsupported counters are shown as unavailable; ANE measurements do
not imply Alloy inference runs on ANE.

`alloy start` runs in the foreground; Ctrl+C stops its owned daemon. Chat uses
a text prompt, while `alloy monitor` opens the dashboard. A compatible daemon
already listening on the selected socket is reused.

## Check the installation

```bash
brew test LayerDynamics/alloy/alloy
alloy doctor
```

The formula test checks installed CPU **and** Metal execution, shared arrays,
CLI/daemon pairing and the unprivileged monitor without downloading a model.
`doctor` reports configuration and API readiness; before setup, a
`setup_required` result is expected. Neither check proves real model
generation. After starting the daemon, test a real response with:

```bash
alloy -m "Say hello in one sentence."
```

## Upgrade

```bash
brew update
brew upgrade LayerDynamics/alloy/alloy
```

Upgrading does not restart a running daemon. Stop the old foreground
`alloy start` with Ctrl+C, then run `alloy start` again to use the new build.
Otherwise, a compatible old listener can still be reused.

Settings, models, SQLite sessions and persisted KV cache stay outside the
Cellar, normally under `~/.alloy/`. `ALLOY_CONFIG_DIR` selects another state
directory; use the same value in every Alloy terminal. Existing explicitly
configured paths remain authoritative. Upgrade and uninstall preserve this
external user data.

## Validation and formula updates

This tap's hosted ARM64 macOS workflow checks the canonical formula's Ruby
syntax, Homebrew loading, HTTPS GitHub source download, actual installation,
shared-command linking and CLI/daemon selection. The hosted VM does **not** establish Metal execution,
so that job does not run the native formula test or claim inference acceptance.

The [Alloy source workflow](https://github.com/LayerDynamics/alloy/blob/main/.github/workflows/ci.yaml)
runs the required physical Apple Silicon acceptance: an actual brew install
and formula test in an owned, uniquely named keg-only installation, followed
by CPU/Metal/shared-array and real-model checks with the original build tree
unavailable. It retains the formula, source identity and complete evidence
without replacing an existing `alloy` installation.

For that candidate CI run, the test formula uses an owned local bare Git
snapshot of the exact checkout, including detached/shallow synthetic PR merge
commits that may be unavailable from GitHub's normal source download. Commit
and tree identities are preserved and the snapshot has its own complete
objects. The public formula continues to use the pinned HTTPS GitHub source;
the hosted installation checks that download path separately.

`Formula/alloy.rb` is generated from the reviewed Alloy source commit using
[`scripts/homebrew/formula.mjs`](https://github.com/LayerDynamics/alloy/blob/main/scripts/homebrew/formula.mjs).
Its source revision is immutable, and official Node/Rust resources have
pinned checksums. Advance the formula version or revision when changing an
untagged source pin so `brew upgrade` selects the new build. npm/Cargo
publication and release-binary automation remain separate work.

See the [startup contract](https://github.com/LayerDynamics/alloy/blob/main/docs/designs/startup.md)
and [packaging contract](https://github.com/LayerDynamics/alloy/blob/main/docs/designs/homebrew.md)
for supported configuration, installed layout and validation boundaries.
