# Sandboxing Agentic AI — Lecture + Lab

Teaching materials for a lecture on **why sandboxing matters for agentic AI**,
and how to do it on Kubernetes with the
[`agent-sandbox`](https://github.com/kubernetes-sigs/agent-sandbox) `Sandbox`
CRD.

Agents run **model-generated, untrusted code**. The code inside the box may be
adversarial (prompt injection, tool misuse). The agent's blast radius is exactly
whatever its sandbox lets it touch — so the sandbox *is* the security boundary.

## Contents

| Path | What it is |
|---|---|
| [`slides/`](slides/) | The main (200-level) Marp deck, `slides.md`: containers vs VMs, the shared-kernel tradeoff, isolation runtimes (gVisor/Kata/Firecracker), warm pools, and the `Sandbox` CRD. Builds to HTML/PDF. |
| [`slides/slides-100.md`](slides/slides-100.md) | Introductory deck for an audience with no Linux or Kubernetes background. Each core idea (agents, sandboxes, kernels, Kubernetes, hardening, isolation runtimes, warm pools) has a plain-language slide with a diagram, then a "100 to 300" slide that goes deeper. It ends with the infographic, the lab commands, and a four-slide glossary. |
| [`infographic/`](infographic/) | The lecture and lab on one page, as SVG (source) and PNG. |
| [`lab/`](lab/) | Hands-on lab: a `kind` cluster where the **same** agent image runs as an unsecured **plain Pod** (defaults) vs a hardened **`Sandbox`** (also offered as a reusable `SandboxTemplate`). The unsecured one is breached; the secured ones contain every attack. |

## The arc

1. **What is a container vs a VM?** Containers share the host kernel (namespaces
   + cgroups); VMs virtualize hardware and bring their own kernel.
2. **Sharing the kernel is great** — millisecond starts, high density, tiny
   portable images, trivial to reproduce an environment.
3. **…and it is the risk** — a shared kernel is a shared attack surface. One
   misconfig (privileged, host mounts, mounted admin tokens, root, no
   NetworkPolicy) or one kernel bug = escape.
4. **Agentic AI makes it urgent** — untrusted code, high volume, multi-tenant.
5. **Isolation is a spectrum** — K8s hardening → gVisor → Kata/Firecracker
   microVMs, trading startup latency for a stronger boundary; **warm pools** hide
   that latency.
6. **The `Sandbox` CRD** makes the secure posture declarative and reusable
   instead of a pile of hand-wired primitives.
7. **Prove it in the lab.**

## Quick start

Slides:

```bash
cd slides
npm run preview      # live preview of both decks in the browser
npm run build:html   # -> slides/slides.html
npm run build:pdf    # -> slides/slides.pdf
npm run build:html:100   # introductory deck -> slides/slides-100.html
npm run build:pdf:100    # introductory deck -> slides/slides-100.pdf
```

Lab:

```bash
cd lab
./scripts/01-up.sh                 # kind + Calico + agent-sandbox controller
./scripts/02-attack-unsecured.sh   # watch the breach
./scripts/03-defend-secured.sh     # watch it get contained
./scripts/03b-defend-secured-template.sh  # same containment, via a reusable SandboxTemplate
./scripts/99-down.sh               # cleanup
```

Add **kernel-level isolation with gVisor** (runs on Apple silicon — no KVM needed):

```bash
cd lab
WITH_GVISOR=1 ./scripts/01-up.sh   # installs runsc into the podman VM + registers the RuntimeClass
./scripts/04-harden-gvisor.sh      # Sandbox under gVisor; proves uname -> *-gvisor, ~17 /dev nodes
```

For the VM-based tiers (Kata, Firecracker — need `/dev/kvm`), run
`./scripts/preflight-kvm.sh` and see `lab/manifests/kata/` and
`lab/manifests/firecracker/`.

See [`lab/README.md`](lab/README.md) for the full walkthrough and the
attack→defense mapping.

## Environment notes

- Built and verified with `kind` on the **podman** provider
  (`KIND_EXPERIMENTAL_PROVIDER=podman`), `kubectl`, and `agent-sandbox` v1.0.6.
- The lab uses the **Calico** CNI so `NetworkPolicy` is actually enforced
  (kind's default kindnet silently ignores it).
- On macOS/Windows the container runtime runs in a Linux VM, so the cluster is
  already one VM boundary from your laptop. That is *incidental* — production
  Linux nodes run pods directly on the shared host kernel. The lab demonstrates
  the **in-cluster** posture, which is what generalizes.

## License

[Apache License 2.0](LICENSE).
