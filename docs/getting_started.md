# Getting Started

## Prerequisites

Before you begin, ensure you have the Rust programming language installed. If you do not have it, you can install it using [rustup](httpss://rustup.rs/).

## Supported Platforms

- ✅ Ubuntu (tested on 22.04)
- ❌ macOS (work in progress)
- ❌ Windows

## Installation

To get started, clone the repository and build with Cargo.

If you want to work on RoboPrec directly:

```bash
git clone https://github.com/robomorphic/roboprec
cd roboprec
cargo build --release
```

The compiled binary will be in `target/release`.

If you want to use RoboPrec as a library in your own project, add it to your `Cargo.toml`:

```toml
[dependencies]
roboprec = { git = "https://github.com/robomorphic/roboprec.git" }
```



## Quick Start

The following example demonstrates how to define a simple computation, analyze it, and inspect the results.

### 1. Define the Computation in Rust

First, we write a Rust program that defines the computation we want to analyze. In this example, we'll compute $x^3$.

We specify:

1.  **Inputs**: Variable `x` with a range of $[0.0, 1.0]$.
2.  **Computation**: `x * x * x`.
3.  **Outputs**: The result is registered as `output`.
4.  **Configuration**: We choose the target precision (e.g., `Float64` or `Fixed`) and output location.

```rust
// 1. Define input 'x' with range [0.0, 1.0] and a default value for testing
let input_range = (Real::from_f64(0.0), Real::from_f64(1.0));
let default_val = 0.5;
let x = add_input_scalar("x", input_range, default_val);

// 2. Perform the computation
let mut res = x * x * x;

// 3. Register the result
register_scalar_output(&mut res, "output");

// (Optional) Verify correctness in Rust
assert!(res.value.to_f64() == default_val.powi(3));

// 4. Configure and run analysis
let config = Config {
    precision: Precision::Float64, // Target precision
    codegen_dir: PathBuf::from("output/"),
    codegen_filename: "codegen".to_string(),
};

analysis(config);
```

### 2. Generate C Code

RoboPrec automatically generates C code based on your configuration.

#### Option A: Double Precision (`Float64`)

If we select `Precision::Float64`, the pipeline generates standard C code using `double`:

```C
typedef struct {
    double output;
} codegen_output_t;

codegen_output_t codegen(
    double x
) {
    double x_mul_x = (x * x);
    double x_mul_x_mul_x = (x_mul_x * x);
    double output = x_mul_x_mul_x;

    return {
        output,
    };
}
```

#### Option B: Fixed-Point Optimization

If we switch the configuration to **fixed-point** (e.g., `Precision::Fixed { total_bits: 16, fractional_bits: -1 }`), RoboPrec performs a powerful optimization:

1.  It automatically selects the optimal number of fractional bits for each variable to maximize precision while preventing overflow.
2.  It generates bit-exact C code using integer arithmetic.

```C
typedef struct {
  int16_t output;
} codegen_output_t;

codegen_output_t codegen(int16_t x) {
    // RoboPrec automatically handles shifting and casting
    int16_t x_mul_x = (int16_t) ((((int32_t) (x) * (int32_t) (x)) >> 14));
    int16_t x_mul_x_mul_x = (int16_t) ((((int32_t) (x_mul_x) * (int32_t) (x)) >> 14));
    int16_t output = x_mul_x_mul_x;

    return { output };
}
```

**Why `int32_t` casts?**

Even though the variables are stored as `int16_t`, intermediate calculations (like multiplication) are cast to `int32_t`. This is necessary because multiplying two 16-bit numbers produces a 32-bit result. RoboPrec handles these promotions automatically to prevent overflow before shifting back down.

**Why choose this approach?**

This version is highly efficient for embedded systems:

*   **Smaller I/O**: Inputs and outputs are 16-bit integers instead of 64-bit doubles.
*   **Faster Arithmetic**: Integer operations are typically faster than floating-point emulation on microcontrollers.

#### Option C: Fixed-Point with Automatic Conversion

If you want the speed of fixed-point internals but need to integrate with an existing system that uses `double`, RoboPrec can generate **automatic conversion wrappers**.

This allows you to pass `double` in and get `double` out, while the heavy lifting is done in optimized fixed-point arithmetic:

```C
typedef struct {
    double output;
} codegen_output_t;

codegen_output_t codegen(double _double_x) {
    // 1. Convert input double to fixed-point (int16_t)
    int16_t x = (int16_t)(_double_x * (1 << 14)); 

    // 2. Perform optimized fixed-point arithmetic
    int16_t x_mul_x = (int16_t) ((((int32_t) (x) * (int32_t) (x)) >> 14));
    int16_t x_mul_x_mul_x = (int16_t) ((((int32_t) (x_mul_x) * (int32_t) (x)) >> 14));
    int16_t output = x_mul_x_mul_x;

    return {
        // 3. Convert result back to double
        ((double)output) / (1 << 14), 
    };
}
```

### 3. Analysis Results

The analysis provides detailed reports on the value range and accumulated error for each variable in the program.

#### Range Analysis

The following table shows the inferred value range for each variable for the example code above.

| Variable        | Range  |
| :-------------- | :----- |
| `output`        | `[0, 1]` |
| `x_mul_x_mul_x` | `[0, 1]` |
| `x_mul_x`       | `[0, 1]` |
| `x`             | `[0, 1]` |

#### Error Analysis

This table presents the worst-case accumulated error for each variable for the example code above.

| Variable        | Accumulated Error (±) |
| :-------------- | :-------------------- |
| `output`        | `4.66e-9`             |
| `x_mul_x_mul_x` | `4.66e-9`             |
| `x_mul_x`       | `2.79e-9`             |
| `x`             | `9.31e-10`            |
