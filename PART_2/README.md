# Static Timing Analysis (STA) - Udemy Course

## Table of Contents
1. [Introduction to Timing Path and Arrival Time](#introduction-to-timing-path-and-arrival-time)  
2. [Timing Concepts](#timing-concepts)  
3. [Setup and Hold Analysis](#setup-and-hold-analysis)  
4. [Types of Timing Analysis](#types-of-timing-analysis)  
5. [Clock Analysis](#clock-analysis)  
6. [Slew, Load, and Clock Checks](#slew-load-and-clock-checks)  
7. [Timing Graph Analysis](#timing-graph-analysis)  

---

## Introduction to Timing Path and Arrival Time

### Timing Path
A **timing path** is formed between:
- **Start point**: Launch flip-flop  
- **End point**: Capture flip-flop  

The path includes:
- CLK to D  
- CLK to output  
- Input to output  

### Key Timing Definitions

**Arrival Time**  
Time required for the signal to reach the end point from the start point (calculated only at end points).

**Required Time**  
System specification defining when the signal is expected to arrive at a given pin.

**Example**  
At the D pin, the signal should arrive after 0.5 ns but before 3 ns (required time).

---

## Timing Concepts

### Slack Analysis
**Slack** = Required time − Arrival time  
Indicates whether data arrives early, on time, or too late.

- **Max Slack** (setup slack) = Required time − Arrival time  
- **Min Slack** (hold slack) = Arrival time − Required time  

### Setup and Hold Analysis

**Setup Time**  
Minimum time before the clock edge that data must be stable.

**Hold Time**  
Minimum time after the clock edge that data must remain stable.

| Analysis Type | Purpose                                   | Violation Condition          |
| ------------- | ----------------------------------------- | ---------------------------- |
| **Setup**     | Checks if data arrives before clock edge  | Arrival time is too late     |
| **Hold**      | Checks if data stays stable after edge    | Data changes too soon        |

---

## Types of Timing Analysis

### 1. Register-to-Register Analysis Types

| Type       | Description                | Path                 |
| ---------- | -------------------------- | -------------------- |
| **reg2reg**| Register to register path  | From flop to flop    |
| **in2reg** | Input to register path     | Input to flop        |
| **reg2out**| Register to output path    | Flop to output       |
| **in2out** | Input to output path       | Input to output      |

### 2. Clock Gating Analysis
Purpose: Reduce power consumption by gating the clock.  
Mechanism: When the gating signal is high, the clock enters the flop; when low, it is blocked.

### 3. Recovery/Removal Analysis
Applies to asynchronous pins (e.g., RESET).  
Ensures signals satisfy timing relative to clock edges.

### 4. Data-to-Data Analysis
Ensures synchronization between data signals (e.g., ‘a’ and ‘ctrl’).  
Requires special constraints for proper timing checks.

### 5. Latch Analysis (Time Borrowing)
Level-triggered latches can borrow or give time between launch and capture.  
- **Time Borrowing**: Latch borrows time when the FF-to-latch path is too long.  
- **Time Giving**: Latch gives time when the latch-to-FF path needs extra slack.

---

## Clock Analysis

### 1. Skew Analysis
Difference in arrival times of the clock at different endpoints.  
Affects both setup and hold checks.

### 2. Pulse Width Analysis
Ensures the clock pulse width remains within specified limits, accounting for degradation due to parasitics.

---

## Slew, Load, and Clock Checks

### Slew (Transition) Analysis
- **Too sharp slew**: Increases short-circuit power.  
- **Too slow slew**: Increases propagation delay.

Check both data slew and clock slew (clock slew constraints are tighter).

### Load Analysis
1. **Fanout** (min/max) – More fanout increases load.  
2. **Capacitance** (min/max) – Ensure capacitance stays within allowable limits.

---

## Timing Graph Analysis

### Setup Analysis Example
- Clock frequency = 1 GHz  
- Clock period = 1 ns  

Components:
- Cell delays  
- Wire delays  
- Signal arrival times  

Converted to a Directed Acyclic Graph (DAG) where gates become nodes.

**Actual Arrival Time (AAT)**  
Latest time a signal transition occurs relative to the clock.

**Required Arrival Time (RAT)**  
Latest allowable time for the transition based on downstream requirements.

**Slack** = RAT − AAT  

### Analysis Methods

| Method | Description                          | Use Case                  |
| ------ | ------------------------------------ | ------------------------- |
| **GBA**| Graph Based Analysis                 | Worst-case path analysis  |
| **PBA**| Path Based Analysis                  | Real-time path checks     |

---

## Key Takeaways

1. **Purpose of STA**: Verify timing constraints without full simulation.  
2. **Slack**: Central metric indicating timing margin.  
3. **Multiple Analyses**: Different checks for registers, inputs, outputs, clocks, and latches.  
4. **Clock Integrity**: Skew and pulse width are critical for reliable timing.  
5. **Graph Approach**: Model circuits as DAGs to systematically compute arrival and required times.  
6. **Optimization**: Resolve negative slack by reducing cell or wire delays.  

