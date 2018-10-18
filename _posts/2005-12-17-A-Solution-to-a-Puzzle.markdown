---
layout: post
title:  "A Solution to a Puzzle"
date:   2005-12-17 14:29:00
categories: [Programming, Prolog, Puzzles]
---

<p>A few months back <a href="http://www.winethirty.com/default.aspx">Randy
Kimmerly</a> posted a <a href="http://www.winethirty.com/blog.aspx?id=146">blog</a>
about one the puzzles on my desk. </p>
<img src="http://www.winethirty.com/images/cube_puzzle.jpg" width="570px">
<p>Twisted in the correct way it will form a 3x3x3 cube. It is a snake cube
puzzle that goes by the commercial name of Cubra<font face="Times New Roman">©</font>.
Randy got excited about the puzzle not for the puzzle itself but by the
challenge of writing a program that will produce the solution. He wrote a very
clever C# program to produce a solution. I thought the solution would be more
concise in Prolog so I volunteered to produce a Prolog version. I had to dust off my <a href="http://en.wikipedia.org/wiki/Prolog">
Prolog</a> skills, I hadn't used Prolog since I used to support Turbo Prolog for
<a href="http://www.borland.com">Borland</a>, so my skills are a bit rusty. Here is my solution,</p>

```
linkTurns([f, f, t, f, t, t, t, t, f, t, t, t, t, t, t, t,
   t, f, t, t, t, f, t, t, t, f]).

inrange(0). inrange(1). inrange(2).
cell(X, Y, Z) :- inrange(X), inrange(Y), inrange(Z).

move(right, cell(X, Y, Z), cell(X1, Y, Z)) :- X1 is X + 1, cell(X1, Y, Z).
move(left, cell(X, Y, Z), cell(X1, Y, Z)) :- X1 is X - 1, cell(X1, Y, Z).
move(down, cell(X, Y, Z), cell(X, Y1, Z)) :- Y1 is Y + 1, cell(X, Y1, Z).
move(up, cell(X, Y, Z), cell(X, Y1, Z)) :- Y1 is Y - 1, cell(X, Y1, Z).
move(in, cell(X, Y, Z), cell(X, Y, Z1)) :- Z1 is Z + 1, cell(X, Y, Z1).
move(out, cell(X, Y, Z), cell(X, Y, Z1)) :- Z1 is Z - 1, cell(X, Y, Z1).

legalTurn(left, X) :- member(X, [up, down, in, out]).
legalTurn(right, X) :- member(X, [up, down, in, out]).
legalTurn(up, X) :- member(X, [left, right, in, out]).
legalTurn(down, X) :- member(X, [left, right, in, out]).
legalTurn(in, X) :- member(X, [left, right, up, down]).
legalTurn(out, X) :- member(X, [left, right, up, down]).

free(X, Y) :- member(X, Y), !, fail.
free(_, _).

solution(Cells, Moves, [], Cells, Moves).
solution([LastCell|UsedCells], [LastMove|OldMoves],
   [t|Turns], ResultCells, ResultMoves) :-
   legalTurn(LastMove, NewMove), move(NewMove, LastCell, NewCell),
   free(NewCell, UsedCells),   solution([NewCell, LastCell|UsedCells],
   [NewMove, LastMove|OldMoves], Turns, ResultCells, ResultMoves).
solution([LastCell|UsedCells], [LastMove|OldMoves],
   [f|Turns], ResultCells, ResultMoves) :-
   move(LastMove, LastCell, NewCell), free(NewCell, UsedCells),
   solution([NewCell, LastCell|UsedCells], [LastMove, LastMove|OldMoves],
   Turns, ResultCells, ResultMoves).

solution(Cells, Moves) :- linkTurns(L), solution([cell(0, 0, 0)],
   [right], L, Cells, Moves).


print_solution([], []).
print_solution([Cell|Cells], [Move|Moves]) :-
   print_solution(Cells, Moves), write(Move), put(9), write('to '),
   write(Cell), nl.

goal :- solution(Cells, Moves), print_solution(Cells, Moves).
```

<p>The clause <span class="code">linkTurns/1</span> represents where the string
of blocks turns and where it doesn't. <span class="code">inrange/1</span>
represents the valid ranges the coordinates of a block can have, 0, 1 or 2. This
is used to build <span class="code">cell/3</span> which represents the locations
the blocks can occupy in a 3x3 result. <span class="code">move/3</span>
calculates how to get from one block to another.&nbsp; <span class="code">
legalTurn/2</span> generates legal turns. <span class="code">free/2</span> helps
decide if a cell is already occupied.&nbsp; <span class="code">solution/5</span>
calculates the solution; and <span class="code">solution/2</span> is a
simplified wrapper around<span class="code"> solution/5</span>. <span class="code">
print_solution/2</span> is a utility clause to help print the solution in a some
what readable form. Finally <span class="code">goal</span> is a clause that
kicks off the whole process (a dead giveaway to Turbo Prolog roots).
</p><p>This will only print one solution (because of the implied cut in using I/O) even though there are two possible. To see
both use,</p><p>
</p>

```
?- solution(Cells, Moves).
```

<p>You will notice that both solutions are reflexive of each other so you could
say there is really only one solution.</p>
<p>I found an interesting site
<a href="http://www.geocities.com/jaapsch/puzzles/snakecube.htm">here</a> that
discusses these puzzles as well as gives a break down of how many of these kinds
of puzzle there can be.</p>
