# Traffic Control Using Model Predictive Control

**Current Thesis Research | EGH400**  
**Supervisor: Dr. Guilherme Fróes Silva**

[Back to Portfolio](../)

---

## The Problem

Traffic kills people. Not just in dramatic crashes—though those happen too—but in the slow accumulation of wasted time, burned fuel, stress-induced health problems, and the thousands of fender-benders caused by stop-and-go congestion.

The root cause? **Traffic shockwaves**: those phantom jams where everyone slams on their brakes for no visible reason, creating a ripple effect that propagates backward through traffic for miles. One driver brakes slightly too hard, the car behind overreacts, the next one even more so, and suddenly you have a traffic jam that exists purely because traffic exists.

<img src="/assets/img/sumo_nocontrol.png" alt="Traffic shockwave visualization without control" style="width:80%;">

*Spatio-temporal density map showing shockwave formation at a highway merge (red = high density, blue = low density)*

This isn't just annoying—it's measurably dangerous. Studies show shockwaves directly increase crash risk, fuel consumption, and emissions. And the problem is getting worse as highways get more congested.

**The research gap**: We know Variable Speed Limit (VSL) signs can help. We know Model Predictive Control (MPC)—a technique that predicts future traffic states rather than just reacting—shows promise. But nobody has systematically figured out which MPC cost function actually works best for reducing shockwaves in real-world scenarios.

That's what this thesis is trying to answer.

---

## The Approach

The challenge with traffic control research: you can't just close down a highway and experiment with different control strategies. That's expensive, dangerous, and politically impossible.

Solution: **Simulate it**. But not just any simulation—you need two models working together:

1. **SUMO (microscopic simulator)**: Simulates every individual vehicle with realistic driver behaviors, lane changes, acceleration patterns. This is the "ground truth"—what would actually happen on a real highway.

2. **LWR model (macroscopic predictor)**: A simplified mathematical model that treats traffic like a fluid flowing through pipes. Fast enough to run hundreds of predictions per second inside the MPC controller.

The architecture looks like this:
- SUMO runs the actual traffic simulation (4000+ vehicles/hour merging from 2 lanes into 1)
- Every 30 timesteps, the MPC controller wakes up
- MPC uses the LWR model to predict: "If I change these speed limits, what happens to traffic density in the next 6 minutes?"
- MPC tries dozens of speed limit combinations, picks the one that minimizes shockwaves
- SUMO applies those speed limits, drivers react, MPC repeats

The hard part isn't the concept—it's getting all the pieces to actually work together.

---

## Building the LWR Model

The Lighthill-Whitham-Richards (LWR) model is a first-order partial differential equation that describes traffic flow as a conservation problem. In plain English: the number of cars on a road segment can only change if cars flow in or out.

**Core equation**:
```
∂ρ/∂t + ∂q(ρ)/∂x = 0
```

Where ρ is density (cars per meter) and q(ρ) is flow (cars passing per second). The flow depends on density through a "fundamental diagram"—at low density, cars flow freely; at high density, flow drops because cars are packed too close to move.

**Implementation challenge**: This continuous equation has to be discretized—broken into road segments and time steps—to run on a computer. Used the Godunov flux method, which calculates how many cars flow across each boundary between segments.

### Calibration Problem

Initial attempt: Set LWR inflow to match SUMO's 4000 veh/h.

<img src="/assets/img/lwr_4000inflow.png" alt="LWR model with low inflow" style="width:80%;">

*LWR prediction with 4000 veh/h inflow—produces stationary shockwave, doesn't match SUMO*

Result: Stationary shockwave at the merge point. Traffic builds up but never propagates downstream. Doesn't match SUMO's behavior at all.

**The fix**: Realized SUMO drivers react to congestion differently than LWR's mathematical model predicts. Had to increase LWR's inflow to 6200 veh/h to produce equivalent congestion patterns.

<img src="/assets/img/lwr_6200inflow.png" alt="LWR model with calibrated inflow" style="width:80%;">

*LWR with 6200 veh/h inflow—shockwave now propagates downstream, matching SUMO's pattern*

Now the shockwave forms at the merge and propagates backward through traffic, just like in SUMO. The leading edge timing and position match reasonably well.

