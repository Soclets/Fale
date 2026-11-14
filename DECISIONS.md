# Fale: design decisions

The owner decides; Fale 2 Design Claude proposes and records. Each entry says what was chosen and whose idea it was.
Numbers are permanent, so a decision is never renumbered. A changed decision gets a new entry that says which one it
replaces.

Fale ("speak!" in Portuguese: expressing yourself in code easily, but clearly) is designed by multifinity, Tankun and
Claude. D1 to D25 keep the numbers they had in the first decisions list (2026-11-14, about 14:55), so they appear here
out of the order they were made, and some don't say whose idea they were. New entries go just before "Open".

The old Fale (a Zig VM, embeddable, beat PUC Lua 5.5.1 on 7 of 11 benchmarks) is now `Soclets/Fale-legacy`.

## Goal

Read it, and you get what the program is trying to do. Normal, everyday names, but clearer than Python.

## Principles

- Words only where a symbol would be a guess (like Python's `//`, `**`, `[1:-1]`).
- Keep symbols everyone knows from school: `+ - = ( ) < >`.
- Full words, but not extra words. Long words are fine, since IDE autocomplete makes them cheap to type.
- Clear first, pretty second.
- Describe it if it's short, otherwise use the learned word (D29). If everyday words can say what something does in
  three words or fewer, without leaving out anything a reader needs, those words are the name: `at_least`,
  `show_in_text`, `max_length`. If they can't, it gets the standard word that other languages or fields already use,
  learned once: `Float`, `HashMap`, `grapheme`. Never invent a new word to learn when a standard one exists.
- Progressive (D30): a simple tool for beginners, and the precise tool right beside it for when it's needed, like
  `Whole` and `i32`, `if` and `match`. Both exist. The simple one is never the precise one renamed.
- Each operator is one fixed word (with underscores if needed), not a free English phrase, so nobody has to guess
  which wording is accepted (the AppleScript problem).

## How to judge it

Write the same small program (for example, splitting a bill between friends) and show it to someone who doesn't code,
or a fresh AI with no docs: "What does this do?" Correct and fast means it's working.

## Example in the current style

```
#name = "Tankun"
#health: Whole = 1000
#people: List of Text = ["Tankun", "Mult", "Claude"]
show_in_text(#people[position: 1])

if #name equals "Tankun":
    ...
end if
if #health at_least 500:
    ...
end if

procedure greet[#name: Text]:
    show_in_text("Hello, " #name)
end procedure

procedure double[#number: Whole] returns Whole:
    return #number + #number
end procedure

greet("Tankun")
```

## Decided

**D1. Names start with `#`.** `$` reads as dollars to normal people.

**D2. `=` stores a value.**

**D3. Types.**

- `Whole`: whole number (i64)
- `Decimal`: exact number with a point (good for money)
- `Float`: approximate, fast number with a point
- `Text`: words and characters
- `List of <Type>`: a collection that can grow, e.g. `List of Text`
- `Array of <Type>, length: N`: a collection with exactly N items, e.g. `Array of Whole, length: 3`
- `List of <Type>, max_length: N`: a list that can grow, but never past N
- `Boolean`: holds `true` or `false`
- `Dictionary, key: <Type>, value: <Type>`: a lookup table that keeps the order you added things:

  ```
  #scores: Dictionary, key: Text, value: Whole = ["Tankun": 1000, "Mult": 850]
  ```

- `HashMap, key: <Type>, value: <Type>`: a lookup table whose order is not promised, and is random on purpose (like
  Go), so nobody leans on an order that was never promised. When order matters, use `Dictionary`. Also protects
  against hash-flooding attacks. "Hash" is a word to learn once, like Float, Array and Boolean. This language never
  promised reproducibility. Tankun: "Reproducibility is a Claude disease".

Not "Text to Whole": it sounds like converting text into a number. Not "of Whole, key: Text": a map holds two things,
so both get a label. The exact wording of the pair syntax, `["key": value]`, is a proposal.

Rule: `of` is for collections holding one kind of thing (List, Array); labels are for anything with more than one
part.

Float, Array and Boolean are programmer words, kept on purpose: they're only three names to learn, and they mean the
same thing in almost every other language.

**D4. Positions count from 1, and say so.** `#people[position: 1]` is the first one. `offset: 0` was dropped as
programmer thinking; "position" settles 1-based counting by itself.

**D5. Comparisons: `at_least`.** One word each. The rest of the family is settled by D26.

**D6. Equality is `equals`, inequality is `not_equal`.** `is` was dropped as fuzzy (and is kept for type checks,
D12); `differs_from` is too long; `differs` hints at "by how much".

**D7. Text inside `"..."` is always literal: no interpolation.** Text and values are joined side by side:
`"Hello, " #name`.

**D8. Every function is a `procedure`, whether it returns something or not.** "Function" means pure in maths and "an
ability" in English, so it would be inaccurate. Proposed: keep the word `function` free for checked-pure procedures
later, like Cyborg's `fn.pure`.

**D9. `returns <Type>` in the header, `return` in the body.** No `returns` means it gives nothing back. Not `->`: an
arrow is a guess for normal people. Not `gives`/`return`: two different words for one idea.

```
procedure double(#number: Whole) returns Whole:
    return #number + #number
end procedure
```

**D10. Interpreted, on a VM, with a GC. The VM is written in Zig.** Tankun's proposal, locked in by multifinity. Why
Zig over C: Tankun's old Fale is already a Zig VM, explicit allocators suit a GC, debug builds catch bounds and
overflow mistakes, cross-compiling and C interop are easy. Much less friction than Rust or SPARK. Watch out for: Zig
is pre-1.0 (upgrades can break code), and memory safety isn't checked at compile time, so GC bugs can slip into
release builds. Test the GC well.

**D11. No NULL. A value that might be missing says so in its type, with `or nothing`.**

```
#phone: Text or nothing = nothing
#name: Text = "Mult"          // can never be nothing
procedure find_phone(#name: Text) returns Text or nothing:
```

`nothing` (no value) is different from `empty` (an empty text or list). Using one without checking is refused, and
the error explains how to check: `if #phone equals nothing:`. Not `Maybe Text`: beginners ask "why not just Text?".
The type answers it in English.

**D12. `or` works between any types (union types).** A spaced phrase invites it anyway. `is` is the type check.

```
#id: Text or Whole = 1042
if #id is Whole:
    ...
end if
```

**D13. Blocks start with `:` and close with `end <kind>`, and indentation is required.** Like `end if`,
`end procedure`, so code always looks tidy and matches its structure. Open: tabs or spaces, and how many.

**D14. `if` uses `else if` and `else`.** Two plain words: not Python's made-up `elif`, not the longer
`otherwise if`.

```
if #health at_least 500:
    show_in_text("Healthy")
else if #health at_least 100:
    show_in_text("Hurt")
else:
    show_in_text("Almost dead")
end if
```

**D15. Conditions combine with plain `and`, `or` and `not`.** `or` also appears in types, like `Text or nothing`;
types come after `:`, conditions after `if`.

```
if #health at_least 100 and #name equals "Mult":
if not #is_ready:
```

**D16. Five loops, each says what it does.** (The `while` example uses `>` since D26.)

```
for each #person in #people:        // go through a collection
    show_in_text(#person)
end for

for each #number from 1 to 10:      // count; 10 is included
    show_in_text(#number)
end for

while #health > 0:                  // repeat while something is true
    #health = #health - 10
end while

repeat 3 times:                     // repeat a set number of times
    show_in_text("Hello")
end repeat

forever:                            // repeat until told to stop
    ...
end forever
```

**D17. `break` leaves the innermost loop; there is no `continue` and no `skip`.** Follows Jernal's D30. Normal
people do say "break out of the loop", while `stop` sounds like stopping the whole program (Tankun's point in
Jernal). An item is skipped by wrapping the work in an `if`, so the skipped path shows as structure instead of a
hidden jump. Open: named loops, like Jernal's D31 `break outer`.

```
for each #item in #list:
    if #item equals "end":
        break
    end if
    if #item not_equal "":
        show_in_text(#item)
    end if
end for
```

**D18. Labels and defaults.** Follows Jernal's D42 (and Cyborg's D113): a declared label is always written, so every
call reads the same.

