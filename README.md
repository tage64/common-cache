# common-cache

A hierarchical cache data structure that prioritizes the most commonly used and recently accessed
items and can dynamically grow and shrink in size.

This is a small, fast-on-average, and intuitive cache-like structure that keeps and promotes the
most commonly and most recently used items.

---

## General Characteristics

The structure roughly has the following properties:
- Elements that are used more often and more recently are placed higher in the cache.
- Items that haven't been used for a while might be discarded from the cache.
- If many elements are used, the cache will grow in size. Conversely, if only a few elements are used, the cache will slowly discard the unused items and shrink in size.
- The cache uses some randomness under the hood to select which items to move down and discard. But don't worry—you can provide your own RNG if you want reproducibility.
- Most operations, like insert, remove, and lookup, run in logarithmic time.

---
## Should I Use This?

You may want to use this data structure if:
- You need a cache that prioritizes the most commonly and most recently used items.
- You want to search the cache, prioritizing the most commonly and most recently used items first.
- You need a cache that dynamically grows when many different items are used and shrinks when only a few items are in use.

One example use case is autocompletion for commands or searches. The cache can remember and autocomplete the most recently and most frequently used commands or queries.

---
## How it Works

The structure roughly works as follows:
- The cache is divided into levels, which grow exponentially. Think of it as a pyramid:
  - At the top, there is a level with 2^0 = 1 item.
  - Below, there is a level with capacity for 2^1 = 2 items.
  - Next, a level with 2^2 = 4 items, then 2^3 = 8 items, and so on.
  - The base value of 2 is not fixed—any integral base greater than 1 can be used. This base is referred to as "base."
- The size of the level with index `n` is `base^n`. However, most levels will usually only be partially filled.
- When initializing the cache, all levels are empty.
- When inserting into the cache, the following steps are performed:
    1. If the item is already in the cache at level `l`, remove it and set the insertion level to be the level above `l` (if it wasn't already at the top). Then go to step 3.
    2. If the item wasn't in the cache, set the insertion level to be the second-lowest level. If the base is 2, this means it will be in the second quarter.
    3. Loop through all levels from the lowest (non-empty) level up to and including the insertion level. For each level:
        1. Generate a random integer `i` in the range `[0, base^l)`, where `l` is the index of the level (with the top level having index 0). This range corresponds to the size of the level.
        2. If `i` is less than the number of actual elements stored at that level, move the `i`-th item on this level to the level below.
    4. Finally, insert the item at the right level. There will always be at least one available slot because if the level were full, an item would have been pushed down to the level below in step 3.2 with 100% certainty.
- When an element is accessed with a `get` operation, it is removed from its current level and inserted into the level above, following the same algorithm described above. The only difference is that if an item is moved down from the lowest level, a new level is not created, and that item is discarded.
- When iterating over the cache, all levels are visited in order. Thus, no element on any level will come after an element on a lower level.
