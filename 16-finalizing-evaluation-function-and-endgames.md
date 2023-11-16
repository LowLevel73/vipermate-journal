# Finalizing the evaluation function and endgames 

*Date: November 16, 2023*

It's hard to imagine that almost all the analysis done to evaluate ViperMate's behavior was done on the opening phase of games.

The reason is that development has been focused on creating a solid infrastructure, and it hasn't been necessary to work on the evaluation function yet. Besides, the "classic" evaluation is only a *placeholder*, because in the foreseeable future I intend to implement an evaluation function based on a neural network. No reason to spend too much time on a placeholder.

This started to change a few weeks ago when I implemented the checkmate logic, and was completed today by adding the last big missing piece: **managing the king correctly in the endgames**.

ViperMate has always relied on the super-basic [Simplified Evaluation Function](https://www.chessprogramming.org/Simplified_Evaluation_Function), which is based on hand-made piece-square tables.

So far I haven't bothered to implement the endgame king table, and this means that the engine *couldn't even see a mate-in-1* when it needed the king's help! The poor thing was determined to stay in its safe position, with no intention of exploring the outside world.

Well, now it can, and of course the new table leads to better endgames in general. Here follows the result of a simple K&R mate-in-8 that couldn't be found before because of the timid king.


```
info depth 5 score cp 50 nodes 8250 nps 11239782 hashfull 2
info depth 7 score cp 50 nodes 44341 nps 11582123 hashfull 2
info depth 9 score cp 55 nodes 134150 nps 9289652 hashfull 8 pv h7g6 h1g1 g6f5 g1f1 f5e4 f1e1 a2b2
info depth 11 score cp 65 nodes 224402 nps 8908553 hashfull 12 pv h7g6 h1g1 g6f5 g1f1 f5e4 
f1e1 e4d3 e1f1
info depth 13 score cp 75 nodes 333490 nps 8261308 hashfull 19 pv h7g6 h1g1 g6f5 g1f1 f5e4 
f1e1 e4d3 e1f1 d3e3 f1g1
info depth 15 score mate 8 nodes 544631 nps 7761826 hashfull 31 pv h7g6 h1g1 g6f5 g1f1 f5e4 f1e1 e4d3 e1f1 d3e3 f1g1 e3f3 g1h1

     a b c d e f g h
     ---------------
8  | . . . . . . . . |  Best move found: Kg6
7  | . . . . . . . . |
6  | . . . . . . K . |
5  | . . . . . . . . |
4  | . . . . . . . . |
3  | . . . . . . . . |
2  | R . . . . . . . |
1  | . . . . . . . k |
```


## A failed attempt

Before finalizing the implementation of the Simplified Evaluation Function, I tried to see if it was feasible to implement [PeSTO's Evaluation Function](https://www.chessprogramming.org/PeSTO%27s_Evaluation_Function) in a quick way instead.

The attempt was a big **failure**. ViperMate was tweaked too much around the simplified tables and forcing the presence of a too extraneous piece of software just made everything worse: quality of play, move order, pruning.

Adding tapered evaluation in the future and auto-tuning the values in the tables is definitely something I'm eager to do, but trying to plug in a piece of alien technology and hoping for the best is not the way to go.

Besides, even if it were an acceptable solution, I couldn't complicate development right now, considering how close I am to the first stable version of the engine.


## Next steps

I have resisted the temptation to add new pruning techniques to the engine.

**Razoring** and **Late Move Reduction** are fairly easy to implement, and I have already written some code, but I have parked it in the "garage" because I want to stick to the plan of getting a releasable beta as soon as possible.

The next steps should be:

* Revisit the overwrite logic of the transposition tables, because it's buggy.
* Sanity checks and *real* testing
* (maybe) automatic tuning of some key parameters


---

*ViperMate* chess engine by *Enrico Altavilla*