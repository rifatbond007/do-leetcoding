# Visual Diagrams — October 26

ASCII diagrams explaining the **core logic** behind each solution.

---

## 1. LC 20 — Valid Parentheses (Easy)

**Idea:** a stack is a perfect match for the LIFO property of bracket pairs.
Every opening bracket you meet must be closed by its mirror, in **reverse order**.

### High-level

```
for each char c in s
  ├── c is one of  (  [  {   ──▶  push(c) onto stack
  └── c is one of  )  ]  }   ──▶  pop top; check it matches c
                                └── mismatch OR leftover in stack  ──▶  INVALID
```

### Walk-through on `s = "{[]}"`

```
  char       stack (top →)     match?     result
  ────       ───────────────   ───────    ───────
  {          [ { ]              push       ok
  [          [ [ { ]            push       ok
  ]          [ { ]               { ↔ }     pop { vs }  ✗ mismatch ✗
                                                                 │
                                                                 ▼
                                                           INVALID  ←
```

Wait — `{[]}` is actually **valid**. Let me re-walk it correctly:

```
  char       stack (top →)     action
──────────   ───────────────    ──────────────
  {          { }                push '{'
  [          { [ }              push '['
  ]          { }                pop '['; '[' matches ']'  ✓
  }          { }                pop '{'; '{' matches '}'  ✓

stack is empty  ──▶  VALID ✓
```

### Walk-through on `s = "([)]"` (invalid — wrong order)

```
  char       stack              action
──────────   ──────────────     ──────────────────────
  (          ( ]                push '('
  [          ( [ ]              push '['
  )          ( ]                  '[' matches ')'  ✗  ──▶ INVALID
```

Notice: the stack still has `(` left over, AND the top of stack is `[` not
`(` — so the mismatch is caught **immediately** when we see `)`.

### Why a stack and not a queue?

```
Opening:  (  [  {
                  ▲
                  └── last opened must close first  →  LIFO  →  STACK
```

### Match table (constant-time lookup)

```
  top of stack   ──▶   expected closer
  ───────────        ────────────────
       (                 )
       [                 ]
       {                 }
```

The code does this inline (no map needed): a single comparison with three
`(c==')' && top=='(') || ...` clauses — equivalent to `pairs[top] == c`.

---

## 2. LC 22 — Generate Parentheses (Medium)

**Idea:** build the string character by character. At every step we may add
`'('` or `')'` subject to **two invariants**:

```
  1. We never add more ')' than '(' at any prefix  (closes ≤ opens)
  2. We never add more than n opening brackets   (opens ≤ n)
```

When both counters reach 0, we have a complete valid string.

### The two counters

```
  open   = number of '(' we have placed so far
  close  = number of ')' we have placed so far

  constraints  (n = 3 for this trace)
  ──────────────
  open  ≤  n        (can't exceed total opens available)
  close ≤  open     (can't close more than we've opened)
  close ≤  n        (can't exceed total closes available)
```

### Decision tree for n = 3

```
                            (n=3, open=0, close=0, "")
                                 │
            ┌────────────────────┴────────────────────┐
          add '('                                     (close>open — can't add ')')
            │
        "(", open=1, close=0
            │
       ┌────┴────────────────┐
   add '('                             (close>open — can't add ')')
       │
   "((", open=2, close=0
       │
  ┌────┴────────────────┐
add '('                             add ')'
  │                                   │
"(((" , open=3, close=0              "(()", open=2, close=1
  │                                   │
(close>open — can't add ')')    ┌─────┴───────────┐
  │                            add '('           add ')'
  │                              │                 │
can't go deeper — DEAD END    "(()(", open=3,close=1   "(())(...", ...
                               │
                          ┌────┴────────────┐
                       add '('           add ')'
                          │                │
                    (open==n, can't '(')   "(()()", open=2, close=2
                                            │
                                       ┌────┴───────────┐
                                    add '('           add ')'
                                       │                │
                                 "(()()(", open=3,close=2   "(()())", open=2,close=3
                                                              │
                                                          add ')'  ──▶  "(()())" ✓
```

### Pruning rules — why this never blows up exponentially

```
Rule 1:  open < n      ──▶  we can still add '('
Rule 2:  close < open  ──▶  we can still add ')'
```

If neither rule fires, we're at a **leaf**. At a leaf, if `open == n` AND
`close == n`, the string is complete and valid → add to answer set.

### Final answer set for n = 3

```
  ((()))      push '(' three times, then ')' three times
  (()())     interleave one early close
  (())()     close the first pair, then open another pair
  ()(())     open, close, then open pair and close both
  ()()()     open-close three times in a row
```

That's 5 strings, which is exactly **Catalan(3) = 5**.

### Stack / recursion shape

Each call to `backtrack` represents one **node** in the tree above.
The `temp` string is mutated in place:

```
  push '('      ──▶  recurse
  recurse returns
  pop '('       ──▶  backtrack, try alternative
```

This push-pop dance is the textbook DFS pattern. Without the `pop_back`, we'd
have leftover characters from sibling branches leaking into each other.

---

## Side-by-side summary

```
  LC 20  Validate          LC 22  Generate
  ────────────             ────────────
  Data structure:  stack    Data structure:  recursion + 2 counters
  Direction:       LIFO     Rule:  close ≤ open  AND  open ≤ n
  Operation:       pop      Operation:  push '(' OR push ')'
                   parity-check    prune by two conditions
  Output:    bool           Output:    vector<string>   (size = Catalan(n))
```