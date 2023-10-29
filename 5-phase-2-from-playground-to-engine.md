# Phase 2: shifting from a playground to an actual chess engine

*Date: October 23, 2023*

Until the few last days, ViperMate has been for me simply a messy playground to test some basic chess engine programming themes, like how to develop a search algorithm and how to optimize the exploration of a tree.

What its `main()` function does is simply to self-play a game and collect statistics on how the engine behaves and what its performances are in terms of pruning and speed.

Don't get me wrong: ViperMate it's *still* messy, but I have made the first steps towards a software destined to behave like an actual engine, by finally implementing the needed infrastructure:

* Iterative Deepening
* A PV-table
* Gameover checks

Here follows a summary of what each of these pieces does at the moment.


## Iterative Deepening

I'll save my rants about Iterative Deepening for another time and just say that I have implemented its basic form: a cycle that searches a position by incrementing the target depth every time.

To make it a bit faster, I have added an option to set the step of each iteration. Instead of doing a depth-1 search, then a depth-2 one, then a depth-3, and so on... the loop can start from a given depth (say, 4) and proceed with increments of N.

All these parameters are configurable but I've added also an option to let the engine decide them by itself.

For now, the information acquired by an iteration, like the best move found at a certain depth, is only partially used by the alpha-beta search.

The reason for that is that currently the search is not very stable, thatnks to a super-basic evaluation function. The best move found at a given iteration is rarely a good prediction of the best move the search will find when a deeper search will be made.

## PV-table

I'm perfectly aware that there are better (or more popular) ways to store the Principal Variation, but still... I have added a PV-table structure to keep track of it.

Apart from helping a bit the search by providing the best move found at a previous iteration, the PV is also shown after each iteration of Iterative Deepening.

I confess that, psychologically, seeing ViperMate print the typical output of a chess engine has felt like a small stepstone in the direction of a more serious project:

```
info depth 4 score cp -73 nodes 7033 nps 4929903 hashfull 669 pv a2a3 b4c3 a3b4 a5c3
info depth 6 score cp -90 nodes 98713 nps 5339937 hashfull 669 pv c1d2 d5e4 d2c3 d5e4 e4d5 b8c6
info depth 8 score cp -103 nodes 2713234 nps 6140275 hashfull 719 pv g1e2 d5e4 a2a3 d8d6 c3e4 g8f6 f1b5 d4c3
```

This information has helped me make some decisions on how to improve pruning; it has also been helpful in spotting unexpected behaviors.


## Gameover checks

At the moment, checking *all* the reasons why a game can end, including fifty move rule and repetitions, is done only between moves in the game and the alpha-beta search only checks for checkmates and stalemates.

This is done for performance purposes, but it's obvious that it's a temporary solution and that sooner or later the search will need to handle all kind of gameover statuses.

### Debugging stale/checkmate checks

In implementing this aspect, I have tested ViperMate with some simple mate-in-3 puzzles. It turned out that there was an issue with a recent decision that I have made to speed-up the search.

At the top of my `negamax()` I checked if the node was a leaf node and immediately return the evaluation of the position if so.

In this way, unfortunately, if a stale/checkmate happens at a leaf note, it will never be detected by the search!

So, I have fixed the issue by moving this check after the generation of the moves, which is required to check for a stale/checkmate. I have also created an option to switch on the no-checks hack, considering how dramatically it increases the speed of the overall search without *necessarily* affecting its outcome.


## Conclusions and next steps

A *lot* more has happened under the hoof in the last days:

* a disproportionately high amount of bug hunting
* implementing the PV-table was tricky
* the self-play default behavior is now just a "mode". Others will be added, like a simple way to play against the engine and, of course, the UCI mode.

The next programming steps will be probably related to providing the engine a way to manage time and to the implementation of the UCI protocol/mode.

---

*ViperMate* chess engine by *Enrico Altavilla*