- Labels are custom words, optional when declaring: the procedure's writer chooses which parameters get one.
- A declared label is required at every call. Unlabelled arguments come first, in order; labelled ones after.
- Labels and defaults are declared at the top of the body, before any code, in this order: `labels`, then
  `defaults` (later, contracts would go after them).
- A default is a labelled parameter's value when the call leaves it off.
- The header stays short (names and types); all "how to call me" info sits in one place. The body uses the parameter
  names.
- Labelled arguments come in the declared order by default. The writer can allow any order with `order: any` on the
  `labels` line.

```
procedure first_word(#text: Text, #limit: Whole) returns Text:
    labels [#limit as limit]
    defaults [#limit = 40]
    ...
end procedure
first_word(#text, limit: 20)
first_word(#text)                    // #limit is 40

procedure transfer(#money: Whole, #payer: Text, #receiver: Text):
    labels [#money as amount, #payer as from, #receiver as to], order: any
    ...
end procedure
transfer(amount: 500, from: "Mult", to: "Tankun")
transfer(to: "Tankun", amount: 500, from: "Mult")    // allowed by order: any
```

**D19. Copy or share, always said out loud, for collections that can change** (List, Array, Dictionary, HashMap).

```
#b = copy #a       // its own list: changing #b never changes #a
#c = share #a      // the very same list: changing #c changes #a
#d = #a            // refused for these types: say copy or share
#x = #y            // fine for Whole, Text, Boolean, Decimal, Float: nothing to share
```

