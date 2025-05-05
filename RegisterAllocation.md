```scala
def example(a: Int, b: Int): Int = {
  var x = a + b
  var y = a - b
  var z = 1

  for (i <- 1 to 3) {
    x = x * i
    y = y + x

    for (j <- 1 to 2) {
      z = z + x - y
    }
  }

  val w = x + y + z
  w
}
```

### Questions

1. **Live‐Range Analysis**
   Compute the live‐range (instruction span) for each variable (`a`, `b`, `x`, `y`, `z`, `i`, `j`, and `w`) in the above code. Show the live‐range as the interval `[first_use, last_use]` in terms of the line numbers of the Scala snippet.

2. **Graph‐Coloring Allocation (No Spills)**
   Build the interference graph for the virtual registers corresponding to the variables (treat each variable as a virtual register) and then assign physical registers by coloring the graph with **K = 4** colors. Show your coloring (i.e., which virtual register gets which color/physical register).

3. **Graph‐Coloring Allocation (With Spills)**
   Repeat the graph‐coloring register allocation with **K = 2** registers. Identify which variable(s) must be spilled (i.e., assigned to memory) and show the final coloring (with spilled variables noted).

4. **Linear‐Scan Allocation (No Spills)**
   Using the same live‐range intervals from Q1, perform a basic linear‐scan register allocation with **K = 4** registers. Show the order in which you allocate/freeze registers and the final assignment.

5. **Linear‐Scan Allocation (With Spills)**
   Perform linear‐scan allocation again with **K = 2** registers. Show which variable(s) get spilled and at what point in the scan they are chosen for spilling.

<details>
<summary>Answer Key</summary>

**Q1. Live‐Range Intervals**
*Assuming line numbers 1–11 for the snippet above:*

* `a`: `[1, 11]` (parameter, used in lines 2 and 3)
* `b`: `[1, 11]`
* `x`: `[2, 10]` (defined line 2, last used in computing `w` on line 10)
* `y`: `[3, 10]`
* `z`: `[4, 10]`
* `i`: `[6, 9]`
* `j`: `[8, 9]`
* `w`: `[10, 11]`

---

**Q2. Graph‐Coloring (K = 4)**

1. Build interference edges by overlapping live‐ranges.
2. With 4 colors, every node has degree < 4, so no spills.
3. One possible coloring:

   * `a` → R0
   * `b` → R1
   * `x` → R2
   * `y` → R3
   * `z` → R0
   * `i` → R1
   * `j` → R2
   * `w` → R3

---

**Q3. Graph‐Coloring (K = 2)**

1. With only 2 colors, nodes of highest degree (e.g. `x`, `y`, `z`) exceed 1 neighbor limit at simplify time.
2. Spill choice (lowest cost): spill `z`.
3. Remove `z`, color remainder with 2 colors:

   * `a`, `x`, `w` → R0
   * `b`, `y`, `i`, `j` → R1
4. `z` is assigned to memory.

---

**Q4. Linear‐Scan (K = 4)**

1. Sort intervals by start: `a`, `b`, `x`, `y`, `z`, `i`, `j`, `w`.
2. Allocate R0–R3 in turn; no interval overlaps more than 4 at once.
3. Final: same as Q2 assignment.

---

**Q5. Linear‐Scan (K = 2)**

1. Sort intervals; start assigning R0, R1.
2. At the 3rd live interval (`x`), no free register → choose the active interval ending farthest away to spill, say `a`.
3. Spill `a` to memory, free its register, and continue.
4. Result:

   * Registers: e.g. `b`→R0, `x`→R1, `y`→R0, `z`→R1, `i`→R0, `j`→R1, `w`→R0
   * Spilled: `a`

</details>
