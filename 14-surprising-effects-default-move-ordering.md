# The surprising effects of the "default" move ordering
*Date: November 13, 2023*

In ViperMate, move ordering is done by assigning a score to each legal move in a position. The score depends on some well-known criteria, like best captures or killer moves.

After this process, some moves can get **the same score**. This can happen because:

* they didn't match with any of the ordering criteria and kept their default value (e.g. 0);
* their default score did change but it matched with the score of other moves by pure chance.

This phenomenon implies that some moves will be presented to the alpha-beta search algorithm as being "equal" but, of course, the move loop in the alpha-beta search will still explore the moves sequentially. So some "equal" moves will be explored before other "equal" ones.

Surprisingly, this unintended, default order can affect the pruning results **in a significant way**.

## A few observations and tests

The first time I noticed this phenomenon was when I was working on the first version of the chess engine, which at that time was written in Python and used the library [Python Chess](https://python-chess.readthedocs.io/en/latest/).

I'm not sure in which order Python Chess generated the list of legal moves, but I observed that changing this default order and **prioritizing certain pieces** over others would increase the number of beta cutoffs, at least in the openings.

Note that this sorting process was done *before applying also the usual move ordering*, so it only affected the order in which the alpha-beta search would evaluate the "equal" moves, **acting as a tie-breaker**.

I also tried randomizing the initial order before doing the move ordering and, as expected, the results were worse: worse pruning and a much slower engine.

The second time I noticed this behavior was just a few days ago. The excellent C++ chess library "[chess-library](https://disservin.github.io/chess-library/)" (yes, that's its name) is what I am using now . It generates and provides the legal moves in the order: KING, PAWNS, KNIGHTS, BISHOPS, ROOKS, QUEEN.

A previous version of my pickMove function didn't apply a *stable* sort to the list of moves, meaning that the moves with the same score were not guaranteed to stay in the same position relative to each other.

After rewriting the code so that the sorting was stable, the chess engine inherited the default piece order set by the chess-library, resulting in about 24% less of evaluated nodes and a noticeable increase in speed.

For reference, here is the new stable `pickMove()`:

```cpp
void Engine::pickMove(Movelist &moves, const int index) {
    int max_index = index;
    int size = moves.size();
    int16_t max_score = moves[max_index].score();

    for (int i = index + 1; i < size; ++i) {
        int16_t score = moves[i].score();
        if (score > max_score) {
            max_index = i;
            max_score = score;
        }
    }

    // If the move with the highest score is not at the "index" position, put it there.
    if (max_index != index) {
        Move temp = moves[max_index];
        std::memmove(&moves[index + 1], &moves[index], (max_index - index) * sizeof(Move));
        moves[index] = temp;
    }
}
```

## Ideas for the future

A few months ago I tried to take advantage of the importance of the initial move order. I added a new move order criterion that assigned a small score to the moves made by the type of pieces that were recently responsible for beta cutoffs. I called it "killer pieces" but it didn't seem to work as hoped.

Still, I think I'll try a similar idea again sooner or later.

## Lessons learned

Regardless of how the list of legal moves is generated, it's important to pay attention to its *initial* order if an engine has a higher chance of assigning the same value to multiple moves in a list.

A useful "default order" would probably depend on the position on the board or the current phase of a game.

At least for ViperMate this seems to be a topic that could be explored and studied more.

---

*ViperMate* chess engine by *Enrico Altavilla*