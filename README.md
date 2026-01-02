# Spindle

A Rust-inspired, memory-safe programming language for quantum computing.

Spindle brings the ownership and borrowing principles that made Rust revolutionary to the quantum realm—where linearity isn't just good practice, it's physics.

## Why Spindle?

Quantum computing introduces constraints that classical languages weren't designed to handle:

- **Qubits cannot be cloned.** The no-cloning theorem is a fundamental law of quantum mechanics.
- **Measurement destroys superposition.** Observing a qubit collapses its state irreversibly.
- **Entanglement creates implicit dependencies.** Operations on one qubit can affect entangled partners.
- **Quantum operations must be reversible.** Unitary transformations are the only non-measurement operations allowed.

These aren't just best practices—they're the laws of physics. Classical type systems let you write programs that violate these laws, leading to runtime errors that are impossible to debug. Spindle makes these constraints compile-time guarantees.

## Core Principles

### 1. Qubit Ownership

In Spindle, qubits have a single owner at any given time, just like Rust's ownership model:

```rust
fn example() {
    let q = Qubit::new();  // q owns this qubit
    hadamard(q);           // ownership moved to hadamard
    // q is no longer valid here - prevented at compile time
}
```

The no-cloning theorem means you can't copy a qubit. Spindle enforces this through move semantics—once a qubit is moved, you can't use the old binding.

### 2. Borrowing for Observation

Measurements require mutable borrows, making it clear when superposition is destroyed:

```rust
fn measure_example() {
    let mut q = Qubit::new();
    hadamard(&q);                    // immutable borrow - preserves superposition
    let result = measure(&mut q);    // mutable borrow - collapses state
    // q is now in a classical state
}
```

### 3. Entanglement Tracking

Entangled qubits must be handled together to prevent accidental decoherence:

```rust
fn bell_state() -> EntangledPair {
    let q1 = Qubit::new();
    let q2 = Qubit::new();
    let q1 = hadamard(q1);
    entangle(q1, q2)  // Returns EntangledPair, preventing independent operations
}
```

### 4. Reversible Operations

All quantum gates are represented as reversible functions. The type system ensures you can only apply unitary transformations:

```rust
trait UnitaryGate {
    fn apply(self, q: Qubit) -> Qubit;
    fn inverse(self) -> Self;
}
```

## Language Features

### Quantum Types

- `Qubit`: A single quantum bit in superposition
- `ClassicalBit`: A measured qubit, stored classically
- `EntangledPair`: Two qubits in an entangled state
- `QuantumRegister<N>`: A fixed-size array of qubits
- `QuantumCircuit`: A composable quantum computation

### Quantum Operations

```rust
// Single-qubit gates
fn hadamard(q: Qubit) -> Qubit;
fn pauli_x(q: Qubit) -> Qubit;
fn pauli_y(q: Qubit) -> Qubit;
fn pauli_z(q: Qubit) -> Qubit;
fn phase(angle: f64, q: Qubit) -> Qubit;

// Multi-qubit gates
fn cnot(control: Qubit, target: Qubit) -> (Qubit, Qubit);
fn toffoli(c1: Qubit, c2: Qubit, target: Qubit) -> (Qubit, Qubit, Qubit);
fn multi_controlled_z<const N: usize>(qubits: [Qubit; N]) -> [Qubit; N];

// Measurement
fn measure(q: &mut Qubit) -> ClassicalBit;
```

### Example: Quantum Teleportation

```rust
fn quantum_teleportation(
    message: Qubit,
    alice_epr: Qubit,
    bob_epr: Qubit
) -> Qubit {
    // Alice's operations
    let (message, alice_epr) = cnot(message, alice_epr);
    let message = hadamard(message);
    
    // Measurements
    let mut msg_mut = message;
    let mut alice_mut = alice_epr;
    let bit1 = measure(&mut msg_mut);
    let bit2 = measure(&mut alice_mut);
    
    // Bob's corrections (based on Alice's measurements)
    let mut result = bob_epr;
    if bit2.is_one() {
        result = pauli_x(result);
    }
    if bit1.is_one() {
        result = pauli_z(result);
    }
    
    result  // Bob now has the original quantum state
}
```

