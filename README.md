# 📡 5G Intent-Driven Autonomous Network Optimisation

> Multi-agent simulation of an **Ericsson-style O-RAN SMO/rApp** — translates plain English business intent into closed-loop RAN parameter optimisation using LLM-based intent interpretation and autonomous KPI-driven control.

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![TinyLlama](https://img.shields.io/badge/LLM-TinyLlama_1.1B-purple)
![LangChain](https://img.shields.io/badge/LangChain-Community-green)
![Colab](https://img.shields.io/badge/Environment-Google_Colab-F9AB00?logo=googlecolab)
![Domain](https://img.shields.io/badge/Domain-5G_O--RAN-red)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

---

## 📌 Overview

Modern 5G networks are expected to self-manage — detecting congestion, interpreting operator intent, and autonomously adjusting radio parameters without human intervention. This is the core idea behind **O-RAN's Zero-Touch Network** and Ericsson's **SMO/rApp** framework.

This project simulates that exact pipeline in Google Colab:

1. An operator states a business intent in plain English
2. A **TinyLlama 1.1B LLM** (via LangChain) interprets it into a structured optimisation goal
3. A **7-cell 5G network simulator** (pure NumPy, 3GPP-standard path loss model) generates realistic KPIs
4. A **TrafficAgent** simulates a downtown peak-hour congestion event
5. A **KPIMonitor** detects SLA violations in real time
6. A **RANOptimizer** autonomously adjusts transmit power and handover margins to resolve the breach
7. Full visualisation of the closed-loop optimisation cycle

---

## 🎯 Key Features

- **Intent interpretation** — TinyLlama converts natural language to structured JSON goals; rule-based fallback ensures robustness
- **Realistic network simulation** — 3GPP Urban Macro path loss model (128.1 + 37.6·log10(d) dB), RSRP-based cell attachment, handover hysteresis
- **Closed-loop optimisation** — autonomous Tx power boosting and HO margin reduction on congested cells and their neighbours
- **SLA monitoring** — worst-cell latency tracked against 20ms threshold per O-RAN SLA definitions
- **Full KPI dashboard** — latency curves, cell load, UE distribution map, energy, throughput, SLA status

---

## 🏗️ System Architecture

```
Business Intent (plain English)
         │
         ▼
┌─────────────────────────────────┐
│   Intent Interpreter Agent      │
│   TinyLlama 1.1B (LangChain)    │
│   → target_metric               │
│   → goal (minimize/maximize)    │
│   → threshold (e.g. "20 ms")    │
│   Fallback: rule-based parser   │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│   NetworkSimulator              │
│   • 7 cells (hexagonal layout)  │
│   • 350–450 UEs                 │
│   • 3GPP path loss model        │
│   • RSRP-based cell attachment  │
│   • HO hysteresis margin        │
└──────┬───────────────┬──────────┘
       │               │
       ▼               ▼
┌────────────┐  ┌──────────────────┐
│  Traffic   │  │   KPI Monitor    │
│  Agent     │  │                  │
│            │  │ • Worst-cell     │
│ • Uniform  │  │   latency        │
│   traffic  │  │ • SLA violation  │
│ • Hotspot  │  │   detection      │
│   (peak    │  │ • History        │
│   hour)    │  │   tracking       │
└────────────┘  └────────┬─────────┘
                         │
                    SLA violated?
                         │
                         ▼
               ┌─────────────────────┐
               │   RAN Optimizer     │
               │                     │
               │ • Boost Tx power    │
               │   on congested      │
               │   cells + neighbours│
               │ • Reduce HO margin  │
               │   → offload UEs     │
               │ • Repeat until SLA  │
               │   restored or max   │
               │   iterations reached│
               └─────────────────────┘
```

---

## 📊 Results

### New Approach — Optimised Pipeline

![New Approach Result](results/new_approach_result.png)

- Worst-cell latency starts at **~17.3ms** and drops to **~15.8ms** after optimisation
- Max cell load reduced from **0.200 → 0.174** — users successfully offloaded to neighbouring cells
- SLA threshold (20ms) maintained throughout
- UE distribution map shows balanced load across all 7 cells post-optimisation

### Original Approach — Full Closed-Loop Cycle

![Old Approach Result](results/old_approach_result.png)

- **Baseline:** latency well below 20ms SLA (uniform traffic)
- **Hotspot:** latency spikes to **~71ms** — SLA violated immediately
- **Optimisation loop:** RANOptimizer triggers, reduces worst latency to ~63ms over 3 iterations
- Demonstrates the complete closed-loop cycle: **problem detected → agent triggered → partial recovery**
- The SLA breach not fully resolved in 3 iterations reflects realistic network behaviour — real O-RAN optimisers require multiple cycles and retraining to fully converge

---

## 🤖 Agent Descriptions

| Agent | Role | Technology |
|---|---|---|
| `IntentInterpreter` | Parses business intent → structured JSON goal | TinyLlama 1.1B via LangChain HuggingFacePipeline |
| `NetworkSimulator` | Simulates 7-cell 5G network with realistic KPIs | NumPy, 3GPP path loss model |
| `TrafficAgent` | Injects traffic hotspot (peak hour simulation) | NumPy random UE placement |
| `KPIMonitor` | Tracks KPIs and detects SLA violations | Pure Python, history tracking |
| `RANOptimizer` | Autonomously adjusts Tx power and HO margins | Rule-based closed-loop control |

---

## 📁 Repository Structure

```
5g-network-optimisation/
├── notebook/
│   └── 5g_autonomous_network.ipynb   # Full self-contained pipeline (run this)
├── results/
│   ├── new_approach_result.png        # Optimised pipeline output
│   └── old_approach_result.png        # Full closed-loop cycle output
├── docs/
│   └── system_notes.md                # Technical background and design notes
├── requirements.txt                   # Dependency reference
└── README.md
```

---

## 🚀 Getting Started

**This project runs entirely on Google Colab. No local setup or API keys required.**

### Step 1 — Open the notebook

Open `notebook/5g_autonomous_network.ipynb` in [Google Colab](https://colab.research.google.com). CPU runtime is sufficient — T4 GPU speeds up TinyLlama loading.

### Step 2 — Run Cell 1 (Installation)

Installs `numpy`, `matplotlib`, `seaborn`, `pandas`, `langchain-community`, `sentence-transformers` automatically.

### Step 3 — Run Cells 2–7 (Class Definitions)

Defines all agent classes: `NetworkSimulator`, `TrafficAgent`, `KPIMonitor`, `RANOptimizer`, and the `plot()` visualisation function. No output expected — just definitions.

### Step 4 — Run Cell 8 (Main Execution)

Runs the full end-to-end pipeline:
- Baseline KPIs (uniform traffic)
- Peak-hour hotspot injection
- Closed-loop optimisation (up to 5 iterations)
- Full KPI dashboard visualisation

> ⏱ TinyLlama takes 20–40s to load on first run.

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| LLM | TinyLlama 1.1B Chat (HuggingFace) |
| LLM Framework | LangChain Community (`HuggingFacePipeline`) |
| Network Simulation | Pure NumPy — 3GPP Urban Macro path loss |
| Visualisation | Matplotlib, Seaborn |
| Environment | Google Colab (CPU or T4 GPU) |
| No API keys | 100% free, fully local |

---

## 📡 Telecom Background

This simulation is inspired by **O-RAN Alliance** specifications and Ericsson's **SMO (Service Management and Orchestration)** framework:

| Concept | This Project |
|---|---|
| rApp (Non-RT RIC) | `RANOptimizer` — closed-loop RAN control |
| Intent-Based Networking | `IntentInterpreter` — NL → structured goal |
| KPI Monitoring | `KPIMonitor` — SLA tracking |
| UE (User Equipment) | Simulated mobile devices |
| RSRP | Received Signal Reference Power — cell selection metric |
| HO Margin | Handover margin — controls cell offloading aggressiveness |
| SLA | Service Level Agreement — 20ms worst-cell latency target |

---

## ⚠️ Limitations & Honest Notes

- **Simulation only** — no real 5G hardware, no actual O-RAN data
- The notebook contains two implementations (original and improved) — run the **New Approach** cells for the cleaner result
- TinyLlama is primarily for demonstration — the rule-based fallback handles most intent parsing in practice
- The optimiser uses simple heuristics; a real rApp would use RL or model-based control

---

## 🔭 Future Work

- [ ] Replace heuristic optimiser with Reinforcement Learning (PPO/SAC)
- [ ] Add xApp layer (Near-RT RIC) for faster control loops
- [ ] Connect to real O-RAN dataset (e.g. OpenRAN Gym)
- [ ] Upgrade intent interpreter to Llama 3.2 for better parsing accuracy
- [ ] Add energy optimisation goal alongside latency
- [ ] Multi-intent handling (latency AND energy simultaneously)

---

## 👤 Author

**Binu Shefield Shifani**
Software Engineer (5 years, Cognizant Technology Solutions)
MS AI & Automation · University West, Trollhättan, Sweden

[![GitHub](https://img.shields.io/badge/GitHub-BinuShefieldShifani-black?logo=github)](https://github.com/BinuShefieldShifani)

---

## 📄 License

MIT License — free to use for research and personal projects.