**Limitation discovered**: LWR can't simulate rarefaction waves (traffic clearing after a jam ends) because it calculates velocity purely from density. Once a shockwave forms under constant inflow, LWR has no mechanism to clear it. This is a fundamental limitation of first-order models, but it's acceptable for MPC because the controller updates frequently with real SUMO data, preventing the model from drifting too far from reality.

---

## Building the MPC Controller

MPC works by repeatedly solving an optimization problem: "What sequence of speed limits minimizes my cost function over the next prediction horizon?"

**The prediction loop**:
1. Get current traffic density from SUMO (every road segment)
2. Linearize the LWR model around current state
3. Use OSQP solver to find optimal speed limits that minimize cost over 12-step horizon (6 minutes)
4. Apply first control action to SUMO
5. Let SUMO run for 30 timesteps
6. Repeat

**Cost function (simplified version)**:
```
J = Σ [α(Δv)² + (ρ - ρ_target)²]
```

Penalizes two things:
- **Density deviation**: Keeps traffic at optimal flow density (60% of jam density)
- **Speed limit changes**: Prevents wild oscillations that confuse drivers (weighted by α)

**Key constraints**:
- Speed limits: 60-110 km/h (legal limits)
- Rate of change: Max 20 km/h change between control steps
- Density bounds: 0 to jam density (physical limits)

**Technical challenge**: OSQP occasionally failed to converge—couldn't find an optimal solution in time. When this happened, kept previous speed limits and tried again next cycle. Occurred ~2-5% of control steps. Not ideal, but acceptable for proof-of-concept.

---

## Results

### Baseline: No Control

<img src="/assets/img/sumo_nocontrol.png" alt="SUMO simulation without control" style="width:80%;">

*Uncontrolled traffic—massive shockwave propagates ~1500m upstream over 1000 seconds*

The shockwave starts at the merge point (~500m) around timestep 750, then propagates backward as a dense diagonal band. Peak density hits 0.35 (35% of maximum). This is what happens when you let drivers react naturally to a bottleneck.

### Naive Control (Reactive)

<img src="/assets/img/old_simple_control.png" alt="Simple reactive control" style="width:80%;">

*Reactive control reduces shockwave somewhat but doesn't prevent formation*

A simple reactive controller (lowers speed limits when density exceeds threshold) helps—the shockwave is less intense—but still propagates upstream. The problem with reactive control: by the time you detect congestion, it's already too late to prevent the shockwave.

### MPC Control (Predictive)

<img src="/assets/img/mpc_best.png" alt="MPC with optimal parameters" style="width:80%;">

*MPC with optimized parameters—main shockwave eliminated, only minor density fluctuations remain*

**Game changer**. The major upstream-propagating shockwave is completely eliminated. There's a small stationary queue at the highway entrance (left edge, red band) that forms around timestep 600, but it doesn't propagate. The rest of the highway shows only minor density fluctuations—traffic flows smoothly.

**Parameter tuning matters**:
- Prediction horizon: 12 steps (optimal)
- α (smoothing weight): 0.1
- Target density: 60% of jam density
- Control frequency: Every 30 timesteps

### What Happens If You Get It Wrong

<img src="/assets/img/h_16.png" alt="MPC with too-long horizon" style="width:45%; display:inline-block;">

*Left: Horizon too short (8 steps)—can't see far enough ahead to prevent shockwave formation*

*Right: Horizon too long (16 steps)—predictions become inaccurate, controller makes suboptimal decisions*