The same rule applies when passing them to a procedure, so a call that can change your list shows it:
`add_guest(share #guests, "Mult")`, `count_guests(copy #guests)`. `copy` and `share` are reserved words.

Why this way, not the mainstream (hidden sharing, the classic "why did #a change?") or Swift's (copy-on-write, which
leans on reference counting): two short words to learn, nothing hidden, and the VM stays simple, since copy just
copies and share just shares. They look like ownership but aren't: the GC still frees memory.

**D20. The GC is generational and fully concurrent.** Like Jernal's D67 and D68, modelled on Java's Generational ZGC.
Tankun: "The best GC, no matter the maintenance. I mean, I haven't tried GC. So, that seems fun".

- New objects go in a small copying nursery (fast bump allocation); survivors move to an old area.
- Both generations are collected while the program runs, so there are no long pauses.
- Needs write barriers (an old object getting a pointer to a new one) and read barriers (objects can move).
- As an interpreted VM it knows its roots exactly (its own stack, registers, globals), so Jernal's D66 stack maps
  aren't needed. Every pointer load and store already goes through the VM's own code, so the barriers live in one
  place.
- This is the hardest part of the whole project. It overrides "maintain little" on purpose.

**D21. Printing is `show_in_text(...)`, not plain `show(...)`.** A normal person might expect `show` to pop up a
whole new window. "In text" says it appears as text output, and leaves room for other ways of showing later.

**D22. Errors use handlers, like Cyborg's, but without families.** Tankun's call. A procedure's header lists what can
go wrong, and the call handles each failure by name. The exact spelling is a proposal:

```
procedure read_whole(#question: Text) returns Whole, fails [not_a_number, cancelled]:
    ...
end procedure

#age = read_whole("Age?") otherwise:
    case not_a_number:
        show_in_text("That's not a number")
    case cancelled:
        show_in_text("Okay, never mind")
end otherwise
```

**D23. No families.** No Cyborg-style narrowings like `fmt_print.stored`. Variants of a procedure are labelled
arguments with defaults instead (D18). Tankun: "Just do a label, lol". Example: `stored:` shows a number's exact
stored value, since a Float's `0.1` isn't really 0.1.

```
procedure show_in_text(#message: Text, #stored: Boolean):
    labels [#stored as stored]
    defaults [#stored = false]
    ...
end procedure
show_in_text(#x)                   // the usual, readable form
show_in_text(#x, stored: true)     // 0.1 as a Float shows
                                   // 0.1000000000000000055511151231257827021181583404541015625
```

**D24. An inexact Float literal must be written with `~` in front, and the compiler checks it.**

```
#a: Float = 0.5        // fine: exact in binary, no ~ needed
#b: Float = ~0.1       // fine: admits it's approximate
#c: Float = 0.1        // refused: 0.1 can't be exact as a Float
#d: Decimal = 0.1      // fine: Decimal is exact
```

Required, not optional: an optional marker can't be trusted, since a missing `~` would tell a reader nothing. Pairs
with `stored:` (D23): `~` warns in the code, `stored: true` shows the truth in the output. The refusal nudges people
toward `Decimal` when they wanted an exact value (money). Planned editor quick fix: "Add ~" or "Use Decimal", one
click.

**D25. Names can't change by default; `mutable` marks the ones that can.**

```
#name = "Tankun"              // fixed
mutable #health = 1000        // can change
#health = #health - 10        // fine
#name = "Mult"                // refused: #name isn't mutable
```

