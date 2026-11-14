# 2. Types

## 2.1 Numbers

| Type | What it holds |
|---|---|
| `Whole` | a whole number: `i64`, a 64-bit signed integer (D3) |
| `Decimal` | an exact number with a point, good for money (D3) |
| `Float` | an approximate, fast number with a point (D3); its size isn't decided |

Sized types like `i32` exist beside `Whole` as the precise layer (D30). Which ones, and how they're spelled, isn't
decided.

A `Whole` that goes past its limit is an error (D44). If every value involved is known before running, it's refused
before running; otherwise the program stops while running. A `+` that only might overflow, like `#age + 1`, is never
refused.

An inexact `Float` literal must be written with `~` in front, and the compiler checks it (D24):

```
#a: Float = 0.5        // fine: exact in binary
#b: Float = ~0.1       // fine: admits it's approximate
#c: Float = 0.1        // refused: 0.1 can't be exact as a Float
#d: Decimal = 0.1      // fine: Decimal is exact
```

The refusal suggests `~` or `Decimal`. `show_in_text(#b, stored: true)` shows the exact stored value (D23).

## 2.2 Text and Boolean

- `Text`: words and characters (D3).
- `Boolean`: `true` or `false` (D3).

## 2.3 Collections

| Type | What it is |
|---|---|
| `List of <Type>` | a collection that can grow (D3) |
| `List of <Type>, max_length: N` | a list that can grow, but never past N (D3) |
| `Array of <Type>, length: N` | exactly N items (D3) |
| `Dictionary, key: <Type>, value: <Type>` | a lookup table that keeps the order you added things (D3) |
| `HashMap, key: <Type>, value: <Type>` | a lookup table whose order isn't promised, and is random on purpose (D3) |

`of` is for collections holding one kind of thing; labels are for anything with more than one part (D3). Positions
count from 1: `#people[position: 1]` is the first (D4).

```
#people: List of Text = ["Tankun", "Mult", "Claude"]
#scores: Dictionary, key: Text, value: Whole = ["Tankun": 1000, "Mult": 850]    // pair syntax: proposal
```

`HashMap`'s random order means nobody leans on an order that was never promised, and it protects against
hash-flooding attacks. Use `Dictionary` when order matters.

## 2.4 `or` and `nothing`

There is no NULL. A value that might be missing says so in its type (D11):

```
#phone: Text or nothing = nothing
#name: Text = "Mult"          // can never be nothing
```

`nothing` (no value) is different from `empty` (an empty text or list). Using a value that might be `nothing` without
checking is refused, and the error shows how to check: `if #phone equals nothing:`.

`or` works between any types (D12). `is` checks which one a value is:

```
#id: Text or Whole = 1042
if #id is Whole:
    ...
end if
```

## 2.5 `one_of`: plain named values

A `one_of` is a type whose value is one of a closed list of plain names (D31). It's declared once, as a block, and
used by its name:

```
one_of Order:
    before
    same
    after
end one_of

#order: Order = before
```

The values are names, not numbers in disguise, so `before + 1` is refused.

## 2.6 `tagged_union`: cases that carry data

A `tagged_union` is the precise layer beside `one_of` (D30, D31): each case may carry its own data, like Rust's enums.

```
tagged_union Result:
    worked
    failed[code: Whole]        // spelling of a case with data: proposal
end tagged_union
```

A field is read with `.`, and declared without `#`, since it's a part of a value, not a name of its own (D41).
Reading a case's field is allowed only where Fale knows the value is that case, such as inside `case failed:`:

```
match #result:
    case worked:
        show_in_text("Done")
    case failed:
        show_in_text("Failed with code " #result.code)
end match
```

A case with data is built with every field labelled, mirroring its declaration (D42):

```
#result: Result = failed[code: 3]
#done: Result = worked
```

## 2.7 Conversion

A value is converted with `as` (D57). How it's written depends on what can go wrong:

```
#label = #count as Text                      // always exact

#age = #input as Whole otherwise:            // can fail
    case not_a_number:
        show_in_text("That's not a number")
end otherwise

#dollars = #price as Whole, rounding: down   // loses information, says how
```

- **Always exact**, written plainly: `Whole` to `Decimal` or `Text`, a sized type to `Whole`.
- **Can fail**, handled with `otherwise:`: `Text` to `Whole`, `Whole` to `i32`, `Decimal` to `Whole`.
- **Loses information**, refused unless it says how: `Float` to `Whole`, `Decimal` to `Float`.

`rounding:` takes one of six words (D59):

| Word | 2.7 | 2.5 | 3.5 | -2.7 |
|---|---|---|---|---|
| `down` | 2 | 2 | 3 | -3 |
| `up` | 3 | 3 | 4 | -2 |
| `nearest` (halves away from zero, as at school) | 3 | 3 | 4 | -3 |
| `toward_zero` (cuts the fraction off) | 2 | 2 | 3 | -2 |
| `nearest_even` (halves to the even number) | 3 | 2 | 4 | -3 |
| `away_from_zero` | 3 | 3 | 4 | -3 |
