# What a gRPC Server Is For When WASM Does the Math

**Part of the Serverless Finance series. Start with [article 1](01-overview.md) for the full context.**

If WASM can run pricing in the browser, why build a gRPC server at all? Because not every financial workload belongs on the client. This project built both — and the distinction taught me where each architecture belongs.

---

## Three things a gRPC backend does that WASM can't

### 1. Heavy compute that doesn't belong in a browser tab

A SABR calibration takes 800ms in WASM. That's 800ms where the browser tab is unresponsive. In native C++, it's 90ms. On a server with 16 cores, it's 90ms that doesn't block anyone's UI.

The gRPC server runs the same `options_evaluator.cpp` as the WASM build, but with the full power of the host machine — multiple cores, native SIMD, no memory constraints. The client fires a request and gets a result asynchronously.

### 2. Shared state across users

Market data, calibrated volatility surfaces, and historical time series are shared resources. If every user's browser downloads and processes their own copy, you're wasting bandwidth and duplicating work. A gRPC server maintains a single source of truth and serves results on demand.

### 3. Batch and background processing

Pricing 10,000 options for a portfolio report makes no sense in a browser. It makes perfect sense as a server-side batch job. The gRPC service handles it, returns the result, and the browser displays it.

---

## Server patterns explored

The project implements three gRPC server variants, each answering a different question:

**Sync server** — Simple, one thread per request. Works fine for sub-100ms pricing requests at moderate concurrency. Easy to reason about, hard to scale.

**Async server** — CompletionQueue-based, non-blocking. A single thread handles hundreds of concurrent requests. Better resource utilization, but the state machine code is harder to read.

**Callback server** — Returns a reactor immediately, schedules work on a thread pool. Cleanest separation of concerns, but requires understanding the reactive pattern.

They all delegate to the same `options_evaluator.cpp`. The server pattern and the business logic are completely decoupled — you can swap one without touching the other.

---

## The hybrid workflow in practice

```
Browser                    gRPC server
  │                           │
  ├─ WASM: price option ──→   │ (no call needed — local)
  ├─ WASM: adjust vol ────→   │ (no call needed — local)
  ├─ WASM: get Greeks ────→   │ (no call needed — local)
  │                           │
  ├─ gRPC: calibrate ─────→   ├─ Runs SABR optimization (multi-threaded)
  │ ←──────────────────────┘  │
  │                           │
  ├─ gRPC: portfolio risk ──→ ├─ Batch prices 5000 options
  │ ←──────────────────────┘  │
```

---

## Running the server

```bash
cd FinFull
mkdir -p build && cd build
cmake .. -DCMAKE_PREFIX_PATH=~/.local    # needs gRPC/Protobuf
make -j$(nproc)

# Three ways to serve the same pricing engine:
./OptionGRPCServer/services/sync_option_server
./OptionGRPCServer/services/async_option_server
./OptionGRPCServer/services/callback_option_server
```

---

## Deployment patterns & considerations

- Co-locate gRPC services with fast storage for market data (NVMe) to reduce I/O bottlenecks for calibrations.
- Use a job queue for batch risk tasks (Celery / RabbitMQ / native work-queues) and let gRPC services pull work items.
