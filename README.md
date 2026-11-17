# Quantum-Hardware Compilation Corpus and Classical GPU Scheduling Benchmarks

*Explore the portfolio: [Prometheus Quantum Overview](https://github.com/PrometheusDynamicsCanada/prometheus-quantum-overview)*

**This repository brings together quantum-hardware measurements and classical GPU benchmarks used to study execution performance.**

Each dataset in this archive measures a distinct execution workflow:

- **Dataset A - Multi-Cloud Quantum Circuit Compilation:** Compares Qiskit SABRE, Quantinuum TKET, and Prometheus on `4`-to-`23`-qubit circuits across six IBM, Rigetti, and IQM quantum processors. Prometheus produced closer agreement with ideal circuit outputs than SABRE in **`80/107` (`74.8%`)** IBM Heron jobs (`+26.31%` mean / `+50.00%` median on `69` dense all-to-all circuits) and **`9/12` (`75.0%`)** Amazon Braket tasks, with all baseline wins retained in full.
- **Dataset B - Classical Problem Compression & NVIDIA H100 Scheduling:** Demonstrates exact QUBO variable compression (`72` down to `4` binary variables with `0.0%` optimality gap) alongside classical `VeloxQ` NVIDIA H100 runs solving industrial schedules up to **`1,029` binary variables in `5.26 seconds`**.

*Comparison boundary:* These two datasets solve different problems - gate-model quantum compilation versus classical scheduling optimization - and **do not represent a head-to-head quantum-versus-GPU advantage comparison** (classical GPU runs are never counted toward quantum job or shot totals).

**[Detailed results](DETAILED_TECHNICAL_REPORT.md) · [Verify the evidence](#verify-the-evidence) · [Discuss a pilot](#discuss-a-pilot)**

![Multi-Cloud Quantum Hardware Corpus vs. Separate Classical H100 Suite](assets/fig1_hybrid_qpu_and_h100_summary.png)
*Opening summary: Left - Three-way quantum compilation win rates across `107` IBM Heron jobs, the `4`-job validation suite, and `12` Amazon Braket tasks. Right - Separate classical QUBO variable compression and NVIDIA H100 scheduling study (`4` to `1,029` binary variables).*

---

## 1. Executive Summary & Principal Comparative Corpus Table


> [!NOTE]
> **Metric & Methodological Conventions:**
> - **Classical Distribution Overlap (F_B, ↑ higher is better) vs. Squared Hellinger Fidelity (F_H = F_B², ↑ higher is better):** In [`ibm_heron_156q_verified_benchmarks.json`](./ibm_heron_156q_verified_benchmarks.json) and [`aws_braket_qpu_12_jobs_ledger.json`](./aws_braket_qpu_12_jobs_ledger.json), `fidelity` denotes F_B = Σ_x √(P_ideal(x) · P_meas(x)). In [`validation_suite_144_executions/`](./validation_suite_144_executions/), `squared_hellinger_fidelity` denotes F_H = F_B².
> - **Total Variation Distance (TVD, ↓ lower is better):** 0.5 · Σ_x |P_ideal(x) - P_meas(x)|. **Linear XEB (↑):** 2ⁿ · Σ_x P_ideal(x) · P_meas(x) - 1.
> - **19Q K_19 Heavy-Output Probability (HOP):** Achieving 70.45% HOP (> 2/3) on deterministic K_19 establishes heavy-output survival on that circuit; it is distinct from formal Quantum Volume (QV) certification.
> - **Complete Retention:** All 119 completed 3-way QPU jobs (107 IBM Heron + 12 AWS Braket) are retained without cherry-picking, including all 27/107 IBM Heron and 3/12 AWS Braket cases where **SABRE O3** achieved higher F_B.

### Table 1: Corpus Overview & Reconciled Execution Accounting

| Benchmark Pillar | Hardware / Environment | Reconciled Scale & Scope | Key Empirical Outcome |
| :--- | :--- | :--- | :--- |
| **Amazon Braket Live QPU Corpus** | Rigetti `Cepheus-1-108Q` (108Q), IQM `Garnet` (20Q), IQM `Emerald` (54Q) | **12 Completed 3-Way `ProgramSet` Tasks** (36 circuits, 119,808 shots) across 4Q-10Q Dense All-to-All, QFT, and QAOA MaxCut ([`aws_braket_qpu_12_jobs_ledger.json`](./aws_braket_qpu_12_jobs_ledger.json)) | **12/12 (100.0%)** lower depth & 2Q CZ gates vs. SABRE O3 & TKET; **9/12 (75.0%)** F_B wins vs. SABRE O3, led by **+379.42% (4.79x)** on Cepheus 9Q Dense, **+116.13% (2.16x)** on Cepheus 6Q Dense, **+14.92%** (-52.6% TVD) on Emerald 6Q QFT, and **+9.39%** (+43.1% XEB) on Garnet 9Q Dense |
| **Pre-Committed 144-Execution Suite** | `ibm_marrakesh`, `ibm_fez`, `ibm_kingston` (156Q Heron) | **4 Packed `SamplerV2` Jobs** (144 circuits, 147,456 shots, 48.0s QPU usage) across 6 families at 6Q, 9Q, 12Q ([`validation_suite_144_executions/`](./validation_suite_144_executions/)) | **16/18 (88.9%)** wins vs. SABRE O3 and **15/18 (83.3%)** vs. TKET on `ibm_marrakesh`; **6/6 (100.0%)** wins at 12Q; pre-committed dispatch (`C:ibm_kingston`) won **6/6 (100.0%)** vs. Fixed A/B and **5/6 (83.3%)** vs. Fixed C |
| **156Q Gate-QPU Compilation Corpus** | `ibm_marrakesh`, `ibm_fez`, `ibm_kingston` (156Q Heron) | **107 Completed 3-Way Jobs** (321 circuits, 4,964,352 completed shots) across 18 qubit scales (4Q-23Q) *(plus 3 documented unrecovered queue dispatches)* | Beat **TKET on 100/107 (93.5%)** and **SABRE O3 on 80/107 (74.8%)** completed hardware triplets; up to **-54.5%** 2Q gate reduction vs. TKET and **-34.8%** vs. SABRE O3 |
| **19Q Dense All-to-All (K_19) Survival** | `ibm_marrakesh`, `ibm_fez`, `ibm_kingston` (156Q Heron) | **19-Qubit Dense All-to-All (K_19)** (171 logical all-to-all pairs, 16,384 shots/compiler) | Achieved **24.26%** (`ibm_kingston`), **12.78%** (`ibm_marrakesh`), and **6.54%** (`ibm_fez`) F_B vs. 18.98%, 6.27%, and 3.57% for SABRE O3; **70.45% HOP** on `ibm_kingston` (vs. 63.28% SABRE O3 and 54.38% TKET) |
| **Hybrid Quantum-HPC QUBO Compression** | VeloxQ NVIDIA H100 (`VeloxQH100_1`) + 156Q IBM Heron | **42 Benchmark Configurations** (150 multi-seed H100 runs) scaling from 4 to 1,029 QUBO binary variables ([`veloxq_h100_hybrid_benchmarks.json`](./veloxq_h100_hybrid_benchmarks.json)) | **94.44% variable compression** (72 to 4 binary variables, 0.0% gap) executed on 4Q IBM Heron (94.04%-96.34% F_B); **10.0x-34.0x** H100 ground-state hit boost; **1,029-variable (15x3)** schedule solved to **26h** horizon (0.0% horizon gap; 25h pure-C_max MILP LB) in **5.26s** |
| **Multi-Architecture Braket Verification** | Amazon Braket SDK (`LocalSimulator`): Rigetti Cepheus (108Q) & IQM Garnet (20Q) | **7 Logical Workloads Verified at 4,096 Shots + 14 Archived 3-Compiler Structural Benchmarks** (8Q-19Q × SABRE O3, TKET, Prometheus) | **7/7 (100.0%)** logical equivalence (1.000000) on `LocalSimulator`; **14/14 (100.0%)** higher modeled execution fidelity vs. SABRE O3 & TKET; **14/14 (100.0%)** lower depth & 2Q gates vs. TKET (**12/14** vs. SABRE O3) |

---

## Verify the Evidence

**Verification Kind:** Deterministic **outcome-ledger consistency & classical schedule-feasibility audit** (`verify_corpus_and_schedules.py`), **local Braket circuit-equivalence check** (`validate_braket_local.py`), and **raw bitstring-payload scoring** on the shared `144`-execution suite:

```bash
python verify_corpus_and_schedules.py
python validation_suite_144_executions/score_counts.py validation_suite_144_executions/results_hardware.json
```

- **Key files:** [`DETAILED_TECHNICAL_REPORT.md`](DETAILED_TECHNICAL_REPORT.md) · [`ibm_heron_156q_verified_benchmarks.json`](ibm_heron_156q_verified_benchmarks.json) · [`aws_braket_qpu_12_jobs_ledger.json`](aws_braket_qpu_12_jobs_ledger.json) · [`veloxq_h100_hybrid_benchmarks.json`](veloxq_h100_hybrid_benchmarks.json) · [`PUBLIC_DISCLOSURE.md`](PUBLIC_DISCLOSURE.md)

---

## Discuss a Pilot

**Bring a representative workload and an acceptance criterion. We can compare Prometheus with your existing execution approach and measure quality, acceptance rate and charged hardware resources.**

- **Contact:** [william@prometheusdynamics.ca](mailto:william@prometheusdynamics.ca) | [prometheusdynamics.ca](https://prometheusdynamics.ca)
