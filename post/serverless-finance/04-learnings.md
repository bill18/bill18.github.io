# What Building a Serverless Finance Platform Taught Me

**Part of the Serverless Finance series. Start with [article 1](01-overview.md) for the full context.**

This project was an exploration, not a production system. Here's what I learned about WASM, gRPC, C++17, and distributed computation in finance — the things I'd carry into a real engineering role.

---

## On WebAssembly

### WASM is ready for interactive compute

The idea that C++ runs at "near-native speed" in the browser is true for compute-bound numerical work. A Monte Carlo path generator in WASM runs at about 25% native speed — fast enough that the user can't tell the difference. For interactive tools (drag a slider, see a price update), WASM is production-ready today.

### WASM is not ready for heavy compute

Any workload that depends on parallelism, large memory, or native SIMD will struggle. Multi-threaded Monte Carlo becomes single-threaded. A 1000×1000 covariance matrix hits heap limits. The browser's security model is inherently at odds with high-performance computing.

The right framing: WASM replaces the **interactive layer** of a server. Not the server itself.

---

## On gRPC and server architecture

### Sync, async, callback — know the tradeoff

| Pattern | Complexity | Scalability | Debuggability |
|---|---|---|---|
| Sync | Low | Low | High |
| Async | Medium | High | Medium |
| Callback | High | High | Low |

### Shared evaluator pattern is the real win

The single design decision I'd keep in any system: **decouple business logic from transport**. The `options_evaluator.cpp` file has zero gRPC dependencies. It's a C++ function that takes a JSON string and returns a JSON string. You can call it from gRPC, from WASM, from a REST API, from a command-line tool — it doesn't care.

---

## On C++17 for financial computation

### The dependency chain pattern works

```
Utils (no dependencies)
  → MathStat (depends on Utils)
    → Excel (depends on MathStat + Utils)
      → Options (depends on Excel + MathStat + Utils)
        → DataFrame / TimeSeries (depends on Options + ...)
```

### Monorepo extraction is worth the effort

The code started with the same library copied into three project directories. Changes had to be synced manually. Divergence crept in. Extracting to a shared `common/` directory fixed the problem.

---

## On the serverless model for finance

### Where it works

- **Interactive risk tools**: Traders and analysts exploring scenarios. WASM handles the math locally. Server costs approach zero.
- **Client-side Monte Carlo**: For moderate path counts (< 100K), the browser outperforms a network round-trip to a server.
- **Educational platforms**: The student's browser runs the simulation. No server provisioning, no scalability problem.

### Where it doesn't (yet)

- **Real-time production trading**: Latency requirements (microseconds) exceed WASM's capabilities. You need native code on dedicated hardware.
- **Large-scale batch risk**: Running 1M+ simulations across thousands of instruments. The server is the right place.
- **Regulated environments**: Computation must be auditable and controlled. Running on user machines introduces compliance challenges.

---

## Recommended next steps

- Add small benchmarks (browser + server) for each model to produce a decision table: when to run local vs remote.
- Create a lightweight orchestrator (client library) that inspects estimated runtime and data footprint, then routes work to WASM or gRPC.
