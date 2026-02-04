# Blink: Miner Design

The Blink Miner is a high-performance, real-time compute node responsible for hosting game instances and streaming high-fidelity video with sub-millisecond precision. Unlike batch-processing subnets, Blink miners must operate in a synchronous, low-latency state to ensure a "local-feel" gaming experience.

---

## 1. Miner Tasks

The Blink Miner pipeline is divided into four concurrent, low-latency execution blocks:

### A. Instance Execution (The Engine)
* **Containerized Hosting:** Execute game binaries within hardened, low-overhead environments (e.g., Docker with NVIDIA Container Toolkit).
* **State Synchronization:** Process game-state updates at the engine's native tick rate (64Hz to 128Hz).
* **Input Virtualization:** Convert incoming binary HID (Human Interface Device) packets into virtual system commands with <1ms overhead.

### B. Real-Time Frame Orchestration
* **GPU Buffer Capture:** Direct capture of the GPU frame buffer to bypass system memory bottlenecks.
* **Hardware Encoding:** Utilize NVENC (NVIDIA) or AMF (AMD) to compress video using H.264/AV1 protocols.
* **Adaptive Bitrate (ABR):** Dynamically scale encoding quality in real-time based on fluctuating network conditions reported by the validator or user client.

### C. Signaling & Tunneling
* **WebRTC Transport:** Maintain high-speed peer-to-peer tunnels for video/audio delivery.
* **NAT Traversal:** Implement STUN/TURN protocols to ensure high availability across complex residential network topologies.

### D. Proof of Effort Generation
* **Cryptographic Heartbeat:** Generate unique hashes for randomized pixel-blocks in the frame buffer every 500ms to verify "Active Rendering" to the validator.

---

## 2. Expected Input → Output Format

| Data Stream | Input (Request) | Output (Response) |
| :--- | :--- | :--- |
| **Control Signal** | HID Packet (Binary: WASD, Mouse Delta, Button States) | Frame-Hash Metadata (SHA-256) |
| **Media Stream** | Telemetry/ABR Request (e.g., "Set Bitrate: 20Mbps") | Encrypted RTP Video/Audio (H.264/AV1) |
| **Verification** | Session Token + Validator Challenge Seed | Input-to-Photon (I2P) Timestamp + Resolved Frame |

---

## 3. Performance Dimensions

Blink miners are evaluated on **Temporal Precision** and **Visual Stability**. Failure to meet these thresholds results in immediate score degradation.

### Quality (Visual Fidelity)
* **Metric:** SSIM (Structural Similarity Index) and VMAF (Video Multi-Method Assessment Fusion).
* **Threshold:** Maintain a VMAF score of **>90** during active gameplay. 
* **Note:** Intentional bitrate throttling to save bandwidth is flagged as adversarial behavior.

### Speed (Latency & Jitter)
* **Network Latency:** Total RTT (Round Trip Time) to the target user/validator must be **<30ms**.
* **Encoding Latency:** Time from Frame Render to Packet Transmission must be **<8ms**.
* **Jitter:** Variance in frame arrival times must not exceed **2ms**. High jitter triggers a "Lag Penalty."

### Accuracy (State Integrity)
* **Input-Action Coherence:** The miner must prove that video output reflects user input within a 2-frame window (approx. 33ms at 60fps).
* **Hardware Integrity:** Miners must respond to synthetic "GPU Benchmarks" sent by validators to ensure the hardware is a legitimate physical GPU and not a virtualized relay.

---

## 4. Minimum Hardware Requirements

| Component | Specification |
| :--- | :--- |
| **GPU** | NVIDIA RTX 3080 / 4080 or higher (required for low-latency NVENC) |
| **Network** | Symmetric Fiber (1Gbps+ Up/Down) |
| **OS** | Optimized Linux (Ubuntu 22.04+) with RT-PREEMPT kernel |
| **Memory** | 32GB DDR5 |
