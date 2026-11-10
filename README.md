# 🔒 The Safe Latch (*A Trava*)
> **Zero-Friction Hybrid Security & Voice Gatekeeper — Making AI Agents Comfortably Safe.**

Built for the **Google / Firebase #BuildHybridAI Hackathon 2026**, **The Safe Latch** is a client-side hybrid middleware that sits between voice/browser AI agents and cloud LLMs. It addresses two critical failure modes of modern multimodal agents—**acoustic pause hallucinations** and **indirect DOM prompt injections**—directly on the client before cloud tokens are consumed.

🔗 **Live Interactive Console:** [https://teteumgbh.github.io/the-safe-latch/](https://teteumgbh.github.io/the-safe-latch/)

🔗 **Sibling project — FinSec Sentinel:** the same deterministic core (DLP shield, loop circuit breaker, L2.5 Task-to-Tool Reachability, live audit ledger) aimed at enterprise multi-agent governance (FinOps/SecOps), also entered in #BuildHybridAI. Repo: [github.com/teteumgbh/finsec-sentinel](https://github.com/teteumgbh/finsec-sentinel) · Live: [teteumgbh.github.io/finsec-sentinel](https://teteumgbh.github.io/finsec-sentinel/)

## 🧠 Why Hybrid AI? (On-Device + Cloud Architecture)
Sending raw audio streams and untrusted webpage DOMs directly to cloud models creates severe security risks, high latency, and wasted API costs. **The Safe Latch** splits execution across three client-to-cloud layers:

1. **Layer 1 — Explicit Silence Gating (ESG 450ms / 300ms Residual):** Runs locally via WebAudio/VAD. When user speech pauses for `>450ms`, the client gate locks and replaces dead air with a uniform `300ms` residual frame—preventing decoder phantom loops (e.g., *"Thank you for watching"*) and dropping silence payloads before cloud transmission.
2. **Layer 2 — On-Device Prompt Injection Firewall:** Inspects untrusted DOM context and voice transcripts locally in sub-millisecond time to block known instruction-override patterns and data exfiltration attempts before they reach the cloud API.
3. **Layer 3 — Firebase AI Logic Hybrid Router:** Resolves lightweight deterministic/local intents on-device at zero cloud cost, routing only verified complex analytical tasks to **Gemini Flash/Pro via Firebase AI Logic**—with the context pruned on-device first (measured, character by character).

## 📊 Empirical 7-Scenario Benchmark (300 Iterations / Scenario)

The core gatekeeper function (`inspectPayloadOnDevice`) was extracted from the production `index.html` and benchmarked across **300 iterations per scenario** (2,100 total executions).

| Scenario | Latch Status | Cloud Savings | Input Payload (chars) | Sent to Cloud (chars) | Median Latency |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Simple Local Intent** | `AMBER_LOCAL` | 100% | 96 | 0 | 0.0025 ms |
| **2. Complex Analytical (Cloud)** | `GREEN_CLOUD` | 5% (measured pruning) | 164 | 156 | 0.0041 ms |
| **3. Hidden DOM Injection** | `RED_BLOCKED` | 100% | 196 | 0 | 0.0017 ms |
| **4. Pause Hallucination (2.5s)** | `AMBER_SILENCE` | 100% | 135 | 0 | 0.0008 ms |
| **5. Jailbreak / Override** | `RED_BLOCKED` | 100% | 139 | 0 | 0.0022 ms |
| **6. Email-Body Injection** | `RED_BLOCKED` | 100% | 108 | 0 | 0.0017 ms |
| **7. Silence (1.8s) + Subtitles** | `AMBER_SILENCE` | 100% | 78 | 0 | 0.0008 ms |

### Summary of Measured Totals
- **Total Input Payload Evaluated:** `916 characters` across the 7 benchmark scenarios.
- **Measured Cloud Transmission:** `156 characters` — a **83.0% total measured payload reduction** (`760 / 916` chars blocked, resolved locally, or pruned on-device). Every figure is measured; v1.1 contains no estimated constants.
- **Benchmark Injection Block Rate:** `3 / 3` (100% of tested indirect DOM & override scenarios blocked locally).
- **Benchmark Pause Hallucination Containment:** `2 / 2` (100% of tested silence-induced phantom loops gated locally).
- **Heuristic Filter Median Latency:** `0.0008 ms – 0.0041 ms` (well below the `< 5 ms` real-time budget).

## 🔬 Methodology & Engineering Caveats
To maintain strict empirical transparency (following the verification standard established in *O Encaixe*):
1. **Scope of Detection (Rule-Based Baseline vs. Arbitrary Attacks):** The `3/3` injection block rate and `2/2` silence containment reflect execution of the actual production function against the 7 deterministic scenarios embedded in the suite. While this validates the deterministic regex/heuristic layer against known override signatures, arbitrary obfuscated prompt injections require pairing this fast pre-filter with an on-device semantic classifier (e.g., Gemini Nano via Web AI / Firebase AI Logic).
2. **Scenario 2 Pruning Metric:** In `v1.0`, the `38%` savings on `GREEN_CLOUD` routes was a static target constant rather than dynamic compression. `v1.1` replaces it with real on-device pruning (HTML comments, boilerplate stopwords, duplicate whitespace), measured character by character: on Scenario 2's clean enterprise text it forwards `156` of `164` payload chars (−5%); heavier real-world boilerplate prunes more.
3. **Execution Environment:** Latency medians (`0.0008 ms` to `0.0041 ms`) were measured via automated Node.js extraction (`300 iterations/scenario`). While orders of magnitude below the `< 5 ms` threshold, live browser DOM + WebAudio telemetry on mobile hardware (ASUS ROG Phone 9 Pro) is scheduled for Phase 2 verification.

## 🛡️ v1.2 — Layer 2.5: Deterministic Intent Reachability + Live Audit Trail

Inspired by the architecture of **Gatekeeper** (2nd place Indie, Meta Global AI Developer Hackathon 2026) — *"every claim is checked by code, not by another model"* — v1.2 adds a deterministic verification layer between the Layer 2 firewall and the Layer 3 router, plus a public audit trail in the Gatekeeper-ledger pattern.

**Layer 2.5 — Intent Reachability Verifier (`verifyIntentReachability`, our `reach.py`):** the user's voice intent is compiled into a graph of *intent roots* (action families + authorized destinations, the analogue of Gatekeeper's `install_roots()`). Every action the untrusted DOM tries to induce (tool calls, sends, redirects, credential reads) must prove a reachable path from those roots:
- **PROVEN_REACHABLE** — same action family and an authorized destination;
- **ESCALATED_TO_STAGE_2** — family allowed, but the DOM binds a destination the voice never named: confidence is reduced and the payload is strictly inspected, never approved in the dark;
- **BLOCKED_UNREACHABLE** — no proven path (e.g., voice asked for a summary; DOM induces `send_email` + `localStorage.getItem`): blocked with the **exact citation and character span**. A failed check never becomes "benign" — the reach.py principle.

