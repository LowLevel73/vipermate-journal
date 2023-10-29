# Refactor: search controller and game modes (benchmark, UCI...)

*Date: October 28, 2023*

A lot has going on in the previous days and I'm quite satisfied with the initial results.

## Search controller

Now the `search()` method, which is the one that implements Iterative Deepening and cyclicly calls the `negamax()` function at different depths, can be completely **controlled by external functions**.

This has been accomplished simply by creating an **infinite loop** in `search()` and by adding some controlling flags to the Engine class, like a flag to decide whether the search should run or not.

New methods set and read the new flags. The net result is, for example, that external functions can call something like `startSearch()` and the search method can check `shouldSearch()` to decide if it has to call `negamax()` or not.

Also, the search method is completely unaware of which conditions need to be met to stop a search or to quit completely.

Extrernal agents can set an end condition by calling `setEndCondition()` and the search method needs just to call a method named `shouldStop()` to understand if the conditions have been met, whatever they are.

```cpp
// Ending conditions for the search.
enum EndCondition {DEPTH, NODES, TIME, INFINITE};

// Sets the end condition for the search.
inline void setEndCondition(const EndCondition condition, const uint64_t value = 0) {
    end_condition = condition;
    end_condition_value = value;
}

// Checks if the search should stop.
inline bool shouldStop() {
    switch (end_condition) {
        case DEPTH:
            return reached_depth >= end_condition_value;
        case NODES:
            return nodes_hits >= end_condition_value;
        case TIME:
            return false; // TODO: implement time management.
        case INFINITE:
            return false;
        default:
            return false;
    }
}
```

And all this added complexity is necessary to implement more easily...

## Game modes

Now ViperMate has "game modes" and each mode can handle the search differently.

At the moment the two existing modes are:

* **Benchmark**: this is the usual self-played short game that I've extensively used to measure how fast the search is. It's a mode that collects useful statistics and it's needed to debug and improve.
* **UCI**: currently half-baked, I have started to implement the UCI mode, which launches a detached (independent) threaded `search()` and by creating the skeleton of the command listening loop.

The benchmark mode just launches the search **synchronously**, for each move of the self-played game. The control returns to the benchmark mode after each ply and, after all the moves have been played, the mode prints the statistics on the console.

On the other hand, the UCI mode launches the search **asynchronously**. `search()` is instructed to stay in a forever loop until the UCI mode calls a `doQuit()` method.

I expect  to create other modes in the future and the new search controlling infrastructure will make their addition easier.

The next steps will be to code the command listening loop in the UCI mode and to add an early exit check in `negamax()`.