A fixed name freezes the inside too (like Jernal's D6): a fixed list can't be added to or changed. `share` can't get
around it: sharing a fixed list into something that would change it is refused. `mutable` over `changeable`: a
programmer word, but universal, learned once, like Float and Array.

**D26. The comparison family: `at_least`, `at_most`, `>`, `<`.** Completes D5, and changes D16's `while` example from
`above` to `>`.

```
if #health at_least 500:    // 500 or more
if #health at_most 100:     // 100 or less
while #health > 0:          // more than 0, not 0 itself
if #age < 18:               // less than 18
```

`>` and `<` are school symbols, so they stay symbols; `>=` and `<=` aren't, so the edge-inclusive pair stays words.
Claude proposed `at_most`, `above` and `below`; the owner chose: "yes, but use `<` and `>` instead". (Owner,
2026-11-14)

**D27. Session rules, printed by a hook.** `RULES.md` holds the rules for every Fale session, and a `SessionStart`
hook in `.claude/settings.json` prints it at startup, clear and after compaction, so the rules survive when a session's
first prompt doesn't. The setup and the rules come from Cyborg's and Jernal's, as Cyborg's design session described
them on 2026-11-14, with Jernal's matcher (no `resume`, which would print them twice). That includes committing and
pushing automatically, with placeholders allowed and marked. The owner said "There's more, but we'll set up a hook for
that" and "Ask Cyborg for rules & hook". (Owner, 2026-11-14)

**D28. Public on Soclets as `Fale`, licensed like Cyborg.** The repository is `Soclets/Fale`, public like its
siblings; the old Fale is `Soclets/Fale-legacy`. Code is under Apache-2.0 WITH LLVM-exception, in `LICENSE`, so
programs run with Fale owe nothing; the repo's Markdown is under CC BY 4.0, in `LICENSE-CC-BY-4.0`. This is Cyborg's
licensing (its D262), as Cyborg's design session described it; both files went in before the first public push. The
owner said: "Make a GH repo, new decisions folder/file. Ask Cyborg for rules & hook". (Owner, 2026-11-14)

**D29. Describe it if it's short, otherwise use the learned word.** A name passes one test: can everyday words say
what it does in three words or fewer, without leaving out anything a reader needs? If yes, those words are the name,
joined with underscores: `at_least`, `not_equal`, `show_in_text`, `max_length`, `compare_text`. If no, it gets the
standard word other languages or fields already use, so it's learned once and pays off everywhere: `Float`, `Array`,
`Boolean`, `HashMap`, `grapheme`. A new word to learn is never invented when a standard one exists. A description
that only fits by dropping something important fails the test, so it isn't used: "number with a point" for `Float`
would hide that it's approximate. This writes down what D3 already did with Float, Array and Boolean. The owner
raised it: "There's a line between "Make words that are able to be described shortly be that, and let more complex
words be learned" vs "Make every word a description""; Claude proposed the three-word test; the owner chose: "Write it
down, and make the rule for clear". (Owner, 2026-11-14)

**D30. Progressive means two layers side by side.** Fale gives beginners a simple tool, and keeps the precise tool
right beside it for when they need it: `Whole` for everyday whole numbers and `i32` when the size matters, `if` for
everyday choices and `match` for handling every case. Both layers exist, so learning the precise one never means
unlearning the simple one, and a beginner never has to meet the precise one first. The simple layer is a real tool of
its own, never the precise one under a friendlier name. Claude had proposed a single `one_of` block whose cases could
carry data; the owner asked "A beginner language, it's supposed to be progressive, right?", then said "I swear, that
looks like `tagged_union` renamed" and explained: "There's `Whole` for simple `i64`, and there's also `i32` there for
you to use, if you want. There's an `if` for simple `match`, and `match` for you later. This is just renaming
`tagged_union`". The owner asked for it to be defined: "we might wanna define `progressive`". (Owner, 2026-11-14)

**D31. `one_of` for plain named values, `tagged_union` for cases that carry data.** Both are declared once as named
blocks, so a header can say `returns Order`:

```
one_of Order:
    before
    same
    after
end one_of

tagged_union Result:
    worked
    failed(#code: Whole)
end tagged_union

#order: Order = before
```

`one_of` is the simple layer (D30): its values are plain names, a closed list, not numbers in disguise, so `before + 1`
is refused. `tagged_union` is the precise layer: each case may carry its own data, like Rust's enums and Cyborg's
`union.tagged` (Cyborg D33). `one_of` is underscored because "one" means nothing on its own; `tagged_union` because it
only means something as a whole, and Fale has no dotted narrowings (D23). The list in `one_of` answers "one of what?":
the owner said "it already fucking said it `[before, same, after]`". The owner asked "Why does the `of` in `one of`
have a space?", then "It seems like it should be a block, tbh", then "Why don't we just use `union.tagged`, then?";
Claude proposed the two layers; the owner chose: "Seems good". The spelling of a case with data,
`failed(#code: Whole)`, is a proposal. (Owner, 2026-11-14)

**D32. A standard, in numbered chapters.** `standard/` describes the language as decided, one numbered chapter per
area, ending with a "Not decided" chapter, like Cyborg's and Jernal's. Each rule names the decisions it comes from, and
`DECISIONS.md` stays the record of why. The owner said: "and have a standard too". (Owner, 2026-11-14)

**D33. Comments: `//`, `///` for docs, and a named `comment` block.** `//` starts a comment that runs to the end of
the line, on a line of its own or after code. `///` is a doc comment. A longer comment is a block, named like any
other block's `end` (D13). (The block's text is indented since D34.)

