# 1. Syntax

## 1.1 Blocks

A line that opens a block ends with `:`. The block closes with `end` and the kind of block it was: `end if`,
`end for`, `end procedure`, `end one_of` (D13). Indentation inside a block is required, so code always looks the way
it's structured. Tabs or spaces, and how many, aren't decided (chapter 8).

```
if #health at_least 500:
    show_in_text("Healthy")
end if
```

Inside a procedure, the first lines may describe it before any code: `labels`, then `defaults` (D18).

## 1.2 Names

- A name starts with `#`: `#name`, `#health`, `#people` (D1). `$` reads as dollars to normal people.
- Types start with a capital letter: `Whole`, `Text`, `List of Text`, and the program's own, like `Order` (D3, D31).
- Keywords, procedures and plain values (like `true`, `nothing`, `before`) are lowercase, with `_` between words:
  `show_in_text`, `at_least`, `one_of`, `max_length` (D5, D21, D31).
- A name is its own description if three everyday words or fewer can say what it does; otherwise it's the standard
  learned word (D29).

## 1.3 Brackets

`( )` runs something; `[ ]` describes or holds something (D36).

- `( )` runs a procedure: `greet("Tankun")`. Grouping in maths, `(2 + 3) * 4`, is the one deliberate exception.
- `[ ]` describes a declaration's parameters, `procedure greet[#name: Text]:`, and a `tagged_union` case's data,
  `failed[code: Whole]` (D41); picks a position, `#people[position: 1]` (D4); holds a list, `["Tankun", "Mult"]`, or
  pairs, `["Tankun": 1000]` *(proposal)* (D3); and holds a procedure's `labels` and `defaults` (D18).
- `< >` are only comparisons (D26): types are written in words, like `List of Text`.

```
procedure greet[#name: Text]:      // [ ] describes what goes in
    show_in_text("Hello, " #name)
end procedure

greet("Tankun")                    // ( ) runs it
```

## 1.4 Text

Text inside `"..."` is always literal: nothing inside it is replaced (D7). Text and values are joined by writing them
side by side:

```
show_in_text("Hello, " #name)
```

## 1.5 Comments

`//` starts a comment that runs to the end of the line, on its own line or after code. `///` is a doc comment. A
longer comment is a named block, and its text is indented like any block's (D33, D34):

```
#health = 1000    // trailing a line
/// Doubles a number.
procedure double[#number: Whole] returns Whole:

comment why_double:
    The old version added the number to itself in a loop.
end comment why_double
```

Comment blocks nest, so code that already holds one can be commented out, but a block can't sit inside one with the
same name (D35). Inside a comment block, everything is free text except lines that open or close a comment block.

```
comment old_scoring:
    procedure score[#points: Whole] returns Whole:
        comment why_double:
            The old version added the number to itself in a loop.
        end comment why_double
        return #points + #points
    end procedure
end comment old_scoring
```