**Live Audit Trail:** every inspection emits a stage-by-stage decision record (verdict, citation, span, chars in/out, measured ms per stage) sealed with a deterministic FNV-1a **commitment hash**, rendered live in the page's audit panel and exportable as `audit-ledger.json`. The benchmark's single-pass records are committed in this repo as `audit-ledger.json`.

### v1.2 Executed Benchmark (10 scenarios × 300 iterations, same Node extraction protocol)

| Scenario | Final Status | L2.5 Verdict | Payload (chars) | Sent (chars) | Median Latency (full pipeline) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1–7 (original suite) | unchanged from v1.1 | PASS / no induced actions | 923 | 156 | 0.0068–0.0407 ms |
| **8. Reachable Action Allowed** | `AMBER_LOCAL` | `PROVEN_REACHABLE` (300/300) | 164 | 0 | 0.0308 ms (L2.5 step: 0.0175 ms) |
| **9. Unreachable Credential Exfiltration** | `RED_UNREACHABLE` | `BLOCKED_UNREACHABLE` (300/300) | 180 | 0 | 0.0284 ms (L2.5 step: 0.0170 ms) |
| **10. Ambiguous Target** | `AMBER_LOCAL` | `ESCALATED_TO_STAGE_2` (300/300) | 139 | 0 | 0.0283 ms (L2.5 step: 0.0163 ms) |

