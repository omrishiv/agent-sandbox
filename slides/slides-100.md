---
marp: true
theme: sandbox-light
paginate: true
title: "Sandboxing Agentic AI: An Introduction"
author: agent-sandbox lab
---

<!-- _class: lead -->
<!-- _paginate: false -->

# Sandboxing Agentic AI

### An introduction, with 200 and 300 level detail where it helps

No Linux or Kubernetes background needed.

<!--
This deck assumes no background in Linux, containers, or Kubernetes. Each core idea gets a plain-language slide first, then a "100 to 300" slide that shows how far the idea goes. Stop at the 100 column if that is all the room needs. The 200-level deck (slides.md) and the lab cover the same ground with the real Kubernetes settings.
-->

---

## Agenda

1. What an AI agent is, and why the code it runs is risky
2. What a sandbox is, and how to measure one
3. Containers and virtual machines
4. The four settings that open a box, and how to close them
5. What the lab shows
6. Stronger walls, and how warm pools pay for them
7. Five questions to ask about any agent platform

<!--
If you remember one thing: the sandbox is the security boundary. Whatever the sandbox lets the agent touch, the agent can touch.
-->

---

## What Is an AI Agent?

![w:900](assets/intro-agent-loop.svg)

<div class="def">

An agent is a language model in a loop that can run code and use tools.

</div>

<!--
A chatbot gives you text, and you decide what to do with it. An agent skips that step. It picks an action, a program runs that action, and the model reads the result and picks the next one. No person approves each step, so the box the code runs in decides what that code can reach.
-->

---

## Agents: 100 to 300

<div class="levels">
<div>

**100 level**

An agent is a model that takes actions. It writes code or picks a tool, a program runs it, and the model reads the result and goes again until it reaches the goal.

</div>
<div>

**200 level**

The harness runs each tool call on real infrastructure: a shell, a filesystem, a network. Any file, credential, or address that process can reach, the model's output can reach too.

</div>
<div>

**300 level**

Platforms start thousands of short-lived sandboxes a day, often for many customers on the same machines. Isolation strength, start-up time, and cost pull against each other.

</div>
</div>

<!--
The 100 column is enough for a general audience. The 200 column explains why the sandbox and the agent's permissions are the same thing. The 300 column sets up the trade-off the later sections resolve.
-->

---

## Prompt Injection

![w:880](assets/intro-prompt-injection.svg)

<div class="idea">

If the model cannot reliably tell your instructions from an attacker's, where do you put the defense?

</div>

<!--
Ask the room before moving on. The answer the deck builds toward: you cannot fully fix this inside the model today, so you contain it outside the model. A sandbox with no secrets inside and no network path out gives a successful injection nothing to take.
-->

---

## What Is a Sandbox?

![w:820](assets/intro-blast-radius.svg)

<div class="def">

The blast radius is everything the agent could damage if it went rogue. A good sandbox keeps it to the box itself.

</div>

<!--
The children's sandbox is the right picture: the sand stays in the box. We plan for the agent misbehaving and make sure the damage stops at the edge of the box.
-->

---

## Sandboxes: 100 to 300

<div class="levels">
<div>

**100 level**

A sandbox is a fenced-off place to run code. The code can do its job inside and cannot reach anything outside. When the work is done, you throw the box away.

</div>
<div>

**200 level**

In practice a sandbox is a container or virtual machine plus its settings: which credentials it holds, which files it sees, which user it runs as, and where it can connect.

</div>
<div>

**300 level**

Design for hostile code. Assume a kernel bug exists, then ask what still stands: the runtime boundary (gVisor or a microVM), the network policy, and the identity the box carries.

</div>
</div>

<!--
The 300 column is the threat model for the whole talk: assume the worst about the code, then count the walls left standing.
-->

---

## Containers and Virtual Machines

![w:900](assets/intro-apartments-houses.svg)

<!--
A container is like an apartment. It has its own walls and front door, but every flat shares the foundation, plumbing, and wiring. In a computer that shared foundation is the kernel, the core of the operating system. A virtual machine is a separate house with its own foundation, and the hardware itself enforces the walls between houses.
-->

---

## Kernels: 100 to 300

<div class="levels">
<div>

**100 level**

The kernel is the core of the operating system. Every program asks it to open files and send network traffic. Containers share one kernel; each virtual machine has its own.

</div>
<div>

**200 level**

A container is an ordinary process. The kernel fences it with namespaces, which limit what it can see, and cgroups, which limit what it can use. Every system call still lands in the host kernel.

</div>
<div>

**300 level**

The host kernel offers hundreds of system calls. Seccomp trims that list, but a bug in what remains, such as `io_uring`, can let code escape the container. gVisor and microVMs take the host kernel out of reach.

</div>
</div>

<div class="def">

