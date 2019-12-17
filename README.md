# NEAT-Columns
Using Neuroevolution to play Columns

The code contained herein allows for the development of an autonomous agent tasked with playing the 1990 16-bit video game classic Columns.  The game is emulated in [openAI](https://openai.com/)'s [Gym Retro](https://retro.readthedocs.io/en/latest/index.html#) library and the neuroevolution is handled by [NEAT-python](https://neat-python.readthedocs.io/en/latest/#)  A bit of [OpenCV](https://opencv.org/) is used to process the game's video output and most of the math is handled by NumPy and SciPy's image library to maximize speed via vectorizaton.
