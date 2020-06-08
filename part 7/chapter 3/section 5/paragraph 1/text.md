# Ball

At this point we have a level full of bricks and a movable player paddle. The only thing missing from the classic Breakout recipe is the ball. The objective is to let the ball collide with all the bricks until each of the destroyable bricks are destroyed, but this all within the condition that the ball is not allowed to reach the bottom edge of the screen.

In addition to the general game object components, a ball has a radius, and an extra boolean value indicating whether the ball is stuck on the player paddle or it's allowed free movement. When the game starts, the ball is initially stuck on the player paddle until the player starts the game by pressing some arbitrary key.

Because the ball is effectively a *GameObject* with a few extra properties it makes sense to create a *BallObject* class as a subclass of *GameObject*:

```cpp
class BallObject : public GameObject
{
    public:
        // ball state	
        float     Radius;
        bool      Stuck;
  

        BallObject();
        BallObject(glm::vec2 pos, float radius, glm::vec2 velocity, Texture2D sprite);

        glm::vec2 Move(float dt, unsigned int window_width);
        void      Reset(glm::vec2 position, glm::vec2 velocity);
}; 
```

The constructor of *BallObject* initializes its own values, but also initializes the underlying *GameObject*. The *BallObject* class hosts a *Move* function that moves the ball based on its velocity. It also checks if it reaches any of the scene's edges and if so, reverses the ball's velocity:

```cpp
glm::vec2 BallObject::Move(float dt, unsigned int window_width)
{
    // if not stuck to player board
    if (!this->Stuck)
    { 
        // move the ball
        this->Position += this->Velocity * dt;
        // check if outside window bounds; if so, reverse velocity and restore at correct position
        if (this->Position.x <= 0.0f)
        {
            this->Velocity.x = -this->Velocity.x;
            this->Position.x = 0.0f;
        }
        else if (this->Position.x + this->Size.x >= window_width)
        {
            this->Velocity.x = -this->Velocity.x;
            this->Position.x = window_width - this->Size.x;
        }
        if (this->Position.y <= 0.0f)
        {
            this->Velocity.y = -this->Velocity.y;
            this->Position.y = 0.0f;
        }
      
    }
    return this->Position;
}  
```

In addition to reversing the ball's velocity, we also want relocate the ball back along the edge; the ball is only able to move if it isn't stuck.

> Because the player is game over \(or loses a life\) if the ball reaches the bottom edge, there is no code to let the ball bounce of the bottom edge. We do need to later implement this logic somewhere in the game code though.

You can find the code for the ball object below:

- BallObject: [header](ball_object_collisions.h), [code](ball_object_collisions.cpp)

First, let's add the ball to the game. Just like the player paddle, we create a BallObject and define two constants that we use to initialize the ball. As for the texture of the ball, we're going to use an image that makes perfect sense in a LearnOpenGL Breakout game: [ball texture](awesomeface.png).

```cpp
// Initial velocity of the Ball
const glm::vec2 INITIAL_BALL_VELOCITY(100.0f, -350.0f);
// Radius of the ball object
const float BALL_RADIUS = 12.5f;
  
BallObject     *Ball; 
  
void Game::Init()
{
    [...]
    glm::vec2 ballPos = playerPos + glm::vec2(PLAYER_SIZE.x / 2.0f - BALL_RADIUS, 
                                              -BALL_RADIUS * 2.0f);
    Ball = new BallObject(ballPos, BALL_RADIUS, INITIAL_BALL_VELOCITY,
        ResourceManager::GetTexture("face"));
}
```

Then we have to update the position of the ball each frame by calling its Move function within the game code's Update function:

```cpp
void Game::Update(float dt)
{
    Ball->Move(dt, this->Width);
}  
```

Furthermore, because the ball is initially stuck to the paddle, we have to give the player the ability to remove it from its stuck position. We select the space key for freeing the ball from the paddle. This means we have to change the *processInput* function a little:

```cpp
void Game::ProcessInput(float dt)
{
    if (this->State == GAME_ACTIVE)
    {
        float velocity = PLAYER_VELOCITY * dt;
        // move playerboard
        if (this->Keys[GLFW_KEY_A])
        {
            if (Player->Position.x >= 0.0f)
            {
                Player->Position.x -= velocity;
                if (Ball->Stuck)
                    Ball->Position.x -= velocity;
            }
        }
        if (this->Keys[GLFW_KEY_D])
        {
            if (Player->Position.x <= this->Width - Player->Size.x)
            {
                Player->Position.x += velocity;
                if (Ball->Stuck)
                    Ball->Position.x += velocity;
            }
        }
        if (this->Keys[GLFW_KEY_SPACE])
            Ball->Stuck = false;
    }
}
```

Here, if the user presses the space bar, the ball's Stuck variable is set to false. Note that we also move the position of the ball alongside the paddle's position whenever the ball is stuck.

Last, we need to render the ball which by now should be fairly obvious:

```cpp
void Game::Render()
{
    if (this->State == GAME_ACTIVE)
    {
        [...]
        Ball->Draw(*Renderer);
    }
}  
```

The result is a ball that follows the paddle and roams freely whenever we press the spacebar. The ball also properly bounces of the left, right, and top edge, but it doesn't yet seem to collide with any of the bricks as we can see:

[no_collisions.mp4](no_collisions.mp4)

What we want is to create one or several function\(s\) that check if the ball object is colliding with any of the bricks in the level and if so, destroy the brick. These so called collision detection functions is what we'll focus on in the [next](../paragraph%202/text.md) chapter.
