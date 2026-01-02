# Spindle

**A Rust-inspired, memory-safe programming language for quantum computing.**

Spindle brings the ownership and borrowing principles that made Rust revolutionary to the quantum realm—where linearity isn't just good practice, it's physics.

---

## Why Spindle?

Quantum computing introduces constraints that classical languages weren't designed to handle:

- **Qubits cannot be cloned.** The no-cloning theorem is a fundamental law of quantum mechanics.
- **Qubits cannot be silently discarded.** Unused quantum state leaks information and corrupts computation.
- **Entanglement creates invisible dependencies.** Operations on one qubit can affect others across your program.

These aren't bugs to catch at runtime—they're errors that should never compile.

Spindle enforces quantum-mechanical correctness at compile time through a linear type system built on Rust's ownership model. If your code compiles, it respects the laws of physics.

---

## Core Principles

### Ownership for Qubits

Every qubit has a single owner. When you pass a qubit to a function, you *move* it. This isn't a convention—it's the type system reflecting reality.

```spindle
fn hadamard(q: Qubit) -> Qubit {
    H(q)
}

fn main() {
    let q = Qubit::new();
    let q = hadamard(q);  // q is moved, then returned
    measure(q);
}
```

### Borrowing for Control

Some operations don't consume qubits—they use them as controls. Spindle distinguishes these with borrowing:

```spindle
fn controlled_not(control: &Qubit, target: Qubit) -> Qubit {
    CNOT(control, target)  // control is borrowed, target is moved
}
```

### Linear Enforcement

You cannot forget a qubit. Every qubit must be explicitly measured or discarded. The compiler will reject code that leaves quantum resources dangling.

```spindle
fn bad() {
    let q = Qubit::new();
    // ERROR: qubit `q` must be consumed (measured or discarded)
}

fn good() {
    let q = Qubit::new();
    let result: Bit = measure(q);  // q is consumed
}
```

### Entanglement Tracking

When qubits become entangled, Spindle's type system tracks the relationship. Operations that would violate entanglement constraints are caught at compile time.

```spindle
fn bell_pair() -> Entangled<Qubit, Qubit> {
    let a = Qubit::new();
    let b = Qubit::new();
    let a = H(a);
    CNOT(&a, b)  // returns an entangled pair
}
```

---

## Quick Example

A simple quantum teleportation protocol:

```spindle
use spindle::prelude::*;

fn teleport(msg: Qubit, alice: Qubit, bob: Qubit) -> Qubit {
    // Entangle Alice and Bob's qubits
    let alice = H(alice);
    let (alice, bob) = CNOT(&alice, bob);

    // Alice interacts message with her qubit
    let (msg, alice) = CNOT(&msg, alice);
    let msg = H(msg);

    // Measure Alice's side
    let (m1, m2) = (measure(msg), measure(alice));

    // Bob applies corrections
    let bob = if m2 { X(bob) } else { bob };
    let bob = if m1 { Z(bob) } else { bob };

    bob  // Bob now holds the teleported state
}
```

---

## Features

| Feature | Description |
|---------|-------------|
| **Linear Types** | Qubits must be used exactly once—no cloning, no forgetting |
| **Ownership & Borrowing** | Rust-style semantics adapted for quantum resources |
| **Entanglement Awareness** | Type-level tracking of qubit correlations |
| **Automatic Uncomputation** | Compiler-assisted cleanup of ancilla qubits |
| **Classical Interop** | Seamless integration with classical control flow |
| **Target Agnostic** | Compile to QASM, QIR, or simulator backends |

---

## Installation

```bash
curl -sSf https://spindle-lang.org/install.sh | sh
```

Or build from source:

```bash
git clone https://github.com/spindle-lang/spindle
cd spindle
cargo build --release
```

---

## Hello, Quantum World

```spindle
use spindle::prelude::*;

fn main() {
    let q = Qubit::new();      // |0⟩
    let q = H(q);              // |+⟩
    let result = measure(q);   // 0 or 1 with equal probability
    
    println!("Measured: {result}");
}
```

Run it:

```bash
spindle run hello.spin
```

---

## Documentation

- [The Spindle Book](https://spindle-lang.org/book) — Comprehensive guide for beginners and experts
- [API Reference](https://spindle-lang.org/docs) — Standard library documentation
- [Language Specification](https://spindle-lang.org/spec) — Formal semantics and type system

---

## Roadmap

- [x] Core language design
- [x] Linear type checker
- [x] Basic gate set (Clifford + T)
- [ ] Entanglement type inference
- [ ] Automatic uncomputation
- [ ] QASM 3.0 backend
- [ ] QIR backend
- [ ] Hardware provider integrations
- [ ] Formal verification tooling

---

## Contributing

Spindle is open source under the MIT license. We welcome contributions of all kinds—code, documentation, examples, and ideas.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

---

## Philosophy

> "The best quantum programs are the ones that couldn't have been wrong."

Classical languages trust programmers to obey quantum rules. Spindle makes breaking them impossible. The compiler is your proof assistant, the type system is your physics engine, and if it compiles, it's correct.

---

## Community

- [Discord](https://discord.gg/spindle)
- [GitHub Discussions](https://github.com/spindle-lang/spindle/discussions)
- [Twitter/X](https://twitter.com/spindle_lang)

---

*Spindle: Where ownership meets superposition.*