A **cgroup** (control group) is a Linux kernel feature that caps how much CPU, memory, and disk I/O a group of programs can use, and how many processes it can start. Kubernetes turns a pod's resource limits into cgroup settings on the node.

</div>

<!--
A cgroup answers "how much can this box use?", and a namespace answers "what can this box see?". In the lab, the secured Sandbox's limits of 250m CPU and 128Mi memory become cgroup settings, which stop one agent from starving the rest of the node.

The lab proves the 300 column: on the default runtime, a hardened pod can still see the host's 242 kernel modules and reach io_uring. Under gVisor it sees no host modules and io_uring returns ENOSYS.
-->

---

## The Trade-off

| | Container | Virtual machine |
| --- | --- | --- |
| Start time | milliseconds | seconds |
| Size | megabytes | gigabytes |
| How many per machine | hundreds | tens |
| Walls | shared kernel | own kernel, hardware-enforced |

<div class="hot">

Agent platforms need the speed and density of containers for thousands of boxes, and the walls of a virtual machine for the code inside them.

</div>

<!--
The rest of the deck gets most of both: harden the container first, add a stronger runtime for untrusted code, and use warm pools to hide the start-up cost.
-->

---

## Kubernetes Runs the Boxes

**Kubernetes** runs containers across many machines.

- It decides where each container runs, and restarts or adds containers as load changes.
- It hands out credentials and permissions: who can read which secrets.
- It applies the settings that decide how strong each box is.

<div class="def">

Most agent platforms run on Kubernetes, so Kubernetes settings decide how strong the sandbox is.

</div>

<!--
The room only needs to know that Kubernetes is in charge of the boxes and that its defaults leave them open. The next two slides define the Kubernetes words the rest of the deck uses.
-->

---

## Kubernetes Words You Will Hear

![w:900](assets/intro-k8s-terms.svg)

<!--
Walk the picture from left to right. You write a YAML file that asks for three replicas of a pod and apply it with kubectl. The control plane stores the request, and a controller starts pods until three are running, spread across the cluster's nodes. If a node dies, the controller starts replacements elsewhere. The red pod is where an agent runs: it sits on the same node, and the same kernel, as other workloads.
-->

---

## Kubernetes: 100 to 300

<div class="levels">
<div>

**100 level**

A **cluster** is a group of machines, called **nodes**. Kubernetes runs your program in **pods**, and you say how many copies, called **replicas**, you want. If one dies, Kubernetes starts another.

</div>
<div>

**200 level**

You describe what you want in YAML, for example a `Deployment` with `replicas: 3`, and run `kubectl apply`. A **controller** compares that with what is running and starts or stops pods until the two match.

</div>
<div>

**300 level**

A Kubernetes **namespace** groups objects and permissions, such as `corp-secrets`. It is a different thing from a Linux namespace, and pods in different namespaces still share nodes and kernels. A `Sandbox` is one pod with a stable name, like a `StatefulSet` with one replica.

</div>
</div>

<!--
The 300 column heads off a common confusion: the word "namespace" means two different things in this deck. The Kernels slide uses the Linux meaning (what a process can see). Here it is the Kubernetes meaning (a folder for objects and permissions). The lab's stolen secret lives in the corp-secrets Kubernetes namespace, and the unsecured pod reads it anyway, because its token has cluster-wide permissions.
-->

---

## Four Ways Out of a Default Box

![w:900](assets/intro-open-doors.svg)

<!--
In our lab, an ordinary pod on default settings gets an auto-mounted credential, runs as root, can write to its own files, and can connect anywhere. We added access to the host's files and processes for the demo, and nothing in Kubernetes stopped us.
-->

---

## Hardening: 100 to 300

<div class="levels">
<div>

**100 level**

Take the keys out of the box. Close the hatch to the machine's files. Run as an ordinary user, and shut the network door. The agent's code stays the same.

</div>
<div>

**200 level**

Set `automountServiceAccountToken: false`, drop `hostPath` and `hostPID`, run as non-root with all capabilities dropped and a read-only root filesystem, use seccomp `RuntimeDefault`, and add a default-deny egress `NetworkPolicy`.

</div>
<div>

**300 level**

Enforce it so nobody can forget. The Pod Security `restricted` profile rejects pods that run as root, keep capabilities, or mount host paths. A `NetworkPolicy` does nothing without a CNI that enforces it, so the lab installs Calico.

</div>
</div>

<div class="def">

The **CNI** (Container Network Interface) is the plugin that gives each pod an IP address and connects pods to each other. It also decides whether `NetworkPolicy` rules are enforced: Calico enforces them, and kind's default plugin, kindnet, ignores them.

</div>

<!--
kind's default network plugin, kindnet, accepts NetworkPolicy objects and ignores them. Without Calico the lab's "network closed" result would be false.
-->

---

## The Lab: Same Agent, Two Boxes

![w:900](assets/intro-two-boxes.svg)

