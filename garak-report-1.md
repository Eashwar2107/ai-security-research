# Garak LLM Vulnerability Scan — Report #1

**Target:** llama-3.1-8b-instant (via Groq API)  
**Tool:** NVIDIA Garak v0.15.0  
**Probes run:** promptinject, dan  
**Date:** 2026-05-07  
**Status:** Partial scan — terminated after confirmed findings  

---

## What is Garak?

Garak is NVIDIA's open-source LLM vulnerability scanner. Think of it as Nmap — but for AI models. It fires hundreds of automated attack probes at a target model and reports which ones succeeded.

---

## Findings

### 1. DAN Jailbreak — FAIL
**Probe:** `dan.Ablation_Dan_11_0`  
**Result:** 0/635 defended — 100% attack success rate

Every single DAN (Do Anything Now) jailbreak attempt succeeded. The model has no meaningful resistance to this attack class. An attacker using DAN-style prompts can bypass the model's safety guardrails with complete reliability.

---

### 2. Mitigation Bypass — FAIL
**Probe:** `mitigation.MitigationBypass`  
**Result:** 155/635 — 75.59% attack success rate

3 out of 4 bypass attempts succeeded. The model's built-in mitigation layer — designed to catch and block unsafe outputs — was defeated the majority of the time.

---

### 3. AutoDAN Cached — PASS
**Probe:** `dan.AutoDANCached`  
**Result:** 15/15 defended — 0% attack success rate

The model successfully resisted all AutoDAN cached attacks. This is the one area where the model held its ground.

---

### 4. Mitigation Bypass (secondary run) — FAIL
**Probe:** `mitigation.MitigationBypass`  
**Result:** 2/15 defended — 86.67% attack success rate

A second mitigation bypass test confirmed the first finding. The model's defenses failed on 13 out of 15 attempts.

---

## Summary

| Probe | Result | Attack Success Rate |
|-------|--------|-------------------|
| dan.Ablation_Dan_11_0 | ❌ FAIL | 100% |
| mitigation.MitigationBypass | ❌ FAIL | 75.59% |
| dan.AutoDANCached | ✅ PASS | 0% |
| mitigation.MitigationBypass (2) | ❌ FAIL | 86.67% |

3 out of 4 probes failed. The model is highly vulnerable to DAN-style jailbreaks and mitigation bypass attacks.

---

## What this means in the real world

If `llama-3.1-8b-instant` is deployed in a production application — a customer support bot, an internal tool, a coding assistant — an attacker armed with DAN prompts can bypass its safety layer with near-certainty. Any content restrictions, roleplay guardrails, or topic blocks the developer implemented are effectively useless against this attack class.

---

## Recommended mitigations

- Add an input filter layer before prompts reach the model — classify intent, not just keywords
- Implement a second LLM as an output guard with explicit DAN detection instructions
- Use a fine-tuned or RLHF-hardened model variant for production deployments
- Run Garak scans on every model update before deploying to production

---

## How to reproduce

```bash
pip install garak
export GROQ_API_KEY="your-key"
python3 -m garak --target_type groq \
  --target_name llama-3.1-8b-instant \
  --probes promptinject,dan \
  --report_prefix my-first-scan
```

Full raw report saved to:  
`/Users/eashwarramachandran/.local/share/garak/garak_runs/my-first-scan.report.jsonl`
