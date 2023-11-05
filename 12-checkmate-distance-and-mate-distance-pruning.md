# Checkmate distance and Mate Distance Pruning
*Date: November 5, 2023*

The decision to tackle the implementation of the UCI protocol opened a positive Pandora's box. It contained many other overlooked aspects of the search algorithm that needed to be studied and finalized before they became unattainable monsters.

My mate distance logic was half-baked, so I made the following changes.

## Depth and ply

The negamax function now uses not only a `depth` parameter, but also a `ply` parameter.

While it's possible to use only depth to understand how far a checkmate is from the root, a `depth` variable is typically used in chess programming as a target value that *decreases* as the search algorithm traverses the tree towards the leaf nodes, which have depth zero.

On the other hand, `ply` is usually a variable with a value that starts at zero and *increases* with each deeper level of the search.

Besides being used to calculate a checkmate distance from the root node, another convenient advantage of having an increasing value is that it can be used to extend the search beyond the originally set target depth. **Search extensions** are a common feature of chess engines and sooner or later they will be implemented in ViperMate as well.

I'm not happy with the decision to pass a ply value through a function parameter, but for the moment it's an acceptable quick solution.

## The checkmate distance

The checkmate distance value must be designed so that a checkmate now is worse for a player than a checkmate later. This is the same concept used for evaluating any position: *a better move isn't necessarily something that maximizes a gain, but also something that minimizes a loss*.

For example, in ViperMate the nominal value of `MATE` is 30000. "White checkmated now" would get something like -30000 (negative values mean that Black wins) and "White checkmated two plies from now" would get something like -29998.

In both cases, White is screwed, but in the second case his bad fate is postponed a bit. Minimax algorithms are masochistic by nature and like to prolong their own agony as much as possible.


## Mate Distance Pruning

Mate Distance Pruning (MDP) is actually a simple concept that allows an alpha-beta search algorithm to do some "forward pruning".

This means that unlike other pruning methods, the kind of pruning achieved by MDP **does not lead to any improvement in finding better moves**, but only helps to speed up the search in case of checkmate.

This kind of pruning calculates the **mate score**, which is the value that a checkmate would have if it happened in the node currently being processed by the search algorithm.

Then it performs two checks:

1. If our (the maximizing player) worst-case score is worse than the score the opponent would get for checkmating us right now, we increase our worst-case score because we can never do worse than getting checkmated right now.
1. If the opponent's (the minimizing player) desired score is higher than the score we would get for checkmating them right now, we reduce opponent's desired score because they can never do better than getting checkmated right now.

Once the alpha or beta values have been updated in this way, Mate Distance Pruning checks for a beta cutoff and returns alpha if it has occurred.

Here is the current code in ViperMate:

```cpp
// Mate Distance Pruning.
if (USE_MDP && !(flags & SearchFlag::ROOT_NODE)) {      
    const int mate_score = ply - MATE;

    if (alpha < mate_score)
        alpha = mate_score;

    else if (beta > -mate_score)
        beta = -mate_score;

    if (alpha >= beta)
        return alpha;
}
```

There is other checkmate-related news about the development of ViperMate, but perhaps I'll address them in a future journal entry.


---

*ViperMate* chess engine by *Enrico Altavilla*