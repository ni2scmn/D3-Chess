# Chess Unwinnability Analyzer 2<img src="https://miguel-ambrona.github.io/img/cha.png" width="70px" align="right">

Chess Unwinnability Analyzer is a free and open-source implementation of a decision procedure
for *checking whether there exists a sequence of legal moves that allows a certain player to checkmate their opponent*
in a given chess position.

Due to an early name of this project, we will refer to this tool as CHA, which stands for
*Chess Helpmate Analyzer*.

(Although the unwinnability decision algorithm implemented in this tool is completely original,
we use [Stockfish](https://github.com/official-stockfish/Stockfish) as a back end
for move generation and chess-related functions.)

## What is this useful for?

This tool can be used to rigorously (and automatically) apply Article 6.9 of the
[FIDE Laws of Chess](https://www.fide.com/FIDE/handbook/LawsOfChess.pdf):

> ...if a player does not complete the prescribed number of moves in the allotted time,
> the game is lost by the player. However, the game is drawn if the position is such that
> the opponent cannot checkmate the player's king by any possible series of legal moves.

Which in shorter English would read as:
*"If you run out of time but your opponent can't mate you, it's a draw!"*

This is a (relatively unknown) generalization of the folklore rule that says that when you
just have the king, you cannot win anymore, not even on the clock.

See [this page](https://elrubiongamma.ddns.net/chess-helpmate-analyzer/about.html)
for more details about the problem of correctly applying Article 6.9 and to know more about this tool.


## This tool

CHA 2 can decide unwinnability in *all chess positions* and is *efficient* enough
to be used by chess servers with a minimal computational overhead.

 * CHA 2 is designed to be *sound*, this means that (ignoring possible implementation bugs)
 *CHA 2 will never declare a position as "unwinnable" if the position is indeed winnable*.

 * CHA 2 is also *complete* (if run without a depth limit), this means that it will always find
 a helpmate sequence if it exists.
 CHA performs extermelly well in real games, but artificially designed positions can
 potentially make the analysis reach the default search limit.
 I challenge you to find a position like this!

## Tests

Running CHA 2.0 on the final position of a test set of 5 million games from the
[Lichess game database](https://database.lichess.org/)
(that were won on time by one of the players) led to identifying
[321 of them](https://github.com/miguel-ambrona/D3-Chess/blob/main/tests/unfair.txt)
which were unfairly classified (they should have ended in a draw).
The tool required an average of 296 μs per position (and never more than 4 ms)
running on a personal laptop, with processor Intel Core i7 of 10th generation.

<img src="https://raw.githubusercontent.com/miguel-ambrona/D3-Chess/d739f88c7e134a62f3df29248651223f2496cd13/tests/results.svg">

Observe that *over 99.92% of the positions were analyzed in less than 1.5 ms*.
These numbers make it completely feasible for an online chess server to run this test on every
game that ends in a timeout. Or even after every single move during the game, to end the
game immediately if a
[dead draw](https://en.wikipedia.org/wiki/Rules_of_chess#Dead_position)
is detected (rigorously applying Article 6.9 of the FIDE Laws of Chess).


## Installation & Usage

After cloning the repository, and from the src/ directory,
get Stockfish by running ```make get-stockfish```.
Then, run ```make``` to compile the tool.

You can test the tool by running ```./cha test```.

Otherwise, simply run ```./cha``` to start a process which waits for commands from stdin.
A command must be a valid [FEN](https://en.wikipedia.org/wiki/Forsyth%E2%80%93Edwards_Notation)
position, followed by the intended winner (**white** or **black**).
If no intended winner is specified, the *last player to make a move is the default intended winner*
(i.e., the player who may still have time on the clock if the position comes from a timeout).

On every query, CHA will produce one line output including:

1. [**winnable** _string_ **|** **unwinnable** **|** **undetermined**], the result of the evaluation.<br>
  When "winnable", a helpmate sequence in UCI format is provided.

1. **nodes** _int_, the total number of positions evaluated.

1. **time** _int_, the total execution time (measured in microseconds).

For example:

> ./cha<br>
> Chess Unwinnability Analyzer (CHA) version 2.2<br>
> Bb2kb2/bKp1p1p1/1pP1P1P1/pP6/6P1/P7/8/8 b - -<br>
> winnable e8d8 b7a6 d8e8 a8b7 e8d8 b7c8 d8e8 c8d7 e8d8 d7e8 d8c8 g4g5 c8d8 e8f7 d8c8 f7g8 c8d8 a6b7 d8e8 b7c8 a5a4 g8f7# nodes 5055 time 5717 (Bb2kb2/bKp1p1p1/1pP1P1P1/pP6/6P1/P7/8/8 b - -)<br>
> 7b/1k5B/7b/8/1p1p1p1p/1PpP1P1P/2P3K1/N7 b - - black<br>
> unwinnable nodes 10101 time 1302 (7b/1k5B/7b/8/1p1p1p1p/1PpP1P1P/2P3K1/N7 b - - black)

There are a few of options you may choose when calling ./cha:

* ```-u``` will only show an output on (u)nwinnable positions, ignoring all winnable.

* ```-quick``` will perform a quick analysis trying to prove that the position is unwinnable,
only producing an output if so is the case.

* ```-min``` will search for the minimum helpmate sequence (at the cost of disabling many
optimizations). This can be used for solving helpmate problems.

* ```-limit```, followed by an integer, can be used to change the #nodes limit in the search.

Other examples:

> ./cha -min -limit 1000000<br>
> Chess Unwinnability Analyzer (CHA) version 2.2<br>
> 8/4K2k/4P2p/8/3b1q2/8/8/8 b - - white<br>
> winnable f4b8 e7f7 d4h8 e6e7 b8f8 e7f8n# nodes 705679 time 155008 (8/4K2k/4P2p/8/3b1q2/8/8/8 b - - white)

Enjoy!


## License

Since this software uses Stockfish, it is also distributed under the
*GNU General Public License version 3* (GPL v3).