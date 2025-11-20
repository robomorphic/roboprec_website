# RoboPrec Framework Overview

RoboPrec is a framework for compiling robotics algorithms to embedded platforms with **provable numerical accuracy guarantees** across mixed-precision datatypes. It combines a Rust-based frontend, formal analysis of numerical error, and optimized C / fixed-point code generation.

RoboPrec was originally introduced in the paper:

> **RoboPrec: Enabling Reliable Embedded Computing for Robotics by Providing Accuracy Guarantees Across Mixed-Precision Datatypes**  
> Alp Eren Yilmaz, Thomas Bourgeat, Lillian Pentecost, Brian Plancher, Sabrina M. Neuman

---

## High-level goals

RoboPrec is designed to help robotics practitioners:

- Safely deploy compute-heavy robotics kernels (e.g., rigid body dynamics) on **low-power embedded hardware**.
- Systematically explore **trade-offs between datatype, performance, and accuracy**.
- Obtain **formal, worst-case error bounds** instead of ad-hoc testing.
- Integrate with existing robotics stacks by generating **plain C or ap_fixed-style code**.

At a high level, RoboPrec answers questions like:

- *"Can I run this algorithm with 32-bit fixed-point on a microcontroller and still be safe?"*
- *"How much performance do I gain by moving from double to fixed-point, and what is the worst-case numerical error?"*

---

## Framework Overview

This page gives you the **mental model** for RoboPrec in one screen.

---

### What RoboPrec does

RoboPrec takes a robotics kernel written in Rust and produces **verified C / fixed-point code**:

1. You write the algorithm using RoboPrec’s linear algebra types (`Scalar`, `Vector`, `Matrix`).
2. RoboPrec **unrolls and simplifies** it into an analysis-friendly DSL.
3. A back-end analysis tool (currently Daisy) computes **ranges and worst‑case errors** for a chosen precision.
4. RoboPrec generates **optimized C code** (float, double, or fixed-point) that respects those bounds.

You get code you can deploy on microcontrollers, CPUs, or FPGAs—with a quantitative story about numerical error.

---

### Three key stages

**1. Transpiler (Rust → DSL)**

- Input: Rust code using `Scalar` / `Vector` / `Matrix` and robot models.
- Actions:
  - Unroll loops.
  - Expand matrix/vector ops into scalar arithmetic.
- Output: a straight‑line program in an analysis DSL.

**2. Error analysis**

- Input: simplified program + input ranges + target precision.
- Uses Daisy to compute for every variable:
    - Value range.
    - Worst-case roundoff error.
- Supports:
    - `Float32`, `Float64`.
    - Fixed-point in **mixed** or **uniform** precision modes.
    - **mixed** version sets the minimum number of integer bits for each variable
    - **uniform** version expects the user to set the integer bits for each variable

**3. Code generation**

- Produces C code matching the chosen datatype:
  - Plain `double` / `float`.
  - Integer-emulated fixed-point (or `ap_fixed`-style for HLS).
- Code is straight‑line and vectorization‑friendly, ideal for embedded targets and SIMD.


### Limitations

RoboPrec can only handle programs with **static dataflow**. Therefore, conditionals and loops must be handled carefully. For example, the following program is valid, because the executed operations are the same regardless of inputs:

```rust
// 1. Define input 'x' with range [0.0, 1.0] and a default value for testing
let input_range = (Real::from_f64(0.0), Real::from_f64(1.0));
let default_val = 0.5;
let mut x = add_input_scalar("x", input_range, default_val);

// 2. Perform the computation
for i in 0..3 {
    println!("Iteration {}", i);
    x = &x * &x;
    if i == 0 {
        x = x + Scalar!(1.0);
    }
    if i == 2 {
        register_scalar_output(&mut x, "intermediate_output");
    }
}
...
```



---

## Examples

- Rigid-body dynamics kernels:
    - Forward kinematics (FK)
    - RNEA
    - RNEA derivatives

- Robot models:
    - RoArm‑M2 (4‑DOF) 
    - RoArm‑M3 (5‑DOF)
    - Indy7 (6‑DOF)
    - Franka Emika Panda (7‑DOF)

These serve both as **benchmarks** (see [Benchmarks](benchmarks.md)) and as **reference examples** for writing your own kernels.

