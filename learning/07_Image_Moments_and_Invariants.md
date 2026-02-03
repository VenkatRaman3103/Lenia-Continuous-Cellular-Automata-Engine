# Image Moments and Invariants in Lenia

## Overview
Image moments and invariants provide powerful mathematical tools for analyzing, classifying, and tracking patterns in Lenia. They enable robust pattern recognition independent of position, orientation, and scale.

## Mathematical Foundations

### Image Moments Definition
For a 2D field f(x,y), the (p,q)-th order moment is:
```
mpq = Σ x,y x^p y^q f(x,y)
```

In continuous form:
```
mpq = ∫∫ x^p y^q f(x,y) dx dy
```

### Central Moments
Moments about the center of mass:
```
μpq = Σ x,y (x-x̄)^p (y-ȳ)^q f(x,y)
```
Where:
- **x̄ = m10/m00**: x-coordinate of center of mass
- **ȳ = m01/m00**: y-coordinate of center of mass

### Normalized Central Moments
Scale-invariant moments:
```
ηpq = μpq / μ00^(1 + (p+q)/2)
```

## Hu's Moment Invariants

### Seven Invariant Moments
Hu derived seven moment invariants that are invariant to translation, rotation, and scale:

1. **φ₁ = η20 + η02**
   - Basic shape descriptor
   - Invariant to all transformations

2. **φ₂ = (η20 - η02)² + 4η11²**
   - Measures shape anisotropy
   - Distinguishes elongated vs. circular shapes

3. **φ₃ = (η30 - 3η12)² + (3η21 - η03)²**
   - Third-order shape information
   - Sensitive to asymmetry

4. **φ₄ = (η30 + η12)² + (η21 + η03)²**
   - Alternative third-order measure
   - Complementary to φ₃

5. **φ₅ = (η30 - 3η12)(η30 + η12)[(η30 + η12)² - 3(η21 + η03)²] + (3η21 - η03)(η21 + η03)[3(η30 + η12)² - (η21 + η03)²]**
   - Complex fifth-order invariant
   - Highly discriminative

6. **φ₆ = (η20 - η02)[(η30 + η12)² - (η21 + η03)²] + 4η11(η30 + η12)(η21 + η03)**
   - Sixth-order mixed invariant
   - Captures complex shape features

7. **φ₇ = (3η21 - η03)(η30 + η12)[(η30 + η12)² - 3(η21 + η03)²] - (η30 - 3η12)(η21 + η03)[3(η30 + η12)² - (η21 + η03)²]**
   - Seventh-order skew invariant
   - Distinguishes mirror images

### Implementation
```javascript
function computeHuMoments(field) {
  const moments = computeAllMoments(field);
  const central = computeCentralMoments(moments);
  const normalized = normalizeCentralMoments(central);
  
  const hu = [];
  hu[0] = normalized.m20 + normalized.m02;
  hu[1] = Math.pow(normalized.m20 - normalized.m02, 2) + 4 * Math.pow(normalized.m11, 2);
  hu[2] = Math.pow(normalized.m30 - 3 * normalized.m12, 2) + 
           Math.pow(3 * normalized.m21 - normalized.m03, 2);
  // ... continue for all 7 invariants
  
  return hu;
}
```

## Complex Moments

### Definition
Complex moments use complex coordinates:
```
cpq = Σ x,y (x + iy)^p (x - iy)^q f(x,y)
```

### Advantages
- **Compact representation**: Fewer moments needed
- **Rotational invariance**: Natural rotation handling
- **Efficient computation**: FFT-based calculation

### Zernike Moments
Orthogonal moments on unit disk:
```
Znm = (n+1/π) ∫∫ f(r,θ) Vnm*(r,θ) r dr dθ
```
Where Vnm are Zernike polynomials.

## Flusser's Moment Invariants

### Extended Invariants
Flusser extended Hu's invariants for better discrimination:

1. **Translation invariants**: Based on central moments
2. **Rotation invariants**: Using complex moments
3. **Scale invariants**: Proper normalization
4. **Blur invariants**: Robust to image degradation

