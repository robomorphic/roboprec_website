# Getting Started

## Prerequisites

Before you begin, ensure you have the Rust programming language installed. If you do not have it, you can install it using [rustup](httpss://rustup.rs/).

## Supported Platforms

- ✅ Ubuntu
- ✅ macOS
- ❌ Windows

## Installation

To get started with the project, you will need to clone the repository and compile the source code. The build process is managed by Cargo, which will automatically handle all dependencies.

```bash
git clone https://github.com/robomorphic/roboprec
cd roboprec
cargo build --release
```

This will create the release binary in the `target/release` directory.


## Quick Start

The following example demonstrates how to define a simple computation, analyze it, and inspect the results.

### 1. Rust Program

First, we define the computation in Rust. This involves specifying inputs, performing operations, and registering outputs.

```rust
// Define the input variable 'x' with its range and a default value.
let input_range: (Real, Real) = (Real::from_f64(0.0), Real::from_f64(1.0));
let default_val = 0.5;
let x = &get_program().add_input_scalar("x", input_range, default_val);

// Perform the computation, in this case, cubing the input.
let mut res = x * x * x;

// Register the result as a scalar output named "output".
register_scalar_output(&mut res, "output");

// Optionally, assert the correctness of the computation in Rust.
assert!(res.value.to_f64() == default_val.powi(3));

// Trigger the main analysis process.
analysis_main();
```

### 2. Generated C Code

The analysis tool compiles the Rust program into the following C code, annotating it with the calculated preconditions and error bounds.

```C
/* @pre: ((x > 0.0) && (x < 1.0)) */
long codegen(long x) {
  int x_mul_x = (int) ((((long) (x) * (long) (x)) >> 30));
  int x_mul_x_mul_x = (int) ((((long) (x_mul_x) * (long) (x)) >> 30));
  int output = x_mul_x_mul_x;
  return output;
} // [0.0, 1.0] +/- 4.6566128765468395e-09
```

### 3. Analysis Results

The analysis provides detailed reports on the value range and accumulated error for each variable in the program.

#### Range Analysis

The following table shows the inferred value range for each variable.

| Variable        | Range  |
| :-------------- | :----- |
| `output`        | `[0, 1]` |
| `x_mul_x_mul_x` | `[0, 1]` |
| `x_mul_x`       | `[0, 1]` |
| `x`             | `[0, 1]` |

#### Error Analysis

This table presents the worst-case accumulated error for each variable.

| Variable        | Accumulated Error (±) |
| :-------------- | :-------------------- |
| `output`        | `4.66e-9`             |
| `x_mul_x_mul_x` | `4.66e-9`             |
| `x_mul_x`       | `2.79e-9`             |
| `x`             | `9.31e-10`            |
