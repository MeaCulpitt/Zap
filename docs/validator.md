# Zap: Validator Architecture & Audit Logic

The Zap validator is a high-frequency auditor that ensures miner performance claims are backed by physical reality. Unlike subnets where output can be checked offline, Zap requires real-time verification — validators must confirm that miners are actually rendering frames, actually located where they claim, and actually delivering the quality they report.

---

## The Verification Challenge

Cloud gaming verification is harder than batch compute verification:

| Challenge | Why It's Hard |
|-----------|---------------|
| **Location claims** | Miners are incentivized to claim proximity they don't have |
| **Rendering claims** | Pre-recorded or relayed streams could fake output |
| **Quality claims** | Miners could report inflated metrics |
| **Availability claims** | Miners could claim uptime while idle |

Zap addresses each through a multi-layer verification stack.

---

## The Verification Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                    VERIFICATION STACK                            │
├─────────────────────────────────────────────────────────────────┤
│  Layer 1: AVAILABILITY (Continuous)                             │
│           Heartbeat pings every 10 seconds                      │
│           Session completion tracking                           │
├─────────────────────────────────────────────────────────────────┤
│  Layer 2: LOCATION (Speed-of-Light)                             │
│           RTT triangulation from watchtower nodes               │
│           Physical distance verification                        │
├─────────────────────────────────────────────────────────────────┤
│  Layer 3: RENDERING (Input-to-Photon)                           │
│           Synthetic gameplay challenges                         │
│           Visual response verification                          │
├─────────────────────────────────────────────────────────────────┤
│  Layer 4: QUALITY (Fidelity Analysis)                           │
│           VMAF/SSIM frame sampling                              │
│           Jitter measurement                                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Layer 1: Availability Verification

**Purpose:** Confirm miners are online and completing sessions.

**Mechanism:**
- Validators ping miners every 10 seconds
- Response required within 100ms
- Session start/end events logged
- Dropped sessions flagged

**Metrics:**

| Metric | Calculation |
|--------|-------------|
| Uptime | (Successful pings / Total pings) × 100% |
| Session completion | (Completed sessions / Started sessions) × 100% |
| Challenge response | (Answered challenges / Issued challenges) × 100% |

**Scoring weight:** 20% of availability component

---

## Layer 2: Speed-of-Light (SoL) Location Verification

**Purpose:** Confirm miners are physically located where they claim.

**The Problem:**
A miner in Virginia could claim to be in São Paulo to capture the Scarcity Bounty. Without verification, geographic incentives become meaningless.

**The Solution:**
Light through fiber travels at approximately 200,000 km/s. Round-trip time imposes a physical floor that cannot be faked.

```
Minimum RTT = (2 × Distance) / Speed of Light in Fiber
            = (2 × Distance) / 200,000 km/s

Example: São Paulo to Miami = 7,500 km
         Minimum RTT = 15,000 km / 200,000 km/s = 75ms
```

**Triangulation:**
Validators operate "watchtower" nodes in known locations. By measuring RTT from multiple watchtowers, they triangulate the miner's actual position.

```
Watchtower A (Miami): RTT = 80ms → Max distance = 8,000km
Watchtower B (Santiago): RTT = 12ms → Max distance = 1,200km  
Watchtower C (Buenos Aires): RTT = 18ms → Max distance = 1,800km

Intersection: São Paulo region ✓
```

**Spoofing Detection:**
If a miner's RTT to watchtowers violates physical constraints, they are flagged:

```python
def verify_location(miner_claimed_location, watchtower_rtts):
    for watchtower, rtt in watchtower_rtts.items():
        max_distance = (rtt / 2) * SPEED_OF_LIGHT_FIBER
        actual_distance = distance(watchtower.location, miner_claimed_location)
        
        if actual_distance > max_distance * 1.1:  # 10% tolerance
            return FLAG_GEOGRAPHIC_SPOOF
    
    return VERIFIED
```

**Scoring weight:** 30% of final score (via latency multiplier)

---

## Layer 3: Input-to-Photon (I2P) Rendering Verification

**Purpose:** Confirm miners are rendering in real-time, not relaying pre-recorded content.

**The Problem:**
A miner could theoretically:
- Pre-render common gameplay sequences
- Relay streams from a remote GPU farm
- Use AI upscaling on low-quality source

**The Solution:**
Validators send unpredictable input sequences and verify the visual response.

**Challenge Structure:**

```json
{
  "challenge_id": "a3f7c2e1",
  "timestamp": 1707408000000,
  "seed": "0x7a3f...",
  "input_sequence": [
    {"t": 0, "action": "move_left"},
    {"t": 100, "action": "jump"},
    {"t": 200, "action": "look_up", "delta_y": -45},
    {"t": 350, "action": "fire"}
  ],
  "expected_response_window_ms": 33
}
```

**Verification Process:**

