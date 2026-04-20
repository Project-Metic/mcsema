# CLAUDE.md — mcsema

McSema: binary lifter that translates x86-64 and AArch64 machine code to LLVM IR.
Foundation component of the Metic binary lifting pipeline (ELF/PE → LLVM IR → Metic IR).
Companion to `remill` (instruction semantics library).

Upstream: lifting-bits/mcsema (open source).

## What This Repo Is

The McSema binary lifter — used by Project-Metic as the first stage of the binary
transformation pipeline. McSema produces LLVM IR from native binaries; Metic's
`metic-lifter` crate then normalizes that IR into Metic IR (typed SSA with named memory
regions and SVID annotations).

## Key Invariants

- **McSema output is not yet Metic IR.** LLVM IR produced by McSema requires a normalization
  pass through `metic-normalizer` before it is valid Metic IR. Never skip normalization.
- **ISA support is gated on test vector coverage.** An ISA with lifting code but no test
  vectors in `tests/vectors/<isa>/` is experimental and must not be used in production.
  See the ISA support table in `README.md`.
- **Never run McSema on untrusted binaries without sandboxing.** Binary lifting is an
  attack surface. Always lift in an isolated environment (container/VM).

## Dev Commands

```bash
# Build (requires LLVM toolchain + remill)
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DLLVM_DIR=/path/to/llvm
make -j$(nproc)

# Run tests
ctest --output-on-failure

# Lift a test binary
mcsema-lift-<version> --os linux --arch amd64 \
  --ir_out output.bc --binary input.elf
```

## Tech Stack

- C++, LLVM/Clang, Python (CFG recovery scripts), remill (instruction semantics)

## Integration with Project-Metic

- Called by `metic-lifter` crate in `project-metic-mono` as the ELF/PE → LLVM IR stage
- LLVM IR output flows into `metic-normalizer` for Metic IR conversion
- Supported ISAs: x86-64, AArch64 (production); RISC-V, WASM (experimental — test vector
  coverage required before production use)

## What NOT to Do

- Do not use McSema output directly as Metic IR without the normalization pass
- Do not lift production binaries without sandboxing
- Do not mark an ISA as production-ready without test vector coverage
