# Serverless Finance: Moving Computational Finance From the Server to the Browser

## At a glance

**Keywords**: C++17, WebAssembly, gRPC, options pricing, Monte Carlo, serverless, edge computing

**The big idea**: What if your browser could run Monte Carlo simulations directly — no server, no cloud bill, no scaling problems? This project explores how far WebAssembly can take computational finance, and when you still need a backend.

**Three projects, one core**: `FinWasm` (browser-only), `FinGrpc` (backend-only), `FinFull` (both at once).

---

## The problem

Financial computations are expensive. A Monte Carlo option price needs 100K–1M simulated paths. A volatility surface calibration needs iterative optimization. A portfolio optimization solves a large linear system.

In a traditional client-server architecture, every user's computation hits your backend. That means:
- You pay for CPU you don't use most of the time
- You scale servers for peak load, not average load
- Every interactive slider drag fires a network round-trip

But WebAssembly runs C++ at near-native speed in the browser. What if the user's machine did the work instead?

---

## What this project explores

### 1. Can WASM replace the server for financial computation?

The `FinWasm` project compiles a full C++17 options pricing library to WebAssembly via Emscripten. The same Black-Scholes, Monte Carlo, and binomial tree code that would run on a server runs in a browser tab. The Flask dashboard is minimal — it serves static files and proxies market data from Yahoo Finance. The math runs client-side.

**What works well**:
- Single-option pricing: essentially instant (< 10ms WASM call)
- Monte Carlo with 10K paths: ~30ms — feels interactive
- Greeks via finite difference: ~50ms for a full set

**Where it struggles**:
- Monte Carlo with 1M+ paths: several seconds (but still faster than a network round-trip to a cold server)
- SABR calibration: the optimizer needs 100+ function evaluations, each running in WASM with no JIT — noticeably slower than native
- Memory: WASM heap is limited; large covariance matrices or long time series are constrained

**Verdict**: WASM is excellent for interactive pricing (what-if sliders, real-time risk). It's not a full server replacement for heavy compute tasks.

### 2. When WASM isn't enough, bring in gRPC

The `FinFull` project pairs the WASM frontend with a gRPC microservice. The browser handles fast interactive pricing locally. Heavy tasks (calibration, portfolio optimization, batch risk) call back to the gRPC server.

This hybrid model means:
- 90% of user interactions never touch a server
- The server only runs when it's genuinely needed
- Each machine pays for its own compute (the user's CPU)
- Server costs scale with data complexity, not user count

### 3. The gRPC service itself explores design options

`FinGrpc` implements the same pricing models as a gRPC microservice — sync, async, and callback server variants. It answers: when you do need a backend, what's the right architecture?

---

## The library

The common C++17 math library powers all three projects.

Utils (date/time, CSV) → MathStat (matrix, stats, interpolation, optimization)
  → Excel (NPV, IRR, bond pricing) → Options (pricing models, Greeks)
  → DataFrame / TimeSeries (columnar data)

15+ option pricing models: Black-Scholes, Monte Carlo, binomial trees, SABR, Heston, jump diffusion, American, Asian, Barrier, Digital, Lookback, and more.

---

## Build and explore

```bash
# Native (test executables for all three)
cd FinWasm && mkdir build && cd build && cmake .. && make

# WASM (needs Emscripten)
source source_em_env.bash && ./build.bash

# gRPC server (needs gRPC/Protobuf)
cd FinGrpc && mkdir build && cd build && cmake .. -DCMAKE_PREFIX_PATH=~/.local && make

# Run any server variant:
./OptionsServer/services/options_server
```

---

## Example user stories

- Trader: runs quick what-if Monte Carlo in the browser to validate an intraday hedging idea; uses the gRPC server only for overnight recalibration jobs.
- Risk analyst: runs batch portfolio risk on the server, uses WASM for interactive stress-testing on subsets of the portfolio.
- Educator / student: runs pricing demos entirely in-browser with no infra costs.

---

## Architecture summary (practical)

- Edge (WASM): interactive pricing, Greeks, small Monte Carlo runs (<100K paths), UI-driven scenarios.
- Backend (gRPC): calibration, batch processing, shared market data, long-running jobs, parallelized compute.

---

## Next steps and ideas

- Add lightweight scheduler that decides client vs server for a task based on estimated runtime and data footprint.
- Provide a provenance layer so calibrations done in-browser can be audited (hash input → server-store result).