```
#health = 1000    // trailing a line
/// Doubles a number.
procedure double(#number: Whole) returns Whole:

comment why_double:
    The old version added the number to itself in a loop.
end comment why_double
```

The owner wrote the design: "Comment syntax ... `// trailing-able too` `/// doc comment` `comment name:` `block`
`end comment name`". (Owner, 2026-11-14)

**D34. A comment block's text is indented.** Like any other block (D13), so a comment can't be mistaken for code
around it. Changes D33's example. Claude asked whether a comment is the one block left free; the owner chose:
"indent, Idk how to format here". (Owner, 2026-11-14)

**D35. Comment blocks nest, but never inside one with the same name.** So code that already holds a comment block can
be commented out: `end comment why_double` can only close `why_double`, never the block around it, unlike C, where an
inner `*/` ends the outer `/*` early. A block inside another block with the same name is refused, and the error asks
for a different name. Inside a comment block, everything is free text except lines that open or close a comment block.

```
comment old_scoring:
    procedure score(#points: Whole) returns Whole:
        comment why_double:
            The old version added the number to itself in a loop.
        end comment why_double
        return #points + #points
    end procedure
end comment old_scoring
```

The owner asked "What do you recom?"; Claude proposed this; the owner chose: "yea". (Owner, 2026-11-14)

**D36. `( )` runs, `[ ]` describes or holds.** Like Jernal's bracket rule. `( )` runs something: a call,
`greet("Tankun")`. `[ ]` describes or holds something: a declaration's parameters, a `tagged_union` case's data, a
list, a position, `labels` and `defaults`. So declarations change from `( )` to `[ ]`, and a declaration can be told
from a call at a glance:

```
procedure greet[#name: Text]:
    show_in_text("Hello, " #name)
end procedure

greet("Tankun")

tagged_union Result:
    worked
    failed[#code: Whole]
end tagged_union
```

Grouping in maths, `(2 + 3) * 4`, is a deliberate exception: `( )` there groups, as taught at school. Jernal's third
rule, `< >` for types, doesn't apply: Fale writes `List of Text`, so `<` and `>` are only comparisons (D26). This
changes the procedure headers in D9, D11, D18, D22 and D23, and the case in D31; their entries keep the spelling they
were decided with. The owner asked "Adopt bracket rules?"; Claude described Jernal's; the owner chose: "yea. Btw,
`()` math grouping is a purposely done exception". (Owner, 2026-11-14)

