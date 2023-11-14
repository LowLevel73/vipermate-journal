# Starting a journal for ViperMate

*Date: October 12, 2023*

I have decided to start keeping a journal for my project ViperMate, a simple chess engine that I'm developing to learn more about chess programming.

The reason for this is that I realized that I was keeping many interesting notes in the comments of my code and that all of them sooner or later disappear, after I refactor the code or eliminate bugs.

Those notes were full of ideas, including bad ones, that at a point in time made sense and from which I learned something new.

I think that keeping a journal will help me to keep track of those ideas. When I'll publish a first decent version of ViperMate on Github, I also hope that my notes will be useful to other people who are starting their journey in chess programming.

## The road that brought me here

I started working on a chess engine on Sept 10 2023, which at the moment of this writing is about a month ago. I started developing it in Python, using the excellent [python-chess](https://python-chess.readthedocs.io/en/latest/) library, well aware that Python is not the best language for chess programming.

The reason of choosing Python is that I wanted to focus on the learning part of the project and I assumed that even a slow chess engine would have let me learn the basics of chess programming.

The assumption was correct, but after implementing the main blocks of the engine (search, pruning, move ordering, evaluation, iterative deepening) I became eager to also learn about other concepts that would require the code to be fast.

Also, without being able to search at deeper depths in acceptable times, I was not able to test in a correct way the effectiveness of the optimizations I was implementing: *some optimizations are effective only at deeper depths*.

For this reason I decided to rewrite the engine in C++, which is not a language I love or I'm familiar with, but it seemed the most reasonable choice, considering that I know a bit of C and the fact that C++ is a language used by many chess engines.

An important note about **my main goal**: I'm here to learn and have fun, with no strong goals, expectations or deadlines. I'm guided by what makes me curious, regardless of how good the resulting chess engine will be. So I'll invest time in features and details I'm aware are secondary or will disappear from the code in the future.

## Where are we now

For the C++ version, which uses the great [Chess Library](https://github.com/Disservin/chess-library), I have implemented most of the features the Python version had and added some that didn't appear in it.

### Negamax search with alpha-beta pruning

It's a very vanilla implementation of negamax and the search stops at a fixed depth. No extensions or reductions are implemented yet.

A simple Principal Variation Search logic is also present, but at the moment I keep it switched off by default, for reasons I'll explain later.

### Move ordering and pruning optimization

Move ordering and pruning optimization is the part of the programming I've enjoied the most so far.

I would say that the amount of time I spent on it is disproportionate to the actual benefit it brings to the engine, but I've learned a lot about pruning optimization in the process. Many choices were made after analyzing the effectiveness of pruning, measured through some simple statistics.

So far the move ordering criteria is:

* Positional gain of a move based on piece-square tables
* Recent best moves
* Best captures: MVV/LVA values, read from a lookup table
* Recent killer moves

Move ordering is also implemented through "move picking", which is a technique that allows to pick the best move from a list of unsorted moves, without the need to sort all of them. This approach has led to about 100% speedup in the search.

### Evaluation function

The evaluation function is very simple and it's based on material and positional balance.

At the moment, I'm using the piece-square tables used in "Simplified Evaluation Function", explained in [this article](https://www.chessprogramming.org/Simplified_Evaluation_Function).

I have both a dynamic and a static version of the evaluation function, the first one used when a move is done/undone, the second one used to initialize the material and positional balance of the board.


### Parts of the Python version that I have not ported yet

* Iterative deepening, and as a consequence its contribution to move ordering
* Mobility criteria for the evaluation of a position (not sure if I'll implement it)


### Not an actual engine, yet

Currently, **ViperMate is only a sandbox** and it doesn't support anything related to actual playing chess games: no full checks for game over, no time management, no UCI protocol.

I test the effectiveness of move ordering, pruning and other optimizations by playing the engine against itself at a fixed depth, collecting data and analyzing it.



## What's the plan

I have a set-in-stone decision for the way I will develop ViperMate: stronger optimizations must not be implemented until I am satisfied with the effectiveness of the simpler ones.

For example, I've observed a curious phenomenon in move ordering, which leads to a bit of less pruning. If I would implement transposition tables now, the pruning would be more effective, but it would also hide the issue that I want to study and possibly fix.

This is also the reason why I have already implemented a very simple Principal Variation Search logic, but I keep it switched off.

In consideration of this decision, I have organized the development of ViperMate in 3 phases:

* **Phase 1** (the current one) is to implement the kind of optimizations that only improve speed without affecting the outcome of the search. As said, the idea is to avoid the strongest optimizations to hide the weakenesses of the more simpler optimizations, so it would also be better to implement transposition tables after getting a good move ordering.

* **Phase 2** is to implement the infrastructure that will make ViperMate an actual chess engine, like UCI protocol, time management, game state management.

* **Phase 3** is to implement the optimizations that can affect the outcome of the search, like strong pruning criteria that could change the outcome of the search.


## Conclusions

I think that's all for now. I'll try to keep this journal updated with the progress of ViperMate and the lessons I learn along the way.

---

*ViperMate* chess engine by *Enrico Altavilla*