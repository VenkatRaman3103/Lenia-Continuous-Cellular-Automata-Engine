# Delta Functions in Lenia

## Overview
Delta functions (or growth functions) determine how cells grow, decay, or change based on their neighborhood sum. They are the crucial "rules" that transform kernel convolution results into field updates.

## Fundamental Role

### Growth/Decay Determination
Delta functions map neighborhood sums to growth rates:
```
growth = d(neighborhood_sum)
```

Where:
- **Input**: Value from kernel convolution (typically 0-1)
- **Output**: Growth rate (typically -1 to +1)
- **Interpretation**: 
  - Positive → Cell growth/increase
  - Zero → Stable state
  - Negative → Cell decay/decrease

### Field Update Integration
The delta function output is integrated over time:
```
f(x, t+1) = f(x, t) + d(k * f(x, t)) · Δt
```

## Core Delta Function Types

### 1. Gaussian Delta
The most commonly used delta function:

```
d(n) = 2e^(-ℓ²/k) - 1 where ℓ = |n-μ|, k = 2σ²
```

Properties:
- **Range**: (-1, +1)
- **Peak**: d(μ) = +1
- **Minimum**: Approaches -1 as distance increases
- **Smooth**: Infinitely differentiable

Parameters:
- **μ (mu)**: Center/peak location (0.1-0.9)
- **σ (sigma)**: Width/standard deviation (0.01-0.5)

JavaScript implementation:
```javascript
function gaussianDelta(n, mu, sigma) {
  const l = Math.abs(n - mu);
  const k = 2 * sigma * sigma;
  return 2 * Math.exp(-l * l / k) - 1;
}
```

### 2. Polynomial Delta
Algebraic alternative:

```
d(n) = 2(1 - ℓ²/k²)^α - 1 if ℓ ≤ k, otherwise -1
```

Where:
- **ℓ = |n-μ|**: Distance from center
- **k**: Width parameter
- **α**: Stiffness parameter (2-16)

Properties:
- **Compact support**: Exactly -1 beyond width
- **Polynomial**: Easier to analyze mathematically
- **Tunable**: α controls shape

```javascript
function polynomialDelta(n, mu, sigma, alpha) {
  const l = Math.abs(n - mu);
  const k = 2 * sigma;
  
  if (l <= k) {
    return 2 * Math.pow(1 - (l * l) / (k * k), alpha) - 1;
  } else {
    return -1;
  }
}
```

### 3. Trapezoidal Delta
Piecewise linear function:

```
d(n) = {
  +1,                    |n-μ| ≤ σ₁
  linear decay,      σ₁ < |n-μ| ≤ σ₂
  -1,                    |n-μ| > σ₂
}
```

Properties:
- **Simple**: Easy to implement and understand
- **Fast**: Computationally efficient
- **Discontinuous**: Non-smooth at boundaries

### 4. Step Delta
For discrete automata compatibility:

```
d(n) = {
  +1,  |n-μ| ≤ σ
  -1,  |n-μ| > σ
}
```

Used for:
- Conway's Game of Life approximation
- SmoothLife step functions
- Binary cellular automata

## Mathematical Analysis

### Key Properties

#### Fixed Points
Values where growth = 0:
```
d(n*) = 0 → n* = equilibrium points
```

For Gaussian delta:
```
d(n*) = 0 → 2e^(-ℓ²/k) - 1 = 0 → ℓ = σ√(ln 2)
```

#### Maximum Growth
Peak value and location:
```
d_max = d(μ) = +1
n_optimal = μ
```

#### Sensitivity
Derivative determines sensitivity:
```
d'(n) = derivative with respect to n
```

For Gaussian:
```
d'(n) = -4(n-μ)e^(-ℓ²/k) / k
```

### Stability Analysis
Linear stability around equilibrium:
```
f ≈ f* + d'(n*)·(k*f-n*)·Δt
```

Stability condition:
```
|1 + d'(n*)·k'(f*)·Δt| < 1
```

## Parameter Effects

### Center Parameter (μ)
- **Range**: 0.1 to 0.9
- **Effect**: Sets optimal neighborhood density
- **Low μ**: Sparse environments favored
- **High μ**: Dense environments favored
- **Critical**: Too low → extinction, too high → explosion

### Width Parameter (σ)
- **Range**: 0.01 to 0.5
- **Effect**: Controls tolerance range
- **Small σ**: Narrow tolerance, sensitive to changes
- **Large σ**: Broad tolerance, robust to variations
- **Trade-off**: Stability vs. responsiveness

### Stiffness Parameter (α)
- **Range**: 2 to 16
- **Effect**: Controls transition sharpness
- **High α**: Abrupt transitions (like step function)
- **Low α**: Smooth, gradual transitions
- **Impact**: Affects pattern complexity and stability

## Behavioral Classification

### Growth Regimes

