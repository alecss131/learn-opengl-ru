# Мяч

На данный момент у нас есть уровень,состоящий из кирпичей и подвижной платформы. Единственное, чего не хватает для классического Арканоида, это мяча. Цель игры - разрушить все кирпичи, и не дать мячу достигнуть нижнего края экрана.

В дополнение к основным игровым компонентам, мяч имеет радиус, и дополнительные значения типа bool которые показывают где находиться мяч: на игровой платформе или в свободном движении. В начале игры мяч изначально находиться на платформе пока игрок не нажмет какую либо произвольную кнопку для начала игры.

Поскольку мяч фактически это объект класса *GameObject* с несколькими допролнительными свойствами имеет смылс создать класс *BallObject* в качестве подкласса *GameObject*:

```cpp
class BallObject : public GameObject
{
    public:
        // состояние мяча	
        float     Radius;
        bool      Stuck;
  

        BallObject();
        BallObject(glm::vec2 pos, float radius, glm::vec2 velocity, Texture2D sprite);

        glm::vec2 Move(float dt, unsigned int window_width);
        void      Reset(glm::vec2 position, glm::vec2 velocity);
}; 
```

В конструкторе класса *BallObject* инициализируются локальные переменные, а также основной объект класса *GameObject*. Класс *BallObject* содержит функцию *Move* которая передвигает мяч в зависимости от скорости.  Также эта функция проверяет коснулся ли мяч какой либо части экрана и если да, то меняет его скорость:

```cpp
glm::vec2 BallObject::Move(float dt, unsigned int window_width)
{
    // если не прилип к платформе игрока
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

В добавок к изменению скорости мяча, мы также хотим перемещать мяч по краям; мяч может двигаться толко если он в игре.

> Поскольку игра окончена \(или жизнь потреряна\) когда мяч косается нижней границы игрового поля, у нас нет кода который позволяет мячу отталкиваться от нижней части игрового поля. хотя позже мы должны будем реализовать эту логику в некоторых случаях.

Код объекта мяч приведен здесь:

- BallObject: [header](ball_object_collisions.h), [code](ball_object_collisions.cpp)

Для начала давайте добавим мяч в игру. по аналагии как мы это делали с платформой, создаем объект класса BallObject и определяем две константы которые используем для инициализации мяча. В качестве текстуры мяча, мы будем использовать изображение: [ball texture](awesomeface.png).

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

Затем мы должны покадрово обновлять положение мяча вызываю функцию Move из функции Update:

```cpp
void Game::Update(float dt)
{
    Ball->Move(dt, this->Width);
}  
```

также, поскольку мяч изначально находится на поверхности ракетки, мы должны дать возможность игроку запустить мяч в игру - для этого будем использовать клавишу пробел. Это означает что нам нужно немного изменить код функции  *processInput*:

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

Далее, если игрок нажимает пробел, переменная начала игры меняет состояние на false. Учтите что мы также двигаем положение мяча  по отношению к платформе когда мяч останавливается

В итоге, нам нужно отрисовать мяч:

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

В результате мы получаем мяч который передвигается вместе с платформой и отталкивается при нажатии пробела. Мяч также соответствующим образом отталкивается от левой, правой, и верхней границ, но как мы видим его еще нужно научить разрушать кирпичи:

[no_collisions.mp4](no_collisions.mp4)

То что мы хотим сделать это создать одну или несколько функций которые будут проверять разрушил ли мяч какие либо кирпичи в текущем уровне и если да, то уничтожать кирпич. На функции определения разрушений мы остановимся в [последующих](../paragraph%202/text.md) уроках.
