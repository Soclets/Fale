# 9. Not decided

The same list as "Open" in [`DECISIONS.md`](../DECISIONS.md), by chapter.

## Syntax (chapter 1)

- Spaces or underscores in multi-word keywords. Proposed rule: spaces when each word stands on its own (`List of Text`,
  `else if`), underscores when the words only mean something together (`at_least`, `one_of`). `for each` breaks it.
- What a doc comment (`///`, D33) attaches to.
- Joining text with maths in the middle: is `"Total: " #a + #b` read as text, then `#a`, then `+ #b`? Maybe require
  brackets: `"Total: " (#a + #b)`.
- The pair syntax `["key": value]`.

## Types (chapter 2)

- Sized number types beside `Whole`, like `i32` (D30).

- Conversion (D57): the full table, the failure names, and the rounding words.

## Names and values (chapter 3)

- Can the type be left off when it's obvious, like `#name = "Tankun"`?

## Control (chapter 4)

- What `<` and `>` do with `Text`. Waits on procedures. The owner's sketch:
  `compare_text("Zebra", "apple", format: grapheme, case: sensitive)`, giving an `Order` (D31).
- Whether `match` should give a value later (D40).
- Named loops, like Jernal's `break outer` (D17).

## Procedures (chapter 5)

- The manifest's name and format, the list of permissions, and whether a package's permissions show in the
  program's manifest (D50).
- The VM's API for hosts (D51).

- Keep `function` for checked-pure procedures later, or drop the word (D8)?

## Failures (chapter 6)

- How a procedure signals a failure from its body (D22).

## Error messages (chapter 8)

- The file extension (`.fale` in examples).
- Exit codes, `NotDesignedYet`, `NotBuiltYet` and the VM's own failures.
- What happens when sized types like `i32` overflow.
