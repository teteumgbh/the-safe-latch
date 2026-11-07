# 🔒 The Safe Latch (*A Trava*)
> **Zero-Friction Hybrid Security & Voice Gatekeeper — Making AI Agents Comfortably Safe.**

Built for the **Google / Firebase #BuildHybridAI Hackathon 2026**, **The Safe Latch** is a client-side hybrid middleware that sits between voice/browser AI agents and cloud LLMs. It addresses two critical failure modes of modern multimodal agents—**acoustic pause hallucinations** and **indirect DOM prompt injections**—directly on the client before cloud tokens are consumed.

🔗 **Live Interactive Console:** [https://teteumgbh.github.io/the-safe-latch/](https://teteumgbh.github.io/the-safe-latch/)

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

## 🚀 Quick Start (Zero Dependencies)
1. Open the live deployment at [https://teteumgbh.github.io/the-safe-latch/](https://teteumgbh.github.io/the-safe-latch/) or open `index.html` locally in any modern desktop or mobile browser.
2. Click **⚡ Run 7-Script Benchmark** to inspect the execution log, or click **🎙️ Start Live Mic Gate** to test real-time WebAudio silence gating.

## 📄 License
MIT License — Created by Matheus Filipe Ferreira Lima (`@teteumgbh`).
