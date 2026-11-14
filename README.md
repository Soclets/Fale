# Fale

**Speak!**

Express yourself in a beginner-friendly but clear way.

Fale ("speak!" in Portuguese) is a readable programming language: normal, everyday names, but clearer than Python.
Words go where a symbol would be a guess, and the symbols everyone knows from school stay. It runs on a VM written in
Zig, with a generational, concurrent GC. It's being designed now, and every decision is in
[`DECISIONS.md`](DECISIONS.md).

## Hello, World!
```
show_in_text("Hello, World!")
```

## Licences

- [`LICENSE`](LICENSE): Apache-2.0 WITH LLVM-exception, for the code. Programs you compile owe nothing.
- [`LICENSE-CC-BY-4.0`](LICENSE-CC-BY-4.0): CC BY 4.0, for this repo's writing: its Markdown files, including this
  README, `DECISIONS.md`, `RULES.md` and `standard/`. Credit them as `Fale GitHub repo text: https://github.com/Soclets/Fale`, say
  they're under CC BY 4.0, and say whether you changed them.

## Design & Impl credit (tools excluded)

Tankun Sriket (Tancum Srigay, TandzInfinity18, TandzNaN, Tandz): Owner and designer.
multifinity: Designer.
Anthropic's Claude (usually the latest Opus, multiple): Design help and impl.
