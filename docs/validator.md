# Zap: Validator Design

The Zap Validator acts as a high-frequency auditor, ensuring that the network's "Edge" claims are backed by physical reality and high-performance hardware.

---

## 1. Scoring and Evaluation Methodology

Validators rank miners based on a composite score derived from three primary verification layers:

### A. Speed-of-Light (SoL) Distance Check (30%)
* **Mechanism:** Round-trip-time (RTT) triangulation.
* **Verification:** Validators use a global network of "watchtower" nodes to ensure a miner's response time matches the physical limits of fiber-optic travel from their claimed IP location.

### B. Input-to-Photon (I2P) Audit (40%)
* **Mechanism:** Synthetic User Simulation.
* **Verification:** The validator sends a sequence of game inputs and verifies that the miner's return video stream reflects these changes within a 33ms window (2-frame delay at 60fps). 

### C. Visual Fidelity & Jitter Analysis (30%)
* **Fidelity:** Automated sampling of frames to calculate VMAF (Video Multi-Method Assessment Fusion) scores.
* **Jitter:** Precise measurement of inter-frame arrival variance. Anything >2ms is penalized as "Stream Stutter."

---

## 2. Evaluation Cadence

Zap's validation operates on a tiered temporal structure:

* **Real-Time (10s):** Latency and heartbeat pings to verify continuous availability.
* **Synthetic Audits (Randomly, 15x/hour):** Short, high-intensity "Ghost Sessions" to perform I2P and VMAF checks.
* **Blockchain Epoch (360 Blocks):** Aggregation of performance metrics and weight commitments to the Subtensor blockchain.

---

## 3. Validator Incentive Alignment

Zap follows the 2026 Bittensor Yuma Consensus (YC) model, reinforced by dTAO market mechanics:

* **Consensus-Driven Dividends:** Validators maximize their TAO earnings by accurately ranking miners. Deviating from the global consensus (e.g., scoring a slow miner highly) results in a "Trust Penalty," reducing validator rewards.
* **Alpha-Stake Accountability:** Validators must hold the subnet's native Alpha token. This ensures that their long-term wealth is tied to the actual utility and user adoption of the Zap network.
* **Anti-Collusion Logic:** Because scoring is based on deterministic physical metrics (Latency/VMAF), any validator providing outlier scores is mathematically identified and excluded from consensus, protecting the network from "bribery" or "lazy validation."
