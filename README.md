Evidence Portfolio Pipeline (v3 — Adversarially Hardened Release)

A computational-journalism ETL and human-in-the-loop verification pipeline designed to classify large-scale news and policy corpora (18,000+ articles / 100,000+ text chunks) on pain care, opioid tapering, veteran healthcare, and drug policy against a fixed 12-tag evidentiary framework.

CORE PRINCIPLE
Machine nominates; human verifies. Every LLM classification is recorded strictly as provisional ("llm_provisional") and must pass automated verbatim-quote verification, scope-pair rules, and cryptographically signed human review before entering the citable evidence graph.

KEY ARCHITECTURAL & SECURITY FEATURES (v3 RELEASE)
1. Exact-Resume SQLite WAL Checkpointing (src/checkpoint.py, src/pipeline_orchestrator.py):
   Content-addressed (SHA-256) article deduplication with atomic temporary-file-to-replace archiving and single-transaction per-chunk SQLite WAL checkpointing. Survives hard process kills (SIGKILL) and power loss without state corruption or duplicate LLM spend.

2. Adaptive AIMD Concurrency & Rate Control (src/rate_controller.py):
   A shared asyncio concurrency controller driven by both observed P99 latency and live vLLM /metrics telemetry (queue depth and KV-cache pressure). Includes window-debounced multiplicative backoff and a starvation-safe 10% retry budget that prevents thundering herds and retry-queue deadlocks under heavy load.

3. Verbatim Quote Enforcement & JSON Repair (src/extraction_prompt.py, src/json_repair.py):
   Deterministic brace-balanced JSON extraction and recorded mechanical repair, paired with normalized character-for-character substring verification. Hallucinated quotes or framework violations are automatically quarantined and routed to a human triage queue.

4. Tamper-Proof HMAC-SHA256 Verification Log (src/audit_verifier.py, src/verify.py):
   Offline CLI review workflow requiring attributable reviewer IDs and typed rationales. Decisions are written to an append-only, hash-chained, HMAC-SHA256 signed log that verifies existing chain integrity prior to appending and supports external head-hash anchoring to detect modification, forgery, deletion, reordering, or truncation.

5. Byte-Capped, Injection-Safe Neo4j Graph Loader (src/neo4j_loader.py):
   Parameterized UNWIND batching with strict label/relationship/epistemic-status whitelists, reserved-property sanitization, shared :Entity index lookups, and a hard 3 MB per-transaction ceiling to prevent database Out-Of-Memory errors.

6. Per-Page Scanned PDF OCR Fallback (src/ocr_fallback.py):
   Detects scanned pages (<100 native characters per page) and falls back to 300-DPI Tesseract OCR with line-preserving cleanup and per-page confidence audit logging.

VERIFICATION & QUICKSTART
Requires Python 3.10+. Includes a zero-network, 18-check unit and end-to-end integration test suite:
  sha256sum -c v3_manifest.sha256
  python -m src.test_redesign

Release Archive SHA-256 (evidence-portfolio-pipeline-v3.zip):
496f96abd6b526a697c14a77f71411ac0fb3404dbb341861c8d4d91b64ad4505

MIT License
