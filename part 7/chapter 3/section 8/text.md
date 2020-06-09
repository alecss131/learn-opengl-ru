# Powerups

Breakout is close to finished, but it would be cool to add at least one more gameplay mechanic so it's not your average standard Breakout clone; what about powerups?

The idea is that whenever a brick is destroyed, the brick has a small chance of spawning a powerup block. Such a block will slowly fall downwards and if it collides with the player paddle, an interesting effect occurs based on the type of powerup. For example, one powerup makes the paddle larger, and another powerup allows the ball to pass through objects. We also include several negative powerups that affect the player in a negative way.

We can model a powerup as a *GameObject* with a few extra properties. That's why we define a class *PowerUp* that inherits from *GameObject*:

```cpp
const glm::vec2 SIZE(60.0f, 20.0f);
const glm::vec2 VELOCITY(0.0f, 150.0f);

class PowerUp : public GameObject 
{
public:
    // powerup state
    std::string Type;
    float       Duration;	
    bool        Activated;
    // constructor
    PowerUp(std::string type, glm::vec3 color, float duration, 
            glm::vec2 position, Texture2D texture) 
        : GameObject(position, SIZE, texture, color, VELOCITY), 
          Type(type), Duration(duration), Activated() 
    { }
};  
```

A *PowerUp* is just a *GameObject* with extra state, so we can simply define it in a single header file which you can find [here](power_up.h).

Each powerup defines its type as a string, a duration for how long it is active, and whether it is currently activated. Within Breakout we're going to feature a total of 4 positive powerups and 2 negative powerups:

![](0.png)

- **Speed**: increases the velocity of the ball by 20%.
- **Sticky**: when the ball collides with the paddle, the ball remains stuck to the paddle unless the spacebar is pressed again. This allows the player to better position the ball before releasing it.
- **Pass-Through**: collision resolution is disabled for non-solid blocks, allowing the ball to pass through multiple blocks.
- **Pad-Size-Increase**: increases the width of the paddle by 50 pixels.
- **Confuse**: activates the confuse postprocessing effect for a short period of time, confusing the user.
- **Chaos**: activates the chaos postprocessing effect for a short period of time, heavily disorienting the user.

You can find the textures here:

- **Textures**: [Speed](powerup_speed.png), [Sticky](powerup_sticky.png), [Pass-Through](powerup_passthrough.png), [Pad-Size-Increase](powerup_increase.png), [Confuse](powerup_confuse.png), [Chaos](powerup_chaos.png). 

Similar to the level block textures, each of the powerup textures is completely grayscale. This makes sure the color of the powerups remain balanced whenever we multiply them with a color vector.

Because powerups have state, a duration, and certain effects associated with them, we would like to keep track of all the powerups currently active in the game; we store them in a vector:

```cpp
class Game {
    public:
        [...]
        std::vector<PowerUp>  PowerUps;
        [...]
        void SpawnPowerUps(GameObject &block);
        void UpdatePowerUps(float dt);
};
```

We've also defined two functions for managing powerups. *SpawnPowerUps* spawns a powerups at the location of a given block and *UpdatePowerUps* manages all powerups currently active within the game.

### Spawning PowerUps

Each time a block is destroyed we would like to, given a small chance, spawn a powerup. This functionality is found inside the game's *SpawnPowerUps* function:

```cpp
bool ShouldSpawn(unsigned int chance)
{
    unsigned int random = rand() % chance;
    return random == 0;
}
void Game::SpawnPowerUps(GameObject &block)
{
    if (ShouldSpawn(75)) // 1 in 75 chance
        this->PowerUps.push_back(
             PowerUp("speed", glm::vec3(0.5f, 0.5f, 1.0f), 0.0f, block.Position, tex_speed
         ));
    if (ShouldSpawn(75))
        this->PowerUps.push_back(
            PowerUp("sticky", glm::vec3(1.0f, 0.5f, 1.0f), 20.0f, block.Position, tex_sticky 
        );
    if (ShouldSpawn(75))
        this->PowerUps.push_back(
            PowerUp("pass-through", glm::vec3(0.5f, 1.0f, 0.5f), 10.0f, block.Position, tex_pass
        ));
    if (ShouldSpawn(75))
        this->PowerUps.push_back(
            PowerUp("pad-size-increase", glm::vec3(1.0f, 0.6f, 0.4), 0.0f, block.Position, tex_size    
        ));
    if (ShouldSpawn(15)) // negative powerups should spawn more often
        this->PowerUps.push_back(
            PowerUp("confuse", glm::vec3(1.0f, 0.3f, 0.3f), 15.0f, block.Position, tex_confuse
        ));
    if (ShouldSpawn(15))
        this->PowerUps.push_back(
            PowerUp("chaos", glm::vec3(0.9f, 0.25f, 0.25f), 15.0f, block.Position, tex_chaos
        ));
}  
```

