---
title: Tic-Tac-Toe (and a baby bot)
date: 2022-03-10 13:21:00 -0800
categories: [projects]
tags: [artificial-intelligence]     # TAG names should always be lowercase
---
I've actually started this project back in 2017, when I was still a novice programmer. When I was still a college student, I was told to
work on personal projects in order to stand out and get precious internship experience. I had finished the command line version of this game back then
and was in the middle of developing a GUI version of the game using Qt, but I got bored with the project and quit halfway. Thus, my portfolio was simply
a simple, unimpressive application and a half-baked Qt game that really should've been implemented using an actual game development framework, like Unity or even PyGame.

I've recently looked back at this project this year, and thought, "Perhaps I could add artificial intelligence to the game." So I went ahead and implemented the minimax algorithm for this game. Here is the pseudocode for the algorithm [1](https://www.neverstopbuilding.com/blog/minimax):

```python
def minimax(game, current_state):
    if game.over:
      return score(game)
    scores = [] # an array of scores, which are numbers
    moves = []  # an array of moves

    # Populate the scores array, recursing as needed
    moves = game.get_available_moves(current_state)
    for move in moves:
        possible_game = game.get_new_state(move)
        game.switch_turn()
        scores.push(minimax(game, possible_game))
        moves.push(move)

    # Do the min or the max calculation
    if game.active_turn == player
        # player aims to maximize
        max_score = max(score)
        max_score_index = scores.index(max_score)
        choice = moves[max_score_index]     # <- important!
        return scores[max_score_index]
    else
        # opponent aims to minimize
        min_score = min(score)
        min_score_index = scores.index(min_score)
        choice = moves[min_score_index]
        return scores[min_score_index]

def score(game):
  if game.win():
    return 10   # just an arbitrary value
  elif game.lose():
    return -10  # same value, opposite sign
  else:
    return 0
```

The minimax algorithm is a relatively simple one, compared to other algorithms such as A*, Dijkstra's, or even machine learning and deep learning. (Part of the reason for my interest in implementing minimax was because I've been interested in AI and machine learning for years. I might pursue graduate school specializing in AI in the future, but for now I'll pursue it as a hobby.)

At a high level, minimax takes a game and its current state, and does two things.

1. If the game is over (either a line of X or O, or a draw), it returns the score. It returns 10 if the AI wins, and -10 if the opponent wins (in other words, the player). It returns 0 if the AI and opponent tie.
2. Otherwise, the algorithm expands the current state by considering all the possible moves. For example, if the current state looked like below and it is X's turn to move:

![img-description](/assets/tictactoe_1.png)
_X's turn to move_

Then the list of possible moves is `[(current state + (row1, column 3)), (current state + (row 2, column 1)), (current state + (row 2, column 2)), ...]`

For each of those new states, the minimax function is called on that new state. The turn is switch, hence it would be O's turn. Since the minimax function ultimately returns a score (an integer), it pushes that value to the list of scores for that state, aptly called `scores`. After minimax is evaluated on that state, the move itself that results in that state is also pushed to a different list called `moves`.

When `scores` and `moves` are completed, the function checks whether the active turn is the player (ie. the AI) or the opponent (ie. the human). Naturally, the player will want to perform a move that will maximize its score, since a high positive score is correlated with a win for the AI. Conversely, the opponent will want to perfrom a move that will minimize the score, since a high negative score is correlated with a win for the opponent. Thus, the function returns the highest score (or lowest) possible, given the current state of the game.

It is also important to note that while minimax returns a number, it also returns `choice`, which is the move that will result in the max score possible given the current state. Since the caller of the initial minimax function (before recursions) will always be the AI, it will return the highest score possible. But **more importantly**, it will return the move that corresponds to the highest score. Since the function is `(game, state) => number`, the optimal `choice` will have to be returned via some other method. In C++, one can use pointers to achieve this effect.

Thank you for reading my first article! I hope to publish more content and projects in the future. The GitHub repository can be found here: <https://github.com/luminouslily35/tictactoe/>
