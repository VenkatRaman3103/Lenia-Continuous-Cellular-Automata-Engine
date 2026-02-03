# Performance Optimization in Lenia

## Overview
Performance optimization is crucial for real-time Lenia simulation, especially at high resolutions and with complex kernels. This document covers techniques for achieving smooth interactive performance.

## Bottleneck Analysis

### Computational Hotspots
1. **Convolution Operations**: O(N² × K²) for direct convolution
2. **FFT Transforms**: O(N² log N) for frequency domain convolution
3. **Rendering Pipeline**: Canvas updates and color mapping
4. **Memory Allocation**: Array creation and garbage collection
5. **UI Updates**: DOM manipulation and event handling

### Performance Metrics
- **Frame Rate**: Target 30-60 FPS for smooth animation
- **Latency**: <100ms response time for interactions
- **Memory Usage**: <500MB for typical configurations
- **CPU Utilization**: <80% for responsive UI

## Convolution Optimization

### FFT vs Direct Convolution
```javascript
class ConvolutionOptimizer {
    constructor() {
        this.fftThreshold = 9; // Switch to FFT for kernels > 9×9
        this.fftCache = new Map();
    }
    
    convolve(field, kernel) {
        const method = this.selectMethod(field, kernel);
        
        switch (method) {
            case 'direct':
                return this.directConvolution(field, kernel);
            case 'fft':
                return this.fftConvolution(field, kernel);
            case 'separable':
                return this.separableConvolution(field, kernel);
        }
    }
    
    selectMethod(field, kernel) {
        const kernelSize = Math.sqrt(kernel.length);
        
        if (kernelSize > this.fftThreshold) {
            return 'fft';
        } else if (this.isSeparable(kernel)) {
            return 'separable';
        } else {
            return 'direct';
        }
    }
}
```

### Separable Kernel Optimization
For kernels that can be expressed as outer products:
```javascript
function optimizeSeparableKernel(kernel) {
    const size = Math.sqrt(kernel.length);
    const singularValues = this.computeSVD(kernel);
    
    // Check if kernel is approximately separable
    if (singularValues[1] / singularValues[0] < 0.01) {
        const [u, s, v] = this.computeSVD(kernel);
        const separableKernel = {
            horizontal: u.map(val => val * Math.sqrt(s[0])),
            vertical: v.map(val => val * Math.sqrt(s[0]))
        };
        
        return separableKernel;
    }
    
    return null;
}

function separableConvolution(field, separableKernel) {
    // Horizontal pass
    const horizontal = this.convolve1D(field, separableKernel.horizontal, 'horizontal');
    // Vertical pass
    const result = this.convolve1D(horizontal, separableKernel.vertical, 'vertical');
    
    return result;
}
```

### Multi-threading with Web Workers
```javascript
class WorkerPool {
    constructor(workerCount = 4) {
        this.workers = [];
        this.taskQueue = [];
        this.busyWorkers = new Set();
        
        for (let i = 0; i < workerCount; i++) {
            const worker = new Worker('convolution-worker.js');
            worker.onmessage = (e) => this.handleWorkerMessage(e, worker);
            this.workers.push(worker);
        }
    }
    
    async convolveParallel(field, kernel) {
        const chunks = this.splitField(field);
        const promises = chunks.map(chunk => this.submitTask(chunk, kernel));
        
        return Promise.all(promises);
    }
    
    submitTask(chunk, kernel) {
        return new Promise((resolve) => {
            const task = {
                chunk,
                kernel,
                resolve,
                id: Math.random()
            };
            
            this.taskQueue.push(task);
            this.processQueue();
        });
    }
    
    processQueue() {
        if (this.taskQueue.length === 0) return;
        
        const availableWorker = this.workers.find(w => !this.busyWorkers.has(w));
        if (!availableWorker) return;
        
        const task = this.taskQueue.shift();
        this.busyWorkers.add(availableWorker);
        
        availableWorker.postMessage({
            chunk: task.chunk,
            kernel: task.kernel,
            id: task.id
        });
    }
}
```

## Memory Optimization

