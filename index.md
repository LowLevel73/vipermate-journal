# A journal for the ViperMate chess engine

![ViperMate logo](/images/vipermate-logo-3.png)

**ViperMate** is my pet project: developing a **chess engine**.

My only goals are to *learn* the topics of chess programming and to have *fun* by concentrating on the activity I like the most: optimizing algorithms and code. The evil practice of premature optimization is welcome here, because this is not a serious project. No expectations, no stress, I just enjoy the journey. This journal documents the path taken so far.

*Enrico Altavilla*

## Status of the chess engine

ViperMate is in its **alpha** stage: the technological backbone is largely implemented. Now it has to learn all the rules of a chess game and how to interact with the outside world. The goal is to release its source code when it becomes a fully functional UCI-compliant engine.

### Technologies implemented at the moment:

* Alpha-beta search with negamax
* Quiescence search
* Iterative Deepening
* Transposition Tables
* Search controllable by external functions
* Engine modes (benchmark, matefinder, UCI)
* UCI support
* Pruning techniques:
    * In Alpha-beta search: Alpha-beta pruning, Principal Variation Search, Null-Move pruning, Mate Distance Pruning
    * In Quiescence search: Alpha-beta pruning, Standing-pat check, Delta pruning
* Move ordering/scoring:
    * Killer moves
    * Positional gain of a move
    * Moves found in transposition tables
    * MVV-LVA captures (Most Valuable Victim - Least Valuable Attacker)
* Evaluation function
    * Incrementally updated at each move
    * Material balance
    * Positional balance, from piece-square tables

## Journal entries

[Finalizing the evaluation function and endgames](16-finalizing-evaluation-function-and-endgames.md)
*November 16, 2023*

[Quiescence search is a game-changer (literally)](15-quiescence-search-game-changer.md)
*November 14, 2023*

[The surprising effects of the "default" move ordering](14-surprising-effects-default-move-ordering.md)
*November 13, 2023*

[ViperMate gets a UCI interface](13-vipermate-gets-uci-interface.md)
*November 11, 2023*

[Checkmate distance and Mate Distance Pruning](12-checkmate-distance-and-mate-distance-pruning.md)
*November 5, 2023*

[The design of internal communications and a nasty G++ -O3 optimizer issue](11-design-of-internal-communication-and-optimizer-issue.md)
*November 3, 2023*

[Silly experiment with parameter optimization for move ordering](10-silly-experiment-parameter-optimization.md)
*October 30, 2023*

[Refactor: search controller and game modes (benchmark, UCI...)](9-search-controller-game-modes.md)
*October 28, 2023*

[Implementing Null-Move Pruning and "search flags"](8-null-move-pruning-search-flags.md)
*October 26, 2023*

[Reorganization: using Git and separating the code into different files](7-reorganization-git-several-files.md)
*October 25, 2023*

[Small optimizations: faster PVS and "colorless" alpha-beta search function](6-faster-PVS-colorless-search.md)
*October 24, 2023*

[Phase 2: shifting from a playground to an actual chess engine](5-phase-2-from-playground-to-engine.md)
*October 23, 2023*

[Improved usage of transposition tables in alpha-beta search](4-improved-usage-transposition-tables.md)
*October 19, 2023*

[The first implementation of transposition tables](3-first-implementation-transposition-tables.md)
*October 16, 2023*

[A curious lump in beta cutoff statistics](2-lump-beta-cutoff-statistics.md)
*October 13, 2023*

[Starting a journal for ViperMate](1-starting-a-journal-vipermate.md)
*October 12, 2023*

---

*ViperMate* chess engine by *Enrico Altavilla*
