# Secure End-to-End Data Transmission with Quantum Key Distribution (BB84 + Shor 9-Qubit QEC)

> A research-grade simulation that demonstrates how to **securely distribute files** by combining the **BB84 quantum key distribution** protocol with a lightweight **XOR symmetric cipher**, while **correcting 1-qubit channel errors** using **Shor’s 9-qubit code**.

---

## 📌 What’s in this repo

- A full **BB84** simulation (basis selection, sifting, sampling, and eavesdropper detection)
- A simple **quantum channel** that injects random single-qubit **X/Y/Z errors**
- **Shor’s 9-qubit error-correction** to recover corrupted qubits (handles X, Z, and thus Y up to a global phase)
- **XOR** symmetric encryption for payload transfer over an insecure classical channel
- **Reproducible experiments** and summary metrics (reconciliation rates, precision/recall of eavesdropper detection, CRC-64 file checks)

> Implementation notes: Python with **Qiskit** (statevector simulator). Experiments used binary payloads from text & image files; verification via **CRC-64** equality of sent/received files.
---

## 🧭 Background (2-minute tour)

- **BB84**: Alice encodes bits on random bases (Z: \|0⟩/\|1⟩; X: \|+⟩/\|−⟩). Bob measures on random bases. They publicly compare bases to **sift** a shared key and **sample** a subset to detect eavesdropping. A single eavesdropper induces ~25% disagreement in sampled bits.  
- **Quantum channel errors**: Environmental interactions can be decomposed into **Pauli errors** (X, Y, Z).
- **Shor’s 9-qubit code**: Encodes a logical qubit into 9 physical qubits; uses 3-qubit repetition for X-error correction, phase kickbacks + Hadamards to convert Z→X, thus correcting **X, Z (and Y up to a global phase)**.

---

## 🛠️ Requirements

- Python 3.13
- Recommended: a fresh virtual environment (Conda or venv)
- Python packages: *see requirements.txt*

## 🚀 Quickstart

```bash
# 1) create & activate an environment (example using conda)
conda create -n qbb84 python=3.13 -y
conda activate qbb84

# 2) install dependencies
pip install -r requirements.txt

# 3) run a demo
python main.py
```

## 🧪 Reproducing the paper’s experiments
**The study evaluated three files of different sizes/types and verified end-to-end integrity using CRC-64:**

- @ logo.png — 163,456 bits

- long_text.txt — 47,936 bits

- short_text.txt — 152 bits
- All received CRC-64 values matched the originals. 

**Core measurements (expected vs observed):**

- Basis agreement: ~50% (theoretical) → 49.993% (observed)

- Key agreement (no Eve, pre-sift): ~75% → 74.956%

- Key agreement (with Eve, pre-sift): ~62.5% → 62.506%

- Sampled key agreement (no Eve, post-sift): 100%

- Sampled key agreement (with Eve, post-sift): ~75% → 74.945%

**All within 95% confidence intervals.**

**Eavesdropper detection:** In runs with randomly injected Eve (p=0.5), the detector achieved 100% precision and 100% recall under the simulation settings.

**QEC sanity checks:** Shor’s code correctly recovered X and Z errors, and Y errors up to a global phase (non-observable).

## 📈 Interpreting results

- Sifted key length ≈ 50% of raw key bits (random, independent bases).

- With Eve, sampled-bit agreement drops to ~75%, revealing interception.

- Without Eve, sampled-bit agreement should be ~100% (ideal simulator).

- CRC-64(original) == CRC-64(received) → end-to-end correctness.

- QEC logs show where X/Z syndromes were triggered and corrected; Y is handled via X+Z (global phase ignored).

## 🔬 How it works (high level)

1. **Key distribution (BB84):** Alice encodes random bits on mixed bases → Bob measures on mixed bases → public basis comparison → sift key → sample subset to detect Eve.

2. **Channel model:** Random single-qubit Pauli errors (X/Y/Z) on transmitted qubits.

3. **Error correction (Shor 9-qubit):**

    - Repetition code for X errors on 3-qubit blocks

    - Hadamards to convert Z→X, correct again

    - Corrects X, Z, and Y (up to a global phase)

4. **Payload encryption:** Use sifted key (post-sampling) with XOR to encrypt the file; send ciphertext over the classical channel; decrypt on Bob’s side.

## Limitations
- Idealized simulator: real hardware introduces loss/noise; BB84 typically tolerates a small QBER (quantum bit error rate) rather than strict 100% agreement.

- Single-qubit error model: Demonstrates principles; multi-qubit or correlated noise is out of scope.