### Object Pooling
```javascript
class ArrayPool {
    constructor(initialSize = 100) {
        this.available = [];
        this.inUse = new Set();
        
        for (let i = 0; i < initialSize; i++) {
            this.available.push(new Float32Array(256 * 256));
        }
    }
    
    acquire(size = 256 * 256) {
        let array = this.available.find(arr => arr.length >= size);
        
        if (!array) {
            array = new Float32Array(size);
        } else {
            this.available = this.available.filter(arr => arr !== array);
        }
        
        this.inUse.add(array);
        return array;
    }
    
    release(array) {
        if (this.inUse.has(array)) {
            this.inUse.delete(array);
            this.available.push(array);
        }
    }
}

// Usage in convolution
class OptimizedConvolution {
    constructor() {
        this.arrayPool = new ArrayPool();
    }
    
    convolve(field, kernel) {
        const result = this.arrayPool.acquire(field.length);
        
        // Perform convolution using pooled array
        this.performConvolution(field, kernel, result);
        
        // Return copy and release pool
        const copy = new Float32Array(result);
        this.arrayPool.release(result);
        
        return copy;
    }
}
```

### Memory-Mapped Arrays
```javascript
class MemoryManager {
    constructor() {
        this.bufferSize = 1024 * 1024 * 64; // 64MB buffer
        this.buffer = new ArrayBuffer(this.bufferSize);
        this.allocator = new StackAllocator(this.buffer);
    }
    
    allocateField(width, height) {
        const size = width * height * 4; // Float32
        const offset = this.allocator.allocate(size);
        
        return {
            data: new Float32Array(this.buffer, offset, width * height),
            offset,
            size
        };
    }
    
    deallocateField(field) {
        this.allocator.deallocate(field.offset, field.size);
    }
}

class StackAllocator {
    constructor(buffer) {
        this.buffer = buffer;
        this.offset = 0;
        this.allocations = [];
    }
    
    allocate(size) {
        const alignedSize = this.align(size, 8);
        
        if (this.offset + alignedSize > this.buffer.byteLength) {
            throw new Error('Out of memory');
        }
        
        const offset = this.offset;
        this.offset += alignedSize;
        this.allocations.push({ offset, size: alignedSize });
        
        return offset;
    }
    
    deallocate(offset, size) {
        // Simple stack deallocation - only works for LIFO
        const lastAllocation = this.allocations[this.allocations.length - 1];
        
        if (lastAllocation.offset === offset) {
            this.offset = offset;
            this.allocations.pop();
        }
    }
    
    align(size, alignment) {
        return Math.ceil(size / alignment) * alignment;
    }
}
```

## Rendering Optimization

### Efficient Canvas Updates
```javascript
class OptimizedRenderer {
    constructor(canvas) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d', { 
            alpha: false,
            desynchronized: true 
        });
        
        this.imageData = this.ctx.createImageData(canvas.width, canvas.height);
        this.dirtyRegions = [];
    }
    
    renderField(field) {
        const pixels = this.imageData.data;
        
        // Use SIMD-like operations for better performance
        this.renderFieldSIMD(field, pixels);
        
        // Only update dirty regions
        this.updateDirtyRegions();
    }
    
    renderFieldSIMD(field, pixels) {
        const length = field.length;
        
        // Process 4 pixels at once for better cache utilization
        for (let i = 0; i < length; i += 4) {
            const value1 = field[i];
            const value2 = field[i + 1] || 0;
            const value3 = field[i + 2] || 0;
            const value4 = field[i + 3] || 0;
            
            const color1 = this.valueToColor(value1);
            const color2 = this.valueToColor(value2);
            const color3 = this.valueToColor(value3);
            const color4 = this.valueToColor(value4);
            
            const pixelIndex = i * 4;
            pixels[pixelIndex] = color1.r;
            pixels[pixelIndex + 1] = color1.g;
            pixels[pixelIndex + 2] = color1.b;
            pixels[pixelIndex + 3] = 255;
            
            if (i + 1 < length) {
                pixels[pixelIndex + 4] = color2.r;
                pixels[pixelIndex + 5] = color2.g;
                pixels[pixelIndex + 6] = color2.b;
                pixels[pixelIndex + 7] = 255;
            }
        }
    }
    
    updateDirtyRegions() {
        if (this.dirtyRegions.length === 0) {
            // Full update
            this.ctx.putImageData(this.imageData, 0, 0);
        } else {
            // Partial updates
            this.dirtyRegions.forEach(region => {
                const subImageData = this.ctx.getImageData(
                    region.x, region.y, region.width, region.height);
                this.ctx.putImageData(subImageData, region.x, region.y);
            });
        }
        
        this.dirtyRegions = [];
    }
}
```

