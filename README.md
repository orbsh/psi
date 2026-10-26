# ψ (psi)

A method-chain language native to wasm. **Status: design draft — no implementation yet.**

Design: [docs/ideas/](docs/ideas/README.md) — one document per idea, no verdicts (written in
Chinese). Syntax shapes live in [docs/reference/syntax.md](docs/reference/syntax.md). The
pre-split long draft stays at [docs/design.md](docs/design.md) as history.

Core proposition: the whole language compresses to two rules — a call is *receiver + name +
the arguments the declaration states*, and a type is *data shape + a set of trait labels* with
no inheritance chain. Pipes, infix operators and modules are notations of those two rules, not
separate mechanisms. Methods are never values and functions only exist as closures, so there is
no currying.

Surface: Rust syntax (`fn`, `let mut`, parentheses and commas, `match`, `|x|`, `#[attr]`, `::`
paths, `;` and expression blocks) over value semantics — no `&`, no `*`, no lifetimes, no borrow
checker. Where Rust's surface fights fluency, fluency wins: the ordering of criteria is
fluency > least surprise > mechanism count > token count, and *terseness is not a criterion*.

One artifact, three hosts:

- **Rust engine extension** — the host embeds a wasm runtime and instantiates the compiled
  `.wasm` module; editing a script means recompiling and reinstantiating.
- **Browser** — the same `.wasm`; JavaScript only supplies the imports.
- **CLI** — `psi run script.psi`; the CLI is itself a wasm host.

Hard constraints that follow from that:

- The artifact is a standard wasm module (Wasm 3.0 GC types plus a few WASI imports) — not a
  private bytecode, not a bundled runtime.
- Compilation is whole-program and linear, no fixed point: script save lands in the edit loop.
- The spec stays small enough for one person to read: no borrow checker, allocator, GC
  implementation, linker, or package version solving.
- The boundary with the host is bytes only: numeric values, linear memory, `externref`;
  everything else is serialized.

## Layout

```
docs/
├── ideas/          一个想法一份文档（允许冲突，没有终态）
├── reference/      语法总表与示例程序
└── design.md       拆分前的长文，保留备查

README.md
```

Read [docs/README.md](docs/README.md) for the layout rule, or jump to
[docs/ideas/README.md](docs/ideas/README.md) for the idea index.

## Name

ψ (psi): ψυχή (breath, soul) and the quantum-mechanical amplitude — light, without body,
but genuinely a thing in a field, matching the physics naming of the `~/world` family
(aura, effector, gravity). On disk everything is ASCII: `psi`, `script.psi`.