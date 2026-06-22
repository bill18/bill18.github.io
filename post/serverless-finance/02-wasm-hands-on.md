# What I Learned Pushing C++ Financial Models Into a Browser Tab

**Part of the Serverless Finance series. Start with [article 1](01-overview.md) for the full context.**

I wanted to know: can WebAssembly really handle computational finance workloads, or is it just a demo technology? I took a real options pricing library and compiled it to WASM. Here's what worked, what didn't, and where the line is.

---

## What running the math in the browser feels like

Open the dashboard, drag a strike slider, and the option price updates in real time. Not from a server — from the C++ code running in the tab. Call it, get a price, adjust vol, get a new price. All client-side, no network calls, no server CPU.

The pipeline: C++17 → Emscripten + Embind (auto-generates JS bindings) → WASM module → JavaScript calls pricing functions directly → Flask serves the page and proxies market data.

---

## The good: interactive pricing is genuinely fast

| Operation | WASM time | Native time | Ratio |
|---|---|---|---|
| Black-Scholes (single price) | ~2μs | ~0.5μs | 4× |
| Binomial tree (500 steps) | ~0.5ms | ~0.1ms | 5× |
| Monte Carlo (10K paths) | ~30ms | ~8ms | 3.75× |
| Finite difference Greeks (full set) | ~50ms | ~12ms | 4× |
| SABR calibration (single expiry) | ~800ms | ~90ms | 9× |

The overhead is consistent (~4×) and comes from WASM's sandbox — no JIT warmup, no native SIMD (without explicit intrinsics), and a 32-bit memory model. But 30ms for a Monte Carlo is **well within interactive territory**.

---

## The bad: heavy compute hits a wall

Monte Carlo with 1M paths: **~3 seconds** in WASM vs ~0.8s native. That's noticeable. The bottleneck isn't WASM speed — it's the memory model. WASM's 32-bit address space limits heap size, and every memory access goes through bounds-checking. For algorithms that are memory-bandwidth-bound (large covariance matrices, time series with thousands of rows), this adds real overhead.

---

## The ugly: what doesn't work

- **Parallelism**: WASM has no threads in the browser (SharedArrayBuffer requires specific headers and still has limitations). A multi-threaded Monte Carlo becomes single-threaded. This is the single biggest limitation.
- **SIMD**: WASM SIMD exists but Emscripten's auto-vectorization is immature. Hand-tuned SIMD code in C++ often doesn't carry over.
- **Large datasets**: Loading a 10-year daily time series into WASM memory is fine (~200KB). Loading a 1000×1000 covariance matrix (8MB) starts straining the heap.
- **Cold start**: The first WASM module load is ~500ms for this codebase (after gzip: ~200KB). Acceptable but noticeable on a fresh page load.

---

## The hybrid answer

This is why the project has two other targets. The `gRPC` project runs the same library as a backend service. The `FinFull` project combines both: WASM for instant interactive work, gRPC for anything that needs parallelism, large memory, or shared state.

The mental model:

```
User drags slider     →  WASM (sub-50ms, no server)
User clicks "calibrate" →  gRPC (server handles optimization)
User requests batch risk → gRPC (server parallelizes)
```

---

## Getting started

```bash
# Clone and build WASM
cd FinWasm
source source_em_env.bash    # activates Emscripten
./build.bash                  # produces .wasm + .js in dashboard/static/lib/

# Start the dashboard
cd dashboard
pip install flask yfinance
python3 app.py
# Open http://localhost:5000
```

---

## Optimization tips

- Use smaller data views: sample time series client-side when testing; send full data to server only for final runs.
- Precompute heavy factors (e.g., Cholesky decompositions) on the server and stream compact results to the client.
- Enable WASM SIMD and Threads where the target browsers allow it; fallback gracefully.

---

No server needed for pricing. Just a browser.
