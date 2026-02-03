# Lenia - UI Controls Documentation

## Overview
This document provides a comprehensive UI/UX guide for all controls in the Lenia artificial life simulator. Controls are organized by panel and functionality, with clear descriptions of actions, results, and user interactions.

---

## 1. Main Panel Navigation

### Panel Switcher Buttons
| Control | Label | Action | Result | Keyboard Shortcut |
|---------|-------|--------|--------|-------------------|
| Button | Controls | Opens the Controls panel | Displays simulation control interface | `1` |
| Button | Calculations | Opens the Calculations panel | Shows calculation details and formulas | `2` |
| Button | Statistics | Opens the Statistics panel | Displays simulation statistics and graphs | `3` |
| Button | 3D Plot | Opens the 3D Plot panel | Shows 3D surface plot visualization | `4` |
| Button | β Cube | Opens the Beta Cube panel | Displays β parameters adjustment interface | `5` |
| Button | About | Opens the About panel | Shows application information and help | ` `` ` (grave accent) |

---

## 2. Parameters Section

### Core Parameter Controls

#### Mu (μ) - Delta Function Center
| Control | Label | Action | Result | Value Range | Keyboard Shortcut |
|---------|-------|--------|--------|-------------|-------------------|
| Button | `+` | Increases mu parameter | Shifts delta function center upward | 0.0 to 1.0 | `Q` or `keypad 7` |
| Button | `−` | Decreases mu parameter | Shifts delta function center downward | 0.0 to 1.0 | `A` or `keypad 4` |
| Display | μ = [value] | Shows current value | Real-time parameter display | - | - |

#### Sigma (σ) - Delta Function Width
| Control | Label | Action | Result | Value Range | Keyboard Shortcut |
|---------|-------|--------|--------|-------------|-------------------|
| Button | `+` | Increases sigma parameter | Widens delta function curve | 0.0 to 1.0 | `W` or `keypad 8` |
| Button | `−` | Decreases sigma parameter | Narrows delta function curve | 0.0 to 1.0 | `S` or `keypad 5` |
| Display | σ = [value] | Shows current value | Real-time parameter display | - | - |

#### R - Space Resolution
| Control | Label | Action | Result | Value Range | Keyboard Shortcut |
|---------|-------|--------|--------|-------------|-------------------|
| Button | `+` | Increases resolution | More cells per kernel radius | 1 to 64 | `R` or `keypad +` |
| Button | `−` | Decreases resolution | Fewer cells per kernel radius | 1 to 64 | `F` or `keypad -` |
| Display | R = [value] | Shows current cells per kernel | Real-time resolution display | - | - |

#### T - Time Steps
| Control | Label | Action | Result | Value Range | Keyboard Shortcut |
|---------|-------|--------|--------|-------------|-------------------|
| Button | `+` | Dilutes time | Increases steps per unit time | 1 to 100 | `T` or `keypad *` |
| Button | `−` | Concentrates time | Decreases steps per unit time | 1 to 100 | `G` or `keypad /` |
| Display | T = [value] | Shows current steps | Real-time time steps display | - | - |

#### Adjustment Step Size
| Control | Label | Action | Result | Value Range | Keyboard Shortcut |
|---------|-------|--------|--------|-------------|-------------------|
| Button | `↑` | Increases adjustment step | Larger parameter changes | 0.001 to 0.1 | `D` or `keypad 6` |
| Button | `↓` | Decreases adjustment step | Smaller parameter changes | 0.001 to 0.1 | `E` or `keypad 9` |
| Display | ± [value] | Shows current step size | Real-time adjustment display | - | - |

### Peak Height Controls (β Parameters)

#### Peak 1 Height
| Control | Action | Result | Value Range | Keyboard Shortcut |
|---------|--------|--------|-------------|-------------------|
| Button `+` | Increases peak 1 height | Adjusts growth function curve | 0.0 to 2.0 | `Y` |
| Button `−` | Decreases peak 1 height | Adjusts growth function curve | 0.0 to 2.0 | `H` |

#### Peak 2 Height
| Control | Action | Result | Value Range | Keyboard Shortcut |
|---------|--------|--------|-------------|-------------------|
| Button `+` | Increases peak 2 height | Adjusts growth function curve | 0.0 to 2.0 | `U` |
| Button `−` | Decreases peak 2 height | Adjusts growth function curve | 0.0 to 2.0 | `J` |

#### Peak 3 Height
| Control | Action | Result | Value Range | Keyboard Shortcut |
|---------|--------|--------|-------------|-------------------|
| Button `+` | Increases peak 3 height | Adjusts growth function curve | 0.0 to 2.0 | `I` |
| Button `−` | Decreases peak 3 height | Adjusts growth function curve | 0.0 to 2.0 | `K` |

#### Peak 4 Height
| Control | Action | Result | Value Range | Keyboard Shortcut |
|---------|--------|--------|-------------|-------------------|
| Button `+` | Increases peak 4 height | Adjusts growth function curve | 0.0 to 2.0 | `O` |
| Button `−` | Decreases peak 4 height | Adjusts growth function curve | 0.0 to 2.0 | `L` |

### Information Displays
| Display | Label | Shows | Update Frequency |
|---------|-------|-------|------------------|
| Text | β = [value] | Beta parameter value | Real-time |
| Text | η = [value] | Eta parameter value | Real-time |
| Text | Δt = [value] μs | Delta time in microseconds | Each calculation |
| Text | t = [value] μs | Total elapsed time | Each calculation |
| Text | Field: [value] cells | Field dimensions | On resize |
| Text | each [value] pixels | Pixel size per cell | On resize |
| Text | CPU: [value] fps | Frames per second | Real-time |

