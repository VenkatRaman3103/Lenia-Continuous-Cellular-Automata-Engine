# Lenia - Artificial Life in 2D Continuous Cellular Automata

## Overview

Lenia is a generalization of continuous cellular automata where naturalistic life forms reside in a virtual world. It extends traditional cellular automata like Conway's Game of Life into continuous space-time with smooth mathematical functions.

## Mathematical Foundation

### Core Equation

The field transition is governed by:
```
Δf(x) = d(k * f(x)) · Δt
```

Where:
- `f(x)` is the field value at position x
- `k` is the kernel function
- `d` is the delta function  
- `*` denotes convolution
- `Δt` is the time step

### Units
- **Space**: 1 μm (micrometer) divided into R cells
- **Time**: 1 μs (microsecond) divided into T steps  
- **Mass**: Mass density per cell (cell value in range [0,1] or [0,∞))

### Kernel Functions

#### Core Functions
1. **Gaussian Bump**: `c(r) = exp(α(1 - 1/k))` where `k = 4r(1-r)`
2. **Polynomial**: `c(r) = k^α` where `k = 4r(1-r)`
3. **Trapezoidal**: 5 equal portions
4. **Step**: For SmoothLife (outer radius = inner radius × 3)
5. **High Step**: For Conway's Game of Life

#### Layer Functions
1. **Unimodal**: `k(u) = c(r)`
2. **Bimodal**: Two peaks with different heights β₁, β₂
3. **Trimodal**: Three peaks with heights β₁, β₂, β₃
4. **Tetramodal**: Four peaks with heights β₁, β₂, β₃, β₄

### Delta Functions

1. **Gaussian**: `d(n) = 2e^(-ℓ²/k) - 1` where `ℓ = |n-μ|, k = 2σ²`
2. **Polynomial**: `d(n) = 2(1 - ℓ²/k²)^α - 1` if `ℓ ≤ k`, otherwise `-1`
3. **Trapezoidal**: Piecewise linear function
4. **Step**: `d(n) = 1` if `ℓ ≤ σ`, otherwise `-1`

### Key Parameters

- **μ (mu)**: Center of delta function
- **σ (sigma)**: Width of delta function  
- **R**: Space resolution (cells per kernel radius)
- **T**: Time resolution (steps per unit time)
- **α (alpha)**: Stiffness parameter
- **β (beta)**: Peak heights for multilayer kernels
- **η (eta)**: Mean peak height
- **Δt**: Time step size

## Life Forms (Solitons)

The system contains dozens of discovered stable life forms organized by taxonomy:

### Orbiformes Order
- **Orbidae Family**
  - *Orbium unicaudatus* (O2): Single-tailed orb
  - *Orbium bicaudatus* (O2): Double-tailed orb
  - *Gyrorbium* (OG2): Rotating orb
  - *Synorbium* (O3): Synthetic orb
  - *Parorbium dividuus* (O4): Divided orb
  - *Triorbium* (~O6): Triple orb

### Scutiformes Order  
- **Scutidae Family**
  - *Scutium solidus* (S1): Solid shield
  - *Scutium gravidus* (S1): Pregnant shield
  - *Scutium valvatus* (S1): Valved shield
  - *Scutium serratus* (2S1): Serrated shield variants
  
- **Discutium** (S2): Double shield variants
- **Triscutium** (S3): Triple shield variants  
- **Tetrascutium** (S4): Quadruple shield variants
- **Pentascutium** (S5): Quintuple shield variants
- **Hexascutium* (S6): Sextuple shield

### Other Orders
- **Pterifera**: Wing-bearing organisms
- **Anuliformes**: Single-ring organisms
- **Heliciformes**: Double-spiral organisms

## User Interface

### Main Display Panels
1. **Field Window** (`f(x)`): Shows current cellular automata state
2. **Delta Window** (`d(k*f(x))`): Shows delta function result
3. **Neighbor Sum Window** (`k*f(x)`): Shows convolution result
4. **Kernel Window** (`k(u)`): Shows kernel visualization

### Control Panels

#### Controls Panel
- **Run Controls**: Start/Stop, Once
- **Presets**: Lenia, SmoothLife, Game of Life, Default, Large
- **Field Controls**: Random, Again, Clear, Center, transformations
- **Soliton Controls**: Add at center/random, Previous/Next, expand/collapse groups
- **Display Options**: Auto-center, Colors, Visual aids, Window sets
- **Copy/Paste**: Format options, GIF generation

#### Calculations Panel
- Mathematical formulas and function visualizations
- Parameter adjustments for kernel and delta functions
- Real-time graph updates

#### Statistics Panel
- Performance metrics (FPS, time step)
- Geometric moments and image invariants
- Period detection and analysis

#### 3D Plot Panel
- Interactive 3D surface visualization
- Plot field, delta, neighbor sum surfaces

