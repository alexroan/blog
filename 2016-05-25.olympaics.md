# OlympAIcs

Rarely a day goes by in an average week where I do not spend at least half an hour sifting through the first few pages of [Hacker News](https://news.ycombinator.com/). A few weeks ago someone posted a link to [The AI Games](http://theaigames.com/), a site where bots compete against each other in a number of two player puzzle games.

I love puzzles. When I was a kid I was obsessed with Rubik's cubes, eventually teaching myself to solve them in just under a minute (I can still solve them but probably not at the same speed as back then). A game I always played throughout childhood was Connect four, also known as Four in a Row. I'd spent many hours on the beach challenging my parents and friends over and over again. So, using one of the sample projects that TheAIGames provides as a starting point, I set out to write a C# Connect four bot.

The game engine that TheAIGames runs on uses standard in and out to issue updates and wait for commands, so once your bot is in a position where it understands the basic commands, it's pretty much good to go.

## How I Did It

The first step was to create a system of calculating the heuristic value of any given state. For example, if each player had played three moves each, player one had three in a row and player two had placed moves dotted around in no particular order, that state would be more valuable to player one than player two. If the opposite were true, then it would be more valuable to player two.

When calculating the heuristic, the bot would check every single position on the board and try to look in every direction (vertical, horizontal, diagonal leaning right, diagonal leaning left), and count how many in a row were equal to the initial position.

The initial calculations looked a bit like this:

- 10,000 for four in a row
- 100 for three in a row
- 10 for two in a row
- 1 for one 

At this point I had some arbitrary values for board states. Next up was implementing an algorithm to decide which state to move towards. At university I'd spent some time looking into search algorithms list Depth first, Breadth first, Greedy best first and A*. These however, are all based on single player puzzles. What I needed was a two player version of these.

### Minimax

The theory behind the Minimax algorithm is to minimize your maximum losses. Choose the path which presents the best possible outcome, assuming that the opposing player will always choose the best possible move.

Here's a pseudo representation of the algorithm:

    int Minimax(state)
        value = -∞
        bestAction
        foreach child of state
            if MinValue(child) > value
                bestAction = child
                value = MinValue(child)
        return bestAction

    int MinValue(state)
        if state.IsTerminal()
            return state.Utility()
        value = ∞
        foreach child of state 
            if MaxValue(child) < value
                value = MaxValue(child)
        return value

    int MaxValue(state)
        if state.IsTerminal()
            return state.Utility()
        value = -∞
        foreach child of state 
            if MinValue(child) > value
                value = MinValue(child)
        return value
            

I fed it some sample data and it worked! Albeit, quite slowly... This was mainly due to the heuristic calculations I had implemented, whereby every single position was checked for four in a row in every possible direction (horizontal, vertical and all diagonals). So, even with a measly depth of two nodes deep, it took a few seconds to come up with a reasonable answer.

To speed up the heuristics, I had to calculate how many four in a row's are possible on a board of seven by six, and then check if any of those are still available to form four by either player. If so, give that state a heuristic value. Doing this enabled the bot to perform searches at a depth of five instead of two in the allotted time.

### Alpha-Beta Pruning

Very similar to Minimax, Alpha-Beta pruning enables a more efficient search by only searching down the tree if that branch has a chance of being more attractive than previous branches. So instead of keeping the 'value' variables at each level as seen in the above code, 'alpha' and 'beta' values are passed deeper into the search and are compared to at each stage. Using this method, some branches are completely disregarded if it means that the opposing player will win immediately for example. In turn, this enabled the depth to be set from five to seven! 

### Refining Heuristic Calculations

At this point my bot was winning more games than it was losing but was still making silly mistakes. I had to sure up my heuristics more. Instead of just counting the two-in-a-rows and three-in-a-rows, I changed the algorithm to only calculate a heuristic for these combinations if they had enough empty spaces next to them to form four in a row, otherwise don't bother.

After this, I noticed that it would still not recognize cases where two in a row was present, then a gap, then one. An obvious winning play. I came up with a way of including this and making the heuristic calculations more robust by counting how many of my, the opposition and free spaces were present in each of the available winning sequences on the entire board, then using these numbers to decide the value of the state. Far more robust, but a bit slower than before, so the depth was reduced to five once more.

I started testing whether certain 'if' statements where better than 'switches', and other nitty gritty stuff like that. One thing that seemed to help in longer matches was that after twenty one rounds, The bot went from searching with a depth of five, to seven. This is only possible because certain branches contain terminal states anyway, so the would be plenty of time left to search branches which would not have been explored otherwise.

## Code and Competition

- Code for this bot can be found [here](https://github.com/alexroan/fourinarow) 
- To see how my bot is doing in the rankings go [here](http://theaigames.com/competitions/four-in-a-row/leaderboard/global/a/) and search for YouShould4Fit.