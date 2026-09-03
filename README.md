# Software Design Plane L2

### From Spec-Driven Development to Intent-Driven Software Synthesis

**Jagadesh B. Munta · White Paper, Version 1.0 · 2026**

📄 **[Read the paper](paper/software-design-plane-l2.pdf)**

---

Spec-driven development moved the unit of human work from code to specification.
It left the architecture where it was: chosen by a human, up front, largely by
precedent, and defended afterwards. This paper asks what happens when
architecture becomes an **output of computation** rather than an input to it.

Humans declare capabilities, constraints, invariants, policies and measurable
outcomes. A design plane searches a bounded space of architectures, evaluates
candidates through independent evaluators, verifies them against a
production-like workload, records what was actually demonstrated, and returns a
promotion decision for a human to govern.

```
Intent ─▶ Synthesize ─▶ Evaluate ─▶ Verify ─▶ Operate ─▶ Observe ─▶ Re-synthesize
   │                                                                      │
   └──────────────────── Software Intent Graph ◀── Evidence Plane ─────────┘
```

The argument is accompanied by a working reference implementation. Everything
quoted below is output from that program, not illustration.

---

## The maturity model

| | | |
|---|---|---|
| **L1** | Code-driven | The artifact is code. Design lives in people's heads and in review. |
| **L1.5** | Spec-driven | The artifact is a specification; an agent writes the code. **The architecture is still chosen by a human, up front.** |
| **L2** | **Intent-driven** | The artifact is *intent*. Architecture is searched, evaluated against evidence, and selected under human governance. |
| **L3** | Continuous design | Runtime evidence feeds back into intent; the design is re-derived when reality contradicts it. |

Most of what is currently called "AI-native development" is L1.5. The move to L2
is not a better code generator — it is putting the architecture decision itself
under evidence.

## The three abstractions

**Intent IR** — outcomes, not mechanisms. The worked example declares 10,000
sustained RPS, p99 gateway overhead under 500 ms, 99.99% availability, strict
tenant isolation, no plaintext credentials, full auditability, and a
$20-per-million cost envelope. It names no technology. Invariants are
machine-checkable because they say which *properties* they require, and
mechanisms declare which properties they provide:

```yaml
- id: inv_tenant_isolation
  statement: Tenant data never crosses a tenant boundary
  requires_properties: [tenant_isolation_strict]
  applies_to_data_class: tenant_payload
  severity: hard
```

The full worked example is in [`example/`](example/enterprise_llm_gateway.yaml).

**Design IR** — architecture as a structured object that can be generated,
compared, scored, rejected and regenerated: components and the mechanism
realising each, interfaces, annotated data flows, deployment topology, expected
failure modes with declared frequency and recovery time, and traceability back
to intent.

**Evidence Plane** — every claim records the method that demonstrated it, and
the method fixes the confidence:

| method | what it means | confidence |
|---|---|---|
| `policy_check`, `static_analysis` | a fact about the Design IR | 1.0 |
| `analytic_model` | closed-form estimate | 0.5–0.7 |
| `load_sim`, `fault_injection` | simulated behaviour | 0.6–0.75 |
| `llm_evaluator` | model judgement | 0.4 |

A model judgement may never disqualify a candidate. Only deterministic checks
may. This separation is what stops "the AI said it was fine" from becoming
evidence.

## What a cycle produces

Five architectures for one intent, each offered the identical simulated trace:

```
design     architecture                        p99ms     rps    err    $/M  score
d1         Latency-optimised request path       10.7   11194  0.00%   6.68  0.890
d2         Operations-minimal plane             23.5   11194  0.00%   9.83  0.834
d3         Isolation-maximal celled plane       20.4   11194  0.00%  10.44  0.880
d4         Cost-floor plane                     10.7   11194  0.00%   3.99  0.814
d5         Availability-maximal redundant plane 15.7   11194  0.00%  10.46  0.893

decision : PROMOTE (approval required)
frontier : d1, d2, d3, d4, d5
  frontier members and the axis each one alone owns:
    d1   security, reliability, performance
    d2   security
    d3   security, reliability
    d4   cost
    d5   security, reliability, complexity, maintainability
```

All five survive as a Pareto frontier and are **not** ranked against each other.
Non-dominance guarantees each one leads somewhere, so the plane says where.
Scalarisation happens exactly once, using the human's declared preference, and
only to order candidates already on the frontier.

Full transcripts: [`results/`](results/).

## Six results worth arguing with

**1 — Presence is not enforcement.** Matching a required property against the
mechanisms present answers "does this design contain something that *could*
enforce the invariant?" For an invariant guarding data, that is the wrong
question. A design that includes per-tenant cells but places them off the
request path contains strict isolation and enforces nothing. The plane certified
exactly that at confidence 1.0 until enforcement was checked by reachability
over the design's own interface graph — upstream of every point the guarded data
leaves the system.