### Blur-Invariant Moments
For robustness to Gaussian blur:
```
Ψpq = Σ (−1)^k (p choose k) (q choose k) μ(p+k)(q+k) / μ00^(1 + (p+q+2k)/2)
```

## Applications in Lenia

### Pattern Classification
Use moment invariants to classify soliton types:
```javascript
class SolitonClassifier {
  constructor() {
    this.referenceMoments = this.loadReferenceData();
  }
  
  classify(field) {
    const moments = computeHuMoments(field);
    const distances = this.referenceMoments.map(ref => ({
      type: ref.type,
      distance: this.computeDistance(moments, ref.moments)
    }));
    
    return distances.sort((a, b) => a.distance - b.distance)[0].type;
  }
  
  computeDistance(m1, m2) {
    // Use log-transformed moments for better comparison
    const log1 = m1.map(m => Math.sign(m) * Math.log(Math.abs(m) + 1e-10));
    const log2 = m2.map(m => Math.sign(m) * Math.log(Math.abs(m) + 1e-10));
    
    return Math.sqrt(log1.reduce((sum, v, i) => 
      sum + Math.pow(v - log2[i], 2), 0));
  }
}
```

### Object Tracking
Track solitons across frames:
```javascript
class SolitonTracker {
  constructor() {
    this.trackedObjects = new Map();
  }
  
  update(field) {
    const detected = this.detectObjects(field);
    const matched = this.matchWithPrevious(detected);
    
    // Update tracking information
    matched.forEach(match => {
      if (match.previous) {
        match.previous.update(match.current);
      } else {
        this.trackedObjects.set(match.current.id, match.current);
      }
    });
  }
  
  matchWithPrevious(detected) {
    // Hungarian algorithm for optimal matching
    return this.hungarianMatch(detected, this.getPreviousObjects());
  }
}
```

### Period Detection
Detect periodic behavior using moment analysis:
```javascript
class PeriodDetector {
  constructor(windowSize = 100) {
    this.momentHistory = [];
    this.windowSize = windowSize;
  }
  
  addFrame(field) {
    const moments = computeHuMoments(field);
    this.momentHistory.push(moments);
    
    if (this.momentHistory.length > this.windowSize) {
      this.momentHistory.shift();
    }
  }
  
  detectPeriod() {
    if (this.momentHistory.length < this.windowSize / 2) {
      return null;
    }
    
    // Autocorrelation analysis
    const periods = [];
    for (let momentIndex = 0; momentIndex < 7; momentIndex++) {
      const series = this.momentHistory.map(m => m[momentIndex]);
      const period = this.findPeriod(series);
      periods.push(period);
    }
    
    // Return most common period
    return this.mode(periods);
  }
  
  findPeriod(series) {
    const autocorr = this.autocorrelation(series);
    const peaks = this.findPeaks(autocorr);
    return peaks.length > 0 ? peaks[0] : null;
  }
}
```

## Implementation Details

### Efficient Moment Computation
```javascript
class MomentCalculator {
  constructor(width, height) {
    this.width = width;
    this.height = height;
    this.precomputeCoordinates();
  }
  
  precomputeCoordinates() {
    this.xCoords = new Float32Array(this.width * this.height);
    this.yCoords = new Float32Array(this.width * this.height);
    
    for (let y = 0; y < this.height; y++) {
      for (let x = 0; x < this.width; x++) {
        const idx = y * this.width + x;
        this.xCoords[idx] = x;
        this.yCoords[idx] = y;
      }
    }
  }
  
  computeMoments(field) {
    const moments = {};
    
    // Zero-th moment (total mass)
    moments.m00 = field.reduce((sum, val) => sum + val, 0);
    
    if (moments.m00 === 0) return moments;
    
    // First moments (center of mass)
    moments.m10 = field.reduce((sum, val, i) => sum + val * this.xCoords[i], 0);
    moments.m01 = field.reduce((sum, val, i) => sum + val * this.yCoords[i], 0);
    
    // Higher moments
    for (let p = 0; p <= 3; p++) {
      for (let q = 0; q <= 3; q++) {
        if (p + q > 3) continue;
        
        const key = `m${p}${q}`;
        moments[key] = field.reduce((sum, val, i) => {
          const xPow = Math.pow(this.xCoords[i], p);
          const yPow = Math.pow(this.yCoords[i], q);
          return sum + val * xPow * yPow;
        }, 0);
      }
    }
    
    return moments;
  }
}
```

