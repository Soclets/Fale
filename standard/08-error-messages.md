# 8. Error messages

## 8.1 Kinds

An error's first words say what happened to the program (D43):

- `Refused before running`: the checker found it, and the program never starts.
- `Stopped while running`: the running program hit it, and stops there.

## 8.2 Layout *(proposal)*

```
Stopped while running: main.fale:4:15, line 4 column 15
 4 |     #more = #big + 1
   |             ^^^^^^^^
 what:   `#big + 1` is too big for a Whole
 why:    a Whole only goes up to 9,223,372,036,854,775,807
 values: `#big` is 9,223,372,036,854,775,807, so `#big + 1` would be 9,223,372,036,854,775,808
```

Not decided yet: the rest of this layout, file extensions and exit codes (chapter 9).