**D37. Who does what: Fale 2 Design Claude designs, writes the oracle and checks; Fale Impl Claude builds the VM.**
Fale 2 Design Claude keeps the decisions and the standard, writes the oracle (`oracle/`) and the tests (`tests/`),
and checks Fale Impl Claude's work against the standard. Fale Impl Claude builds the Zig VM (D10) in its own clone.
The oracle and the VM never read each other's code, so their bugs can't agree; where they differ, the standard decides
which is wrong, and if it's silent, that's a question for the owner (like Jernal's rule 18). The owner proposed it:
"Now, impltors. You do oracle and checking and design? While `Fale Impl Claude` does the other stuff?". Claude noted
the cost, that oracle code slows design talk, so the oracle can move to its own session later. (Owner, 2026-11-14)

**D38. The oracle is written in OCaml.** OCaml's variants and checked `match` suit a syntax tree, its GC makes `share`
just two names holding the same list, zarith's exact fractions fit `Decimal` and `stored: true`, uuseg handles
graphemes, and it isn't Zig, so it can't share the VM's mistakes. JernOrac uses it and would pick it again. From
JernOrac's experience, on day one: a language integer never touches OCaml's 63-bit `int` (`Int64`, or zarith's `Z`
for exact steps, then a range check); steps with side effects are put in order with `let ... in`, since OCaml may
evaluate arguments right to left; a case that can't happen raises `Impossible`, never quietly returns something; the
checker produces a checked tree that the interpreter runs; the parser recovers from errors from the start; and a
fuzzer is built early. Claude recommended it and asked JernOrac first; the owner chose: "yea". (Owner, 2026-11-14)

**D39. Host code traces decisions and uses Fale's words.** The VM and the oracle mark code that carries out a
decision with a `trace: D<number>` comment, and a check fails on a tag naming no decision and lists decisions nothing
traces yet. Their names are full words, one per idea, in Fale's own words over textbook ones (`procedure`, `position`,
`Whole`, `share`), in the host language's casing, and tests are named after their decision. Both sections come from
Jernal's rules. The owner asked "What are the rules for writing host source code?"; Claude proposed adding Jernal's
Tracing and Naming; the owner chose: "yes". (Owner, 2026-11-14)

**D40. `match` names every case, and several cases can share a line.** `match` is the precise layer beside `if`
(D30), shaped like `otherwise:` (D22), with `case` lines:

```
match #order:
    case before, same:
        show_in_text("Not after")
    case after:
        show_in_text("After")
end match
```

A missed case is refused, and the error lists the missing ones, so adding a case later forces every `match` to handle
it. There is no catch-all: several cases on one line, `case before, same:`, give the brevity while still naming every
case. For now `match` is a statement and gives no value, since nothing has shown it needs one. Cyborg's design session
advised multi-case lines over a catch-all (Cyborg never designed them) and said the shared shape with failure
handling made both easier to learn. Claude proposed it; the owner chose: "yea". (Owner, 2026-11-14)

**D41. Fields are read with `.` and declared without `#`.** A field is a part of a value, not a name of its own, so
it has no `#` (D1): it's declared as `failed[code: Whole]` and read through its value, `#result.code`. Reading a
`tagged_union` case's field is allowed only where Fale knows the value is that case, such as inside `case failed:`;
anywhere else it's refused, like using a value that might be `nothing` (D11). A field inherits its value's mutability
(D25). This changes D31's and D36's `failed[#code: Whole]`.

```
tagged_union Result:
    worked
    failed[code: Whole]
end tagged_union

match #result:
    case worked:
        show_in_text("Done")
    case failed:
        show_in_text("Failed with code " #result.code)
end match
```

The owner proposed the dot: "can't it just be `#r.r`?"; Claude proposed dropping `#` from declarations; the owner
asked "Is it really a variable?", then chose: "yea". (Owner, 2026-11-14)

**D42. A case with data is built with its fields labelled in `[ ]`.** Building mirrors declaring, with the type
swapped for a value, and every field is labelled, so `failed[3]` is refused, like a missing declared label (D18):

```
#result: Result = failed[code: 3]
#done: Result = worked
```

Claude proposed it, following D36 (`[ ]` holds) and D41; the owner chose: "yea". (Owner, 2026-11-14)

**D43. Error kinds say what happened to the program.** Fale's two main kinds of error are written in words, not as
`CompileTimeError` and `RuntimeError`, since "compile time" and "run time" are programmer jargon (D29):

```
Refused before running: main.fale:1:13, line 1 column 13
Stopped while running: main.fale:4:15, line 4 column 15
```

Claude proposed them after laying out Cyborg's and Jernal's formats; the owner chose: "Yes". (Owner, 2026-11-14)

**D44. A `Whole` that goes past its limit is an error, found as early as it can be.** When the checker knows every value
involved, because they're written in the program or held by fixed names with known values, an overflow is refused
before running. When it depends on values only the running program knows, like typed input, the program stops while
running. A `+` that only might overflow, like `#age + 1`, is never refused. The owner asked "But `+ 1` is known there,
shouldn't it be a compiler error?", then "if the fucking variable is known at comp, that's what I meant". (Owner,
2026-11-14)

**D45. `context:` gives the facts at the moment of an error.** An error message may have a `context:` line with
facts about what the program had at that point: values, which type something is, what's known. It never holds advice
or guesses; those belong to `fix:`, and earlier lines that caused the error to `because:`.

