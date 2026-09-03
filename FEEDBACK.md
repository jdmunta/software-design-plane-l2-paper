# What I would most like challenged

This is a version 1.0 white paper with one case study. It is published now
because the parts most likely to be wrong are the parts I cannot check alone.
Below is what I would rather have attacked than praised.

**Open an issue for any of these**, or take the whole thing apart in a comment.
Disagreement with a specific number or mechanism is more useful than agreement
with the framing.

---

## 1. Is the baseline a strawman?

Arm A of the experiment encodes a conventional human prescription — Go, Redis,
PostgreSQL, Kubernetes, Prometheus — and L2 beats it on requirements satisfied,
p99 and measured capacity.

**The obvious objection is that a good architect would not have produced arm
A.** I think that objection is at least partly right. The implementation takes
a `--baseline-a` override precisely so a reader can substitute a stronger human
design and re-run, and I would genuinely like to see someone do that and report
that the treatment loses.

If you have a design for the [worked example](example/enterprise_llm_gateway.yaml)
that you think beats what search found, that is the single most valuable piece
of feedback available.

## 2. Is 5x review cost fatal?

The experiment's own verdict includes `NEGATIVE — human review cost scaled
5.0x`. A reviewer must compare five frontier candidates rather than approve one.

I argue this is worth it because each frontier member is the best available on
something and the plane names which. I am not certain. It is possible that:

- review cost scales worse than linearly with candidate count in practice,
- five options produce worse decisions than one via choice overload,
- or the naming of owned axes does not actually reduce the burden it claims to.

None of these are things a simulation can settle.

## 3. Does the catalog carry the whole result?

Every quantitative conclusion is a function of a hand-written mechanism catalog,
and 20 of its 27 rows are my own estimates with a declared 35% relative sigma.
That sigma is itself a judgement.

The paper's defence is that the uncertainty is propagated and that 27% of
candidate/dimension orderings are reported as *not separable* given it. But:

- Are the point estimates in the right order of magnitude at all?
- Is 35% too generous or too tight for this class of component?
- Does correlated error across rows — my systematically over- or under-rating a
  whole technology family — break the resampling, which treats rows as
  independent?

The last one is the objection I have the least good answer to.

## 4. Is "presence is not enforcement" general, or a patch?

The reachability check requires that enforcement of a data-guarding invariant
lie on the request path, upstream of every point that data leaves the design. It
found a real class of false certification.

But it is ordering-and-presence analysis over a *declared* graph. It says
nothing about whether the running system matches the graph, and invariants
without a data class are deliberately exempt. Is that exemption principled, or
is it drawing the boundary around the cases the check happens to handle?

## 5. Do the research questions survive?

The paper works through eight. The ones I consider genuinely open:

| | |
|---|---|
| **RQ1** | Requirements that resist property decomposition entirely — is intent representation a general solution or one that works on the well-behaved subset? |
| **RQ3** | Evaluator independence is measured structurally. Two evaluators can share a blind spot without sharing a formula. |
| **RQ6** | Safe evolution is only partially addressed. State migration, compatibility and rollback mechanics are out of scope, and that may be where the real difficulty lives. |

## 6. Is L2 the right cut?

The maturity model claims spec-driven development (L1.5) leaves the architecture
decision untouched and that this is the thing worth automating next.

An alternative reading: architecture selection is not the bottleneck, and the
same effort spent on verification of *implemented* systems would pay better. If
you think the level boundary is drawn in the wrong place, say so — that is an
argument about the premise, not the details, and it matters more.

---

## What is already known to be missing

Not defects to report — stated so feedback can go elsewhere:

- Nothing is deployed. Shadow execution, fault injection and the capacity ramp
  are simulation. Model calibration therefore measures the analytic models
  against the simulator, not against reality.
- Formal methods are implemented only at the modest end (reachability), not as
  proof.
- Security invariants are checked structurally, never by attacking a running
  system.
- One intent, one case study. Every conclusion is scoped to it.
- The implementation stops at a promotion decision and does not generate code.
