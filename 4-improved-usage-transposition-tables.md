# Improved usage of transposition tables in alpha-beta search

*Date: October 19, 2023*

I've improved the usage of transposition tables, both generally and in particular in my alpha-beta search method, `negamax()`.

## The general improvements

### Storing/reading the best move

Now the table entries include the typical field for the best move found in a position.

When a move is found in an entry, the engine prioritizes it for move ordering, so that it will be checked before any other move when doing the move evaluation loop in alpha-beta search.

### Aging

Until now, transposition tables were always cleared just before starting a new search from the root. This had at least two disadvantages:

* Subsequent moves in the game didn't have access to all the potentially useful information stored during previous searches.
* The cleaning operation can have a cost, depending on how large the table is.

Now the tables are never cleared, only overwritten. The entries in the tables use a simple version of "**depth aging**", to prevent too old entries to stay in the table for too much time and steal precious space that could be used to store information about the ongoing ply.

Depth aging is achieved simply adding to the depth value a number that depends on the current ply, meaning the half-move reached in the game that the engine is playing.

This is the code:

```cpp
// Return an "aged" version of the depth passed to the transposition table get/store functions.
inline int agedDepth(const int depth) {
    return depth + Engine::game_plies * TT_AGINGFACTOR;
}
```
, where `TT_AGINGFACTOR` is a constant currently set to 4 in the config file.

This value has been determined empirically, after conducting a few tests, but I have not invested much time on its optimization because I have more stringent priorities and at the moment my tests are too superficial and flawed anyway.

There are more sophisticated and more space-saving methods to handle aging in transposition table entries but, again, what I'm doing is simple and good enough for now.


## TT usage in alpha-beta search

### Better starting values

Beside using the stored move to prioritize its search, my `negamax()` method now uses both this information and the value found in the TT entry for better initialization of the corresponding variable used in the move loop:

```cpp
int bestValue = (alpha != alphaOrig ? alpha : -BIGNUMBER);
```

In this way, if I know that the value found in the entry contributed to update alpha, I can simply use it as a starting value better than `-BIGNUMBER`.

### Moving table lookups after the check for leaf node

Previously, table lookups were done at the very top of the alpha-beta search, even for leaf nodes (i.e. when depth is 0). This did increase beta cutoffs a bit but, upon further tests, it turned out that it also contributed to **slow down the engine considerably**.

Now I have moved the check for depth-0 to the top of `negamax()`, so that the function could return immediately instead of doing a table lookup. This is the code:

```cpp
if (depth == 0) {
    return color * evaluate(board);
}
```

I was **astonished** to discover that this simple optimization led to a 20-25% increase in speed!

In part, this happens because the current evaluation logic is super-simple and implemented in a quite fast way, so calling `evaluate()` costs really nothing compared to a transposition table lookup and subsequent checks.

I expect this to change when I'll start working on a serious evaluation function, but for the moment I'll just enjoy the bonus speed!

## Conclusions

I'm still not particularly happy with how (relatively) little the transposition tables have contributed to speed so far, but again, it's early days and I'll have to see what happens when I implement iterative deepening and additional pruning methods.

---

*ViperMate* chess engine by *Enrico Altavilla*