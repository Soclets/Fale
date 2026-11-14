# 3. Names and values

## 3.1 Storing

`=` stores a value in a name (D2). A type goes after `:`:

```
#health: Whole = 1000
#name = "Tankun"            // whether the type can be left off isn't decided
```

## 3.2 Fixed and `mutable`

Names can't change by default. `mutable` marks the ones that can (D25):

```
#name = "Tankun"              // fixed
mutable #health = 1000        // can change
#health = #health - 10        // fine
#name = "Mult"                // refused: #name isn't mutable
```

A fixed name freezes the inside too: a fixed list can't be added to or changed.

## 3.3 `copy` and `share`

For collections that can change (List, Array, Dictionary, HashMap), storing or passing one always says whether it's a
copy or the same one (D19):

```
#b = copy #a       // its own list: changing #b never changes #a
#c = share #a      // the very same list: changing #c changes #a
#d = #a            // refused for these types: say copy or share
#x = #y            // fine for Whole, Text, Boolean, Decimal, Float: nothing to share
```

The same goes for passing them to a procedure, so a call that can change your list shows it:

```
add_guest(share #guests, "Mult")
count_guests(copy #guests)
```

`share` can't get around a fixed name: sharing a fixed list into something that would change it is refused (D25).
`copy` and `share` aren't ownership: the GC still frees memory (chapter 7).