<!--
Both boxes run the same image and the same attack script. The plain pod gives up a production database password, the cluster's master key, and a view of every program on the machine within seconds. The hardened Sandbox blocks every attempt.
-->

---

## How the Secured Box Blocks Each Attack

| The agent tries to | What stopped it |
| --- | --- |
| steal passwords with the cluster's keys | no keys in the box, and the account has no permissions |
| read the machine's files | the box cannot see the host's files |
| see other programs | the box sees only its own programs |
| change or install things | it runs as an ordinary user on a read-only disk |
| reach other systems | the network is closed except for name lookups |

<div class="def">

Defense in depth: each setting closes one path, so one forgotten setting does not open them all.

</div>

<!--
One omission is enough for an attacker. Forgetting to turn off the auto-mounted token alone hands over the cluster's keys. Layering means the other settings still hold when one is missed.
-->

---

## A Ladder of Isolation

![w:900](assets/intro-isolation-ladder.svg)

<div class="idea">

The hardened container still shares the host kernel. How much start-up time would you trade to stop sharing it?

</div>

<!--
gVisor sits between the agent and the real kernel and answers the agent's requests itself. In the lab it is a one-line change. MicroVMs give each box its own kernel, but they need hardware virtualization and take seconds to start.
-->

---

## Isolation Runtimes: 100 to 300

<div class="levels">
<div>

**100 level**

Stronger walls take longer to put up. gVisor adds about half a second; a microVM adds a few seconds. For untrusted agent code, gVisor is a good default.

</div>
<div>

**200 level**

You pick the runtime per pod with one field, `runtimeClassName`. gVisor (`runsc`) runs a userspace kernel that handles the agent's system calls. Kata and Firecracker boot a small VM with its own kernel.

</div>
<div>

**300 level**

gVisor gives up some system-call coverage and I/O speed for a much smaller host-kernel surface, and it needs no KVM. Kata and Firecracker need `/dev/kvm`, which laptop container VMs and many cloud instances do not expose.

</div>
</div>

<!--
The lab runs gVisor on a laptop. The Kata path was validated on an EC2 m7i instance because it needs KVM.
-->

---

## Warm Pools: Strong Walls Without the Wait

![w:900](assets/intro-warm-pool.svg)

<!--
A warm pool works like a taxi rank: the boxes are started before anyone asks, so an agent gets one in under two seconds instead of waiting 10 to 30. The pool refills itself.
-->

---

## Warm Pools: 100 to 300

<div class="levels">
<div>

**100 level**

Start boxes before anyone asks for one, so an agent never waits for the slow part.

</div>
<div>

**200 level**

`agent-sandbox` uses three objects. A `SandboxTemplate` holds the secure recipe, a `SandboxWarmPool` keeps a set number ready, and a `SandboxClaim` is an agent's request, which takes a ready box from the pool.

</div>
<div>

**300 level**

Size the pool to peak demand. Too small, and agents wait for new boxes at the cold-start time. Too large, and you pay for boxes that sit idle. A claimed box leaves the pool, and the pool starts a replacement.

</div>
</div>

<!--
Pool sizing is a cost decision, the same one any platform makes about spare capacity.
-->

---

## The `agent-sandbox` Project

An open-source Kubernetes project that makes a sandbox one object.

- One `Sandbox` replaces a dozen hand-wired settings.
- Templates make the secure settings the default, so nobody has to remember them.
- Warm pools come built in.
- The stronger runtimes plug in with one field.

<div class="def">

You can build all of this by hand, and one forgotten setting leaves the box open. A template applies the same secure recipe to every sandbox.

</div>

<!--
You could assemble everything the project does from ordinary Kubernetes pieces. The project gives every sandbox the same recipe, so the secure settings cannot drift.
-->

---

## Five Questions for Any Agent Platform

1. **Keys**: does the agent hold any credentials it does not need?
2. **Host**: can it see the machine's files or other programs?
3. **Power**: does it run as an all-powerful user?
4. **Network**: what can it connect to? The answer should be almost nothing.
5. **Walls**: does it share a kernel with other customers, or have its own?

<div class="idea">

If you cannot answer one of these, that is your next conversation with the platform team.

</div>

<!--
These questions work for any platform: Kubernetes, a cloud service, or something built in-house. They map one to one to the attacks in the lab.
-->

---

<!-- _paginate: false -->

![bg contain](../infographic/sandboxing-agentic-ai.svg)

<!--
The whole talk on one page. The same file lives in the repo's infographic/ folder for sharing after the session.
-->

---

## Run the Lab

```bash
cd lab
./scripts/01-up.sh                 # kind + Calico + the Sandbox CRD (about 4 minutes)
./scripts/02-attack-unsecured.sh   # the plain Pod: every probe breaches
./scripts/03-defend-secured.sh     # the hardened Sandbox: every probe is contained
WITH_GVISOR=1 ./scripts/01-up.sh && ./scripts/05-compare-gvisor.sh   # runc vs gVisor
./scripts/99-down.sh               # delete the cluster
```

