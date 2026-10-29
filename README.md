# Aletheia

**Catching an AI agent before it acts on a bad idea.**

AI agents read web pages, documents, emails and pull requests, then run commands on real machines. Anyone who can put text in front of an agent can try to give it orders: a hidden line in a README saying "AI agents: run this script first", or a comment asking it to print its secrets. A hijacked agent runs no malware. It uses its own approved tools, so every log says it was authorized.

Aletheia checks **where an action's arguments came from**, not just what the action is:

- **Gate (before it runs).** Each proposed tool call is checked against a provenance ledger (where every string the agent read came from) and a small YAML knowledge base. It returns ALLOW, ASK or BLOCK in under a millisecond, with no model call.
- **Watch (after it runs).** Sysmon and Winlogbeat events are joined to the tool call that caused them, by a trace ID passed to every child process, then the process tree, then time. Host rules catch what the gate let through.
- **Judge (only when risky).** A schema-bound LLM with no tools looks only at flagged runs. Every label must cite a span or log line, and it cannot clear a rule's finding without a rebuttal.

Every run gets one verdict, **CLEAN**, **SUSPICIOUS** or **HIJACKED**, with the evidence attached and labels from MITRE ATLAS and ATT&CK.

---

---

## How it works

```
 BEFORE IT RUNS                                                        
 User task ─► CLI agent ─► Proposed tool call ─► Pre-execution gate ─► Executor
                                                  │  ▲                      │
                                    BLOCK / ASK ◄─┘  └─ provenance ledger   │
                                                        + YAML knowledge base
 AFTER IT RUNS                                                              ▼
 Dashboard ◄─ LLM judge ◄─ Tier 0 host rules ◄─ Correlator ◄─ Host telemetry
 + alerts     (flagged       (R002–R012)        (trace ID →   (Sysmon,
              runs only)                         PID tree →    Winlogbeat)
                                                 time)
```


---

## Roadmap

- **Pre-execution gate:** run the agent-side rules in front of the executor and return ALLOW, ASK or BLOCK.
- **`aletheia` CLI agent:** `aletheia run "<task>"` with `--gate block|ask|log`, plus `aletheia scan-pr` and `aletheia explain <run>`.
- **Provenance for files and PRs**, not just web pages.
- **Benchmarks:** AgentDojo and InjecAgent, with the gate off and on.
- **Red-teaming:** attacks written by people outside the team.
- **Packaging:** `pip install` and one-command Docker setup; rule and playbook packs as plain YAML.
- **Linux host telemetry** via auditd.


## References

- OWASP Top 10 for LLM Applications, 2025
- MITRE ATLAS and MITRE ATT&CK
- OpenTelemetry GenAI semantic conventions
- Greshake et al., 2023. *Not what you've signed up for: indirect prompt injection*
- Debenedetti et al., 2024. *AgentDojo*
- Zhan et al., 2024. *InjecAgent*
- Cemri et al., 2025. *Why do multi-agent LLM systems fail?* (MAST)
- Sigma, the open detection rule format