The literature predicted this: too short means the controller is myopic (can't anticipate problems), too long means predictions drift from reality (garbage in, garbage out). The optimal horizon is where you can still predict accurately but far enough to actually prevent problems.

---

## What This Actually Means

**Quantitative improvements** (preliminary—full analysis pending):
- Shockwave propagation: Eliminated vs. 1500m backward propagation
- Peak density: Reduced from 0.35 to <0.15 in main highway sections
- Traffic smoothness: Density variance significantly lower across all segments

**Real-world translation**: Fewer hard braking events means fewer rear-end collisions. Smoother traffic flow means lower fuel consumption and emissions. Reduced congestion means people get home faster.

**The cost function question**: This thesis used a simple cost function (density deviation + speed limit smoothing). Does adding safety-specific terms (like penalizing density gradients or velocity variance) work better? That's the next phase—systematically test different cost function formulations and measure their impact on both congestion and safety metrics.

---

## Technical Implementation

**Software Stack**
- Python (main control logic and analysis)
- SUMO (microscopic traffic simulation)
- TraCI API (SUMO ↔ Python interface)
- NumPy (numerical computing for LWR)
- OSQP (quadratic programming solver for MPC)
- Matplotlib (density heatmap visualization)

**LWR Model Implementation**
- Godunov flux method for numerical stability
- CFL condition satisfied: Δt/Δx · max|dq/dρ| < 1
- Greenshields fundamental diagram: q(ρ) = ρ · vf · (1 - ρ/ρj)
- Finite difference linearization for MPC integration
- Real-time calibration: LWR state updated from SUMO every control step

**MPC Implementation**
- Quadratic cost function in standard form: xᵀQx + uᵀRu
- State normalization: All variables scaled to [0,1] for numerical stability
- Linearized dynamics: ρₖ₊₁ = Aρₖ + Bvₖ (A and B computed via finite differences)
- Constraints: Box constraints on speed limits and densities, rate-of-change limits
- Control frequency: 30 SUMO timesteps = ~30 seconds simulation time

**Computational Performance**
- LWR prediction: ~0.01s per horizon step
- MPC optimization: ~0.1-0.5s per control cycle (12-step horizon)
- SUMO simulation: Real-time (4000 veh/h inflow, 2500m highway, 1000 timesteps)
- Total simulation time: ~10-15 minutes for full scenario

---

## Challenges & Lessons

**Code complexity**: Building LWR and MPC from scratch rather than using pre-built libraries was initially overwhelming. The temptation to directly copy others' implementations was strong. Lesson: Step back, understand the math first, then code incrementally. Test each component in isolation before integration.

**Model limitations**: LWR's inability to simulate rarefaction waves was frustrating—kept trying to "fix" it before realizing it's a fundamental first-order model limitation. Lesson: Understand your tools' theoretical constraints before blaming implementation bugs.

**Convergence issues**: OSQP occasionally failing to converge was a persistent annoyance. Tried adjusting solver tolerances, warm-starting, different constraint formulations. Lesson: For proof-of-concept, occasional failures are acceptable; for deployment, would need a more robust solver or fallback strategy.

**Parameter sensitivity**: Small changes in prediction horizon, target density, or α dramatically affected results. Spent weeks manually tuning parameters. Lesson: Need systematic parameter search methodology (grid search, Bayesian optimization) rather than manual tweaking.

---

## Next Steps

**Immediate (Thesis Completion)**:
- Implement higher-order macroscopic model (METANET or CTM) to capture rarefaction waves
- Test alternative cost function formulations:
  - Safety-focused: Penalize density gradients and velocity variance
  - Efficiency-focused: Minimize total time spent
  - Hybrid: Multi-objective optimization with different weightings
- Add realistic driver non-compliance factor (70-90% compliance rather than 100%)
- Quantify improvements: Total Time Spent (TTS), crash risk metrics, fuel consumption estimates

**Future Directions**:
- Test on different scenarios: Lane drop, on-ramp metering, weather-related capacity drops
- Incorporate connected vehicle data (if available) for better state estimation
- Multi-agent MPC: Coordinate control across multiple highway segments
- Real-world validation: Compare simulation predictions to actual VSL deployment data

---

## Why This Matters

Traffic congestion costs the US economy $166 billion annually in lost time and wasted fuel. Traffic crashes kill ~40,000 Americans per year. These aren't abstract problems—they're daily realities for millions of people.

MPC for traffic control isn't science fiction—Variable Speed Limit signs already exist on most major highways. The infrastructure is there. The question isn't "can we do this?" but "how do we do it optimally?"

This research is one small step toward answering that question: which cost function formulation actually reduces shockwaves most effectively? Get that right, and you could save lives, hours, and fuel across every highway that implements it.

For autonomous systems and intelligent infrastructure, the lesson is broader: predictive control beats reactive control, but only if you understand your model's limitations and tune your objectives correctly. The difference between "works in simulation" and "works in the real world" is often just a matter of getting those details right.


[Back to Portfolio](../)
