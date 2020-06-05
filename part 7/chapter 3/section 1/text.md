# Breakout

Over these chapters we learned a fair share about OpenGL's inner workings and how we can use them to create fancy graphics. However, aside from a lot of tech demos, we haven't really created a practical application with OpenGL. This is the introduction of a larger series about creating a relatively simple 2D game using OpenGL. The next chapters will demonstrate how we can use OpenGL in a larger, more complicated, setting. Note that the series does not necessarily introduce new OpenGL concepts but more or less show how we can apply these concepts to a larger whole.

Because we rather keep things simple, we're going to base our 2D game on an already existing 2D arcade game. Introducing Breakout, a classic 2D game released in 1976 on the Atari 2600 console. Breakout requires the player, who controls a small horizontal paddle, to destroy all the bricks by bouncing a small ball against each brick without allowing the ball to reach the bottom edge. Once the player destroys all bricks, he completes the game.

Below we can see how Breakout originally looked on the Atari 2600:

![](0.png)

The game has the following mechanics:

- A small paddle is controlled by the player and can only move horizontally within the bounds of the screen.
- The ball travels across the screen and each collision results in the ball changing its direction based on where it hit; this applies to the screen bounds, the bricks, and the paddle.
- If the ball reaches the bottom edge of the screen, the player is either game over or loses a life.
- As soon as a brick touches the ball, the brick is destroyed.
- The player wins as soon as all bricks are destroyed.
- The direction of the ball can be manipulated by how far the ball bounces from the paddle's center.

Because from time to time the ball may find a small gap reaching the area above the brick wall, it will continue to bounce up and forth between the top edge of the level and the top edge of the brick layer. The ball keeps this up, until it eventually finds a gap again. This is logically where the game obtained its name from, since the ball has to break out.

# OpenGL Breakout

We're going to take this classic arcade game as the basis of a 2D game that we'll completely implement with OpenGL. This version of Breakout will render its graphics on the GPU which gives us the ability to enhance the classical Breakout game with some nice extra features.

Other than the classic mechanics, our version of Breakout will feature:

- Amazing graphics!
- Particles
- Text rendering
- Power-ups
- Postprocessing effects
- Multiple \(customizable\) levels

To get you excited you can see what the game will look like after you've finished these chapters:

![](1.png)

These chapters will combine a large number of concepts from previous chapters and demonstrate how they can work together as a whole. Therefore, it is important to have at least finished the Getting started chapters before working your way through these series.

Also, several chapters will require concepts from other chapters \([Framebuffers](../../../part%204/chapter%205/text.md) for example from the Advanced OpenGL section\) so where necessary, the required chapters are listed.

If you believe you're ready to get your hands dirty then move on to the [next chapter](../../chapter%203/section%202/text.md).