```
 context: `#people` has 3 items, and `#i` is 4
 context: `#id` is Text here, not Whole
```

Cyborg and Jernal call a narrower version `when:`, for values only; the owner asked "The fuck is `when`?", Claude
proposed `values:`, and the owner asked "What about `context:`? So it could be used generallt". Claude proposed this
definition; the owner chose: "Yea". (Owner, 2026-11-14)

**D46. Every error is shown at once, without knock-on errors.** Like Cyborg's D207 and Jernal's D13. The checker
finds an error and its fix, applies the fix in memory only, never in the file, and checks again; an error that
disappears was caused by the first one, so it isn't shown on its own but as a `because:` pointing back. The checking
must really carry on as if the code were fixed. If there's more than one real fix, the first is applied. With no
certain fix, only the broken line is skipped, and nothing that's only wrong because something else is unknown is
reported. Claude described Cyborg's and Jernal's technique; the owner asked "Did we adopt the anti cascade err tech?",
then chose: "yea". (Owner, 2026-11-14)

**D47. Error messages are templates in the standard, filled from the real program.** Only a message's sentence shape
is fixed. Its slots are filled from the program: names and expressions exactly as written, types, values, the place,
and fixes built from the actual code (`~0.1`, not `~<number>`):

```
what:    `{expression}` is too big for a {type}
what:    `#big + 1` is too big for a Whole
```

Every template lives in the standard, in one catalogue. The oracle and the VM both follow it, so their messages match
byte for byte without either reading the other's code (D37), and the owner can read and approve every message Fale can
show in one place. The owner asked "should we make dynamic source code? ... Like, `variable name` in error actually
shows the variable name, etc"; Claude proposed the catalogue; the owner chose: "yea". (Owner, 2026-11-14)

**D48. The error layout.** The kind and the place come first, written twice: `main.fale:1:13` for tools, then
`line 1 column 13` for people, with the column counted in graphemes. Then the source line, with a `^` run under
exactly the part in question. Then one label per line: `what:` says what's wrong, and stays even when there's a fix;
`why:` gives the rule in plain words, never a decision number; `context:` gives the facts (D45); `because:` points at
earlier lines that caused it, each with its own place and source line; `fix:` lists every real fix, never a guess.

```
Refused before running: main.fale:1:13, line 1 column 13
 1 | #c: Float = 0.1
   |             ^^^
 what: 0.1 can't be stored exactly as a Float
 why:  an approximate Float is written with ~ in front
 fix:  `~0.1`, or make it a `Decimal`
```

It's Cyborg's and Jernal's layout (from Radium), with three beginner changes: no decision numbers ("Yo, chill with
decision numbers in there"), `what:` kept beside `fix:`, and more than one fix allowed, as long as each is real. The
owner chose: "yes". (Owner, 2026-11-14)

**D49. Base Fale touches nothing outside the program; the standard library does.** Base Fale only computes: maths,
`Text`, collections, `Decimal` and the like. Anything that touches the outside world comes from the standard library
and has to be brought in: the terminal (including `show_in_text`, with no exception), files and folders, time,
randomness, the network, and the program's surroundings (arguments, environment variables, other programs). So the
top of a program says what it can touch, and a program that brings nothing in can't touch anything. How things are
brought in isn't designed yet. The owner proposed it: "I think it should be some external things, and base Fale can't
touch anything external"; Claude asked whether `show_in_text` is the one exception; the owner chose: "no exception".
(Owner, 2026-11-14)

**D50. The manifest is the program's permission list.** What a program may touch outside itself (D49) is listed in
its manifest, not in `use` lines in the code, so the code stays straight to its idea. Anyone asking "what can this
touch?" reads one short file, like a phone app's permissions. A new project's manifest is made for it with the
terminal already allowed, so a first program is one line of code. Using something the manifest doesn't allow is
refused before running, with the fix pointing at the manifest. Jernal also keeps all bringing-in in its manifest
(Jernal D59, D114). Claude had proposed `use terminal` at the top of the file; the owner said "Idk about that instead
of manifest, man. Like, it's supposed to be straight to the point, to the idea of the code. And you want it to be
sitting there importing stuff?". (Owner, 2026-11-14)

**D51. Fale is embeddable as well as standalone.** Since base Fale touches nothing outside the program (D49), a host
decides what a script may touch: standalone, the `fale` program grants what the manifest lists; embedded, a game or
app grants its own abilities (say `game_world`, and no `files`), like Lua's host-opened libraries. The VM keeps no
global state, so several scripts can run side by side, and offers a clean API for hosts. The owner asked "Okay, so
your idea is for Fale to become an embeddable, contained scripting language?"; Claude said it falls out of D49 and
asked whether to aim for it; the owner chose: "Seems good". (Owner, 2026-11-14)

**D52. SIML, redesigned: named blocks, `key = value`, `( )` comments, quoted text and bare host words.** Fale's
manifest (D50) is written in SIML, the owner's own format from Cyborg (Cyborg D228 to D231, also used by Jernal),
redesigned from scratch on the owner's sketch:

```
details:
    name = "Hello"
    description = "Says hello to the world"    (this is a comment)
