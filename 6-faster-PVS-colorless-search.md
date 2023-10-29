# Small optimizations: faster PVS and "colorless" alpha-beta search function

*Date: October 24, 2023*

While working on the more important additions to the engine, I took the opportunity to tweak a few aspects.

## Colorless alpha-beta search function

My alpha-beta search function is currently called "negamax" and it used a "color" parameter to automatically update (via a `-color` expression) the player associated with the next search depth.

I have removed it from the list of parameters and now the color is determined within the function by querying the board object. Overall, the search seems to be a bit faster.

## A faster Principal Variation Search

The Principal Variation Search starts with a null window search for the PV nodes, the important nodes for which we need an exact evaluation. A null-window search is faster than a full search, but sometimes it fails (the resulting value is between alpha and beta), and in this case it's necessary to perform a full search.

I did an analysis of the values of alpha, beta, and the evaluation, and it turns out that the re-search can be made faster by increasing alpha by 1, which makes the condition that leads to the re-search a little rarer.

This change has led to **more pruning** (about 3%) and, *apparently*, no problem with the result of the search.

To be on the safe side, I implemented this change by using a new constant `PVS_LESSRESEARCH = 1` in the config file. The code now looks like this:

```cpp
if (USE_PVS && move_number > 0) {
    // If this is not the first move, do a null window search.
    val = -negamax(board, depth-1, -alpha-1, -alpha);

    // If the value is within the window, do a full search.
    if (val > (alpha+PVS_LESSRESEARCH) && val < beta && depth > PVS_NORESEARCHDEPTH) {
        val = -negamax(board, depth-1, -beta, -alpha);
    }
}
```

I just hope it won't be a problem when I start seriously debugging the search over many different positions.

---

*ViperMate* chess engine by *Enrico Altavilla*