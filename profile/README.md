# 🔄 Reconcile‑Loop Toolkit

> **Declarative, self‑healing control‑plane built from two lightweight services.**

&nbsp;

<p align="center">
  <img src="https://img.shields.io/badge/license-Apache%202.0-blue" alt="License" />
</p>

## 📚 Table of Contents

- [Why Reconcile?](#-why-reconcile)
- [Architecture](#-architecture)
- [Repositories](#-repositories)
  - [`state-manager`](#state-manager)
  - [`controlloop`](#controlloop)
- [Contributing](#-contributing)
- [License](#-license)

---

## ❓ Why Reconcile?

Modern platforms thrive on **declarative resource definitions** and **automatic drift correction**.  
The _reconciliation‑loop_ pattern continuously drives the _actual_ system state toward the _desired_ state you declare, yielding:

- **Self‑healing** 🔧 — detects & corrects drift automatically.
- **Predictability** ✅ — idempotent operations with built‑in retry/back‑off.
- **Extensibility** 🧩 — add new resource types by writing plug‑in reconcilers.

---

## 🏗️ Architecture

```
┌──────────────┐  CRUD -- REST   ┌────────────────┐
│  Clients ✅   │ ──────────────▶ │ state-manager │◄────────────┐
│ (   API )    │                 │   (Postgres)  │              │
└──────────────┘                 └──────┬────────┘              │
                                       │ write-through cache    │
                                       ▼                        │
                                  ┌────────┐                    │
                                  │ Redis  │                    │
                                  └──┬─────┘                    │
                                     │ pub/sub events           │
                                     ▼                          │
                             ┌────────────────┐                 │
                             │  controlloop   │                 │
                             │   (operator)   │                 │
                             └──────┬─────────┘                 │
                                    │ reconcile status          │
                                    └───────────────────────────┘
```

1. **Clients** create or update resource specs in **state‑manager**.
2. **controlloop** streams change events into its work‑queue.
3. Each item is processed until system reality matches the spec; status is persisted back.

---

## 📦 Repositories

### `state-manager` 💾

High‑performance, durable store for all resource specs **and** status.

| Feature | Details |
|---------|---------|
| Tech Stack | PostgreSQL (strong consistency) + Redis (sub‑ms cache) |
| APIs | REST with CRUD|
| Extras | Event stream (Redis `STREAMS`), schema migrations, optimistic locking |

➡️ **Repo:** [`state-manager`](https://github.com/reconcile-kit/state-manager)

### `controlloop` ⚙️

Operator/agent that implements the reconcile loop using the state stored in **state‑manager**.

| Feature | Details |
|---------|---------|
| Work Queue | Rate‑limited, exponential back‑off, idempotent handlers |
| Extensibility | Plug‑in reconcilers per resource kind (Go interfaces) |


➡️ **Repo:** [`controlloop`](https://github.com/reconcile-kit/controlloop)

---

## 🤝 Contributing

PRs are welcome! Please read [`CONTRIBUTING.md`](./CONTRIBUTING.md) to get started.

1. Fork 🍴 & clone the repo.
2. Create a feature branch.
3. Commit with conventional commits.
4. Open a pull request.

---

## 📄 License

Distributed under the **Apache 2.0** license. See [`LICENSE`](./LICENSE) for full text.
