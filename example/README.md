# The worked example, as Intent IR

[`enterprise_llm_gateway.yaml`](enterprise_llm_gateway.yaml) is the intent every
result in this repository is computed from: an OpenAI-compatible inference
gateway fronting multiple model providers for 400 tenants at 10,000 sustained
RPS.

Note what is **not** in it — Go, Redis, PostgreSQL, Kubernetes, Prometheus,
microservices, serverless. Those are candidate mechanisms, and choosing among
them is the design plane's job. A human who writes them here has re-entered
L1.5, and the implementation has a test that fails if any catalog mechanism name
appears anywhere in an intent.

What is in it: capabilities with the properties they require, SLOs with
comparators and severities, invariants that name required *properties* rather
than technologies, a cost envelope, a preference vector, and policies governing
promotion.
