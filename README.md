# search a 2d matrix ii: the corner-elimination trick, why flattening breaks, and how to explain it in an interview

Problem 240 shows up in interview loops more often than its difficulty tag suggests. It's rated Medium, it looks like a warm-up, and then a surprising number of candidates either flatten the matrix into one sorted array (wrong) or start a binary search from the middle (also wrong). The fix is a four-line loop, but the reasoning behind it is what interviewers actually want to hear.

Here's the problem, the two approaches that work, the ones that don't, and the edge cases that quietly cost people the round.

## The two properties you get, and the one you don't

You're given an `m x n` integer matrix where two things are true:

- Integers in each row are sorted in ascending order, left to right.
- Integers in each column are sorted in ascending order, top to bottom.

Return `true` if `target` appears anywhere in the matrix.

The LeetCode example:


[ 1,  4,  7, 11, 15]
[ 2,  5,  8, 12, 19]
[ 3,  6,  9, 16, 22]
[10, 13, 14, 17, 24]
[18, 21, 23, 26, 30]


`target = 5` → `true`. `target = 20` → `false`.

Constraints are worth reading before you code: `1 <= m, n <= 300`, and values (including the target) range from `-10^9` to `10^9`. No empty matrix inside the test data, but you'll still want a guard for it in an interview — the interviewer notices.

Now the part people skip. Problem 74 ("Search a 2D Matrix") adds a third property: the first element of each row is greater than the last element of the previous row. That extra property means the whole grid is one globally sorted sequence, so you can pretend it's a flat array of length `m * n`, binary search it in `O(log(mn))`, and map an index back with `row = mid // n, col = mid % n`.

Problem 240 does not have that property. Look at row 0, column 4: `15`. Row 1, column 0: `2`. Fifteen sits *above* two in reading order. There's no total ordering across the grid, so flattening it produces an unsorted array and your binary search returns garbage.

> The single most common wrong answer to 240 is the problem-74 solution. The second most common is running the problem-74 solution and then being confused when it fails on a test case with a small number in the bottom-left region.

## Start in a corner where one direction shrinks and the other grows

Pick the top-right corner. In that cell, two directions have opposite meanings:

- moving **left** gives you smaller values
- moving **down** gives you larger values

That asymmetry is the whole solution. From `(0, n-1)`:

- If the cell equals the target, return `true`.
- If the cell is **greater** than the target, everything below it in that column is even bigger (columns increase downward), so the entire column can be discarded: `col -= 1`.
- If the cell is **less** than the target, everything to its left in that row is even smaller (rows increase rightward), so the entire row can be discarded: `row += 1`.

Every iteration permanently removes one row or one column. That's at most `m + n` steps before you fall off the grid. No recursion, no extra memory.

Trace `target = 5` from the top-right to see the elimination in action:

| Step | Cell | Value | Decision |
| --- | --- | --- | --- |
| 1 | (0, 4) | 15 | 15 > 5 → drop column 4 |
| 2 | (0, 3) | 11 | 11 > 5 → drop column 3 |
| 3 | (0, 2) | 7 | 7 > 5 → drop column 2 |
| 4 | (0, 1) | 4 | 4 < 5 → drop row 0 |
| 5 | (1, 1) | 5 | match → `true` |

And `target = 20`, which isn't in the matrix:

15 < 20 (row 0 gone) → 19 < 20 (row 1 gone) → 22 > 20 (column 4 gone) → 16 < 20 (row 2 gone) → 17 < 20 (row 3 gone) → 26 > 20 → 23 > 20 → 21 > 20 → 18 < 20 (row 4 gone) → row index is now 5, loop ends, return `false`.

The bottom-left corner works exactly as well, with the comparisons mirrored. Here it is in Python:

python
def searchMatrix(matrix, target):
    if not matrix or not matrix[0]:
        return False

    rows, cols = len(matrix), len(matrix[0])
    r, c = rows - 1, 0          # bottom-left

    while r >= 0 and c < cols:
        if matrix[r][c] == target:
            return True
        if matrix[r][c] > target:
            r -= 1              # cell is the row minimum; drop the row
        else:
            c += 1              # cell is the column maximum; drop the column
    return False



