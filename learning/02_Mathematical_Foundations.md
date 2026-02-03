# Mathematical Foundations of Lenia

## Overview
Lenia extends traditional cellular automata into continuous space-time using differential equations and smooth mathematical functions. This document covers the core mathematical framework that makes Lenia capable of generating naturalistic life forms.

## Core Equation

### Field Evolution
The fundamental equation governing Lenia's evolution is:

```
Δf(x) = d(k * f(x)) · Δt
```

Where:
- **f(x)**: Field value at position x (cell state)
- **k**: Kernel function (neighborhood weighting)
- **d**: Delta function (growth rule)
- **\***: Convolution operation
- **Δt**: Time step size

### Discrete Time Implementation
In practice, we use discrete time steps:
```
f(x, t+1) = f(x, t) + d(k * f(x, t)) · Δt
```

## Units and Scaling

### Physical Interpretation
Lenia uses biologically-inspired units:

1. **Space**: 1 μm (micrometer) = R cells
2. **Time**: 1 μs (microsecond) = T steps  
3. **Mass**: Cell density per cell

### Normalization
- Field values typically range from **[0, 1]** or **[0, ∞)**
- Time steps are usually small: **Δt ≈ 0.1 - 0.5**
- Space resolution affects computational cost

## Mathematical Components

### 1. Kernel Functions (k)
Kernels define neighborhood influence and are radially symmetric:

```javascript
k(r) = f(distance_from_center / kernel_radius)
```

Key properties:
- **Radial symmetry**: k(r) depends only on distance from center
- **Normalization**: Σk(u) = 1 (or other constant)
- **Smoothness**: Continuous derivatives
- **Finite support**: k(r) = 0 for r > 1

### 2. Delta Functions (d)
Delta functions determine growth/decay based on neighborhood sum:

```javascript
growth = d(neighborhood_sum)
```

Key properties:
- **Range**: Typically outputs [-1, 1]
- **Center**: μ determines optimal growth point
- **Width**: σ controls sensitivity
- **Smoothness**: Continuous derivatives preferred

### 3. Convolution Operation
The neighbor sum uses circular convolution:

```
(k * f)(x) = (1/N) Σ|u|≤1 k(u) · f(x + u)
```

Where N is the normalization factor:
```
N = Σ|u|≤1 k(u)
```

## Parameter System

### Core Parameters

| Parameter | Symbol | Range | Physical Meaning |
|-----------|--------|-------|------------------|
| Space resolution | R | 8-64 | Cells per kernel radius |
| Time resolution | T | 1-8 | Steps per unit time |
| Time step | Δt | 0.1-0.5 | Integration step size |

### Kernel Parameters

| Parameter | Symbol | Range | Effect |
|-----------|--------|-------|--------|
| Stiffness | α | 2-16 | Kernel shape sharpness |
| Peak heights | β₁, β₂, ... | 0.1-2.0 | Relative influence |
| Mean height | η | Calculated | Average peak height |

### Delta Parameters

| Parameter | Symbol | Range | Effect |
|-----------|--------|-------|--------|
| Center | μ | 0.1-0.9 | Optimal growth point |
| Width | σ | 0.01-0.5 | Sensitivity range |
| Stiffness | α | 2-16 | Growth curve sharpness |

## Mathematical Analysis

### Stability Analysis
Life forms (solitons) exist when:
1. **Fixed points**: d(k*f*) = 0 for equilibrium states
2. **Linear stability**: Eigenvalues of Jacobian have negative real parts
3. **Nonlinear stability**: Bounded perturbations return to equilibrium

### Conservation Laws
Some Lenia configurations exhibit:
- **Mass conservation**: Σf(x) remains constant
- **Energy conservation**: Certain functionals remain invariant
- **Momentum conservation**: For symmetric configurations

### Bifurcation Theory
Parameter changes cause qualitative behavior changes:
- **Saddle-node bifurcations**: Creation/annihilation of fixed points
- **Hopf bifurcations**: Emergence of oscillatory behavior
- **Period-doubling cascades**: Route to chaotic dynamics

## Numerical Methods

### Integration Schemes
1. **Explicit Euler**: Simple but can be unstable
   ```
   f_{n+1} = f_n + Δt · d(k * f_n)
   ```

2. **Semi-implicit**: Better stability
   ```
   f_{n+1} = f_n + Δt · d(k * f_n) + O(Δt²)
   ```

3. **Higher-order methods**: Runge-Kutta, etc.

### Discretization Effects
1. **Spatial aliasing**: Insufficient resolution
2. **Temporal aliasing**: Large time steps
3. **Boundary conditions**: Wrap-around, fixed, reflective

### Stability Constraints
CFL-like conditions for numerical stability:
```
Δt ≤ C / (max|d'(·)| · max|k(·)|)
```
Where C is a constant depending on discretization.

## Practice Problems

### Beginner Level
1. **Parameter Understanding**
   - Given R=16, what does this mean physically?
   - If μ=0.5 and σ=0.1, sketch the delta function
   - How does changing α affect kernel shape?

2. **Basic Calculations**
   - Compute normalization factor N for simple kernels
   - Calculate derivative of delta functions
   - Verify units consistency in the core equation

3. **Numerical Implementation**
   - Implement 1D convolution manually
   - Create explicit Euler integration
   - Test different time step sizes

### Intermediate Level
1. **Kernel Analysis**
   - Derive analytical expressions for common kernels
   - Compute Fourier transforms of kernels
   - Analyze multi-modal kernel properties

2. **Stability Analysis**
   - Linearize around equilibrium points
   - Compute Jacobian matrices
   - Determine eigenvalues analytically

3. **Parameter Studies**
   - Map bifurcation diagrams
   - Create stability regions in parameter space
   - Analyze sensitivity to parameter changes

### Advanced Level
1. **Rigorous Mathematics**
   - Prove existence/uniqueness of solutions
   - Analyze long-term behavior
   - Develop analytical approximations

2. **Computational Methods**
   - Implement adaptive time stepping
   - Use spectral methods for spatial derivatives
   - Develop multi-scale algorithms

3. **Theoretical Extensions**
   - Higher-dimensional analysis
   - Stochastic extensions
   - Coupled systems analysis

## Learning Resources

### Mathematics Background
1. **"Partial Differential Equations"** by Walter Strauss - PDE foundations
2. **"Numerical Recipes"** by Press et al. - Computational methods
3. **"Dynamical Systems"** by Steven Strogatz - Bifurcation theory

### Specialized Topics
1. **"Convolutional Neural Networks"** - Understanding convolution operations
2. **"Fourier Analysis and Applications"** - Spectral methods
3. **"Stability Theory"** - Dynamical systems analysis

### Online Courses
1. **MIT OpenCourseWare**: Differential Equations
2. **Coursera**: Dynamical Systems and Chaos
3. **Khan Academy**: Linear Algebra and Calculus

### Software Tools
1. **MATLAB/Octave**: Symbolic and numerical computation
2. **Python (NumPy/SciPy)**: Scientific computing
3. **Mathematica**: Symbolic mathematics

### Research Papers
1. "Analysis of continuous cellular automata" - Mathematical foundations
2. "Stability of localized structures in reaction-diffusion systems"
3. "Numerical methods for integro-differential equations"

### Practice Platforms
1. **Project Euler**: Mathematical problem solving
2. **LeetCode**: Algorithm and numerical problems
3. **arXiv**: Latest mathematical research

## Next Steps
After mastering these mathematical foundations:
1. Study specific kernel functions in detail
2. Understand delta function implementations
3. Learn about convolution and FFT optimization
4. Analyze specific life forms mathematically