### WebGL Acceleration
```javascript
class WebGLRenderer {
    constructor(canvas) {
        this.canvas = canvas;
        this.gl = canvas.getContext('webgl2') || canvas.getContext('webgl');
        
        this.setupShaders();
        this.setupBuffers();
        this.setupTextures();
    }
    
    setupShaders() {
        const vertexShaderSource = `
            attribute vec2 a_position;
            attribute vec2 a_texCoord;
            varying vec2 v_texCoord;
            
            void main() {
                gl_Position = vec4(a_position, 0.0, 1.0);
                v_texCoord = a_texCoord;
            }
        `;
        
        const fragmentShaderSource = `
            precision highp float;
            uniform sampler2D u_field;
            uniform float u_colorMode;
            varying vec2 v_texCoord;
            
            vec3 fieldToColor(float value) {
                if (u_colorMode == 0.0) {
                    // Green mode
                    return vec3(0.0, value, value * 0.5);
                } else if (u_colorMode == 1.0) {
                    // Heat map
                    return vec3(value, value * 0.5, 1.0 - value);
                } else {
                    // Grayscale
                    return vec3(value, value, value);
                }
            }
            
            void main() {
                float value = texture2D(u_field, v_texCoord).r;
                vec3 color = fieldToColor(value);
                gl_FragColor = vec4(color, 1.0);
            }
        `;
        
        this.program = this.createShaderProgram(vertexShaderSource, fragmentShaderSource);
    }
    
    renderField(field) {
        // Update texture with field data
        this.gl.bindTexture(this.gl.TEXTURE_2D, this.fieldTexture);
        this.gl.texSubImage2D(this.gl.TEXTURE_2D, 0, 0, 0, 
            this.fieldWidth, this.fieldHeight, 
            this.gl.RED, this.gl.FLOAT, field);
        
        // Render
        this.gl.drawArrays(this.gl.TRIANGLE_STRIP, 0, 4);
    }
}
```

## Algorithmic Optimization

### Adaptive Resolution
```javascript
class AdaptiveResolution {
    constructor(targetFPS = 30) {
        this.targetFPS = targetFPS;
        this.currentFPS = 60;
        this.resolution = 256;
        this.minResolution = 64;
        this.maxResolution = 512;
    }
    
    updateResolution(actualFPS) {
        this.currentFPS = actualFPS;
        
        if (actualFPS < this.targetFPS * 0.9) {
            // Decrease resolution
            this.resolution = Math.max(this.minResolution, 
                Math.floor(this.resolution * 0.9));
        } else if (actualFPS > this.targetFPS * 1.1) {
            // Increase resolution
            this.resolution = Math.min(this.maxResolution, 
                Math.floor(this.resolution * 1.1));
        }
    }
    
    getOptimalParameters() {
        return {
            fieldSize: this.resolution,
            kernelSize: Math.min(17, Math.floor(this.resolution / 16)),
            timeStep: this.currentFPS < this.targetFPS ? 0.2 : 0.1
        };
    }
}
```

### Level of Detail (LOD)
```javascript
class LODManager {
    constructor() {
        this.levels = [
            { resolution: 64, kernelSize: 5, quality: 'low' },
            { resolution: 128, kernelSize: 9, quality: 'medium' },
            { resolution: 256, kernelSize: 13, quality: 'high' },
            { resolution: 512, kernelSize: 17, quality: 'ultra' }
        ];
        
        this.currentLevel = 2; // Start at high quality
    }
    
    selectLevel(performanceMetrics) {
        const { fps, memory, cpu } = performanceMetrics;
        
        if (fps < 20 || memory > 0.8 || cpu > 0.9) {
            this.currentLevel = Math.max(0, this.currentLevel - 1);
        } else if (fps > 50 && memory < 0.5 && cpu < 0.7) {
            this.currentLevel = Math.min(this.levels.length - 1, 
                this.currentLevel + 1);
        }
        
        return this.levels[this.currentLevel];
    }
}
```

## Profiling and Monitoring

