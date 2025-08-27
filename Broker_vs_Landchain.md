# Control Plane vs. LangChain

> **LangChain is an application framework; the Front-End MCP Broker + AAP EDA is an operations/control-plane.**  
> They solve different problems and can complement each other.

---

## TL;DR

- **Use the Front-End MCP Broker + AAP EDA** when you need **enterprise messaging guarantees**:
  - durable queues  
  - per-key ordering  
  - deadlines & cancel  
  - idempotency & DLQs  
  - sagas/compensation  
  - multi-tenant RBAC & audit  

- **Use LangChain** to build **LLM apps/pipelines** (RAG, tools, agents) that *consume* or *produce* messages.

In practice, you often need both: LangChain runs *inside* adapters/workers, while the broker enforces cross-system guarantees.

---

## Where LangChain Shines

- **LLM application dev**: chains, agents, tools, RAG, evaluators, memory.
- **Developer ergonomics**: fast prototypes, batteries-included SDKs.
- **Tracing/eval**: LangSmith and related tooling for app teams.

---

## What LangChain Does *Not* Provide

The broker layer adds **control-plane semantics** that LangChain alone cannot guarantee:

- **Durable command bus** with per-key ordering (e.g. `provision → configure → enable`).
- **End-to-end SLA controls**: deadlines, cancel, backpressure, priorities.
- **Exactly-once/idempotency** transport semantics.
- **Sagas/compensation** across heterogeneous estates (CLI, HTTP, MCP servers, SAP, MQ, etc.).
- **Enterprise RBAC & audit** (who did what, when).
- **Ops scale**: AAP Controllers/EEs, change windows, credential governance.
- **Observability at platform layer**: OTel traces, SLO dashboards, DLQ depth.

---

## Why This Matters in the Enterprise

- **Reliability > clever prompts**: Workflows must be ordered, durable, compensatable.
- **Least privilege & audit**: Centralized policy and immutable event logs.
- **Heterogeneous backends**: AAP already automates mainframes, SAP, MQ, firewalled zones.
- **Org structure**: Platform teams own the bus/broker; feature teams own adapters.

---

## Interop Pattern (Best of Both)

1. **Keep MCP ↔ Broker** at the edge for LLM clients (ChatGPT, Copilot).
2. **EDA rulebooks → AAP job templates** route work.
3. **Inside adapters/workers**, call LangChain chains/agents (RAG, summarizers, evaluators).
4. **Report progress/results** back to broker for SSE streaming and Kafka audit.

### Example Adapter Using LangChain

```python
# adapter.py (runs in AAP EE)
from my_langchain_app import summarize_ticket_chain

def run(ticket_id, correlation_id, broker_url):
    for pct, msg in summarize_ticket_chain.iter_progress(ticket_id):
        post(f"{broker_url}/{correlation_id}/progress",
             json={"percent": pct, "msg": msg})
    result = summarize_ticket_chain.run(ticket_id)
    post(f"{broker_url}/{correlation_id}",
         json={"status":"ok", "result": result})
```

---

## Decision Checklist

**Use LangChain alone when:**
- Single-team app.
- Best-effort reliability acceptable.
- Minimal RBAC/audit needs.

**Use MCP Broker + AAP EDA when:**
- Need durability, ordering, deadlines, idempotency, DLQ.
- Multi-step sagas & compensation.
- Multi-tenant RBAC, compliance, regulated environments.
- Air-gapped / hybrid deployments.

---

## Bottom Line

- **LangChain helps you build smart tasks.**  
- **The Front-End MCP Broker makes those tasks safe, ordered, auditable, and operable at scale.**  
- Use both: LangChain for app-level logic, Broker for control-plane guarantees.