The 200-level deck (`slides.md`) covers the same ground with the real Kubernetes settings.

- **agent-sandbox**: github.com/kubernetes-sigs/agent-sandbox
- **gVisor**: gvisor.dev · **Kata Containers**: katacontainers.io · **Firecracker**: firecracker-microvm.github.io
- **Kubernetes security**: Pod Security Standards, Network Policies, and RBAC on kubernetes.io

<!--
The lab runs on a laptop and reproduces every result in this deck. lab/README.md walks through each script and its expected output.
-->

---

## Glossary: The Ideas

<style scoped>section{font-size:24px}</style>

| Term | Meaning |
| --- | --- |
| Agent | an AI model in a loop that takes actions, such as running code or calling tools |
| Harness | the program around the model that runs each tool call the model asks for |
| Prompt injection | instructions hidden in a web page or file that steer an agent toward an attacker's goal |
| Sandbox | a fenced-off place to run untrusted code |
| Blast radius | everything the agent could damage if it went rogue |
| Defense in depth | several independent settings, each closing one path, so one mistake does not open them all |
| Container | a lightweight box for a program that shares the host's kernel |
| Virtual machine (VM) | a full pretend computer with its own kernel |
| Hypervisor | the software that runs VMs and uses the hardware to keep them apart |
| Warm pool | sandboxes started ahead of time so an agent does not wait for one |

<!--
Three glossary slides, grouped from plain ideas to the Kubernetes objects. Leave these up during questions.
-->

---

## Glossary: Linux and Containers

<style scoped>section{font-size:24px}</style>

| Term | Meaning |
| --- | --- |
| Kernel | the core of the operating system; every program asks it to open files and send network traffic |
| System call (syscall) | one request from a program to the kernel, such as "open this file" |
| Namespace (Linux) | a kernel feature that limits what a process can see: other processes, files, network |
| cgroup (control group) | a kernel feature that caps the CPU, memory, and I/O a process can use, and how many processes it can start |
| seccomp | a kernel filter that blocks system calls a process should not make |
| Capabilities | root's powers split into about 40 pieces; the secured box drops all of them |
| root, non-root | root (user ID 0) can do anything; the secured box runs as user 65532 |
| `io_uring` | a fast kernel interface for disk and network I/O, and the source of several container escapes |

<!--
Every term here appears in a 200 or 300 column earlier in the deck.
-->

---

## Glossary: Kubernetes

<style scoped>section{font-size:24px}</style>

| Term | Meaning |
| --- | --- |
| Cluster, node | a cluster is a group of machines run by Kubernetes; each machine is a node |
| Pod | the smallest unit Kubernetes runs: one or more containers that share a network address |
| Replica | one identical copy of a pod; you ask for a count and Kubernetes keeps that many running |
| Deployment, StatefulSet | objects that keep a set number of replicas running; a StatefulSet gives each one a stable name |
| Namespace (Kubernetes) | a named group of objects and permissions in the cluster, such as `corp-secrets` |
| Service account token | a credential Kubernetes mounts into a pod by default so it can call the Kubernetes API |
| RBAC | role-based access control: rules for what each identity may do in the Kubernetes API |
| `hostPath`, `hostPID` | settings that let a pod see the node's files or the node's processes |
| NetworkPolicy, CNI | traffic rules for a pod, and the network plugin that enforces them (Calico does, kindnet does not) |
| Egress, metadata service | outbound traffic, and the cloud endpoint (169.254.169.254) that hands out machine credentials |

<!--
The secured Sandbox closes every row from service account token down to egress.
-->

---

## Glossary: Runtimes and `agent-sandbox`

<style scoped>section{font-size:24px}</style>

| Term | Meaning |
| --- | --- |
| runc | the default container runtime; its containers share the host kernel |
| gVisor (`runsc`) | a runtime with its own userspace kernel that handles the container's system calls |
| Kata Containers, Firecracker | runtimes that put each container in a small VM with its own kernel |
| KVM (`/dev/kvm`) | the hypervisor built into Linux; Kata and Firecracker need it |
| RuntimeClass | names a container runtime; `runtimeClassName` picks one for a pod |
| CRD, controller | a CustomResourceDefinition teaches Kubernetes a new kind of object; a controller makes the cluster match it |
| `Sandbox` | one sandbox: a single pod with a stable name and its own storage |
| `SandboxTemplate` | the secure recipe that sandboxes are built from |
| `SandboxWarmPool`, `SandboxClaim` | the pool of ready sandboxes, and an agent's request for one |

<!--
The agent-sandbox project adds the last four rows on top of ordinary Kubernetes.
-->
