# origami-fold

A type class of pure and recursive monadic origami-like lazy monoidal mapping folds.

Performs an analysis, monoidally, and transformation, top-down or bottom-up, in a single pass; traverses a structure to produce a monoidal result where the intermediate and final monoidal values can be used to transform the values of the structure, [tying the knot](https://wiki.haskell.org/Tying_the_Knot).

For instance, `foldo` can be used to calculate variance of a series and label each member of the series with its deviation in a single pass.

Evaluation is driven by the choice of monoid. In pure computations, evaluation order is based on the monoid chosen. In monadic compuations, evaluation order is driven by the underlying recursive monad.

`foldo` can be seen to simultaneously traverse both left-to-right and right-to-left or top-down and bottom-up. To be productive, however, one approach must be chosen at the use site; choosing both top-down and bottom-up simultaneously would result in ```<<loop>>```. There are derivative methods to prevent such misuse:

These methods (and their (M)onadFix implementations) are safe:
* foldll/M; fold from the left and combine monoidal values from the left; access ancestors for monoidal production
* foldlr/M; fold from the left and combine monoidal values from the right; access ancestors for monoidal production
* foldrl/M; fold from the right and combine monoidal values from the left; access descendants for monoidal production
* foldrr/M; fold from the right and combine monoidal values from the right; access descendants for monoidal production

There are currently instances for `[]`, `Tree`, and `Seq`.

If you know the name of this style of fold or find where I got the laziness or implementation wrong, I'd love to hear about it.

### Example

As an example, imagine walking a structure and, for each element, `c`, labeling `c` with the length of the path from the starting element to `c` plus the count of elements after `c` plus one for `c` divided by the total number of elements in the structure - a relative weight of `c` within the structure.

```haskell
foldll (\ancestors@as descendants@ds total@t current@c -> (getSum (as <> ds <> Sum 1) / getSum t)) (\_ _ -> Sum 1)
```

For a tree, this can be seen as isolating the path to an element plus possible future paths and weighing that relative to the entire tree. This is a powerful lazy traversal operating in a single pass.

```
 1               ==>        1.0
 |                          |
 +- 2                       +- 0.3
 |  |                       |   |
 |  `- 3                    |   `- 0.3
 |                          |
 +- 4                       +- 0.2
 |                          |
 `- 5                       `- 0.7
    |                           |
    `- 6                        `- 0.7
       |                            |
       `- 7                         `- 0.7
          |                             |
          +- 8                          +- 0.5
          |                             |
          +- 9                          +- 0.5
          |                             |
          `- 10                         +- 0.5
```

Note that the above fold doesn't utilize the previous monoidal value or the current element in the production `(\_ _ -> Sum 1)`. Origami folds are capable of generating their monoidal values relative to historical monoidal values and the current element before transformation; this production just happens to be relatively simple.