Time:  O(m + n)
Space: O(1)


Two corners are useless: top-left and bottom-right. From the top-left, both right and down *increase*, so when the cell is smaller than the target you can't rule out either direction. Same problem at the bottom-right in reverse. If you can't name a direction that strictly eliminates a whole line, you don't have the trick yet.

## The other approaches that get accepted

The staircase is the expected answer, but it isn't the only one.

**Row-wise binary search.** Each row is individually sorted, so binary search every row independently. You're looking for the first index where `row[mid] >= target`; if that element equals the target, return `true`, otherwise advance to the next row. That's `O(m log n)` time and `O(1)` space. It's slower than the staircase but it's a completely legitimate answer, and it has a real advantage: it reuses a generic binary search template instead of a problem-specific gimmick.

This is the route AlgoMonster's own editorial for problem 240 takes — the "find the first true index" monotonic template, applied once per row. Same pattern that handles "find the first bad version," "find the boundary in a boolean array," and the rest of the first-true family. You can read their walkthrough of that approach here: 👉 [see how AlgoMonster solves Search a 2D Matrix II](https://algo.monster?fpr=li69).

**Divide and conquer.** LeetCode tags 240 with `Divide and Conquer` alongside `Binary Search` and `Matrix`. The textbook-ish version compares the middle element against the target and tries to eliminate a rectangle. The catch, noted in standard algorithms literature going back to Richard Bird's lecture notes on this exact question, is that a middle comparison eliminates one quadrant and leaves an L-shaped region — which doesn't fit a clean recursive call. You can patch it up, but you end up with a slower algorithm and a messier explanation.

| Approach | Time | Space | Worth using? |
| --- | --- | --- | --- |
| Brute force scan | O(m·n) | O(1) | Only as a baseline to describe before improving |
| Binary search each row | O(m·log n) | O(1) | Yes — simple, reuses a standard template |
| Staircase from top-right or bottom-left | O(m + n) | O(1) | Yes — this is the answer interviewers expect |
| Divide and conquer on quadrants | Awkward to bound cleanly | O(log) recursion | Technically valid, rarely worth the explanation |

## What AlgoMonster actually does with this problem

AlgoMonster is a text-and-diagram interview prep platform built by ex-Google engineers, organized around 48 explicit coding patterns rather than a giant problem list. Each pattern comes with a written explanation, a decision flowchart for recognizing when it applies, and template code you adapt — then a set of problems mapped to that pattern. The binary search module is where 240 lands, alongside the boundary-search and first-true variants.

Two things make it a reasonable fit for someone stuck on this specific problem:

- The editorial for 240 doesn't just print a solution. It builds the row-wise monotonic binary search out of the "find the first index where `row[mid] >= target`" template, so the code you write for 240 is the same shape of code you'd write for a dozen other problems.
- Recognition is usually the actual bottleneck, not implementation. If you've read the solution to 240 five times and still don't spot the pattern in a new problem, a flowchart that asks "is the input sorted? are you looking for a boundary?" is more useful than a sixth reading.

Honest limits, because they matter before you pay: there's no video. Everything is text, diagrams, and an in-browser editor. The problem bank is in the low hundreds, which is plenty for learning patterns and not enough if your gap is raw speed from volume — most people pair it with LeetCode for that. System design coverage is introductory, and there's no coaching or mock interviews with a human. If any of those is your main gap, this isn't the tool for it.

### Current plans and pricing