#### Growth Zone
```
d(n) > 0 → |n-μ| < threshold
```
Cells increase in this range.

#### Decay Zone
```
d(n) < 0 → |n-μ| > threshold
```
Cells decrease in this range.

#### Equilibrium Zone
```
d(n) = 0 → |n-μ| = threshold
```
Cells remain stable at these densities.

### Pattern Formation
Delta function shape determines:

1. **Single peak**: Simple growth/decay dynamics
2. **Multiple peaks**: Complex multi-stable states
3. **Asymmetric**: Directional growth preferences
4. **Flat regions**: Plateaus of stable growth

## Implementation Details

### Numerical Stability

#### Clamping
```javascript
function safeDelta(n, mu, sigma, alpha) {
  n = Math.max(0, Math.min(1, n)); // Clamp input
  let result = gaussianDelta(n, mu, sigma);
  return Math.max(-1, Math.min(1, result)); // Clamp output
}
```

#### Smoothing
Add small epsilon to prevent division by zero:
```javascript
function smoothDelta(n, mu, sigma, alpha) {
  const l = Math.abs(n - mu);
  const k = Math.max(2 * sigma * sigma, 1e-10);
  return 2 * Math.exp(-l * l / k) - 1;
}
```

#### Vectorization
For efficient batch processing:
```javascript
function batchDelta(values, mu, sigma, alpha) {
  return values.map(n => gaussianDelta(n, mu, sigma));
}
```

### Optimization Techniques

#### Lookup Tables
Pre-compute delta values:
```javascript
function createLookupTable(mu, sigma, resolution = 1000) {
  const table = new Array(resolution);
  for (let i = 0; i < resolution; i++) {
    const n = i / (resolution - 1);
    table[i] = gaussianDelta(n, mu, sigma);
  }
  return table;
}

function lookupDelta(n, table) {
  const index = Math.round(n * (table.length - 1));
  return table[index];
}
```

#### Approximation
Use cheaper approximations for real-time:
```javascript
function fastDeltaApprox(n, mu, sigma) {
  const x = (n - mu) / sigma;
  if (Math.abs(x) < 2) {
    return 1 - 0.25 * x * x; // Quadratic approximation
  } else {
    return -1;
  }
}
```

## Practice Problems

### Beginner Level
1. **Basic Implementation**
   - Implement all four delta function types
   - Create visualizations of function shapes
   - Test parameter effects on function behavior

2. **Parameter Exploration**
   - Systematically vary μ and σ
   - Document behavior changes
   - Find optimal ranges for stability

3. **Fixed Point Analysis**
   - Find equilibrium points analytically
   - Verify numerically
   - Test stability through simulation

### Intermediate Level
1. **Stability Analysis**
   - Compute derivatives analytically
   - Analyze linear stability
   - Create stability diagrams

2. **Multi-Modal Deltas**
   - Design delta functions with multiple peaks
   - Study multi-stable behavior
   - Create complex pattern dynamics

3. **Optimization**
   - Implement lookup tables
   - Compare accuracy vs. performance
   - Profile different implementations

### Advanced Level
1. **Custom Delta Design**
   - Design delta functions for specific behaviors
   - Use optimization to find optimal parameters
   - Create automated delta function generation

2. **Theoretical Extensions**
   - Non-monotonic delta functions
   - Time-varying delta parameters
   - Spatially-varying delta functions

3. **Research Applications**
   - Connect delta shape to pattern formation theory
   - Study bifurcations in delta parameter space
   - Explore coupled delta-kernel systems

## Learning Resources

### Mathematics Background
1. **"Ordinary Differential Equations"** - Fixed points and stability
2. **"Nonlinear Dynamics and Chaos"** - Bifurcation theory
3. **"Mathematical Biology"** - Growth function applications

### Signal Processing
1. **"Digital Signal Processing"** - Function approximation
2. **"Approximation Theory"** - Optimal function representation
3. **"Numerical Analysis"** - Stability and convergence

### Programming Resources
1. **SciPy documentation**: Optimization and interpolation
2. **NumPy documentation**: Vectorized operations
3. **WebGL tutorials**: GPU function evaluation

### Research Papers
1. "Stability analysis of continuous cellular automata"
2. "Optimal growth function design for pattern formation"
3. "Bifurcation analysis in reaction-diffusion systems"

### Visualization Tools
1. **Desmos**: Interactive function plotting
2. **Matplotlib**: Python plotting library
3. **D3.js**: Web-based interactive visualizations

### Code Examples
1. **Lenia repository**: Original delta implementations
2. **Reaction-diffusion simulators**: Related growth functions
3. **Neural network activation functions**: Similar mathematical forms

## Next Steps
After mastering delta functions:
1. Study convolution and FFT for efficient kernel application
2. Understand how kernel-delta interactions create life forms
3. Learn about pattern analysis and recognition
4. Explore advanced optimization and performance techniques