# Agent Instructions for agent-sandbox

## Overview

This repo holds teaching material for a lecture on sandboxing agentic AI on Kubernetes: two Marp slide decks (`slides/`), a one-page infographic (`infographic/`), and a hands-on lab (`lab/`). The lab runs the same agent image as a deliberately unsecured plain Pod and as a hardened `Sandbox` (from the [`kubernetes-sigs/agent-sandbox`](https://github.com/kubernetes-sigs/agent-sandbox) CRD, `agents.x-k8s.io/v1beta1`), runs attack probes against both, and shows the breach versus the containment.

There is no application code, no Python, and no CI. The deliverables are Bash scripts, Kubernetes manifests, Markdown docs, SVG pictures, and the decks with their built HTML/PDF. The rules below fit that.

## Core Principles

- Keep it simple. The audience is students, so someone new to Kubernetes should be able to read every script and manifest top to bottom.
- The lab is a teaching argument: "security is opt-in." Every change has to keep the unsecured path visibly breached and the secured paths visibly contained.
- Prefer one clear command over a clever pipeline. If a probe needs explaining, the explanation belongs in a comment next to it.

## Repository Layout

```
README.md                    # the lecture arc + quick start
slides/
  slides.md                  # 200-level Marp deck (speaker notes live in <!-- --> comments)
  slides.html, slides.pdf    # built artifacts, committed
  slides-100.md              # introductory deck: plain-language slides + "100 to 300" slides
  slides-100.html, .pdf      # built artifacts, committed
  themes/sandbox.css         # theme for slides.md (gaia with smaller font + padding)
  themes/sandbox-light.css   # theme for slides-100.md (callout boxes, three-column levels)
  diagrams/*.mmd             # Mermaid sources for slides.md
  assets/*.svg               # rendered Mermaid diagrams; assets/intro-*.svg are hand-written for slides-100.md
  package.json               # npm scripts wrapping marp-cli / mermaid-cli via npx
infographic/
  sandboxing-agentic-ai.svg  # one-page summary, hand-written SVG (source)
  sandboxing-agentic-ai.png  # 2x export of the SVG
lab/
  kind-config.yaml           # default cluster: CNI disabled so Calico can be installed
  manifests/
    00-setup.yaml            # namespaces + the fake "loot" secret in corp-secrets
    unsecured/               # plain Pod + cluster-admin RBAC (sandbox/ = same posture via the CRD)
    secured/                 # hardened Sandbox + zero-RBAC SA + deny-all NetworkPolicy
    secured/template/        # same posture as SandboxTemplate + SandboxWarmPool + SandboxClaim
    gvisor/                  # runsc RuntimeClass, gVisor-aware kind config, gVisor Sandbox
    kata/, firecracker/      # microVM tiers, need /dev/kvm (docs + validate-kind.sh)
  scripts/
    lib.sh                   # shared constants, version pins, output helpers
    01-up.sh ... 06-vm-proof.sh, demo.sh, 99-down.sh
    install-runsc.sh, preflight-kvm.sh
```

## Running the Lab

The lab was built on `kind` with the podman provider. `lib.sh` exports `KIND_EXPERIMENTAL_PROVIDER=podman` unless it is already set; export `KIND_EXPERIMENTAL_PROVIDER=docker` to use Docker instead.

```bash
cd lab
./scripts/01-up.sh                        # kind + Calico + agent-sandbox controller (~3-4 min)
./scripts/02-attack-unsecured.sh          # every probe should print BREACH
./scripts/03-defend-secured.sh            # every probe should print CONTAINED
./scripts/03b-defend-secured-template.sh  # same containment via SandboxTemplate
./scripts/99-down.sh                      # delete the cluster

# Optional gVisor tier (no KVM needed)
WITH_GVISOR=1 ./scripts/01-up.sh
./scripts/04-harden-gvisor.sh
./scripts/05-compare-gvisor.sh
./scripts/06-vm-proof.sh                  # podman machine only (macOS)
```

`01-up.sh` deletes and recreates the `agent-sandbox-demo` cluster. Never point these scripts at a shared or real cluster: `unsecured/` binds a ServiceAccount to `cluster-admin` and mounts the node's `/` on purpose.

`install-runsc.sh`, `06-vm-proof.sh` and the macOS branch of `preflight-kvm.sh` assume a podman machine VM. On a Linux host, install gVisor on the host instead (see the comment at the top of `install-runsc.sh`). `lab/manifests/kata/validate-kind.sh` is meant for a KVM-capable Linux box (for example an EC2 `m7i`) and uses `sudo`.

## Building the Slides

```bash
cd slides
npm run preview          # live preview at http://localhost:8080
npm run build:html       # -> slides.html
npm run build:pdf        # -> slides.pdf
npm run build:diagrams   # diagrams/*.mmd -> assets/*.svg (downloads headless Chromium on first run)
npm run build:html:100   # -> slides-100.html
npm run build:pdf:100    # -> slides-100.pdf
```

PDF builds (and `--images png`) need Chrome or Chromium. If none is installed, `npx @puppeteer/browsers install chrome-headless-shell@stable` downloads one, and `CHROME_PATH` points Marp at it. Ubuntu 23.10 and later block Chrome's own sandbox through AppArmor (`No usable sandbox!`). For local renders of this repo's files, point `CHROME_PATH` at a small wrapper script that runs the binary with `--no-sandbox`.

Marp does not render fenced `mermaid` blocks, so diagrams are always pre-rendered to SVG and embedded as images. Edit the `.mmd`, regenerate, and commit both the source and the SVG.

The built HTML and PDF are committed. Rebuild a deck's HTML and PDF in the same change as any edit to its Markdown, its theme, or an asset it uses, so the artifacts never drift from the source.

In `slides-100.md`, every core idea gets a plain-language slide with a picture, followed by an "X: 100 to 300" slide. Keep the 100 column free of jargon, put the real setting names in the 200 column, and keep the 300 column to production details you can check against the lab or upstream docs. Callouts use `<div class="idea">` (a question or key idea), `<div class="def">` (a definition), and `<div class="hot">` (a warning). Leave a blank line after each opening tag so Marp renders the Markdown inside. When a 200 or 300 column uses a term the audience may not know (as with cgroup and CNI), define it in a `def` box on that slide and add it to the glossary slides at the end of the deck.

To check a deck for overflow, render every slide to PNG and look at each one:

```bash
npx -y @marp-team/marp-cli@^4 slides-100.md --theme-set themes/sandbox-light.css \
  --html --allow-local-files --images png -o /tmp/deck/s.png
```

Dense slides overflow the theme easily. Split the slide, or add a one-line `<style scoped>section{font-size:...}</style>` as the existing dense slides do. Check the result in the preview before committing.

`npm run preview` starts a long-running server. Do not start it unless asked, and stop it when done.

## Infographic

`infographic/sandboxing-agentic-ai.svg` is the source and `sandboxing-agentic-ai.png` is a 2x export. Re-export the PNG in the same change as any SVG edit (the command is in `infographic/README.md`). The palette follows the explainer style recorded in that README: teal and yellow on cream, with dark red reserved for BREACH. Keep the facts on it in step with the lab: if a probe, a control, or a timing changes, update the infographic too.

## Bash Script Conventions

Match the existing scripts:

- Shebang `#!/usr/bin/env bash`, then a header comment saying what the script does and why, then `set -euo pipefail`, then `source "$(dirname "${BASH_SOURCE[0]}")/lib.sh"`.
- Use the `lib.sh` helpers rather than raw `echo`: `step`, `info`, `good`, `bad`, `warn`, `run` (echoes the command, then runs it), `sbexec` (kubectl exec into the demo namespace), `probe`, `breached`, `contained`, `require`.
- Check required tools up front with `require <tool>`.
- Name environment-variable knobs in caps with a default, for example `"${WITH_GVISOR:-0}"`, and document them in the header comment.
- Put tunable values (names, timeouts that repeat, versions) in constants at the top of the script or in `lib.sh`, not inline.
- Quote every variable expansion. Use `[[ ... ]]` for tests.
- Use `kubectl wait` / `rollout status` with an explicit `--timeout`, never a bare `sleep` to wait for readiness.
- Every attack probe follows the same shape: `probe "description"`, attempt the action, then `breached` or `contained` with a sentence that names the control responsible. In the unsecured script the `else` branch says "unexpected for the unsecured box", and the secured scripts do the mirror image.
- Keep scripts short and linear. If a script grows past roughly 100 lines, move repeated logic into a function in `lib.sh`.
- Remote heredocs that must not expand locally use single quotes and a `# shellcheck disable=SC2016` comment, as in `install-runsc.sh`.

The `⚠ BREACH`, `🛡 CONTAINED` and similar markers in `lib.sh` are part of the documented output in the READMEs. Leave them as they are, but do not add new emojis to scripts, comments, or docs.

## Manifest Conventions

- Every manifest opens with a comment block explaining the posture and listing each control next to the attack it closes (see `secured/sandbox.yaml`). Keep that list in sync when you add or remove a field.
- The unsecured and secured variants share the same image, command, and namespace, so the only difference is the isolation posture. Preserve that parity in any new variant.
- Pin image tags. Never use `:latest` in a manifest.
- Resource names, labels and NetworkPolicy selectors are coupled across files and scripts (for example `app: secured-agent` in `secured/sandbox.yaml` and `secured/networkpolicy.yaml`, and the `POD=` names in the scripts). Rename them everywhere at once or not at all.
- The secret in `00-setup.yaml` is fake demo loot. Keep any new example credentials unmistakably fake.

## Version Pins: Change Them Everywhere

Versions are pinned in `lab/scripts/lib.sh` but also repeated in other files. When you bump one, grep for it and update every copy:

| Pin | Defined in | Also appears in |
|---|---|---|
| `nicolaka/netshoot:v0.13` | `lib.sh` (`NETSHOOT_IMAGE`) | every Sandbox/Pod manifest, `kata/README.md`, `firecracker/README.md` |
| agent-sandbox `v1.0.6` | `lib.sh` (`AGENT_SANDBOX_VERSION`) | `kata/validate-kind.sh` default, root `README.md` environment notes |
| Calico `v3.28.2` | `lib.sh` (`CALICO_VERSION`) | (only there) |
| cluster name `agent-sandbox-demo` | `lib.sh` (`CLUSTER_NAME`) | `lab/kind-config.yaml`, `lab/manifests/gvisor/kind-config.yaml` |

```bash
grep -rn 'netshoot:v\|v1\.0\.6\|agent-sandbox-demo' lab README.md
```

Bumping the agent-sandbox controller can change CRD fields or controller defaults (for example what `SandboxTemplate` injects). Re-run the full lab after a bump, not just a syntax check.

## What to Run After Changing What

| You changed | Run before opening a PR |
|---|---|
| Any `lab/scripts/*.sh` | `bash -n <file>`, plus `shellcheck <file>` if installed, then the script itself against a fresh `01-up.sh` cluster |
| A manifest | `kubectl apply --dry-run=client -f <file>` (needs the CRDs for `Sandbox*` kinds, so use a live lab cluster), then the script that applies it |
| Secured or unsecured posture | `./scripts/demo.sh` end to end; confirm every probe still prints the expected BREACH / CONTAINED |
| gVisor files | `WITH_GVISOR=1 ./scripts/01-up.sh`, then `04-harden-gvisor.sh` and `05-compare-gvisor.sh` |
| Kata / Firecracker files | `kata/validate-kind.sh` on a KVM host, or say in the PR that it was not run and why |
| `slides.md`, theme, assets | `npm run build:html && npm run build:pdf`, eyeball for overflow, commit the artifacts |
| A `.mmd` diagram | `npm run build:diagrams`, then rebuild HTML/PDF |
| `slides-100.md`, `sandbox-light.css`, `assets/intro-*.svg` | `npm run build:html:100 && npm run build:pdf:100`, render to PNG and check every slide, commit the artifacts |
| The infographic SVG | re-export the PNG; rebuild `slides-100` (its summary slide embeds the SVG) |
| Observed output quoted in a README | Re-capture it from a real run; do not hand-edit "verified output" blocks |

If you could not run a check (no KVM, no podman machine), say so in the PR description rather than implying it passed.

## Documentation Guidelines

- READMEs quote real, observed output. Keep those blocks honest: re-run and paste, do not paraphrase.
- When you add a script or manifest, update the file tree in `lab/README.md` and, if it changes the arc, the root `README.md`.
- `slides/README.md` refers to `.vscode/settings.json` for the Marp VS Code theme, but that file is not committed. Either add it or fix the reference if you touch that area.
- Existing Markdown is hard-wrapped at about 80 columns. For new paragraphs, write one paragraph per line. Do not reflow existing paragraphs you are not otherwise changing, because it buries the real diff.
- Do not add emojis to documentation.
- Do not use em-dashes or double hyphens as dashes in anything you write. Use a comma, colon, parentheses, or two sentences.

### Prose

Slides, speaker notes, READMEs, and PR text should read like a person wrote them. Use short words and active voice, cut padding such as `in order to`, and give exact numbers, commands, and file paths. Avoid the patterns that mark machine-written text: "not X, but Y" contrasts, sentences that only tee up the next one, closing lines that restate the point, three-item lists where two facts will do, and bold on every bullet. If your tooling has a writing or prose-lint pass, run it on every file you change.

## Scratchpad

Put plans, analyses and drafts in `.scratchpad/` at the repo root (`plan-<feature>.md`, `analysis-<topic>.md`, `draft-github-issue.md`). This repo has no `.gitignore`, so `.scratchpad/` is ignored through `.git/info/exclude` (local only). Do not add a `.gitignore` to an upstream PR unless that is the point of the PR.

## Contributing Upstream

This checkout is a fork:

- `origin`: `aarora79/agent-sandbox` (the fork)
- upstream: `omrishiv/agent-sandbox` (the source repo PRs go to)

Workflow:

```bash
# one-time
git remote add upstream https://github.com/omrishiv/agent-sandbox.git

# per change
git fetch upstream
git switch -c <topic-branch> upstream/main
# ... make changes, run the checks above ...
git commit -s -m "Short imperative summary"
git push -u origin <topic-branch>
gh pr create -R omrishiv/agent-sandbox --base main --head aarora79:<topic-branch>
```

- One topic per branch and per PR. Do not open PRs from the fork's `main`; keep `main` as a clean mirror of `upstream/main` (`git fetch upstream && git switch main && git merge --ff-only upstream/main && git push origin main`).
- Upstream commits carry a `Signed-off-by:` trailer, so sign off with `git commit -s`.
- Stage files by name. Do not `git add -A`; it picks up `.scratchpad/`-style leftovers and local `node_modules/`.
- Keep commit messages and PR descriptions professional and technical. No "Generated with Claude Code" lines and no `Co-Authored-By: Claude` trailers.
- The PR description should say what changed, why, and which of the checks above were run (and on what: podman on macOS, Linux kind, KVM host).