1. Validator sends signed challenge with input sequence
2. Miner executes inputs in game instance
3. Miner returns rendered frames with timestamps
4. Validator checks:
   - Frame shows correct visual response to each input
   - Response occurs within 33ms window (2 frames at 60fps)
   - Frame hashes are consistent with claimed rendering

**Why Pre-Recording Fails:**
- Input sequences are generated from random seeds
- Combinatorial explosion makes pre-rendering infeasible
- Timing requirements prevent buffering

**Challenge Frequency:** ~15 audits per hour per miner (randomized intervals)

**Scoring weight:** 40% of verification confidence

---

## Layer 4: Quality (Fidelity Analysis)

**Purpose:** Confirm visual quality meets claimed standards.

**Metrics:**

| Metric | What It Measures | Target |
|--------|------------------|--------|
| VMAF | Perceptual video quality (0-100) | >90 |
| SSIM | Structural similarity (0-1) | >0.95 |
| Jitter | Frame arrival variance | <2ms |

**VMAF Sampling:**
During I2P challenges, validators capture frames and compute VMAF against reference renders. Miners cannot predict which frames will be sampled.

**Jitter Measurement:**
Validators track inter-frame arrival times:

```python
frame_times = [t1, t2, t3, t4, ...]
intervals = [t2-t1, t3-t2, t4-t3, ...]
jitter = std_dev(intervals)

if jitter > 2ms:
    quality_penalty = 2ms / jitter  # Linear degradation
```

**Scoring weight:** 30% of quality component

---

## Composite Scoring

Validators compute a final score for each miner:

```
S = Q × L × A

Where:
  Q = Quality score (VMAF/SSIM weighted average, 0.0-1.0)
  L = Latency multiplier (based on RTT tier, 0.1-2.0)
  A = Availability score (uptime × completion × response, 0.0-1.0)
```

**Example Calculation:**

| Miner | VMAF | RTT | Uptime | Completion | Response | Score |
|-------|------|-----|--------|------------|----------|-------|
| A (São Paulo) | 94 | 12ms | 99.8% | 99.5% | 100% | 0.94 × 2.0 × 0.993 = **1.87** |
| B (Miami) | 96 | 45ms | 99.9% | 99.8% | 100% | 0.96 × 1.0 × 0.997 = **0.96** |
| C (Virginia) | 98 | 120ms | 100% | 100% | 100% | 0.98 × 0.1 × 1.0 = **0.10** |

Miner A earns ~19x more than Miner C despite lower quality, because proximity matters.

---

## Evaluation Cadence

Zap operates on three temporal scales:

### Real-Time (Continuous)
- Availability pings every 10 seconds
- Session event logging
- Immediate flags for dropped connections

### Synthetic Audits (Random, ~15/hour)
- I2P challenges with input sequences
- VMAF frame sampling
- Jitter measurement
- SoL triangulation refresh

### Epoch Commitment (360 blocks, ~1 hour)
- Aggregate all metrics into weight matrix
- Normalize scores across miner set
- Commit weights to Subtensor
- Yuma Consensus determines final emissions

---

## Validator Incentive Alignment

### Consensus-Driven Dividends

Validators earn TAO by matching the stake-weighted median of all validator scores. Deviating from consensus reduces Trust and emissions.

```
If validator_score differs from median by >10%:
    Trust penalty applied
    Emissions reduced proportionally
```

**Why This Works:**
- Honest validators converge on similar scores (same physical measurements)
- Colluding validators can't shift consensus without majority stake
- Lazy validators (random scores) are mathematically detectable

### Alpha-Stake Accountability

Validators must hold the subnet's Alpha token. Their wealth is tied to network utility:
- Good validation → healthy network → Alpha value increases
- Poor validation → network degrades → Alpha value drops

### Discovery Rewards

Validators earn bonuses for:
- Onboarding miners in underserved regions
- First verification of new geographic zones
- Identifying and flagging cheaters

---

## Anti-Collusion Mechanisms

**Problem:** Validators could collude with miners to inflate scores.

**Defense:** Physical metrics are deterministic.

| Metric | Collusion Difficulty |
|--------|---------------------|
| RTT | Cannot fake physics; watchtower triangulation catches lies |
| VMAF | Reference renders are deterministic; inflated scores detectable |
| Jitter | Timestamp variance is measurable by any validator |
| I2P | Input-output relationship is verifiable by any party |

A colluding validator would produce scores inconsistent with physics. Other validators would score the same miner differently, and Yuma Consensus would penalize the outlier.

---

## Validator Infrastructure

### Minimum Requirements

| Component | Requirement |
|-----------|-------------|
| CPU | 8-core, 3.0GHz+ |
| RAM | 32GB |
| Network | 1Gbps, low latency |
| Storage | 500GB SSD |

### Watchtower Network

Validators benefit from operating watchtower nodes in multiple regions:
- Improves SoL triangulation accuracy
- Enables independent RTT verification
- Increases challenge coverage

Watchtower locations should be:
- Geographically distributed
- On low-latency backbone connections
- In regions with high miner density

---
