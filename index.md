# A journal for the ViperMate chess engine

![ViperMate logo](/images/vipermate-logo-2.png)

**ViperMate** is my pet project: developing a **chess engine**.

My only goals are to *learn* the themes of chess programming and *have fun* by focusing on the activity that I like most: algorithm and code optimization. No expectations, no stress, just enjoying the journey.

This journal documents the path taken so far.

*Enrico Altavilla*

## Status

Alpha version. Towards the first public release of a beta version.

### Technologies implemented at the moment:

* Alpha-beta search with negamax.
* Iterative Deepening
* Transposition Tables
* Search controllable by external functions
* Engine modes (benchmark, UCI, etc.)
* Pruning techniques:
    * Alpha-beta pruning
    * Principal Variation Search
    * Null-Move pruning
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