The *SpawnPowerUps* function creates a new *PowerUp object* based on a given chance \(1 in 75 for normal powerups and 1 in 15 for negative powerups\) and sets their properties. Each powerup is given a specific color to make them more recognizable for the user and a duration in seconds based on its type; here a duration of 0.0f means its duration is infinite. Additionally, each powerup is given the position of the destroyed block and one of the textures from the beginning of this chapter.

### Activating PowerUps

We then have to update the game's *DoCollisions* function to not only check for brick and paddle collisions, but also collisions between the paddle and each non-destroyed PowerUp. Note that we call *SpawnPowerUps* directly after a block is destroyed.

```cpp
void Game::DoCollisions()
{
    for (GameObject &box : this->Levels[this->Level].Bricks)
    {
        if (!box.Destroyed)
        {
            Collision collision = CheckCollision(*Ball, box);
            if (std::get<0>(collision)) // if collision is true
            {
                // destroy block if not solid
                if (!box.IsSolid)
                {
                    box.Destroyed = true;
                    this->SpawnPowerUps(box);
                }
                [...]
            }
        }
    }        
    [...] 
    for (PowerUp &powerUp : this->PowerUps)
    {
        if (!powerUp.Destroyed)
        {
            if (powerUp.Position.y >= this->Height)
                powerUp.Destroyed = true;
            if (CheckCollision(*Player, powerUp))
            {	// collided with player, now activate powerup
                ActivatePowerUp(powerUp);
                powerUp.Destroyed = true;
                powerUp.Activated = true;
            }
        }
    }  
}
```

For all powerups not yet destroyed, we check if the powerup either reached the bottom edge of the screen or collided with the paddle. In both cases the powerup is destroyed, but when collided with the paddle, it is also activated.

Activating a powerup is accomplished by settings its Activated property to true and enabling the powerup's effect by giving it to the *ActivatePowerUp* function:

```cpp
void ActivatePowerUp(PowerUp &powerUp)
{
    if (powerUp.Type == "speed")
    {
        Ball->Velocity *= 1.2;
    }
    else if (powerUp.Type == "sticky")
    {
        Ball->Sticky = true;
        Player->Color = glm::vec3(1.0f, 0.5f, 1.0f);
    }
    else if (powerUp.Type == "pass-through")
    {
        Ball->PassThrough = true;
        Ball->Color = glm::vec3(1.0f, 0.5f, 0.5f);
    }
    else if (powerUp.Type == "pad-size-increase")
    {
        Player->Size.x += 50;
    }
    else if (powerUp.Type == "confuse")
    {
        if (!Effects->Chaos)
            Effects->Confuse = true; // only activate if chaos wasn't already active
    }
    else if (powerUp.Type == "chaos")
    {
        if (!Effects->Confuse)
            Effects->Chaos = true;
    }
} 
```

The purpose of *ActivatePowerUp* is exactly as it sounds: it activates the effect of a powerup as we've described at the start of this chapter. We check the type of the powerup and change the game state accordingly. For the "sticky" and "pass-through" effect, we also change the color of the paddle and the ball respectively to give the user some feedback as to which effect is currently active.

Because the sticky and pass-through effects somewhat change the game logic we store their effect as a property of the ball object; this way we can change the game logic based on whatever effect on the ball is currently active. The only thing we've changed in the *BallObject* header is the addition of these two properties, but for completeness' sake its updated code is listed below:

- **BallObject**: [header](ball_object.h), [code](ball_object.cpp).

We can then easily implement the sticky effect by slightly updating the *DoCollisions* function at the collision code between the ball and the paddle:

```cpp
if (!Ball->Stuck && std::get<0>(result))
{
    [...]
    Ball->Stuck = Ball->Sticky;
}
```

Here we set the ball's Stuck property equal to the ball's Sticky property. If the sticky effect is activated, the ball will end up stuck to the player paddle whenever it collides; the user then has to press the spacebar again to release the ball.

A similar small change is made for the pass-through effect within the same *DoCollisions* function. When the ball's PassThrough property is set to true we do not perform any collision resolution on the non-solid bricks.

