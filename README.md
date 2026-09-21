# csa-python-mcp-foundation

**The shared foundation for CSA's Python API libraries and the MCP servers that deliver them.**

> **Status: parked.** This repository exists to hold the docs, the method and the open
> questions while the evidence accumulates. No code yet. See [Why parked](#why-parked).

---

## What this is for

CSA is building a lot of these. Each one is **two artifacts, not one**:

```
  library      the whole vendor API - tested, versioned, installable,
               and useful with no agent present
  MCP server   a curated subset, shaped for a model: names, descriptions,
               capability gates, safety defaults
```

**The library is the product. The server is a delivery layer.** That ordering is not tidiness —
it is the difference between a finite tool set and an unbounded API. The moment a user wants
something outside the tools, a tools-only server has nothing to offer and a library-first one has
the entire API.

This repository is where the parts that recur across every such pair will live: the shared code,
the method for producing one, and the opinionated defaults for building on top.

## Why two artifacts matters more in the AI era

A traditional API client assumes the caller is code with fixed intent. Here the caller is a model
that can be **influenced by the data it reads**, will **attempt things nobody enumerated**, and
**fails in plausible-looking ways** rather than crashing. Three consequences follow, and they are
the reason this foundation is worth having at all:

**1. Coverage turns a technical question into a policy one.** At partial coverage, every refusal
is ambiguous — is this forbidden, or merely unwritten? Nobody can audit the shape of what someone
got round to. At near-total coverage, every restriction becomes a *named decision*. `csa-zendesk`'s
scoping triage put 822 operations into 335 now / 387 later / **48 never** / 52 blocked; the
`never` bucket only means anything because the alternative was possible.

**2. The agent can extend a local server itself.** With the library present, a request outside the
tool set is about fifteen lines of Python — and **the user does not write it, the agent does**.
For a local server that escape hatch is free: the interpreter is already on the machine. For a
hosted server it is a product decision with a sandbox behind it, and it buys something local
cannot — scheduled, unattended, centrally-audited runs.

**3. Where the agent cannot extend, it must be able to ask.** A hosted server's users cannot write
against the library. So the server carries a feedback tool, and requests for specific workflows
land as input to the next version. Several CSA servers already ship this (`report_a_problem`,
`c360_feedback`); the foundation should make it standard rather than incidental.

## The identity problem this has to live with

**MCP servers run as you.** There is no delegation model, no service principal, no scope-down.
Every server you install holds your full authority at that vendor, and most vendors cannot issue
a narrower identity even when asked.

So scoping has to happen **client-side** — an install declares which objects it may touch, and the
default is nothing. That is not a stopgap pending a better answer; for most vendors it is the only
answer available, and the foundation should treat it as a first-class feature.

**Stated honestly, because the honesty is what makes it useful:** a client-side allowlist is a
**blast-radius control, not a security boundary**. Anyone who can edit the environment can widen
it. What it reliably prevents is the dominant real failure — an agent acting on the wrong object —
and, because it is *declared*, a reviewer can read what a given install may reach. It must never
be described as isolation, or people stop asking for the real thing.

## Opinionated defaults: refuse, warn, and let them override in the open

We are open source; anyone can fork or ignore us. So the defaults are not enforcement, they are
**an opinion with a visible escape hatch**:

- **Refuse by default** where the failure is irreversible or reaches a third party.
- **Warn, do not block**, where the action is destructive but instantly recoverable.
- **Override per control, never by one master switch.** `--unsafe` gets set once during a bad
  afternoon and lives forever; a named variable reads as alarming in a config file, which it
  should.
- **The server reports its own overrides at runtime.** An override recorded only in an environment
  nobody re-reads is how a fleet quietly opts out. If the server can be asked what it may do, it
  can be asked what it has been told to ignore — and the model operating without a guard should
  be told so too.

**We ship the taxonomy, not the verdicts.** Which operations are dangerous is a judgement only the
people who know that vendor can make. What the foundation provides is the vocabulary to make it
in: **Effect · Reversibility · Reach · Authority**, with reversibility *ternary* —
`reversible · reversible-for-a-period · irreversible`.

The axis that earns its place in the guidance is **Reach**, because it is the one people miss
unaided: a public reply is trivially reversible in the database and completely irreversible in the
world. An author reasoning only from "can I undo this?" will file it as safe.

## Candidate scope

Nothing here is committed. Each line needs the projects behind it named before it ships.

| Area | Evidence today |
|---|---|
| The `Backend` / `PolicyBackend` / `exceptions` seam — one enforcement point, failing closed | Converged independently in two servers |
| Untrusted-data handling — **both** delimiter wrapping and terminal-control-sequence stripping | Two servers, one half each |
| Token store, refresh discipline, the stdout hazard, loopback callback | Three servers, three different OAuth grants |
| Client-side scoping / allowlists | One server |
| Attribution — so a vendor's audit log can tell an agent's actions from a human's | None yet |
| Shared-quota rate limiting across many servers against one vendor | None yet |
| Upstream drift detection | Required by CSA MCP goals; unbuilt |
| Live-verification harness (demo as end-to-end test) | One server; needs a second adopter |
| The method: API and website survey, prior-art review of existing servers, scoping triage, checklists | Applied once, end to end |

## Why parked

CSA's engineering knowledge base prohibits building structure *in advance of the experience that
would fill it*, and offers a test: **can you name the projects behind each item?** For the seam,
yes — two. For attribution and rate limiting, no — zero.

So this repository collects evidence and open questions until each line can name its projects.
Extracting a framework from two instances is the classic premature abstraction; the rule of three
exists for a reason.

## Where the thinking lives

The cross-cutting practice — what *every* CSA MCP server owes, the three server classes, and the
engineering discipline that is not MCP-specific — lives in **CINO-Platform-Engineering**
(`surfaces/mcp/` and `research/mcp-servers/`). It is not duplicated here; a forked copy is how two
divergent versions start.

This repository is for the specifics: the shared code, the build method, and the defaults.