**2 — Disagreement is only evidence if the evaluators are independent.** An
earlier version of the adversarial reviewer scored `complexity` with the
identical formula over the identical field as the analytic evaluator. Every
aggregate looked better-supported than it was. The disagreement metric caught it
— exactly 0.000 on that dimension — and the plane now classifies each dimension
as `independent`, `correlated`, `duplicate` or `degenerate`. This was a real
defect found in the implementation, not a hypothetical.

**3 — A throughput objective is a claim about capacity.** Replaying 10k rps and
observing 10k rps served demonstrates only that the design met the load it was
given. Offered load is binary-searched upward until the design sheds, and the
objective is demonstrated by the measured knee and the component that limits it.

**4 — The plane's own models are held to the standard it holds intent to.**
Where a closed-form estimate and the simulation disagree, the measurement wins
*and the size of the correction is kept*. On this intent, latency and capacity
track to within 0.3% and single-region availability is materially optimistic —
a located instruction about which model needs work, and exactly the number that
disappears if measurement silently overwrites the estimate.

**5 — Roughly a quarter of the orderings are not evidence.** Twenty of the
catalog's 27 rows are the author's estimates. Each declares its own uncertainty;
the catalog is resampled 64 times and the evaluation re-run on each draw:

```
         frontier  rejected      security     performance          cost
  d1        100%        0%    1.00 (fact)     0.77-0.84      0.72-0.81
  d2         98%        2%    1.00 (fact)     0.78-0.88      0.72-0.81
  d5        100%        0%    1.00 (fact)     0.80-0.85      0.62-0.72
```

Security and correctness have zero-width intervals because they rest on property
checks — facts, not estimates. Two candidates sit *on* a hard constraint
boundary rather than clear of it, which a point estimate reports as a clean
pass. And 27% of candidate/dimension orderings cannot be separated at all given
the catalog's own uncertainty. That is reported as a finding rather than buried.

**6 — The falsifiable experiment returns a mixed result.** Three arms under
identical tests: **A** a human prescription (Go, Redis, PostgreSQL, Kubernetes,
Prometheus), **B** one AI-generated architecture with a single evaluator, **T**
multi-candidate search with independent evaluation.

```
              req%    p99ms   cap rps     err     $/M  infra  cands  evals  review
arm A         90.0     20.2    26,094  0.00%    6.37     10      1      1       1
arm B        100.0     10.8    35,469  0.00%    6.68     10      1      1       1
arm T        100.0     15.5    44,844  0.00%   10.46     10      5     10       5

* L2 rejected a constraint violation the human baseline carried into the design:
  ['inv_tenant_isolation'].
* Treatment vs baseline A — better on requirements satisfied, p99 latency,
  measured capacity; worse on cost.
* Treatment vs baseline B — better on measured capacity; worse on p99, cost.
* NEGATIVE — human review cost scaled 5.0x.
```

Single-shot AI generation already satisfied every hard requirement here. Search
bought 27% more measured capacity and a frontier, and paid for it in cost, p99
and **five times the review burden**. That is the honest result, and it is
reported rather than smoothed.

## What this deliberately does not claim

- **Nothing is deployed.** Shadow execution, fault injection and the capacity
  ramp are simulation. Real environments would change every number.
- **The catalog is hand-written.** Order-of-magnitude estimates. The declared
  35% uncertainty is itself a judgement.
- **The search space is bounded by the catalog.** Search composes declared
  mechanisms; it does not invent them.
- **Reachability is graph analysis, not proof.** It checks ordering and presence
  over a *declared* interface graph. It cannot tell you the running system
  matches that graph.
- **One case study.** Every conclusion is scoped to a single intent and says so.
- **The plane stops at a decision.** Implementation — coding agents, IaC,
  generated tests — is where spec-driven tooling already works, and is left to
  it.

## Reference implementation

A working implementation accompanies the paper: the three IRs, a mechanism
catalog as the bounded search space, archetype-seeded search with repair and
diversity filtering, analytic and adversarial evaluators, workload simulation
with a capacity ramp and fault injection, catalog resampling, bounded continuous
design, the falsifiable experiment, and a CLI, HTTP API and dashboard.

It is **not yet public** — it is being validated before release. If you would
like access for review, please open an issue here.

## Feedback

This is a version 1.0 white paper and the whole point of publishing it now is to
be argued with. [`FEEDBACK.md`](FEEDBACK.md) names the specific questions I would
most like challenged — the research questions, the negative results, and the
catalog assumptions everything else rests on.

Open an issue, or reach me on LinkedIn.

## Citation

```bibtex
@techreport{munta2026l2,
  author = {Munta, Jagadesh B.},
  title  = {Software Design Plane L2: From Spec-Driven Development to
            Intent-Driven Software Synthesis},
  year   = {2026},
  type   = {White Paper},
  number = {Version 1.0}
}
```

## License

The paper and its text are released under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — share and adapt with
attribution. The Intent IR example in [`example/`](example/) is Apache-2.0,
matching the reference implementation.
