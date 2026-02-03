# Convolution and FFT in Lenia

## Overview
Convolution is the core mathematical operation that applies kernel functions to the field in Lenia. Fast Fourier Transform (FFT) optimization makes real-time simulation possible by reducing computational complexity.

## Convolution Fundamentals

### Mathematical Definition
Circular convolution in 2D:
```
(k * f)(x, y) = (1/N) Σᵢⱼ k(i, j) · f(x-i, y-j)
```

Where:
- **k**: Kernel function (typically small, like 13×13 or 17×17)
- **f**: Field function (large, like 256×256 or 512×512)
- **N**: Normalization factor
- **\***: Convolution operator

### Physical Interpretation
At each point, convolution:
1. **Overlays** kernel on field
2. **Multiplies** kernel weights by field values
3. **Sums** weighted contributions
4. **Normalizes** result

### Boundary Conditions
Common approaches:
- **Wrap-around (periodic)**: Toroidal topology
- **Zero-padding**: Edges have zero contribution
- **Reflective**: Mirrored boundary conditions
- **Fixed**: Constant boundary values

## Direct Convolution Implementation

### Naive Approach
```javascript
function directConvolution(field, kernel) {
  const [fieldWidth, fieldHeight] = getFieldSize(field);
  const [kernelWidth, kernelHeight] = kernel.shape;
  const result = new Float32Array(fieldWidth * fieldHeight);
  
  const kCenterX = Math.floor(kernelWidth / 2);
  const kCenterY = Math.floor(kernelHeight / 2);
  
  for (let y = 0; y < fieldHeight; y++) {
    for (let x = 0; x < fieldWidth; x++) {
      let sum = 0;
      
      for (let ky = 0; ky < kernelHeight; ky++) {
        for (let kx = 0; kx < kernelWidth; kx++) {
          const fx = (x + kx - kCenterX + fieldWidth) % fieldWidth;
          const fy = (y + ky - kCenterY + fieldHeight) % fieldHeight;
          
          sum += kernel[ky * kernelWidth + kx] * field[fy * fieldWidth + fx];
        }
      }
      
      result[y * fieldWidth + x] = sum;
    }
  }
  
  return result;
}
```

### Complexity Analysis
- **Time**: O(N² × K²) where N is field size, K is kernel size
- **Space**: O(N²) for result
- **Example**: 512×512 field with 17×17 kernel ≈ 76 million operations

## Fast Fourier Transform Optimization

### Convolution Theorem
The key insight: convolution in spatial domain = multiplication in frequency domain:
```
FFT(k * f) = FFT(k) × FFT(f)
k * f = IFFT(FFT(k) × FFT(f))
```

### FFT Implementation
```javascript
function fftConvolution(field, kernel) {
  // Convert to arrays of appropriate size
  const fieldFFT = fft2d(field);
  const kernelFFT = fft2d(kernel);
  
  // Multiply in frequency domain
  const resultFFT = multiplyComplex(fieldFFT, kernelFFT);
  
  // Convert back to spatial domain
  return ifft2d(resultFFT);
}

function multiplyComplex(a, b) {
  return {
    real: a.real * b.real - a.imag * b.imag,
    imag: a.real * b.imag + a.imag * b.real
  };
}
```

### Complexity Reduction
- **FFT**: O(N² log N) each
- **Multiplication**: O(N²)
- **Total**: O(N² log N) vs. O(N² × K²)

### Performance Gains
For typical Lenia parameters:
- **Field**: 512×512
- **Kernel**: 17×17
- **Speed improvement**: ~10-100x depending on implementation

## Implementation Details

### Zero Padding
For circular convolution, embed in larger array:
```javascript
function padForConvolution(field, kernel) {
  const fieldSize = field.length;
  const kernelSize = kernel.length;
  
  // Find next power of 2 for FFT efficiency
  const paddedSize = nextPowerOf2(fieldSize + kernelSize - 1);
  
  // Pad both arrays
  const paddedField = new Float32Array(paddedSize * paddedSize);
  const paddedKernel = new Float32Array(paddedSize * paddedSize);
  
  // Copy data (centered)
  copyToCenter(field, paddedField);
  copyToCenter(kernel, paddedKernel);
  
  return { paddedField, paddedKernel, size: paddedSize };
}
```

### Real FFT Optimization
Since inputs are real-valued, use specialized algorithms:
- **Half-size FFT**: Only compute half the frequency domain
- **Hermitian symmetry**: Exploit conjugate symmetry
- **Memory savings**: 50% reduction in storage

### Multi-dimensional FFT
For 2D convolution:
```javascript
function fft2d(real) {
  const width = Math.sqrt(real.length);
  const height = width;
  
  // FFT on rows
  const rowFFT = new Array(height);
  for (let y = 0; y < height; y++) {
    const row = real.slice(y * width, (y + 1) * width);
    rowFFT[y] = fft1d(row);
  }
  
  // FFT on columns (transpose)
  const transposed = transpose(rowFFT);
  const colFFT = new Array(width);
  for (let x = 0; x < width; x++) {
    colFFT[x] = fft1d(transposed[x]);
  }
  
  return transpose(colFFT);
}
```

## Memory and Performance Optimization

