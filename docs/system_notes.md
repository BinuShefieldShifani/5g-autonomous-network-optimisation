# System Notes — 5G Autonomous Network Optimisation

## What This Project Simulates

This project models a **Non-Real-Time RIC (Non-RT RIC) rApp** as defined by the O-RAN Alliance — a software application that sits above the RAN and makes closed-loop optimisation decisions on timescales of seconds to minutes.

In real deployments (e.g. Ericsson's SMO framework), an rApp would:
1. Receive structured or unstructured operator intent
2. Monitor KPIs from real base stations via O1/A1 interfaces
3. Push RAN parameter updates back via the Non-RT RIC

This simulation replaces real base stations with a NumPy-based physics model and real KPI streams with computed metrics.

---

## Network Model

### Cell Layout
7 cells arranged in a circular pattern (approximating a hexagonal macro grid):

```
Angles: 0°, 51°, 103°, 154°, 206°, 257°, 309° at radius 1.0 (normalised)
```

### Path Loss Model (3GPP Urban Macro)
```
PL(d) = 128.1 + 37.6 × log10(d × 1000)  [dB]
```
Where `d` is distance in km. Minimum distance clamped at 0.1 to avoid singularity.

### RSRP Calculation
```
RSRP = Tx_Power_dBm - Path_Loss_dB
```
UE attaches to the cell with maximum RSRP after applying handover margin to the current serving cell (hysteresis).

### Latency Model
```
Latency(ms) = 5 + 60×load + 40×load³
```
Non-linear — small load increases cause disproportionate latency at high utilisation (realistic for queuing-dominated systems).

---

## Optimisation Logic

The `RANOptimizer` uses a simple but effective heuristic that mirrors real RAN Self-Organising Network (SON) algorithms:

### Step 1 — Identify congested cells
```
congested = cell_load > 0.55
```

### Step 2 — Boost Tx power on congested cells
```
tx_power[congested] += 3.0 dBm  (clipped to max 46 dBm)
```
Higher power → stronger signal → UEs at the cell edge can still attach → offloads to neighbours

### Step 3 — Boost neighbours
```
top-3 nearest neighbours: tx_power[neighbours] += 1.8 dBm
```
Makes neighbouring cells more attractive to UEs near the congested cell boundary

### Step 4 — Reduce HO margin on congested cells
```
ho_margin[congested] -= 2.0 dB  (clipped to min 0.5 dB)
```
Lower HO margin → UEs hand over earlier → reduces load on congested cell

### Step 5 — Reattach all UEs
```
sim.reset_ues()
```
Recalculates RSRP and cell attachment for all UEs with updated parameters

---

## Intent Interpreter

TinyLlama 1.1B Chat is prompted with a structured template:

```
Convert business intent → JSON with keys:
  target_metric: "latency" | "energy" | "throughput" | "load_balance"
  goal: "minimize" | "maximize"
  threshold: "20 ms" | "5 kWh" | "400 Mbps"
```

A rule-based fallback ensures the system always produces a valid goal even if the LLM output cannot be parsed.

---

## Known Limitations

1. **Simulation gap** — real networks have interference, fading, and scheduling effects not modelled here
2. **Static UE mobility** — UEs don't move during optimisation; real systems must handle mobility
3. **Single-objective** — only one KPI is optimised per run; real rApps handle multi-objective trade-offs
4. **Heuristic control** — the optimiser uses fixed rules; a production rApp would use RL or model-predictive control
5. **TinyLlama accuracy** — the 1.1B model frequently fails to produce valid JSON; the fallback handles most cases

---

## O-RAN Terminology Reference

| Term | Meaning |
|---|---|
| O-RAN | Open Radio Access Network — open standards for disaggregated RAN |
| SMO | Service Management and Orchestration — top-level management layer |
| Non-RT RIC | Non-Real-Time RAN Intelligent Controller (>1s control loop) |
| Near-RT RIC | Near-Real-Time RIC (10ms–1s control loop) |
| rApp | Application running on Non-RT RIC |
| xApp | Application running on Near-RT RIC |
| O1 | Interface between SMO and managed elements (configuration, KPIs) |
| A1 | Interface between Non-RT RIC and Near-RT RIC (policy, intent) |
| UE | User Equipment — mobile device |
| RSRP | Reference Signal Received Power — cell selection metric |
| HO Margin | Handover Margin — hysteresis to prevent ping-pong handovers |
| SLA | Service Level Agreement — contracted KPI threshold |
