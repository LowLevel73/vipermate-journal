# Implementing Null-Move Pruning and "search flags"

*Date: October 26, 2023*

I decided to postpone the tedious activity of adding the UCI interface to ViperMate, because I realized that a way sweeter programming candy was just around the corner: adding Null-Move Pruning (NMP).

I don't regret doing it. During my research in the last weeks, it was clear that it was one of the kind of puning techniques that could increase significantly the overall pruning performance of the engine. And so it did.

The quantity of evaluated nodes and the speed have changed by a factor of 0.5 and 2 respectively, letting me finally use depth 10 in the self-play games that I use to test the performance of the software.

The current version of NMP is a "vanilla" one, meaning that the **depth reduction factor** (*R*) is fixed and currently set at 2 in the config file.

More sohisticated chess engines calculate *R* dinamically, so that the search can be less deep in some circumstances, but I don't have the infrastructure to identify automatically the best values, yet, so I'll keep it simple for the time being.

## Search flags

Maybe even more importantly, alongside the implementation of NMP it was necessary to add a logic that has been on my radar for some time: **different types of nodes and their usage in recursive search**.

I have create the following enumeration:

```cpp
enum SearchFlag {
    PV_NODE = 1,
    ROOT_NODE = 2,
    NULLMOVE = 4
};
```
The values are, of course, powers of two so that they can be used and combined as flags. In other words: a **bitmask**.

I have then added a parameter to my `negamax()` method, called simply `flags`, with two advantages.

**First**, every call to `negamax()` can now specify the kind of node that the call has to inspect. For example, the method of ViperMate that implements Iterative Deepening and that calls `negamax()` can do it in the following way:

```cpp
besteval = negamax(board, i, alpha, beta, SearchFlag::ROOT_NODE);
```

**Second**, `negamax()` can use these flags to understand what to do or not to do. For example, doing a null-move search immediately after another is something that needs to be **avoided** and the method does it by checking the flags:

```cpp
    if (USE_NMP && depth >= NMP_DEPTH
       && !(flags & (SearchFlag::NULLMOVE | SearchFlag::PV_NODE | SearchFlag::ROOT_NODE))
       && !board.inCheck()
       && board.hasNonPawnMaterial(board.sideToMove())) {

        board.makeNullMove();

        // Get the evaluation of the position after the null move.
        int null_value = -negamax(board, depth - NMP_REDUCTION - 1, -beta, -beta + 1, SearchFlag::NULLMOVE);

        // Rest of the NMP logic...
```

By using a bitmask, it's possible to check multiple flags just with one operation, which is both convenient and fast.

---

*ViperMate* chess engine by *Enrico Altavilla*