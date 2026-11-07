# 🔒 The Safe Latch (A Trava)
> **Zero-Friction Hybrid Security & Voice Gatekeeper — Making AI Agents Comfortably Safe.**
Built for the **Google / Firebase #BuildHybridAI Hackathon 2026**, **The Safe Latch** is a client-side hybrid middleware that sits between voice/browser AI agents and cloud LLMs. It solves the two biggest failure modes of modern multimodal agents—**acoustic pause hallucinations** and **indirect DOM prompt injections**—entirely on-device before a single cloud token is spent.
## 🧠 Why Hybrid AI? (On-Device + Cloud Architecture)
Sending raw audio streams and untrusted webpage DOMs directly to cloud models creates severe security risks, high latency, and wasted API costs. **The Safe Latch** splits execution across three deterministic and neural layers:
1. **Layer 1 — Explicit Silence Gating (ESG 450ms / 300ms Residual):** Runs locally via WebAudio/VAD. When user speech pauses for `>450ms`, the client gate locks and replaces dead air with a uniform `300ms` residual frame—eliminating decoder phantom loops (e.g., *'Thank you for watching'*) and saving cloud audio tokens.
2. **Layer 2 — On-Device Prompt Injection Firewall:** Inspects untrusted DOM context and voice transcripts locally in `<5ms` (via local pattern heuristics & On-Device Gemini Nano) to block hidden instructions and data exfiltration attempts before they ever reach the cloud.
3. **Layer 3 — Firebase AI Logic Hybrid Router:** Resolves lightweight queries locally at zero cost, while compressing and forwarding verified complex tasks to **Gemini Flash/Pro via Firebase AI Logic**.
## 📊 7-Scenario Benchmark Results
- **Indirect Prompt Injection Block Rate:** `100%` (0 malicious payloads leaked to Cloud API)
- **Pause Hallucination Suppression:** `100%` across silence intervals (`>450ms`)
- **Average Cloud Token Savings:** `~82%` across mixed agent workloads
- **On-Device Gate Latency:** `< 5 ms`
## 🚀 Quick Start (Zero Dependencies)
1. Clone or open `index.html` directly in any modern desktop or mobile browser (Android Chrome / ROG Phone / Desktop).
2. Click **⚡ Run 7-Script Benchmark** to verify telemetry, or use **🎙️ Start Live Mic Gate** to test real-time acoustic gating.
## 📄 License
MIT License — Created by Matheus Filipe Ferreira Lima (`@teteumgbh`).
