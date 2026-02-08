# Zap: Incentive & Mechanism Design

Zap's incentive structure commoditizes low-latency compute by rewarding miners who provide high-fidelity gaming sessions at the physical edge of the network. Unlike batch-processing subnets, Zap requires synchronous, real-time performance — every frame matters.

---

## Worked Example: The Edge Gaming Pipeline

### Scenario: Competitive Player in São Paulo

**Setup:**
1. Player in São Paulo wants to play a competitive shooter
2. Nearest centralized cloud gaming server is in Virginia (120ms RTT)
3. Local Zap miner is 8km away in the same city (12ms RTT)

**Session Flow:**

1. **Connection:** Player requests session via Zap client. Network routes to nearest available miner based on latency profile.

2. **Gameplay:** Miner runs game instance on RTX 4080, captures frames directly from GPU buffer, encodes via NVENC (H.264/AV1), streams via WebRTC.

3. **Verification:** Validator issues synthetic input challenge mid-session. Miner must render the correct visual response within 33ms (2 frames at 60fps).

4. **Scoring:**
   - RTT: 12ms → Latency multiplier: 1.5x (sub-15ms tier)
   - VMAF: 94 → Quality score: 0.94
   - Jitter: 1.2ms → No penalty
   - Uptime: 99.8% → Availability: 0.998

5. **Final Score:** 0.94 × 1.5 × 0.998 = **1.41**

**Comparison:** A miner in Virginia serving the same player would score:
   - RTT: 120ms → Latency multiplier: 0.3x
   - Same quality metrics
   - Final Score: 0.94 × 0.3 × 0.998 = **0.28**

**Result:** The São Paulo miner earns 5x more TAO for serving the same player. The network naturally incentivizes geographic proximity.

---

## Emission and Reward Logic

Zap utilizes the Bittensor dTAO emission schedule:

* **Incentive Distribution:** 41% to Miners, 41% to Validators, 18% to Subnet Owner
* **Epoch Length:** 360 blocks (~1 hour)

### The Scoring Formula

```
S = Q × L × A

Where:
  Q = Quality score (0.0 - 1.0)
  L = Latency multiplier (0.1 - 2.0)
  A = Availability score (0.0 - 1.0)
```

### Quality Score (Q)

Measured via automated visual fidelity benchmarks:

| Metric | Weight | Threshold |
|--------|--------|-----------|
| VMAF (Video Multi-Method Assessment Fusion) | 70% | >90 for Tier 1 |
| SSIM (Structural Similarity Index) | 30% | >0.95 for Tier 1 |

Quality is sampled during synthetic audit sessions. Miners cannot predict which frames will be evaluated.

### Latency Multiplier (L)

Round-trip time determines the multiplier:

| RTT | Multiplier | Tier |
|-----|------------|------|
| <15ms | 2.0x | Edge |
| 15-30ms | 1.5x | Regional |
| 30-50ms | 1.0x | Standard |
| 50-100ms | 0.5x | Degraded |
| >100ms | 0.1x | Disqualified |

The multiplier creates strong economic pressure toward geographic distribution. A miner cannot compete by being far away.

### Availability Score (A)

| Metric | Weight |
|--------|--------|
| Uptime (responding to pings) | 50% |
| Session completion rate | 30% |
| Challenge response rate | 20% |

Miners who drop sessions or fail to respond to validator challenges see their availability score decay.

---

## Scarcity Bounties

To bootstrap geographic coverage, Zap implements regional multipliers:

| Region Status | Bounty Multiplier |
|---------------|-------------------|
| Zero-Node (first miner in urban area) | 2.5x |
| Low-Density (<3 miners per 100km²) | 1.5x |
| Standard Density | 1.0x |
| High-Density (>10 miners per 100km²) | 0.8x |

Scarcity Bounties decay as more miners join a region. The network self-balances toward optimal geographic distribution.

---

## Incentive Alignment

### For Miners

Miners maximize earnings by:
1. **Geographic positioning** — Being close to players (latency multiplier)
2. **Hardware quality** — High-end GPUs for better VMAF scores
3. **Reliability** — Consistent uptime and session completion
4. **Regional scarcity** — First-mover advantage in underserved areas

