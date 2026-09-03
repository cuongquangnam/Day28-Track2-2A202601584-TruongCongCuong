# Answers — Day 28 Track 2

## Current Submission Status

This repository has completed the coding checkpoint and the basic-system flow.
The local evidence pack is intentionally marked partial where the current machine
cannot prove a full integration point without the full Airflow/Spark profile,
a real GPU-backed vLLM endpoint, or LangSmith credentials.

- Fast suite: `87 passed` in `evidence/fast-suite-output.txt`.
- Basic stack: Kafka, API, Envoy gateway, Feast, Qdrant, MLflow, Prometheus,
  Grafana, Jaeger and OTEL collector started successfully with `ports.local`.
- Seed data: 25 records were accepted through the gateway after throttling to
  stay below Envoy's 10 requests/second local rate limit.
- Readiness: `ready` for the basic flow with `LAB28_VLLM_REQUIRE_REAL=false`.
- Integration report: `integration-report.json` is generated, but not fully
  ready because Delta/Airflow and full trace evidence are not available locally.

## Trade-offs

- The four student-owned functions are kept small and contract-driven. Feature
  names come from `FEATURE_REFS` instead of being duplicated in the Feast request.
- `dedupe_latest` orders output by `idempotency_key` so Kafka replay produces a
  deterministic Delta merge source.
- The basic flow uses local port overrides in `ports.local` because this machine
  already had services on ports 5000, 8000 and 6333.
- The basic flow treats vLLM as optional by setting `LAB28_VLLM_REQUIRE_REAL=false`;
  this is acceptable for local readiness but does not satisfy the real GPU gate.

## Production Gaps

- IP02 and IP03 need the full profile to run Airflow and Spark/Delta, then capture
  DAG run ID, asset event, Delta history and time-travel evidence.
- IP04 needs Feast online materialization from the Delta-derived feature snapshot.
- IP07 needs a real vLLM endpoint that proves `/version`, `/v1/models` and `vllm:`
  metrics.
- IP10 needs a full trace containing gateway, API, Kafka, Airflow, Spark, Feast,
  Qdrant, MLflow and vLLM spans.
- Failure/recovery, no-data-loss proof, GitOps drift/self-heal and rollback
  evidence still need to be recorded in a live demo environment.

## Load Profile

The local load profile is in `evidence/load-profile.json`.

- Requests: 200
- Workers: 8
- Status counts: 15 successful responses, 185 rate-limited/failed responses
- P50: 0.92 ms
- P95: 290.84 ms
- P99: 412.59 ms

The main bottleneck in this basic run is the Envoy local rate limit. The gateway
is configured for 10 requests/second, so concurrent `/ready` probes quickly hit
the limiter. This is expected and is also useful evidence for IP08.

## Contributions

- Truong Cong Cuong: implemented the student-owned integration functions,
  ran the fast suite, started the basic platform flow, handled local port
  conflicts, generated the current evidence pack and documented remaining gaps.
- AI assistant: helped inspect the assignment, implement and verify code, run
  local checks and produce truthful submission artifacts.