```cpp
Direction dir = std::get<1>(collision);
glm::vec2 diff_vector = std::get<2>(collision);
if (!(Ball->PassThrough && !box.IsSolid)) 
{
    if (dir == LEFT || dir == RIGHT) // horizontal collision
    {
        [...]
    }
    else 
    {
        [...]
    }
}  
```

The other effects are activated by simply modifying the game's state like the ball's velocity, the paddle's size, or an effect of the *PostProcesser* object.

### Updating PowerUps

Now all that is left to do is make sure that powerups are able to move once they've spawned and that they're deactivated as soon as their duration runs out; otherwise powerups will stay active forever.

Within the game's *UpdatePowerUps* function we move the powerups based on their velocity and decrease the active powerups their duration. Whenever a powerup's duration is decreased to 0.0f, its effect is deactivated and the relevant variables are reset to their original state:

```cpp
void Game::UpdatePowerUps(float dt)
{
    for (PowerUp &powerUp : this->PowerUps)
    {
        powerUp.Position += powerUp.Velocity * dt;
        if (powerUp.Activated)
        {
            powerUp.Duration -= dt;

            if (powerUp.Duration <= 0.0f)
            {
                // remove powerup from list (will later be removed)
                powerUp.Activated = false;
                // deactivate effects
                if (powerUp.Type == "sticky")
                {
                    if (!isOtherPowerUpActive(this->PowerUps, "sticky"))
                    {	// only reset if no other PowerUp of type sticky is active
                        Ball->Sticky = false;
                        Player->Color = glm::vec3(1.0f);
                    }
                }
                else if (powerUp.Type == "pass-through")
                {
                    if (!isOtherPowerUpActive(this->PowerUps, "pass-through"))
                    {	// only reset if no other PowerUp of type pass-through is active
                        Ball->PassThrough = false;
                        Ball->Color = glm::vec3(1.0f);
                    }
                }
                else if (powerUp.Type == "confuse")
                {
                    if (!isOtherPowerUpActive(this->PowerUps, "confuse"))
                    {	// only reset if no other PowerUp of type confuse is active
                        Effects->Confuse = false;
                    }
                }
                else if (powerUp.Type == "chaos")
                {
                    if (!isOtherPowerUpActive(this->PowerUps, "chaos"))
                    {	// only reset if no other PowerUp of type chaos is active
                        Effects->Chaos = false;
                    }
                }                
            }
        }
    }
    this->PowerUps.erase(std::remove_if(this->PowerUps.begin(), this->PowerUps.end(),
        [](const PowerUp &powerUp) { return powerUp.Destroyed && !powerUp.Activated; }
    ), this->PowerUps.end());
}  
```

You can see that for each effect we disable it by resetting the relevant items to their original state. We also set the powerup's Activated property to false. At the end of *UpdatePowerUps* we then loop through the PowerUps vector and erase each powerup if they are destroyed **and** deactivated. We use the *remove_if* function from the *algorithm* header to erase these items given a lambda predicate.

> The *remove_if* function moves all elements for which the lambda predicate is true to the end of the container object and returns an iterator to the start of this removed elements range. The container's *erase* function then takes this iterator and the vector's end iterator to remove all the elements between these two iterators.

It may happen that while one of the powerup effects is active, another powerup of the same type collides with the player paddle. In that case we have more than 1 powerup of that type currently active within the game's PowerUps vector. Whenever one of these powerups gets deactivated, we don't want to disable its effects yet since another powerup of the same type may still be active. For this reason we use the *IsOtherPowerUpActive* function to check if there is still another powerup active of the same type. Only if this function returns false we deactivate the powerup. This way, the powerup's duration of a given type is extended to the duration of its last activated powerup:

```cpp
bool IsOtherPowerUpActive(std::vector<PowerUp> &powerUps, std::string type)
{
    for (const PowerUp &powerUp : powerUps)
    {
        if (powerUp.Activated)
            if (powerUp.Type == type)
                return true;
    }
    return false;
}  
```

The function checks for all activated powerups if there is still a powerup active of the same type and if so, returns true.

The last thing left to do is render the powerups:

```cpp
void Game::Render()
{
    if (this->State == GAME_ACTIVE)
    {
        [...]
        for (PowerUp &powerUp : this->PowerUps)
            if (!powerUp.Destroyed)
                powerUp.Draw(*Renderer);
        [...]
    }
}    
```

Combine all this functionality and we have a working powerup system that not only makes the game more fun, but also a lot more challenging. It'll look a bit like this:

[](powerups.mp4)

You can find the updated game code here \(there we also reset all powerup effects whenever the level is reset\):

- **Game**: [header](game.h), [code](game.cpp).
