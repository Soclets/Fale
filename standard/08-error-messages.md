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
 what:    `#big + 1` is too big for a Whole
 why:     a Whole only goes up to 9,223,372,036,854,775,807
 context: `#big` is 9,223,372,036,854,775,807, so `#big + 1` would be 9,223,372,036,854,775,808
```

`context:` gives facts about what the program had at that point: values, which type something is, what's known
(D45). It never holds advice or guesses.

## 8.3 Every error at once

Every error is shown at once, without knock-on errors (D46). The checker applies an error's fix in memory only, never
in the file, and checks again; an error that disappears was caused by the first one, and is shown as a `because:`
pointing back. With more than one real fix, the first is applied. With no certain fix, only the broken line is
skipped, and nothing that's only wrong because something else is unknown is reported.

## 8.4 Templates

Every message is a template with slots, filled from the program: names and expressions exactly as written, types,
values, the place, and fixes built from the actual code (D47). The catalogue of templates is section 8.5, and both the
oracle and the VM follow it word for word.

```
what:    `{expression}` is too big for a {type}
what:    `#big + 1` is too big for a Whole
```

## 8.5 Catalogue

*(Empty so far: templates are added as each error is designed.)*

Not decided yet: the rest of this layout, file extensions and exit codes (chapter 9).