### Memory Management
```javascript
class FFTConvolution {
  constructor(maxSize) {
    // Pre-allocate working buffers
    this.workBuffer = new Float32Array(maxSize * maxSize * 2); // Complex
    this.freqBuffer = new Float32Array(maxSize * maxSize * 2);
    this.fft = new FFT(maxSize);
  }
  
  convolve(field, kernel) {
    // Reuse pre-allocated buffers
    return this.fftConvolutionInternal(field, kernel);
  }
}
```

### Batch Processing
For multiple fields with same kernel:
```javascript
function batchFFTConvolution(fields, kernel) {
  const kernelFFT = fft2d(kernel); // Compute once
  
  return fields.map(field => {
    const fieldFFT = fft2d(field);
    const resultFFT = multiplyComplex(fieldFFT, kernelFFT);
    return ifft2d(resultFFT);
  });
}
```

### GPU Acceleration
WebGL implementation for browser:
```glsl
// Fragment shader for FFT
precision highp float;
uniform sampler2D u_texture;
uniform vec2 u_resolution;
uniform float u_direction;

vec2 complexMultiply(vec2 a, vec2 b) {
  return vec2(a.x * b.x - a.y * b.y, a.x * b.y + a.y * b.x);
}

void main() {
  vec2 coord = gl_FragCoord.xy / u_resolution;
  
  // FFT butterfly operations
  // ... implementation details
  
  gl_FragColor = vec4(result, 0.0, 1.0);
}
```

## Numerical Considerations

### Precision Issues
- **Floating-point errors**: Accumulate over iterations
- **Wrap-around artifacts**: From circular convolution
- **Edge effects**: From padding strategies

### Accuracy vs. Speed
```javascript
function adaptiveConvolution(field, kernel, accuracy) {
  if (kernel.length < 9 || accuracy === 'fast') {
    return directConvolution(field, kernel); // Small kernels
  } else {
    return fftConvolution(field, kernel);    // Large kernels
  }
}
```

### Error Analysis
Compare direct vs. FFT results:
```javascript
function validateFFT(field, kernel, tolerance = 1e-6) {
  const direct = directConvolution(field, kernel);
  const fft = fftConvolution(field, kernel);
  
  let maxError = 0;
  for (let i = 0; i < direct.length; i++) {
    const error = Math.abs(direct[i] - fft[i]);
    maxError = Math.max(maxError, error);
  }
  
  return maxError < tolerance;
}
```

## Practice Problems

### Beginner Level
1. **Basic Convolution**
   - Implement direct 2D convolution
   - Test with simple kernels (3×3, 5×5)
   - Verify with known results

2. **FFT Basics**
   - Implement 1D FFT from scratch
   - Verify with sinusoidal inputs
   - Test inverse FFT (FFT⁻¹(FFT(x)) = x)

3. **Boundary Conditions**
   - Implement different boundary conditions
   - Compare effects on results
   - Test with various field patterns

### Intermediate Level
1. **FFT Optimization**
   - Implement 2D FFT using 1D FFTs
   - Optimize memory usage
   - Compare performance with direct convolution

2. **Kernel Frequency Analysis**
   - Compute FFT of various kernels
   - Analyze frequency content
   - Design kernels with specific frequency properties

3. **Multi-scale Processing**
   - Implement multi-resolution convolution
   - Use different kernel sizes at different scales
   - Create pyramid representations

### Advanced Level
1. **GPU Implementation**
   - Implement FFT using WebGL/WebGPU
   - Optimize for parallel processing
   - Compare with CPU performance

2. **Advanced Algorithms**
   - Implement Winograd convolution
   - Use separable kernel decomposition
   - Develop mixed-precision algorithms

3. **Research Applications**
   - Study convolution in different mathematical spaces
   - Explore non-uniform FFT algorithms
   - Analyze numerical stability in long simulations

## Learning Resources

### Signal Processing Foundations
1. **"Digital Signal Processing"** by Oppenheim & Schafer - Classic text
2. **"The Scientist and Engineer's Guide to Digital Signal Processing"** - Practical approach
3. **"Fast Fourier Transform and Its Applications"** - FFT-specific

### Mathematical Background
1. **"Fourier Analysis and Applications"** - Mathematical theory
2. **"Numerical Linear Algebra"** - Matrix operations
3. **"Computational Science and Engineering"** - Practical numerical methods

### Programming Resources
1. **FFTW Documentation**: Fastest Fourier Transform in the West
2. **NumPy/SciPy FFT**: Python implementations
3. **Kiss FFT**: Simple FFT library

### GPU Computing
1. **CUDA Programming Guide**: NVIDIA GPU programming
2. **WebGL/WebGPU Documentation**: Browser GPU computing
3. **OpenCL**: Cross-platform GPU computing

### Online Resources
1. **Better Explained**: Intuitive FFT explanations
2. **Steve Brunton**: YouTube channel on signal processing
3. **3Blue1Brown**: Visual explanations of Fourier transforms

### Code Libraries
1. **scikit-image**: Image processing with convolution
2. **OpenCV**: Optimized computer vision operations
3. **TensorFlow/PyTorch**: GPU-accelerated convolution

## Next Steps
After mastering convolution and FFT:
1. Study life forms and soliton stability analysis
2. Learn about image moments for pattern recognition
3. Understand user interface implementation details
4. Explore advanced optimization techniques