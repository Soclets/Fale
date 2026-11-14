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

procedure greet(#name: Text):
    show_in_text("Hello, " #name)
end procedure

procedure double(#number: Whole) returns Whole:
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

## Open

- What do `<` and `>` do with Text? Most languages compare letter codes, so `"Zebra" < "apple"` is true. Waits until
  procedures are fully designed. The owner's sketch: `compare_text("Zebra", "apple", format: grapheme, case: sensitive)`.
  Its result (before, same, after) needs a type that doesn't exist yet.
- Is the type optional when it's obvious, like `#name = "Tankun"`?
- Joining with maths in the middle: is `"Total: " #a + #b` read as text then `#a` then `+ #b`? Maybe require
  brackets: `"Total: " (#a + #b)`.
- Keep `function` for checked-pure procedures later, or drop the word?
- Tabs or spaces, and how many (D13).
- Named loops, like Jernal's D31 `break outer` (D17).