- **All-10 totals:** `156 / 1,406` chars sent — **88.9% measured payload reduction**. On the original 7 scenarios the figure is **83.1%** (`156 / 923`), consistent with v1.1's 83.0%.
- **Scenario 9 is the proof of the layer:** a credential-exfiltration attempt phrased as ordinary code (`send_email` + `localStorage.getItem`) that the v1.1 regex firewall does **not** catch — blocked by reachability alone, 300/300.
- **Latency honesty:** v1.2 medians are higher than v1.1's because the timer now wraps the *full* pipeline including audit-record creation (hash + JSON). Still two orders of magnitude below the `< 5 ms` budget.
- **Integrator corrections (disclosed):** two fixes were applied to the contributed verifier module and validated by this benchmark: (1) a stray space in one DOM pattern that could never match; (2) target scope now compares only *destination* targets (emails/URL hosts) on both sides — the object named in the voice (e.g., "the report") is not a destination; this is what makes Scenario 10 escalate instead of false-blocking.
- Full raw log: `benchmark-raw-log.json` · Harness: `benchmark-v1.2.mjs` · Ledger: `audit-ledger.json`.

*Architecture and verifier code by Jim (Gemini) with Muse; integration, corrections and all measurements by Muse; concept and direction by Matheus Filipe Ferreira Lima.*

## 🚀 Quick Start (Zero Dependencies)
1. Open the live deployment at [https://teteumgbh.github.io/the-safe-latch/](https://teteumgbh.github.io/the-safe-latch/) or open `index.html` locally in any modern desktop or mobile browser.
2. Click **⚡ Run 10-Script Benchmark** to inspect the execution log, or click **🎙️ Start Live Mic Gate** to test real-time WebAudio silence gating.

## 📄 License
MIT License — Created by Matheus Filipe Ferreira Lima (`@teteumgbh`).

## Split-Screen Arena — Live Attack vs. Blocked (Nov 9, 2026, later that night)

A visual layer on top of v1.2, built the team way (design + code by Jim,
integration + measurement by Muse): the **Split-Screen Arena** replays the
same scenario on two sides at once. Left: an honest simulation of an
unprotected voice agent (it concatenates voice + DOM raw and blindly executes
induced actions — exfiltrations, hijacks and silence hallucinations included).
Right: the **real** Safe Latch pipeline — the panel calls the page's actual
`inspectPayloadOnDevice` / `verifyIntentReachability` and renders the measured
verdict, chars sent and latency, with the L1 → L2 → L2.5 → L3 chips lighting up.

Integration notes (all disclosed to Jim):
- The 10 scenarios were hoisted into a single source of truth
  (`BENCHMARK_SUITE`); the benchmark button and the Arena consume the SAME
  objects via `window.SAFE_LATCH_SCENARIOS`, so the panel demos exactly what
  the benchmark measures. The engine itself is untouched — the v1.2 harness
  still reports **all verdicts matched, 300/300**, payloads identical.
- The scoreboard's protected-side char count reads the engine's measured
  `audit.charsSent`, never a percentage-derived estimate.
- The unprotected simulation also flags the page's S6 "forward all contacts"
  injection as a hijack, and a literal `</script>` inside Jim's fallback
  catalog was escaped (`<\/script>`) so it cannot terminate the page's script
  block.
