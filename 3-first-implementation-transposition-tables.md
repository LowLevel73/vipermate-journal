# The first implementation of transposition tables

*Date: October 16, 2023*

I've started to implement transposition tables.

As usual, I've adopted my preferred method of introducing new technology into the engine: adding one feature at a time. This will let me understand how each feature affects the engine and how to optimize it before moving to the more powerful ones, which could hide the weaknesses of the simpler ones. The approch could also simplify debugging.

Applied to transposition tables, this means that they will acquire the following features gradually:

1. Storing only the evaluation of a position, the depth at which it was made and what kind of value it is (exact, lowerbound, upperbound)
2. Storing also the best move found for a position, which will be used to speed-up search
3. Maybe adding an aging mechanism to the table, to avoid keeping too many old positions in it

## The current implementation

Currently, I have made the first step of the list.

Here is how the structure of the table looks like:

```cpp
struct TT_Entry {
    uint64_t key = 0;
    uint8_t type = NodeType::NONE;
    uint8_t depth = 0;
    int16_t value = 0;
};
```

I've prepared a possible second version of the structure in case I decide in the future to optimize the code by checking if the key matches only partially the requested hash. It's a dangerous business, because the new version includes some nasty **preprocessor checks for the endianness of the CPU**, so for now I'll keep this idea dormant.

Reading and writing from/to the tables has been implemented into my alpha-beta search. Before starting the usual move loop, I check if the current position is already in the table. If so, I use the stored evaluation to directly return it if it's exact, or to update apha or beta and then checking if the update fires a beta cutoff.

```cpp
uint64_t hash = board.hash();
    TT_Entry* t_entry = getTTentry(hash);

    if (t_entry != nullptr && t_entry->depth >= depth) {
        int t_type = t_entry->type;
        int t_value = t_entry->value;

        if (t_type == NodeType::EXACT) {
            return t_value;
        }
        else if (t_type == NodeType::LOWERBOUND)
            alpha = max(alpha, t_value);
        else if (t_type == NodeType::UPPERBOUND)
            beta = min(beta, t_value);

        if (alpha >= beta) {
            return t_value;
        }
    }
```

After the move loop, I store the evaluation of the position in the table, with the depth at which it was made and the type of value. I prioritize the creation of `LOWERBOUND` nodes over `UPPERBOUND` ones, because the former generally lead to more beta cutoffs.

```cpp
const NodeType nodeType = bestValue >= beta ? NodeType::LOWERBOUND : (bestValue <= alphaOrig ? NodeType::UPPERBOUND : NodeType::EXACT);

    // Transposition table storing.
    storeTTentry(hash, bestValue, depth, nodeType);
```

The storing method doesn't blindly overwrite any entry, but applies a simple overwrite logic:

```cpp
    if  ((entry->type <= NodeType::UPPERBOUND) ||
        ((depth >= entry->depth) && (type >= entry->type || entry->key != hash))) {
            entry->key = hash;
            entry->value = value;
            entry->type = type;
            entry->depth = depth;
    }
```

The new value gets stored if one of the following conditions is true:
* If the entry slot is empty or contains an `UPPERBOUND` value, just write on it.
* If there is a collision with a different position, overwrite it only if the new depth is greater (or equal) than the stored one.
* If it's the entry of the same position, overwrite it only if the new depth is greater (or equal) than the stored one, and the new type is "better" or the same.

**What does "better" mean, in this context**? I have assigned priorities to each type of evaluation and I overwrite an existing node only if the type of the new evaluation has a priority higher than the one in the existing evaluation.

`EXACT` has a greater priority that `LOWERBOUND`, which in turn has a greater priority that `UPPERBOUND`. This logic is implemented simply by sorting the value of the NodeType enum from the least important to the most important:

```cpp
// Type of node for the transposition table.
// NOTE: the order of the values is important!
enum NodeType {
    NONE,
    UPPERBOUND,
    LOWERBOUND,
    EXACT
};
```

## The improvements so far

When transposition tables are switched on, the number of visited/evaluated nodes gets reduced by about 50% and the overall speed of the engine has also improved by about 50%.

It's definitely not bad, especially considering that the current implementation doesn't even include the storing of the best moves found, but somehow I expected more.

All the literature on transposition tables explains how a gamechanger they are for a chess engine, so I more speedup opportunities will be observed after their full implementation.


## A possible bug and transposition table behavior

When I tried to adjust the number of stored killer moves, I noticed that the final moves chosen by the engine can change. This doesn't happen when using the transposition tables.

At first I thought this was a bug, because my (wrong) understanding of how transposition tables work was that they should only speed up the process, not the behavior of the search algorithm.

Upon further research, it turned out that transposition tables can introduce **search instability** if they create cutoffs at earlier depths in the search (which is the standard way they are implemented).

A simple way to understand if the observed behavior was actually a byproduct of earlier cutoffs was to limit the extraction of values from the tables in `negamax()` only if the current search depth matched the depth stored in the table entry.

When I tried this, the final moves chosen by the engine became exactly those chosen when transposition tables were disabled. So it turns out that this wasn't a bug after all.

My understanding of transposition table behavior has definitely improved mainly thanks to [a course by Professor Jonathan Schaeffer](https://webdocs.cs.ualberta.ca/~jonathan/PREVIOUS/Courses/657/index.html). Among other things, he explains how transposition tables can introduce instability in the search:

> TT Anomalies
> * TT may give a more accurate result than needed
>   * May result in a solution that normally should not be found
>   * But this may depend on the order in which successors are expanded

---

*ViperMate* chess engine by *Enrico Altavilla*