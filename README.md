# NEAT-Columns: Using neuroevolution to play Columns

## Overview

The project described herein allows for the execution of an autonomous agent tasked with playing the 1990 16-bit video game classic [Columns](https://store.steampowered.com/app/34285/Columns/), available on Steam for purchase. (*Please note that I have no financial interest in Steam; I merely include the link as the only place known to me to legally acquire the ROM file*).  The game is emulated in [openAI](https://openai.com/)'s [Gym Retro](https://retro.readthedocs.io/en/latest/index.html#) library and the neuroevolution is handled by [NEAT-python](https://neat-python.readthedocs.io/en/latest/#).  A bit of [OpenCV](https://opencv.org/) is used to process the game's video output and most of the math is handled by NumPy and SciPy's ndimage.measurements library to maximize speed via vectorized methods. Gym Retro provides a script to [automate the import of ROMs obtained from Steam](https://retro.readthedocs.io/en/latest/getting_started.html#importing-roms).

Each frame of the game is rendered in in the gym-retro environment. Depending on the model used, each frame of the game window may be reduced to a simplified representation of the game board and that game state is then fed into the NEAT-Python neural net (these implementations use a [recurrent neural network (RNN)](https://en.wikipedia.org/wiki/Recurrent_neural_network) architecture); and the ouput of the neural network is then translated, either directly or indirectly, into one or more simulated button presses on the virtual controller to move the game piece in play.  

## The Fitness Function  

As the game proceeds, the fitness of the network is evaluated based on the results it generates.  There are two major contributors to the fitness function: those portions that are derived from a scoring event and those portions that come from "nudges" toward better gameplay.  

### Determining Fitness from Scoring Events

Scoring events are scaled (or not) to encourage the agent/RNN under evaluation to favor certain behaviors.  The game awards the player a single point for pressing the "down" key to hasten the falling of the game piece in play into final position.  Since this behavior can largely be automated with little thought for an artificial player, this fitness bonus is not scaled, *i.e.*, it recieves a x 1 multiplier.  Making matches increases the player's score by 10 x the number of blocks matched x (the level + 1).  This is the action we want to reward most, and so we scale it by a factor of x 500.  As levels increase with longer game play periods, this makes scoring matches *extremely* rewarding. A third scoring event involves a wild-card block, which destroys all blocks of the color on which it lands. This is counted as any other match event would.  However, if the wild-card block falls to the bottom of the game board without landing on top of any other gem, the player scores a flat 10,000 points.  This is very desireable in the early game, but becomes much less so at higher levels.  The scaling of this behavior is also encouraged, but is only given a x 5 multiplier. 

### Using Behavioral "Nudges" to Affect Fitness Between Scoring Events

The second set of fitness adjustment terms consists of two kinds of behavioral "nudges" toward desireable play.  The first nudge evaluates the game board for 8-way contiguity of pieces of matching color, *i.e.*, color "island" detection. having colored blocks of increasing size makes matching events more probable when the game piece in play can increase their size. Each of the six possible colors are evaluated for island size, and the island sizes are then cubed, summed, and scaled with a x 30 multiplier to encourage their formation.  As the islands become especially large, matching events occur and are scored accordingly. 

The second nudge is actually a penalty for undesireable play.  As the gem blocks stack towards the top of the board, the game approaches a game-over state resulting from the inability of a game piece to entirely fit on the game board.  To discourage this, each column of the game board is evaluated for gem height, raised to the fourth power, and the difference between each term and 3<sup>4</sup> is summed and scaled with a x -0.2 multiplier to encourage the agent to keep the gems closer to the bottom of the board.  Since blocks in play are 3 gems high, it is used as a kind of neutral "break-even" point.  Note that the term is not always negative for occupied columns - after a matching event some 1- or 2- high columns may be left with negative terms that become bonuses when the negative multiplier is applied.  This is by design, as these short columns can serve as "stubs" for matching with the next block in play.
 
 ## The Models
 
At this point, two neuroevolutionary models are considered, which I describe as "emergent" and "decisive."  A third model, which I plan to call "predictive", is also under development.

### Emergent Model

The emergent model seeks to make the neural network of each genome do as much of the work as possible in terms of working out efficient play.  The neural net is fed the state of the board each frame of the game and the output is used to give direct inputs to the controller.  This method, while emotionally rewarding when it succeeds, is also very difficult to train, as many frames of the game require no action on the part of the genome.  This method also has a nasty tendency to fall into infinite loops by alternately pressing left and right in each successive frame.  When this occurs, the block in play drifts slowly down into place until it lands on top of another gem. When this occurs, the game engine grants the player a bit of extra time to nudge the block before falling down further or locking into place.  This leads to a kind of infinite "Wile E. Coyote" behavior of vibrating on and off of a gem column "cliff".  The behavior is managed by counting the number of frames since earning a reward and forcing the selection of the down arrow key to either lock the gem down or force it to fall below the "cliff" height to simply move the game forward.

### Decisive Model

The decisive model takes a more focused approach.  It only feeds the neural network when a new game piece is in play, therefore only feeding the neural network when a newly actionable state of the board is available for consideration.  When this model processes the board state, the output is a series of relative favorabilities for a series of possible button presses which move decisively (hence the name) to achieve one of the eighteen possible ending positions (six columms x three block rotation states) of the new block in play.  This method has proven much more proficient in achieving match events much more quickly than the emergent model, with the infinite loop behavior effectively eliminated.

### Predictive Model (planned)

The predictive model will be based on the decisive model, but with the additional imput of the preview block displayed just to the right of the upper right corner of the board.  This will give the model a view into what the next block in play will be and allow the model to use that information to plan the placement of the current block, though the model will not explicitly evaluate the possibilites arising from the placement of the upcoming block block after the play of the current block.

Code is provided in [Jupyter Notebook](https://jupyter.org/) format (.ipynb).
