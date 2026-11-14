# 5. Procedures

## 5.1 Declaring

Every function is a `procedure`, whether it gives something back or not (D8). One that does says `returns <Type>` in
its header and uses `return` in its body (D9). No `returns` means it gives nothing back.

```
procedure greet[#name: Text]:
    show_in_text("Hello, " #name)
end procedure

procedure double[#number: Whole] returns Whole:
    return #number + #number
end procedure

greet("Tankun")
```

The word `function` is kept free, perhaps for checked-pure procedures later.

## 5.2 Labels and defaults

The procedure's writer chooses which parameters get a label (D18). A declared label is written at every call.
Unlabelled arguments come first, in order; labelled ones come after.

Labels and defaults are declared at the top of the body, before any code: `labels`, then `defaults`. A default is a
labelled parameter's value when the call leaves it off. The header stays short, with names and types only; the body
uses the parameter names.

```
procedure first_word[#text: Text, #limit: Whole] returns Text:
    labels [#limit as limit]
    defaults [#limit = 40]
    ...
end procedure

first_word(#text, limit: 20)
first_word(#text)                    // #limit is 40
```

Labelled arguments come in the declared order, unless the `labels` line says `order: any`:

```
procedure transfer[#money: Whole, #payer: Text, #receiver: Text]:
    labels [#money as amount, #payer as from, #receiver as to], order: any
    ...
end procedure

transfer(amount: 500, from: "Mult", to: "Tankun")
transfer(to: "Tankun", amount: 500, from: "Mult")
```

## 5.3 Variants are labels

There are no families or narrowings (D23). A variant of a procedure is a labelled argument with a default.

## 5.4 `show_in_text`

`show_in_text(...)` shows something as text output (D21). It comes from the standard library, since output touches
the outside world (D49). Plain `show` might suggest a new window.

```
show_in_text(#x)                   // the usual, readable form
show_in_text(#x, stored: true)     // the exact stored value: a Float's ~0.1 shows
                                   // 0.1000000000000000055511151231257827021181583404541015625
```