#### β Cube Panel
- 4D parameter space visualization
- Soliton existence marking system

## Keyboard Shortcuts

### Basic Controls
- `Enter/Shift+Enter`: Start/Stop running
- `Space`: Calculate one step
- `` ` `` (grave): Show About panel

### Number Keys
- `1-5`: Switch between control panels
- `6`: Lenia preset
- `7`: SmoothLife preset  
- `8`: Game of Life preset
- `9`: Default settings
- `0`: Large field

### Field Manipulation
- `N/Shift+N`: Add random cells
- `M/keypad .`: Repeat with same seed
- `Del`: Clear field
- `' (quote)/Shift+'`: Center/rotate object
- `Arrow keys/Shift+Arrows`: Move field
- `PgUp/PgDn/fn+↑/fn+↓`: Flip field
- `Home/End/fn+←/fn+→`: Rotate field
- `/ (slash)/Shift+/`: Double/halve field
- `, (comma)/Shift+,/. (dot)/Shift+.`: Mirror operations

### Parameter Adjustment
- `Q/W/A/S`: Decrease/increase delta function center/width
- `R/F/Shift+R/Shift+F`: Increase/decrease space resolution
- `T/G/Shift+T/Shift+G`: Dilute/concentrate time
- `E/D`: Decrease/increase adjustment step
- `Y/U/I/O/H/J/K/L`: Adjust peak heights
- `; (semicolon)`: Random peak heights

### Soliton Controls
- `Z`: Add soliton at center
- `X`: Add soliton randomly
- `C/keypad 1`: Previous soliton
- `V/keypad 2`: Next soliton
- `B/keypad 3`: Toggle enlarge option
- `P/Shift+P/Alt+P`: Mark soliton status

### Display Options
- `Alt+A`: Auto center/rotate/mirror
- `Alt+S`: Change color scheme
- `Alt+D`: Toggle visual aids
- `Alt+F`: Switch window set
- `Alt+C/V/X`: Copy/paste operations
- `Alt+G/H/K/N/Q/R/T/U/W/Y`: Various calculation and display options

### Advanced
- `INT+1/2`: Adjust kernel stiffness
- `INT+3/4`: Expand/collapse all soliton groups
- `ALT+F`: Click windows to switch display

## Technical Implementation

### File Structure
- `Lenia.html`: Main HTML interface
- `Lenia-LifeForms.js`: Core JavaScript implementation and life form data
- `libs/`: External libraries
  - `MersenneTwister19937.js`: Random number generation
  - `jsgif/`: GIF encoding functionality
  - `plotly-latest.min.js`: 3D plotting

### Core Algorithms
- FFT-based convolution for efficient neighbor sum calculation
- Image moment calculations for statistics
- Hu's and Flusser's image invariants
- Period detection algorithms
- GIF encoding for animation export

### Performance Features
- Variable resolution (R) for space discretization
- Time stepping (T) for temporal resolution
- GPU acceleration potential through canvas operations
- Optimized FFT algorithms

## Preset Configurations

### Lenia
- Continuous cellular automata with smooth kernels
- Default parameters for stable life forms

### SmoothLife
- Continuous version of Game of Life
- Larger neighborhoods, smooth transitions
- Step kernel function

### Game of Life  
- Discrete cellular automata
- High step kernel, specific parameters
- 8-neighbor Moore neighborhood

## Mathematical Background

### Convolution
The neighbor sum uses circular kernel convolution:
```
(k * f)(x) = (1/N) Σ|u|≤1 k(u) · f(x + u)
```
Where N is the normalization factor Σ|u|≤1 k(u).

### Image Moments
- **Raw moments**: `mpq = Σ x,y x^p y^q f(x,y)`
- **Central moments**: `μpq = Σ x,y (x-x̄)^p (y-ȳ)^q f(x,y)`
- **Complex moments**: `cpq = Σ x,y (x+iy)^p (x-iy)^q f(x,y)`

### Invariants
The system uses Hu's and Flusser's image invariants for:
- Rotation invariance
- Scale invariance  
- Pattern recognition
- Period detection

## Extensions and Customization

### Custom Life Forms
- Life forms are stored as compressed cell patterns
- Unicode-based encoding for sharing
- Parameters included in each life form definition

### Parameter Spaces
- 4D β cube for exploring parameter combinations
- Systematic soliton discovery
- Stability marking system

### Export Features
- GIF animation generation
- Data export in various formats
- Copy/paste functionality for sharing patterns

## References

### Academic Foundations
- Cellular Automata theory
- SmoothLife and Larger than Life
- Continuous spatial dynamics
- Image moment theory

### Libraries Used
- KaTeX for mathematical rendering
- Plotly.js for 3D visualization  
- jsgif for GIF encoding
- Mersenne Twister for randomization

This comprehensive documentation covers all aspects of the Lenia artificial life system, from mathematical foundations to practical usage and technical implementation details.