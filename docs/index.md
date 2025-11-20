# Welcome to RoboPrec

RoboPrec is a verification‑aware compiler pipeline for **numerical workloads**, built and evaluated on **robotics kernels** (rigid body dynamics, kinematics, etc.), but applicable to any computation that can be written in its Rust frontend.

It is especially useful when you want to:

- Run robotics or other numeric kernels on **microcontrollers or embedded SoCs** with **fast fixed-point** instead of emulated `double`.
- Still benefit from fixed-point or mixed precision on **desktop/server CPUs**, where some kernels can be faster than `double`.
- Know **exactly how much numerical error you introduce** and prove it stays within bounds.
- Generate **optimized C code** that drops into existing C/C++ stacks.

Under the hood, RoboPrec is a Rust library and analysis + codegen pipeline that connects your code to formal numerical analysis tools, then back to C code.

---

## Why RoboPrec

- **Robotics‑first, numerics‑general**: designed around rigid body dynamics and robot models, but the analysis + codegen pipeline works for any scalar/vector/matrix computation.
- **Embedded and desktop**:
	- On microcontrollers, fixed-point often gives **order‑of‑magnitude speedups** over `double` with formal guarantees.
	- On modern CPUs, we show that fixed‑point can still **match or beat `double`** on some hot kernels.
- **From types to guarantees**:
	- Static **range and error analysis** per variable.
	- Support for `Float32`, `Float64`, and **fixed-point** (mixed or uniform).
	- Verified code ready for **CPUs, microcontrollers, or FPGAs**.

Headline results from the paper:

- Up to **122x faster** than `double` on some microcontrollers.
- On desktop CPUs, 32‑bit fixed-point can **match or beat `double`** on key kernels.
- For many dynamics workloads, **fixed32 is *more accurate* than `float`** in worst‑case bounds.

---

## Next steps

- **New here?** Start with [Getting Started](getting_started.md).
- **Curious about the internals?** See [Framework Overview](framework.md).
- **Want the numbers?** Check [Benchmarks](benchmarks.md).
- **Citing RoboPrec?** See [Publications](publications.md).