end details

permissions:
    files = allowed
    terminal = allowed
end permissions
```

- The host defines everything: block names, keys, and which values are allowed. SIML only knows blocks,
  `key = value`, quotes and comments.
- A block opens with `name:` and closes with `end name`; lines inside are indented (like D13).
- `( )` is a comment, anywhere outside quotes, including on a line of its own. Brackets inside a comment balance, so
  `(see (this))` is one comment. Inside quotes, `(` is just a character.
- Quotes mean text, even without a space: `"Hello"`. A bare word is one the host knows, like a `one_of` value
  (`allowed`), and the host refuses a word it doesn't know.

The owner said "Let's make an siml standard actually, scratch syntax. Let's get it, propose"; Claude proposed a
line-based design; the owner sketched this instead: "Things like `name` and other settings are definiable by that
SIML's host, rather than fixed"; Claude filled in indentation, comment and quoting rules; the owner chose: "yea".
(Owner, 2026-11-14)

**D53. SIML lives in its own repository, `Soclets/SIML`.** Its standard and decisions moved there (SIML D1 to D3), and
Fale points at it; D52 stays here as the record of how the redesign started. Claude proposed a shared repo, as
Jernal's design session had suggested; the owner chose: "yup. SIML in big letters". (Owner, 2026-11-14)

**D54. Indentation is 4 spaces per level, and a tab character is refused.** Settles D13's open question. A tab and
spaces can look identical on screen while meaning different things, the classic invisible Python error. The error
offers to convert the tab; editors insert spaces when the Tab key is pressed. SIML uses the same rule (SIML D9). Claude
proposed it for both; the owner asked "What does that mean?", then chose: "yea". (Owner, 2026-11-14)

**D55. Inside text, `\"` is a quote, `\\` is a backslash, and a backslash before anything else is refused.** The
same rules as SIML (SIML D6, D7), so a text value can be copied between Fale code and a manifest unchanged. Text is
otherwise still literal (D7): nothing else inside it is replaced, and `"line one\nline two"` is refused rather than
quietly meaning a backslash and an `n`. Claude proposed matching SIML; the owner chose: "yea". (Owner, 2026-11-14)

## Open

- What do `<` and `>` do with Text? Most languages compare letter codes, so `"Zebra" < "apple"` is true. Waits until
  procedures are fully designed. The owner's sketch: `compare_text("Zebra", "apple", format: grapheme, case: sensitive)`.
  Its result (before, same, after) needs a type that doesn't exist yet.
- Is the type optional when it's obvious, like `#name = "Tankun"`?
- Joining with maths in the middle: is `"Total: " #a + #b` read as text then `#a` then `+ #b`? Maybe require
  brackets: `"Total: " (#a + #b)`.
- Keep `function` for checked-pure procedures later, or drop the word?
- Named loops, like Jernal's D31 `break outer` (D17).
- Spaces or underscores in multi-word keywords. Claude's proposed rule: spaces when each word stands on its own
  (`List of Text`, `else if`, `end if`), underscores when the words only mean something together (`at_least`,
  `one_of`). `for each` breaks it, since `for` never appears alone.
- Sized number types beside `Whole`, like `i32` (D30): which ones, and how they're spelled.
- Whether `match` should give a value later (D40).
- What a doc comment (`///`, D33) attaches to.
- Exit codes, `NotDesignedYet`, `NotBuiltYet` and the VM's own failures (D43). The file extension (`.fale` in
  examples).
- What happens when sized types like `i32` overflow (D30, D44).
- The manifest's file name and its blocks and keys (D50, D52).
- Whether bringing in a package that touches files needs the program's own manifest to allow files too (D50).
- The VM's API for hosts (D51).