Consumer-grade hardware (RTX 4080+) is sufficient. No industrial infrastructure required.

### For Validators

Validators maximize earnings by:
1. **Accurate scoring** — Matching consensus with other validators
2. **Challenge coverage** — Running sufficient audits to catch underperformers
3. **Geographic diversity** — Operating watchtower nodes for Speed-of-Light verification

Under Yuma Consensus, validators who deviate from the stake-weighted median lose Trust and see reduced emissions.

---

## Mechanisms to Discourage Adversarial Behavior

### Speed-of-Light (SoL) Checks

**Problem:** Miner claims to be in São Paulo but is actually in Virginia.

**Solution:** Validators triangulate RTT from multiple watchtower nodes. If a miner's response times violate the physical limits of light through fiber, they are flagged and slashed.

```
Max Distance = RTT × (Speed of Light in Fiber) / 2
             = 12ms × (200,000 km/s) / 2
             = 1,200 km radius

If miner claims São Paulo but RTT suggests >5,000km, flag as geographic spoof.
```

### Input-to-Photon (I2P) Audits

**Problem:** Miner relays pre-recorded video instead of rendering in real-time.

**Solution:** Validators send unpredictable input sequences (synthetic gameplay). The miner must return video showing the correct visual response within 33ms. Pre-recorded content cannot pass.

```
Challenge: Move character left, jump, look up
Expected: Frame shows character mid-jump, facing up
Tolerance: 2 frames (33ms at 60fps)
```

### Jitter Penalties

**Problem:** Miner over-provisions hardware, causing inconsistent frame delivery.

**Solution:** Frame arrival times are measured. Variance >2ms triggers a Jitter Penalty that reduces the quality score.

```python
jitter = std_dev(frame_arrival_times)
if jitter > 2ms:
    Q = Q × (2ms / jitter)  # Linear penalty
```

### Cryptographic Heartbeats

**Problem:** Miner claims to be rendering but is idle.

**Solution:** Every 500ms, miners generate SHA-256 hashes of randomized pixel blocks from the frame buffer. Validators can request proof of any historical heartbeat. Missing heartbeats indicate idle hardware.

---

## Proof of Useful Effort

Zap qualifies as a **Proof of Useful Effort (PoUE)** subnet:

| Requirement | Implementation |
|-------------|----------------|
| **Real work** | GPU rendering at 60fps is computationally intensive and immediately useful |
| **Verifiable** | Visual output can be checked against input challenges |
| **Non-fakeable** | I2P audits prevent pre-rendering; SoL checks prevent location spoofing |
| **Economically aligned** | Only real, nearby, high-quality compute earns meaningful rewards |

The "intelligence" in Zap is the **Dynamic Orchestration Layer** — miners must intelligently manage:
- Resource allocation across concurrent sessions
- Network congestion and adaptive bitrate
- Thermal throttling to maintain stable performance
- Geographic positioning for optimal coverage

---

## High-Level Algorithm

```
┌─────────────────────────────────────────────────────────────────┐
│  1. ASSIGNMENT     Validator issues synthetic game challenge    │
│                    (input sequence + expected visual outcome)   │
├─────────────────────────────────────────────────────────────────┤
│  2. EXECUTION      Miner renders scene, streams via WebRTC     │
│                    Returns frame hashes + I2P timestamp         │
├─────────────────────────────────────────────────────────────────┤
│  3. VERIFICATION   Validator checks:                            │
│                    - Visual response matches input (I2P)        │
│                    - RTT matches claimed location (SoL)         │
│                    - VMAF/SSIM meet quality thresholds          │
│                    - Jitter within tolerance                    │
├─────────────────────────────────────────────────────────────────┤
│  4. SCORING        Validator computes S = Q × L × A             │
│                    Aggregates into weight matrix                │
├─────────────────────────────────────────────────────────────────┤
│  5. CONSENSUS      Weights committed to Subtensor               │
│                    Emissions distributed via Yuma Consensus     │
└─────────────────────────────────────────────────────────────────┘
```