### Numerical Stability
```javascript
function stableMoments(field) {
  // Use log-space computation for large values
  const logField = field.map(v => Math.log(v + 1e-10));
  
  // Compute moments in log space
  const logMoments = computeLogMoments(logField);
  
  // Transform back with proper error handling
  return logMoments.map(m => Math.exp(m) - 1e-10);
}
```

### GPU Acceleration
WebGL shader for moment computation:
```glsl
precision highp float;
uniform sampler2D u_field;
uniform vec2 u_resolution;
uniform int u_order;

void main() {
  vec2 coord = gl_FragCoord.xy / u_resolution;
  float value = texture2D(u_field, coord).r;
  
  // Compute moments based on order
  if (u_order == 0) {
    gl_FragColor = vec4(value, 0.0, 0.0, 1.0);
  } else if (u_order == 1) {
    vec2 pos = gl_FragCoord.xy;
    gl_FragColor = vec4(value * pos.x, value * pos.y, 0.0, 1.0);
  }
  // ... higher orders
}
```

## Practice Problems

### Beginner Level
1. **Basic Moment Computation**
   - Implement raw moment calculation
   - Compute center of mass
   - Calculate basic shape descriptors

2. **Hu's Invariants**
   - Implement all seven Hu invariants
   - Test invariance properties
   - Classify simple shapes

3. **Pattern Recognition**
   - Create reference moment database
   - Implement simple classifier
   - Test on known patterns

### Intermediate Level
1. **Advanced Invariants**
   - Implement Flusser's invariants
   - Add blur-invariant moments
   - Compare classification performance

2. **Tracking Systems**
   - Implement object tracking
   - Handle object merging/splitting
   - Optimize matching algorithms

3. **Period Detection**
   - Implement autocorrelation analysis
   - Detect periodic behaviors
   - Analyze soliton dynamics

### Advanced Level
1. **Optimization Techniques**
   - GPU acceleration of moment computation
   - FFT-based complex moments
   - Real-time performance optimization

2. **Machine Learning Integration**
   - Use moments as features for ML
   - Train neural network classifiers
   - Implement deep learning approaches

3. **Research Applications**
   - Develop new invariant measures
   - Study moment-based pattern evolution
   - Create automated discovery systems

## Learning Resources

### Image Processing Theory
1. **"Digital Image Processing"** by Gonzalez & Woods - Classic text
2. **"Image Analysis and Recognition"** - Advanced techniques
3. **"Pattern Recognition"** by Duda, Hart & Stork - Classification theory

### Mathematical Background
1. **"Moments and Moment Invariants in Pattern Recognition"** by Jan Flusser
2. **"Statistical Pattern Recognition"** - Mathematical foundations
3. **"Computer Vision: Algorithms and Applications"** - Practical applications

### Programming Resources
1. **OpenCV Documentation**: Moment functions and examples
2. **scikit-image**: Python image processing library
3. **ImageJ**: Open-source image analysis software

### Research Papers
1. "Visual Pattern Recognition by Moment Invariants" by Hu (1962)
2. "Moment Invariants for Image Analysis" by Flusser et al.
3. "Zernike Moments for Pattern Recognition" applications

### Online Courses
1. **Coursera**: Image and video processing
2. **Udacity**: Computer vision nanodegree
3. **MIT OpenCourseWare**: Computer vision courses

### Code Libraries
1. **OpenCV**: Comprehensive computer vision library
2. **Mahotas**: Python image processing
3. **SimpleCV**: Simplified computer vision interface

## Next Steps
After mastering image moments and invariants:
1. Study user interface implementation details
2. Learn about performance optimization techniques
3. Explore advanced research topics and extensions
4. Apply these techniques to other pattern recognition problems