### Performance Profiler
```javascript
class PerformanceProfiler {
    constructor() {
        this.metrics = {
            convolution: [],
            rendering: [],
            total: [],
            memory: []
        };
        
        this.frameCount = 0;
        this.startTime = performance.now();
    }
    
    startFrame() {
        this.frameStartTime = performance.now();
    }
    
    endFrame() {
        const frameTime = performance.now() - this.frameStartTime;
        this.metrics.total.push(frameTime);
        
        this.frameCount++;
        
        if (this.frameCount % 60 === 0) {
            this.reportMetrics();
        }
    }
    
    measureOperation(name, operation) {
        const startTime = performance.now();
        const result = operation();
        const endTime = performance.now();
        
        if (!this.metrics[name]) {
            this.metrics[name] = [];
        }
        
        this.metrics[name].push(endTime - startTime);
        return result;
    }
    
    reportMetrics() {
        const avgTotal = this.average(this.metrics.total);
        const avgConvolution = this.average(this.metrics.convolution);
        const avgRendering = this.average(this.metrics.rendering);
        const fps = 1000 / avgTotal;
        
        console.log(`Performance Report:
            FPS: ${fps.toFixed(1)}
            Frame Time: ${avgTotal.toFixed(2)}ms
            Convolution: ${avgConvolution.toFixed(2)}ms
            Rendering: ${avgRendering.toFixed(2)}ms
            Memory: ${(performance.memory?.usedJSHeapSize / 1048576).toFixed(1)}MB
        `);
        
        // Clear old metrics
        Object.keys(this.metrics).forEach(key => {
            this.metrics[key] = this.metrics[key].slice(-100);
        });
    }
    
    average(array) {
        return array.reduce((sum, val) => sum + val, 0) / array.length;
    }
}
```

## Practice Problems

### Beginner Level
1. **Basic Profiling**
   - Add performance timing to existing code
   - Identify bottlenecks using browser dev tools
   - Measure frame rates and memory usage

2. **Simple Optimizations**
   - Optimize loop structures
   - Reduce unnecessary calculations
   - Implement basic caching

3. **Memory Management**
   - Fix memory leaks
   - Implement object pooling
   - Optimize array allocations

### Intermediate Level
1. **Algorithm Optimization**
   - Implement FFT convolution
   - Add separable kernel detection
   - Create adaptive resolution system

2. **Rendering Optimization**
   - Implement WebGL rendering
   - Add dirty region tracking
   - Optimize color mapping

3. **Parallel Processing**
   - Use Web Workers for convolution
   - Implement task scheduling
   - Optimize worker communication

### Advanced Level
1. **Advanced GPU Computing**
   - Implement compute shaders
   - Use WebGL2 for better performance
   - Create GPU-based FFT

2. **System-Level Optimization**
   - Optimize memory layout
   - Implement SIMD operations
   - Create custom allocators

3. **Performance Engineering**
   - Build comprehensive profiling system
   - Implement automatic optimization
   - Create performance regression tests

## Learning Resources

### Performance Optimization
1. **"High Performance JavaScript"** by Nicholas Zakas
2. **"Web Performance in Action"** by Jeremy Wagner
3. **"JavaScript: The Definitive Guide"** - Performance chapters

### Computer Architecture
1. **"Computer Organization and Design"** by Patterson & Hennessy
2. **"Computer Architecture: A Quantitative Approach"**
3. **"What Every Programmer Should Know About Memory"**

### Parallel Computing
1. **"Parallel Programming in OpenMP"**
2. **"Programming Massively Parallel Processors"**
3. **Web Workers API Documentation**

### Graphics Programming
1. **"WebGL Programming Guide"** by Kouichi Matsuda
2. **"OpenGL SuperBible"** - General GPU programming
3. **"GPU Gems"** series - Advanced techniques

### Profiling Tools
1. **Chrome DevTools**: Performance profiling
2. **Firefox Profiler**: Advanced profiling
3. **WebPageTest**: Performance analysis

### Online Resources
1. **MDN Web Performance**: Browser optimization guides
2. **Web.dev**: Modern web performance best practices
3. **GPU.js**: GPU computing in JavaScript

## Next Steps
After mastering performance optimization:
1. Study advanced research topics and extensions
2. Explore deployment and distribution options
3. Learn about mobile optimization techniques
4. Investigate cloud-based simulation scaling