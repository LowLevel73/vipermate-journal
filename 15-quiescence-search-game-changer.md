# Quiescence search is a game-changer (literally)

*Date: November 14, 2023*

I finally developed the quiescence search method.

Previously, ViperMate simply evaluated the position as soon as a leaf node was reached, causing very erratic changes in evaluations and PV moves after each iteration of iterative deepening.

Even though the current version of the `quiescence()` method does not include some important checks and does not take advantage of transposition tables, the difference in **quality of moves** and search speed has transformed ViperMate into a much more serious chess engine.

## Extending search can make it faster

An interesting observation is how an *extension* to the main search can actually *reduce* the total number of nodes visited and evaluated. Since quiescence search produces more accurate evaluations, the main search also benefits from this reliable information and uses it to achieve more meaningful pruning and better moves.

This observation also applies to another kind of search extension that is done in `quiescence()` itself: a king in check deserves more attention, and in this case ViperMate increases the quiescence search depth for that node/branch. Again, the tests show that by investing in more precise evaluations, the overall pruning, speed and move quality increases.

## The current implementation

Here is the code for the current version of `quiescence()`. I have added the usual **standing pat check for beta cutoffs** and a simple form of **delta pruning**.

Tests show that delta pruning could be improved and optimized a bit more, at the cost of more code clutter. For now, I have decided to keep the function simple and clean, and drop the extra pruning.

```cpp

int Engine::quiescence(int depth, int ply, int alpha, int beta) {
    Engine::nodes_hits++;

    int color = (board.sideToMove() == Color::WHITE ? 1 : -1);

    // Stand-pat position evaluation.
    int eval = color * evaluate(board);

    if (eval >= beta) {
        return eval;
    }

    bool inCheck = board.inCheck();

    // If the king is in check, increase the depth.
    if (inCheck) {
        depth++;
    }

    if (depth <= 0) {
        return eval;
    }

    if (eval > alpha) {
        alpha = eval;
    }


    // The moves.
    Movelist moves;
    generateMoves<MoveGenType::CAPTURE>(moves, 0, color, nullptr);

    int best_value = eval;
    int moves_size = moves.size();

    for (int move_number = 0; move_number < moves_size; ++move_number) {
        pickMove(moves, move_number);
        Move& move = moves[move_number];

        // Delta pruning.
        int delta = piece_material_value[static_cast<int>(board.at<chess::PieceType>(move.to()))] + QS_DELTA_MARGIN;

        if ((best_value + delta) < alpha && !inCheck) {
            continue;
        }

        doMove(move);
        eval = -quiescence(depth-1, ply+1, -beta, -alpha);
        undoMove(move);

        if (eval >= beta) {
            best_value = eval;
            break;
        }

        if (eval > best_value) {
            best_value = eval;

            if (best_value > alpha) {
                alpha = best_value;
            }
        }
    }

    return best_value;
}
```

## Other improvements

Perhaps the most subtle but important change made to implement quiescence search is the modification of the `generateMoves()` method, which can now be called with a **C++ template argument** that tells the function what kind of moves to generate. For ViperMate quiescence search, only capture moves need to be generated. 

I used `constexpr if(...)` statements inside the method to make sure that the compiler, during the optimization phase, creates only the code needed for the type of moves to be generated. Specialized instantiations of a function lead to faster code at the expense of a little more space.

## Juicy next steps

### Quiescence search affects iterative deepening and PV-moves

Now that quiescence search provides more stable evaluations, this feature could be used to improve other aspects of the engine.

One key piece of information is **how much the PV move has changed in the previous iterations** of iterative deepening. Calmer positions usually see the same best move reported by all iterations, while more tactical positions have a greater chance of seeing the best move change from one iteration to another.

How "sharp" or "tactical" a position is can help with:

* tweaking existing euristics, such as killer moves
* deciding how much to apply search reduction or extension
* deciding how much time to devote to a search

### Using transposition tables

I haven't been able to find a satisfactory use of the transposition table for quiescence search.

The main problem seems to be that the current implementation of the transposition table storage logic is buggy. Other doubts have arisen about whether and how to distinguish the transposition table entries of the quiescence search from the entries created in the main search.

For now, I'm satisfied with the current results, and I'll try to add TT usage to quiescence search again in the future, probably after fixing the problematic storage criteria.


---

*ViperMate* chess engine by *Enrico Altavilla*