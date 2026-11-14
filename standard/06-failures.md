# 6. Failures

A procedure's header lists what can go wrong with `fails [...]`, and the call handles each failure by name with
`otherwise` (D22). There are no failure families. The exact spelling is a proposal.

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

How a procedure signals a failure from its body isn't decided.
