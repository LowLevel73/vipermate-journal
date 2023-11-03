# The design of internal communications and a nasty G++ -O3 optimizer issue

*Date: November 3, 2023*

A few days have passed and the amount of time invested in the project has not diminished, but... all the activity has been focused on design decisions and fixing a nasty "bug".

## Decisions about how internal communication will work

I decided to separate the `search()` method from the output that the chess engine will send, for example after finding a move or when it has updated information about an ongoing search. In other words, the goal is to have an iterative deepening loop that doesn't write directly to `std::out`.

This way, the search thread will focus on... searching.

This decision will require some more work and, unexpectedly, I have wisely decided that this task must not interfere with my initial goal of getting the UCI implementation **done**.

So for now, I will cook up a less-than-ideal but fast direct write to `std::out` to avoid any delay. Asynchronous implementation can wait.

## A nasty G++ -O3 optimization bug

It turns out that `search()` wasn't able to read the correct value of a private class variable when the `-O3` (or `-Ofast`) optimization option of G++ was enabled.

**Note**: There are no concurrent accesses in the entire code of the application. The variable is defined and initialized with a value; then the only read is performed by the iterative loop, and the read value is different from the one used to initialize the variable.

Everything works perfectly with the `-O2` optimization level, but not with the stronger levels.

So I started to add to `-O2` every single optimization option that defines the `-O3` level. There were no problems with enabling the following flags:

```
CPPFLAGS = -O2 -fgcse-after-reload -fipa-cp-clone -floop-interchange -floop-unroll-and-jam -fpeel-loops -fpredictive-commoning -fsplit-paths -ftree-loop-distribution -ftree-partial-pre -funswitch-loops -fvect-cost-model=dynamic -fversion-loops-for-strides -ftree-vectorize
```

... but all hell broke loose after adding **`-fsplit-loops`**.

Long story short: for some reason, the `while(true) {...}` loop of the iterative deepening cycle is "optimized" by this flag in a way that prevents a variable from being read correctly!

Fortunately, the problem disappears if the variable is declared as `atomic`. And, no, I will not renounce to `-O3`, considering how much it contributes to speed.

Of course, I can't consider this a real "fix". The same optimizer flag might also interfere with other pieces of code, and as a programmer I can do very little to make the exoteric behavior of the optimizer exactly what I want. Nevertheless, I haven't observed any other problems, and for now I can live with the workaround, perhaps extending it to other variables that might face the same fate.

The positive aspect of this problem is that it has prompted me to look for possible **undefined behaviors**. I started with [Dr. Memory](https://drmemory.org/), and when the engine is compiled with the optimization flags turned off, it just finds an out-of-stack read in a silly function that prints the board.

I'll debug it. More substantial sanity checks will be done later and will require code changes.


---

*ViperMate* chess engine by *Enrico Altavilla*