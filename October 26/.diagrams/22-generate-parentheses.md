# LC 22 — Generate Parentheses (Medium)

**Core idea:** Build the string character by character. At every step we may
add `'('` or `')'` subject to **two invariants**:

```
  1. We never add more ')' than '(' at any prefix   (close ≤ open)
  2. We never add more than n opening brackets      (open  ≤ n)
```

When both counters reach 0, we have a complete valid string.

## The two counters

```
  open   = number of '(' placed so far
  close  = number of ')' placed so far

  constraints  (n = 3 for this trace)
  ──────────────
  open  ≤  n        (can't exceed total opens available)
  close ≤  open     (can't close more than we've opened)
  close ≤  n        (can't exceed total closes available)
```

## Decision tree for n = 3

```
                            (n=3, open=0, close=0, "")
                                 │
                  ┌──────────────┴──────────────┐
                add '('                       add ')'  ✗  (close>open — invalid prefix)
                  │
            "(", open=1, close=0
                  │
         ┌────────┴────────┐
       add '('          add ')'  ✗  (close>open)
         │
     "((", open=2, close=0
         │
    ┌────┴────────────────┐
  add '('              add ')'
    │                     │
  "(((", o=3,c=0       "(()", o=2,c=1
    │                     │
  (close>open — DEAD)  ┌──┴─────────────┐
                       │                │
                    add '('          add ')'
                       │                │
                  "(()(", o=3,c=1   "(())(", o=2,c=2
                       │
                  ┌────┴───────┐
               add '('     add ')'
                  │            │
             (open==n,       "(()()", o=2,c=2
              can't '(')        │
                          ┌────┴─────────┐
                       add '('        add ')'
                          │              │
                  (open==n,          "(()())", o=2,c=3
                   can't '(')           │
                                    add ')'  ──▶  "(()())"  ✓  ANSWER
```

(Full expansion produces all 5 answers; this excerpt shows the spine that
leads to `"(()())"`. Other valid outputs: `((()))`, `(())()`, `()(())`, `()()()`.)

## Pruning rules — why this never blows up exponentially

```
  Rule 1:  open  < n      ──▶  we can still add '('
  Rule 2:  close < open   ──▶  we can still add ')'

  If neither rule fires, we're at a LEAF.
    ├─ open == n  AND  close == n  ──▶  complete valid string → add to answer
    └─ otherwise                   ──▶  dead end, discard
```

Without the rules, the brute-force tree would be 2^(2n) leaves — only a
fraction of those are valid. The two rules prune at every node.

## Final answer set for n = 3

```
  ((()))      push '(' ×3, then ')' ×3
  (()())     interleave one early close
  (())()     close the first pair, then open another pair
  ()(())     open, close, then open pair and close both
  ()()()     open-close three times in a row
```

That is **Catalan(3) = 5**, matching the formula:

```
  Catalan(n) = (1/(n+1)) * C(2n, n)
             = (2n)! / (n! * (n+1)!)
```

```
  n  Catalan(n)
  ─  ──────────
  1      1
  2      2
  3      5     ←  this problem
  4     14
  5     42
```

## Stack / recursion shape

Each call to `backtrack` represents one **node** in the tree above.
The `temp` string is mutated in place:

```
  push '('      ──▶  recurse
  recurse returns
  pop '('       ──▶  backtrack, try alternative
```

This push-pop dance is the textbook DFS pattern. Without the `pop_back`,
leftover characters from sibling branches would leak into each other.

```
  Frame 1:  temp = ""           ─┐
                                  │  push '('
  Frame 2:  temp = "("          ─┤
                                  │  push '('
  Frame 3:  temp = "(("         ─┤
                                  │  push ')'
  Frame 4:  temp = "(()"        ─┘
                                  base case  (open=1, close=2, n=3)
                                  pop ')'   ←  unwind
                                  ...
```

## Code ↔ diagram mapping

```
  if (open > 0) {              ◀──  Rule 1: still have opens to place
      temp.push_back('(');
      backtrack(temp, open-1, close);
      temp.pop_back();         ◀──  backtrack (un-mutate)
  }
  if (close > open) {          ◀──  Rule 2: can still close
      temp.push_back(')');
      backtrack(temp, open, close-1);
      temp.pop_back();
  }

  if (open == 0 && close == 0) ◀──  LEAF: complete valid string
      sol.push_back(temp);
```

## Complexity

```
  Time   O(Catalan(n) · n)  — number of valid outputs × length of each
  Space  O(n)               — recursion depth + `temp` string
```

The total number of recursive calls is bounded by the total number of nodes
in the pruned tree, which is O(4^n / sqrt(n)) by the Catalan generating
function — exponential in n, but dramatically less than the unpruned 4^n.
