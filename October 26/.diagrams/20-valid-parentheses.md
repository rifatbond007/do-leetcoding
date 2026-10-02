# LC 20 — Valid Parentheses (Easy)

**Core idea:** A **stack** is a perfect match for the LIFO property of
bracket pairs. Every opener must be closed by its mirror, in **reverse order**.

## High-level flow

```
for each char c in s
  ├── c ∈ { ( [ {   ──▶  push(c) onto stack
  └── c ∈ { ) ] }   ──▶  pop top; check it matches c
                       └── mismatch OR leftover in stack  ──▶  INVALID
```

## Walk-through on `s = "{[]}"` (valid)

```
  char    stack (top →)    action
───────── ──────────────── ─────────────────────────────────
   {       { }              push '{'
   [       [ { }            push '['
   ]       { }              pop '[';  '[' matches ']'  ✓
   }       { }              pop '{';  '{' matches '}'  ✓

stack is empty  ──▶  VALID ✓
```

## Walk-through on `s = "([)]"` (invalid — wrong nesting order)

```
  char    stack (top →)    action
───────── ──────────────── ─────────────────────────────────
   (       ( }              push '('
   [       [ ( }            push '['
   )       [ ( }            top is '[' but we saw ')'
                                ──▶ mismatch  ──▶  INVALID
```

Notice: the stack still has `(` left over, AND the top is `[` not `(` — so
the mismatch is caught **immediately** when we see `)`.

## Why a stack and not a queue?

```
Opening:  (  [  {
                  ▲
                  └── last opened must close first  →  LIFO  →  STACK
```

A queue would force FIFO, which means the *first* opened bracket would be
the first to close — wrong for nested structures.

## Match table (constant-time lookup)

```
  top of stack   ──▶   expected closer
  ───────────        ────────────────
       (                 )
       [                 ]
       {                 }
```

The code does this inline (no map needed): a single comparison with three
`(c==')' && top=='(') || ...` clauses — equivalent to `pairs[top] == c`.

## Edge cases the stack handles for free

```
  ""         ──▶  empty stack at end                  ──▶  VALID
  "((("      ──▶  leftover '{' in stack at end         ──▶  INVALID
  "))"       ──▶  pop on empty stack                   ──▶  INVALID
  "()"       ──▶  push, pop, empty                      ──▶  VALID
```

## Complexity

```
  Time   O(n)   — one pass, each char push/pop at most once
  Space  O(n)   — worst case all openers, stack holds n chars
```

## Code ↔ diagram mapping

```
  if (c=='(' || c=='{' || c=='[')  ◀──  "push branch" in the flow chart
      st.push(c);

  else {                            ◀──  "pop branch"
      if (st.empty()) return false; ◀──  "pop on empty"  → INVALID
      if ( (c==')' && st.top()=='(') ||   ◀──  "match table" check
           (c==']' && st.top()=='[') ||
           (c=='}' && st.top()=='{') )
          st.pop();
      else
          return false;             ◀──  mismatch  → INVALID
  }

  return st.empty();                ◀──  "leftover in stack"  → INVALID
```