---

## 3. Calculation Statistics Section

### Statistics Controls
| Control | Label | Action | Result | Keyboard Shortcut |
|---------|-------|--------|--------|-------------------|
| Button | Reset | Resets all statistics | Clears calculation history and counters | `Alt + B` |
| Button | [Calc Level] | Toggles calculation level | Changes detail level of calculations | `Alt + N` |
| Button | [Display Values] | Changes displayed values | Switches between different value displays | `Alt + M` |
| Display | Period: [value] μs | Shows calculation period | Updates with each calculation cycle | - |

---

## 4. Controls Panel

### Run Controls
| Control | Label | Action | Result | Keyboard Shortcut |
|---------|-------|--------|--------|-------------------|
| Button | Start/Stop | Toggles simulation | Starts or stops continuous calculation | `Shift + Enter` or `Enter` |
| Button | Once | Performs single step | Calculates one time step only | `Space` |

### Preset Rules
| Control | Label | Action | Result | Keyboard Shortcut |
|---------|-------|--------|--------|-------------------|
| Button | Lenia | Loads Lenia rules | Switches to Lenia cellular automaton | `6` |
| Button | SmoothLife | Loads SmoothLife rules | Switches to SmoothLife CA | `7` |
| Button | Game of Life | Loads GoL rules | Switches to Conway's Game of Life | `8` |
| Button | Default | Resets to default | Restores default settings | `9` |
| Button | Large | Expands field | Creates larger simulation field | `0` |

### Field Manipulation Controls

#### Basic Field Operations
| Control | Label | Action | Result | Keyboard Shortcut |
|---------|-------|--------|--------|-------------------|
| Button | Random | Adds random cells | Inserts random patterns in field | `Shift + N` |
| Button | Again | Repeats random | Uses same seed for reproducibility | `M` or `keypad .` |
| Button | Clear | Clears field | Removes all cells from field | `Delete` |
| Button | Center | Centers/rotates object | Centers and/or rotates field content | `Shift + '` |
| Button | Repeat | Repeats last action | Redo last add/paste operation | `keypad 0` |

#### Field Transformation Controls
| Control | Label | Action | Result | Keyboard Shortcut |
|---------|-------|--------|--------|-------------------|
| Button | ↔ | Flip horizontally | Mirrors field left-right | `Page Up` or `fn + ↑` |
| Button | ↺ | Rotate anti-clockwise | Rotates field 90° left | `Shift + Home` or `fn + ←` |
| Button | ↑ | Move field up | Shifts field upward | `Shift + ↑` |
| Button | ↻ | Rotate clockwise | Rotates field 90° right | `Shift + End` or `fn + →` |
| Button | ×2 | Double field | Scales field size by 2x | `/` |
| Button | ↖ | Mirror diagonal | Mirrors horizontally/diagonally | `Shift + ,` |

#### Resolution Controls
| Control | Label | Action | Result | Keyboard Shortcut |
|---------|-------|--------|--------|-------------------|
| Button | − | Decrease resolution | Lowers space resolution | `[` |
| Button | + | Increase resolution | Raises space resolution | `]` |
| Button | 0 | Reset resolution | Returns to default resolution | `\` |

---

## 5. User Interaction Guidelines

### Visual Feedback
- **Button States**: Buttons show hover effects with color changes
- **Disabled States**: Some buttons show `btnAddOff` class when not applicable
- **Real-time Updates**: All parameter displays update immediately
- **Color Coding**: Parameters use blue color (μ, σ, R, T, β, η)

### Keyboard Navigation
- **Direct Access**: Most controls have keyboard shortcuts
- **Number Keys**: Quick access to panel switches (1-5)
- **Arrow Keys**: Field movement and parameter adjustment
- **Modifier Keys**: Shift and Alt modify button behaviors

### Performance Considerations
- **CPU Monitor**: Real-time FPS display for performance tracking
- **Resolution Impact**: Higher R values require more processing power
- **Field Size**: Larger fields impact calculation time
- **Calculation Level**: Higher detail levels affect performance

---

## 6. Tips and Best Practices

### For Beginners
1. Start with **Start/Stop** to see basic animation
2. Use **Random** to generate initial life forms
3. Try different **Preset** rules to explore behaviors
4. Adjust **R** and **T** to balance quality vs performance

### For Advanced Users
1. Use **β parameters** for fine-tuning life forms
2. Combine **field transformations** for complex patterns
3. Monitor **statistics** for scientific analysis
4. Use **keyboard shortcuts** for efficient workflow

### Troubleshooting
- **Low FPS**: Decrease R or field size
- **No Activity**: Try different presets or add random cells
- **Unstable Patterns**: Adjust μ and σ parameters
- **Performance Issues**: Use calculation level settings

---

## 7. Control Summary

### Essential Controls
- **Start/Stop**: Primary simulation control
- **Random**: Quick pattern generation
- **Clear**: Reset workspace
- **Presets**: Switch between CA rules

### Fine-Tuning Controls
- **μ, σ**: Core shape parameters
- **β peaks**: Growth function tuning
- **R, T**: Resolution and time balance

### Advanced Controls
- **Field transformations**: Geometric manipulations
- **Statistics**: Scientific analysis tools
- **3D Plot**: Visualization of parameter spaces

This documentation provides a complete reference for all UI controls in Lenia, enabling users to fully utilize the simulator's capabilities for exploring artificial life forms.