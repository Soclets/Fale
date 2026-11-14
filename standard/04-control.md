# 4. Control

## 4.1 Comparisons

| Write | Means |
|---|---|
| `#a equals #b` | the same (D6) |
| `#a not_equal #b` | not the same (D6) |
| `#a at_least #b` | `#a` is `#b` or more (D5, D26) |
| `#a at_most #b` | `#a` is `#b` or less (D26) |
| `#a > #b` | more than, not equal (D26) |
| `#a < #b` | less than, not equal (D26) |
| `#a is Whole` | a type check (D12) |

`>` and `<` are school symbols, so they stay symbols; `>=` and `<=` aren't, so the pair that includes the edge is
words (D26). What `<` and `>` do with `Text` isn't decided.

Conditions combine with `and`, `or` and `not` (D15):

```
if #health at_least 100 and #name equals "Mult":
if not #is_ready:
```

## 4.2 `if`

`if`, then any number of `else if`, then an optional `else` (D14):

```
if #health at_least 500:
    show_in_text("Healthy")
else if #health at_least 100:
    show_in_text("Hurt")
else:
    show_in_text("Almost dead")
end if
```

## 4.3 `match`

`match` handles every case of a `one_of` or `tagged_union`. It's the precise layer beside `if` (D30, D40):

```
match #order:
    case before, same:
        show_in_text("Not after")
    case after:
        show_in_text("After")
end match
```

A missed case is refused, and the error lists the missing ones. There is no catch-all: several cases can share one
`case` line. `match` is a statement, and gives no value.

## 4.4 Loops

Five loops, each saying what it does (D16):

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

## 4.5 `break`

`break` leaves the innermost loop (D17). There is no `continue` and no `skip`: an item is skipped by wrapping the work
in an `if`, so the skipped path shows as structure.

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