### Example: Grover's Search Algorithm

```rust
use std::f64::consts::PI;

fn grovers_search<const N: usize>(oracle: impl Fn([Qubit; N]) -> [Qubit; N]) -> ClassicalBit {
    // Initialize qubits in superposition
    let qubits: [Qubit; N] = std::array::from_fn(|_| Qubit::new());
    let qubits = qubits.map(hadamard);
    
    // Apply Grover iterations
    let iterations = (PI / 4.0 * f64::sqrt(2_f64.powi(N as i32))).floor() as usize;
    let mut qubits = qubits;
    
    for _ in 0..iterations {
        // Oracle
        qubits = oracle(qubits);
        
        // Diffusion operator
        qubits = qubits.map(hadamard);
        qubits = qubits.map(pauli_x);
        qubits = multi_controlled_z(qubits);
        qubits = qubits.map(pauli_x);
        qubits = qubits.map(hadamard);
    }
    
    // Measure result - destructure to extract first qubit
    let [mut first_qubit, ..] = qubits;
    measure(&mut first_qubit)
}
```

## Safety Guarantees

Spindle provides compile-time guarantees for quantum-specific safety:

1. **No Cloning**: The type system prevents copying qubits
2. **No Use-After-Measure**: Measured qubits cannot be used in quantum operations
3. **Entanglement Safety**: Entangled qubits must be operated on together
4. **Reversibility**: Only unitary operations are allowed (except measurement)
5. **Resource Cleanup**: All qubits must be either measured or explicitly released

## Getting Started

### Installation

```bash
# Install Spindle compiler
curl --proto '=https' --tlsv1.2 -sSf https://spindle-lang.org/install.sh | sh

# Verify installation
spindle --version
```

### Hello Quantum World

```rust
// hello_quantum.sp
fn main() {
    let q = Qubit::new();
    let q = hadamard(q);
    let mut q = q;
    let result = measure(&mut q);
    println!("Measured: {}", result);
}
```

Compile and run:
```bash
spindle build hello_quantum.sp
./hello_quantum
```

## Comparison with Other Quantum Languages

| Feature | Spindle | Q# | Qiskit | Cirq |
|---------|---------|-----|--------|------|
| No-cloning enforcement | ✅ Compile-time | ⚠️ Runtime | ⚠️ Runtime | ⚠️ Runtime |
| Memory safety | ✅ Ownership system | ❌ GC-based | ❌ Python | ❌ Python |
| Type-safe entanglement | ✅ Yes | ❌ No | ❌ No | ❌ No |
| Reversibility checking | ✅ Compile-time | ⚠️ Limited | ❌ No | ❌ No |
| Classical integration | ✅ Zero-cost | ✅ Good | ⚠️ Python overhead | ⚠️ Python overhead |

## Roadmap

- [x] Core type system with ownership
- [x] Basic quantum gates
- [x] Entanglement tracking
- [ ] Quantum circuit optimization
- [ ] Simulator backend
- [ ] Hardware backend integration (IBM Quantum, IonQ, Rigetti)
- [ ] Standard library of quantum algorithms
- [ ] IDE support and language server
- [ ] Formal verification tools

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

Spindle is dual-licensed under MIT and Apache 2.0. See [LICENSE-MIT](LICENSE-MIT) and [LICENSE-APACHE](LICENSE-APACHE) for details.

## Research

Spindle is based on research in linear type systems and quantum programming languages. Key papers:

- "Linear types for quantum programming" - Peter Selinger
- "The quantum IO monad" - Thorsten Altenkirch & Jonathan Grattage
- "Quantum programming with inductive datatypes" - Andrea Vezzosi et al.

## Community

- [Discord](https://discord.gg/spindle)
- [Forum](https://forum.spindle-lang.org)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/spindle)

---

*Spindle: Where type safety meets quantum mechanics.*
