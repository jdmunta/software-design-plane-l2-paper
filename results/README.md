# Sample outputs

Console transcripts from the reference implementation, on the worked example in
[`../example/enterprise_llm_gateway.yaml`](../example/enterprise_llm_gateway.yaml).
Regenerated from the implementation rather than transcribed by hand.

| file | command | what it shows |
|---|---|---|
| [`full-cycle.txt`](full-cycle.txt) | `l2 run --candidates 6` | one complete cycle: search, diversity, evaluator disagreement and independence, measured capacity, model calibration, catalog resampling, and the promotion decision |
| [`experiment.txt`](experiment.txt) | `l2 experiment` | the falsifiable experiment — arms A/B/T, findings, verdict and caveats |
| [`review-external-design.txt`](review-external-design.txt) | `l2 review --design ... --against demo` | a hand-authored architecture submitted to the plane, rejected for placing tenant isolation off the request path, and placed against the searched candidates |

Every figure here is produced by a model or a simulation. **Nothing is
deployed.** Each demonstration is tagged with the method that produced it and
carries a confidence derived from that method.
