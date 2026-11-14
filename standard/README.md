# The Fale standard

This describes Fale as decided so far. Each rule names the decisions it comes from, and
[`DECISIONS.md`](../DECISIONS.md) says why. Anything marked *(proposal)* has a shape on paper but isn't decided.
Anything not here is in chapter 8. (D32)

1. [Syntax](01-syntax.md): lines, blocks, names, brackets, text
2. [Types](02-types.md): numbers, text, collections, `or`, `one_of`, `tagged_union`
3. [Names and values](03-names.md): storing, `mutable`, `copy` and `share`
4. [Control](04-control.md): comparisons, `if`, loops, `break`
5. [Procedures](05-procedures.md): `procedure`, `returns`, labels, defaults, `show_in_text`
6. [Failures](06-failures.md): `fails` and `otherwise`
7. [Memory and the VM](07-memory.md): the VM, the GC
8. [Not decided](08-not-decided.md)

## Goal

Read it, and you get what the program is trying to do. Normal, everyday names, but clearer than Python.

## Principles

- **Words only where a symbol would be a guess**, like Python's `//`, `**` and `[1:-1]`.
- **Symbols everyone knows from school stay**: `+ - = ( ) < >`.
- **Full words, but not extra words.** Long words are fine, since autocomplete makes them cheap to type.
- **Clear first, pretty second.**
- **One fixed word per operator**, with underscores if needed, never a free English phrase, so nobody guesses which
  wording is accepted.
- **Describe it if it's short, otherwise use the learned word** (D29). If everyday words can say what something does
  in three words or fewer, without leaving out anything a reader needs, those words are the name. If not, it gets the
  standard word other languages or fields already use, learned once.
- **Progressive** (D30). A simple tool for beginners, and the precise tool right beside it for when it's needed:
  `Whole` and `i32`, `if` and `match`, `one_of` and `tagged_union`. Both exist. The simple one is never the precise one
  renamed.
