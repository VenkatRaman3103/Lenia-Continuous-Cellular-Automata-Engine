# Kernel Functions in Lenia

## Overview
Kernel functions define the neighborhood structure and influence pattern in Lenia. They determine how cells interact with their neighbors and are crucial for creating stable life forms and interesting dynamics.

## Fundamental Properties

### Radial Symmetry
Kernels in Lenia are radially symmetric:
```
k(u) = k(|u|) = k(r)
```
Where `r = |u|` is the distance from the kernel center.

### Normalization
Most kernels are normalized to preserve mass:
```
∫ k(u) du = 1
```
In discrete form:
```
Σ k(u) = 1
```

### Finite Support
Kernels typically have compact support:
```
k(r) = 0 for r > 1
```
This defines the neighborhood radius.

## Core Kernel Types

### 1. Gaussian Bump
The most commonly used kernel in Lenia:

```
k(r) = exp(α(1 - 1/k)) where k = 4r(1-r)
```

Properties:
- **Smooth**: Infinitely differentiable
- **Peak**: Maximum at r = 0.5
- **Zero**: k(0) = k(1) = 0
- **Parameter**: α controls peak sharpness

JavaScript implementation:
```javascript
function gaussianBump(r, alpha) {
  if (r < 0 || r > 1) return 0;
  const k = 4 * r * (1 - r);
  return Math.exp(alpha * (1 - 1/k));
}
```

### 2. Polynomial Kernel
Simpler alternative with similar properties:

```
k(r) = k^α where k = 4r(1-r)
```

Properties:
- **Algebraic**: Polynomial behavior
- **Simpler**: Easier to compute derivatives
- **Similar**: Qualitatively similar to Gaussian

```javascript
function polynomialKernel(r, alpha) {
  if (r < 0 || r > 1) return 0;
  const k = 4 * r * (1 - r);
  return Math.pow(k, alpha);
}
```

### 3. Trapezoidal Kernel
Piecewise linear kernel:

```
k(r) = {
  5r,           0 ≤ r ≤ 0.2
  1,            0.2 < r ≤ 0.8
  5(1 - r),     0.8 < r ≤ 1
}
```

Properties:
- **Linear**: Simple piecewise function
- **Flat**: Constant middle region
- **Fast**: Computationally efficient

### 4. Step Kernel
For SmoothLife compatibility:

```
k(r) = {
  1,  r_inner ≤ r ≤ r_outer
  0,  otherwise
}
```

Where `r_outer = 3 × r_inner`

### 5. High Step Kernel
For Conway's Game of Life approximation:

```
k(r) = {
  1,  r = 1 (8-neighbor Moore)
  0,  otherwise
}
```

## Multi-Modal Kernel Functions

### Bimodal Kernel
Two peaks at different radii:

```
k(r) = β₁·c₁(r) + β₂·c₂(r)
```

Where:
- `c₁(r)` and `c₂(r)` are core functions at different radii
- `β₁`, `β₂` are peak heights
- Typically peaks at r ≈ 0.3 and r ≈ 0.7

### Trimodal Kernel
Three peaks for complex interactions:

```
k(r) = β₁·c₁(r) + β₂·c₂(r) + β₃·c₃(r)
```

### Higher-Modal Kernels
Up to tetramodal (4 peaks) kernels are supported for increasingly complex behaviors.

## Parameter Analysis

### Stiffness Parameter (α)
- **Range**: Typically 2-16
- **Effect**: Higher α = sharper peaks
- **Trade-off**: Sharpness vs. numerical stability

### Peak Heights (β)
- **Normalization**: Σβᵢ = 1 (common convention)
- **Mean height**: η = (Σβᵢ)/n
- **Variation**: Controls relative influence of different modes

### Spatial Placement
Peak locations affect interaction ranges:
- **Inner peaks**: Short-range interactions
- **Outer peaks**: Long-range coupling
- **Spacing**: Determines characteristic length scales

## Mathematical Properties

### Fourier Transform
Understanding kernels in frequency domain:

```
K(ω) = ∫ k(r) e^(-iω·r) dr
```

Key insights:
- **Low frequencies**: Determine bulk behavior
- **High frequencies**: Control fine details
- **Cutoff frequency**: Related to kernel support

### Moments
Moments characterize kernel shape:

```
m_n = ∫ r^n k(r) dr
```

- **m₀**: Total mass (normalization)
- **m₁**: Mean radius
- **m₂**: Variance (spread)

### Derivatives
Smoothness affects dynamics:

```
k'(r) = derivative with respect to radius
k''(r) = second derivative
```

Higher derivatives exist for smooth kernels (Gaussian) but not for piecewise kernels.

## Implementation Considerations

### Discretization
For discrete implementation:

```javascript
function discretizeKernel(kernel, R) {
  const size = 2 * R + 1;
  const discrete = new Array(size * size);
  
  for (let i = 0; i < size; i++) {
    for (let j = 0; j < size; j++) {
      const x = (i - R) / R;
      const y = (j - R) / R;
      const r = Math.sqrt(x * x + y * y);
      discrete[i * size + j] = kernel(r);
    }
  }
  
  // Normalize
  const sum = discrete.reduce((a, b) => a + b, 0);
  return discrete.map(v => v / sum);
}
```

### Performance Optimization
- **Pre-computation**: Store kernel values
- **Separation**: For separable kernels
- **FFT**: Use convolution theorem for large kernels

### Numerical Stability
- **Avoid zeros**: Prevent division by zero
- **Smoothing**: Add small epsilon where needed
- **Clipping**: Ensure reasonable ranges

## Practice Problems

### Beginner Level
1. **Basic Kernel Implementation**
   - Implement Gaussian bump kernel from scratch
   - Create visualization of kernel shapes
   - Test normalization property

2. **Parameter Exploration**
   - Vary α from 2 to 16 and observe changes
   - Create animation of kernel shape evolution
   - Measure computational time for different kernels

3. **Discretization Practice**
   - Convert continuous kernels to discrete grids
   - Test different grid resolutions (R values)
   - Compare accuracy vs. computation cost

### Intermediate Level
1. **Multi-Modal Kernels**
   - Implement bimodal and trimodal kernels
   - Find optimal peak locations for specific behaviors
   - Analyze interaction between different modes

2. **Mathematical Analysis**
   - Compute Fourier transforms analytically
   - Calculate kernel moments
   - Derive optimal parameter relationships

3. **Optimization Techniques**
   - Implement FFT-based kernel application
   - Compare direct vs. FFT convolution
   - Profile performance for different kernel sizes

### Advanced Level
1. **Custom Kernel Design**
   - Design kernels for specific life forms
   - Use optimization algorithms to find parameters
   - Create automated kernel generation systems

2. **Theoretical Extensions**
   - Extend to non-radial kernels
   - Implement time-varying kernels
   - Develop kernel composition frameworks

3. **Research Applications**
   - Connect kernel properties to pattern formation
   - Study kernel-induced phase transitions
   - Explore multi-scale kernel hierarchies

## Learning Resources

### Mathematical Background
1. **"Fourier Analysis and Applications"** - Frequency domain analysis
2. **"Radial Basis Functions"** - Mathematical theory of radial functions
3. **"Numerical Methods for PDEs"** - Discretization techniques

### Signal Processing
1. **"Digital Image Processing"** by Gonzalez & Woods - Convolution operations
2. **"The Scientist and Engineer's Guide to Digital Signal Processing"**
3. **"Wavelets and Subband Coding"** - Multi-scale analysis

### Programming Resources
1. **NumPy/SciPy documentation**: Convolution and FFT functions
2. **GLSL shader tutorials**: GPU kernel implementation
3. **WebGL performance guides**: Browser optimization

### Research Papers
1. "Radial basis function interpolation" - Mathematical foundations
2. "Multi-scale kernel methods for pattern formation"
3. "Optimal kernel design for cellular automata"

### Visualization Tools
1. **Desmos**: Interactive function plotting
2. **GeoGebra**: 3D function visualization
3. **Matplotlib/Plotly**: Python/Javascript plotting libraries

### Code Repositories
1. **scikit-image**: Image processing kernels
2. **OpenCV**: Optimized convolution implementations
3. **TensorFlow/PyTorch**: GPU-accelerated convolution

## Next Steps
After mastering kernel functions:
1. Study delta functions for growth rules
2. Learn about convolution and FFT optimization
3. Understand how kernel-delta interactions create life forms
4. Explore advanced multi-scale and time-varying kernels