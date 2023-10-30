# Silly experiment with parameter optimization for move ordering

*Date: October 30, 2023*

Parameter optimization needs to be done in a serious way. You need to define which parameters to test, how to evaluate the results, and of course you need different data sets to train the model and validate the results.

This requires some work that I have no intention of doing right now, but I wanted a quick distraction from programming the UCI interface, so I said to myself: "Fuck that. I'll just create a dumb loop that changes the values of the MVV-LVA array to minimize the amount of nodes evaluated by the engine for the first 8 plies of a new game".

To be clear, this is pure idiocy. There is **zero chance** that such an attempt will lead to generalization. The process will only find the values that best fit that particular situation. Moreover, without using a serious methodology, there is no way to know if the process has fallen into the comfortable depths of the first [local minimum](https://en.wikipedia.org/wiki/Maximum_and_minimum) it found.

Still, it's possible to learn a lot from intentionally silly experiments.

## Tweaking the values of the MVV-LVA array

MVV-LVA stands for "Most Valuable Victim - Least Valuable Attacker", meaning that not all captures are equal, and their value should depend on which type of piece captures which other type of piece. For example, **a pawn capturing a queen** is usually considered more meaningful (for move ordering and search exploration) than a queen capturing a pawn.

ViperMate uses a simple 2D table to determine what score to assign to each possible combination of attacker-victim pairs. This is what the table looked like before "optimization"; the values were chosen arbitrarily, by hand:

```cpp
// Rows are the victims, columns are the attackers.
// The order is PAWN, KNIGHT, BISHOP, ROOK, QUEEN, KING and NONE.
const uint16_t mmv_lva[7][7] = {
    { 900,  700,  700,  500, 200,  80, 0},
    {1100,  900,  900,  700, 400, 280, 0},
    {1100,  900,  900,  700, 400, 280, 0},
    {1300, 1100, 1100,  900, 600, 480, 0},
    {1600, 1400, 1400, 1200, 900, 780, 0},
    {0, 0, 0, 0, 0, 0, 0},
    {0, 0, 0, 0, 0, 0, 0}
};
```

The optimization loop simply changes each of the values by a certain amount, starts a new game with 8 plies, and checks if the chosen KPI has improved: the minimization of the evaluated nodes. It continues until no improvement is found. The amount of change decreases over time: ±300, then ±100, then ±50.

"Evaluated Nodes" is not a good KPI to understand if the pruning has actually improved, but remember: this whole experiment is a visit to the realm of irrational programming.

This is the table after the optimization:

```cpp
uint16_t mmv_lva[7][7] = {
    { 900,  100,  100,   50,  50,  80, 0},
    {1100,  900,  900,  900, 400, 180, 0},
    {1100,  900, 1350,  850, 200, 280, 0},
    {1000,  800,  850,  900, 950, 480, 0},
    {1400, 1400, 1400, 1600, 900, 880, 0},
    {0, 0, 0, 0, 0, 0, 0},
    {0, 0, 0, 0, 0, 0, 0}
};
```

## Results and Observations

* The new values lead to fewer evaluated nodes only by a small amount: about -2.26%. There is a slight speed improvement.
* I have no idea why prioritizing "**bishop takes bishop**" so much (value 1350) resulted in fewer evaluated nodes. All the other values in the diagonal of the table didn't change, which means that other same-piece captures didn't seem to have the same importance.
* Rook takes Queen" is the type of capture with the highest value. There is a good chance that this is an error in the flawed process I followed.
* The optimization was done at depth 8. The new values cause a **significant slowdown** for searches done with depth 10. This is to be expected: the values found don't work well for situations other than the one tested.
* An interesting question that has arisen is whether the values should be chosen to generalize well regardless of position, or whether they should **change dynamically** during the search, depending on the depth/ply/position on the board.

## Final thoughts and next steps

This silly activity was far from useless: I'll keep all the observations and thoughts from the experiment in the back of my mind, especially the one about *dynamically updating* the values. I'm sure they'll come in handy when I get serious about optimizing the parameters.

---

*ViperMate* chess engine by *Enrico Altavilla*