AlgoMonster runs annual or lifetime access — no monthly option. Prices move with promotions, which is why third-party write-ups quote anything from roughly $119 to $459 depending on when they were published. What the subscription page itself currently shows:

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Limited access to a sample of patterns and problems | $0 | No commitment | [start with the free tier](https://algo.monster?fpr=li69) |
| Pro — Yearly | Full pattern, lesson and problem library, 3,000+ editorials, unlimited AI assistant, unlimited Speedrun, questions collected from top tech companies and updated monthly, member discount on live bootcamps | $99 (~$8.25/mo billed annually) | Annual, cancel or upgrade anytime | [get Pro Yearly](https://algo.monster/subscribe?fpr=li69) |
| Pro — Lifetime | Everything in the annual plan, lifetime content updates, internal referral to a top tech company for qualified candidates | $189 | One-time | [get Pro Lifetime](https://algo.monster/subscribe?fpr=li69) |

A couple of practical notes. The subscribe page has advertised a 3-day trial before the annual charge kicks in, so read the checkout text rather than assuming. Discount aggregator pages list the pre-sale prices as $198/year and $378 lifetime, which would make the current figures roughly half off — but treat any specific percentage as seasonal, because I found no single discount code that verifies as working today. Foundation Course and a Build-an-LLM course are bundled with a subscription; specialist courses developed with outside partners, such as Quant Interview Prep, are sold separately.

The break-even question is simple arithmetic. If you're prepping for one interview cycle, yearly at $99 is the cheaper bet. If you expect to interview again in two or three years — which most engineers do — lifetime at $189 is under two years of annual renewals, so it wins on math alone.

## Edge cases that cost people the round

These are the ones that come up in this problem specifically, not generic advice.

**The loop bound.** From the bottom-left, the condition is `r >= 0 and c < cols`. Writing `r > 0` drops the top row and produces a wrong answer on inputs where the target sits in row 0 — a bug that only shows up on certain test cases, which is the worst kind of bug to debug live.

**Empty input.** `if not matrix or not matrix[0]: return False`. LeetCode's test data doesn't include this for 240, but interviewers ask about it, and `len(matrix[0])` throws on an empty list.

**Single row or single column.** One row degenerates into a plain linear walk, one column into a walk down the column. The staircase handles both without special-casing, which is a good reason to prefer it over row-wise binary search in a live setting.

**Negative values.** The constraints allow values down to `-10^9`, so don't write an early exit that assumes non-negative input.

**Range shortcut.** If `target < matrix[0][0]` or `target > matrix[m-1][n-1]`, you can return `false` immediately. It's a nice touch to mention, not a substitute for the loop.

**Confusing `m` and `n`.** Only relevant if you're doing the row-binary-search version, where the index mapping matters. You're indexing into a single row, so you're searching `0 .. len(row) - 1` — not `m * n`. Mixing the two coordinate systems inside one loop is the classic source of wrong answers here.

## The 90-second version to say out loud

If you want the phrasing, it's roughly this, in order:

1. "Each row and each column is sorted, but the rows don't chain — the first element of a row isn't necessarily greater than the last element of the row above. So I can't flatten this and binary search it the way problem 74 works."
2. "Instead I'll start at the top-right corner. Moving left decreases the value, moving down increases it."
3. "If the current cell is larger than the target, nothing below it in that column can be the target, so I discard the column. If it's smaller, nothing to its left in that row can be the target, so I discard the row."
4. "Each step removes a full row or column, so the loop runs at most `m + n` times. Time is `O(m + n)`, space is `O(1)`."

That's it. State the properties, name the corner, justify one elimination step, give the complexity. The code is four lines; the reasoning is the part being graded.

## Quick answers

**Why is `O(m + n)` the target and not something faster?** Because there's no global ordering to binary search against. The staircase gets its speed from eliminating a whole line per comparison, and `m + n` total eliminations is the natural ceiling for that argument.

**Can I use the staircase on problem 74?** Yes, it works there too. You just give up the `O(log(mn))` that the extra sorting property buys you.

**Does it matter whether I start top-right or bottom-left?** Not for correctness — both eliminate one row or column per step. Pick whichever you can explain without hesitating. Just don't start at top-left or bottom-right; neither gives you a direction that rules anything out.

**Is the row-wise binary search version wrong?** No. It's `O(m log n)` instead of `O(m + n)` and interviewers accept it. If your binary search template is shaky, the row-wise version may actually be the safer choice under pressure — and it's the one you'll find in 👉 [AlgoMonster's editorial walkthrough for this problem](https://algo.monster?fpr=li69), built from the same first-true template used across their binary search pattern.

**Is it worth paying for a prep platform to learn one problem?** No, and nobody should sell it that way. The case for a pattern-based course is the other 47 patterns and the recognition step in a live interview. If 240 is the only thing standing between you and a job, you don't need a subscription — you need twenty minutes with the trace table above.
