# Уровни

Игра Арканоид, увы не только о создании одного счастливого зеленого шарика, она также содержит полные уровни с большим количеством разноцветных кирпичей. Мы хотим, чтобы эти уровни были настраиваемыми таким образом, чтобы они могли поддерживать любое количество ячеек и /или столбцов, мы хотим, чтобы уровни имели нерушимые кирпичи \(которые не могут быть уничтожены\), мы хотим, чтобы была возможность настраивать различные цвета для кирпичей, а также сохранять настройки во внешний \(текстовый\) файл.

В этой главе мы кратко пройдемся по коду объекта игрового уровня, который используется для управления большим количеством кирпичей. Сначала мы должны определить объект кирпича.

Мы создаем компонент, называемый игровым объектом, который действует как базовое представление объекта внутри игры. Такой игровой объект содержит такие данные, как его положение, размер и скорость. Он содержит цвет, компонент вращения, состояние не разрушен и\/или уничтожен, а также сохраняет переменную *Texture2d* в качестве спрайта.

Каждый объект в игре представлен как *Gameobject* или наследник этого класса. Код класса *Gameobject* приведен ниже:

- GameObject: [header](game_object.h), [code](game_object.cpp)

Уровень в Арканоиде состоит полностью из кирпичей, так что мы можем представлять уровень как коллекцию кирпичей. Поскольку кирпич требует того же состояния, что и игровой объект, мы собираемся представлять каждый кирпич уровня как *Gameobject*. Объявление класса *Gamelevel* выглядит следующим образом:

```cpp
class GameLevel
{
public:
    // level state
    std::vector<GameObject> Bricks;
    // constructor
    GameLevel() { }
    // loads level from file
    void Load(const char *file, unsigned int levelWidth, unsigned int levelHeight);
    // render level
    void Draw(SpriteRenderer &renderer);
    // check if the level is completed (all non-solid tiles are destroyed)
    bool IsCompleted();
private:
    // initialize level from tile data
    void init(std::vector<std::vector<unsigned int>> tileData, 
              unsigned int levelWidth, unsigned int levelHeight);
};  
```

Так как уровень загружается из внешнего файла \(текстовый\), нам нужно предложить некую структуру уровня. Вот пример того, как может выглядеть уровень игры в текстовом файле:

```
1 1 1 1 1 1 
2 2 0 0 2 2
3 3 4 4 3 3
```

Уровень сохраняется в матричной структуре, где каждое число представляет собой тип кирпича, каждый из которых отделен пробелом. В коде для каждого уровня мы можем обозначить, что представляет каждое число. Мы выбрали следующее представление:

- Число 0: без кирпича, пустое пространство внутри уровня.
- Число 1: цельный кирпич, кирпич, который нельзя уничтожить.
- Число больше 1: разрушаемый кирпич; каждое последующее число отличается только цветом.

Приведенный выше пример уровеня после обработки *Gamelevel* будет выглядеть так:

![](0.png)

Класс *Gamelevel* использует две функции для генерации уровня из файла. Сначала он загружает все числа в двумерном векторе в функцию *Load* которая затем обрабатывает эти числа \(для создания всех игровых объектов\) в функции *init* .

```cpp
void GameLevel::Load(const char *file, unsigned int levelWidth, unsigned int levelHeight)
{
    // clear old data
    this->Bricks.clear();
    // load from file
    unsigned int tileCode;
    GameLevel level;
    std::string line;
    std::ifstream fstream(file);
    std::vector<std::vector<unsigned int>> tileData;
    if (fstream)
    {
        while (std::getline(fstream, line)) // read each line from level file
        {
            std::istringstream sstream(line);
            std::vector<unsigned int> row;
            while (sstream >> tileCode) // read each word separated by spaces
                row.push_back(tileCode);
            tileData.push_back(row);
        }
        if (tileData.size() > 0)
            this->init(tileData, levelWidth, levelHeight);
    }
} 
```

Загруженная tileData затем передается на игровой уровень *init* функции:

```cpp
void GameLevel::init(std::vector<std::vector<unsigned int>> tileData, 
                     unsigned int lvlWidth, unsigned int lvlHeight)
{
    // calculate dimensions
    unsigned int height = tileData.size();
    unsigned int width  = tileData[0].size();
    float unit_width    = lvlWidth / static_cast<float>(width);
    float unit_height   = lvlHeight / height;
    // initialize level tiles based on tileData		
    for (unsigned int y = 0; y < height; ++y)
    {
        for (unsigned int x = 0; x < width; ++x)
        {
            // check block type from level data (2D level array)
            if (tileData[y][x] == 1) // solid
            {
                glm::vec2 pos(unit_width * x, unit_height * y);
                glm::vec2 size(unit_width, unit_height);
                GameObject obj(pos, size, 
                    ResourceManager::GetTexture("block_solid"), 
                    glm::vec3(0.8f, 0.8f, 0.7f)
                );
                obj.IsSolid = true;
                this->Bricks.push_back(obj);
            }
            else if (tileData[y][x] > 1)	
            {
                glm::vec3 color = glm::vec3(1.0f); // original: white
                if (tileData[y][x] == 2)
                    color = glm::vec3(0.2f, 0.6f, 1.0f);
                else if (tileData[y][x] == 3)
                    color = glm::vec3(0.0f, 0.7f, 0.0f);
                else if (tileData[y][x] == 4)
                    color = glm::vec3(0.8f, 0.8f, 0.4f);
                else if (tileData[y][x] == 5)
                    color = glm::vec3(1.0f, 0.5f, 0.0f);

                glm::vec2 pos(unit_width * x, unit_height * y);
                glm::vec2 size(unit_width, unit_height);
                this->Bricks.push_back(
                    GameObject(pos, size, ResourceManager::GetTexture("block"), color)
                );
            }
        }
    }  
}
```

Функция *init* проходит по каждому загруженному числу и добавляет *Gameobject* к вектору уровней Bricks на основе обработанного числа. Размер каждого кирпича рассчитывается автоматически \(unit_width и unit_height\) на основе общего количества кирпичей, таким образом чтобы каждый кирпич идеально вписывался в рамки экрана.

Здесь мы загружаем игровые объекты с двумя новыми текстурами, текстурой [блока](block.png) и текстурой [твердого блока](block_solid.png).

![](1.png)

Небольшой секре т заключается в том, что эти текстуры полностью в отмасштабированы в сером цвете. Эффект достигается за счет того, что мы можем аккуратно манипулировать их цветами внутри игрового кода, умножая цвета серого на определенный вектор цветов; точно так же, как мы сделали в *Spriterenderer*. Таким образом, настройка цвета не выглядит слишком странной или несбалансированной.

Класс *Gamelevel* также содержит несколько других функций, таких как рендеринг всех неразрушенных кирпичей, или проверка, если все разрушаемые кирпичи уничтожены. Вы можете найти исходный код класса *Gamelevel* ниже:

- GameLevel: [header](game_level.h), [code](game_level.cpp)

Класс игрового уровня дает нам большую гибкость, поскольку поддерживается  любое количество ячеек и столбцов, и пользователь может легко создать новые уровни, путем модификации текстового файла уровней.

## В игре

Мы хотели бы иметь возможность поддержки нескольких уровней в игре Арканоид, так что нам придется немного расширить класс игры, добавив вектор, который содержит переменные типа *Gamelevel*. Мы также добавим возможность сохранения текущего активного уровня:


```cpp
class Game
{
    [...]
    std::vector<GameLevel> Levels;
    unsigned int           Level;
    [...]  
};
```

Эта версия игры Арканоид включает в себя 4 уровня:

- [Standard](one.lvl)
- [A few small gaps](two.lvl)
- [Space invader](three.lvl)
- [Bounce galore](four.lvl)

Затем каждая из текстур и уровней инициализируется в рамках игрового класса *Init* функции:

```cpp
void Game::Init()
{
    [...]
    // load textures
    ResourceManager::LoadTexture("textures/background.jpg", false, "background");
    ResourceManager::LoadTexture("textures/awesomeface.png", true, "face");
    ResourceManager::LoadTexture("textures/block.png", false, "block");
    ResourceManager::LoadTexture("textures/block_solid.png", false, "block_solid");
    // load levels
    GameLevel one; one.Load("levels/one.lvl", this->Width, this->Height / 2);
    GameLevel two; two.Load("levels/two.lvl", this->Width, this->Height / 2);
    GameLevel three; three.Load("levels/three.lvl", this->Width, this->Height / 2);
    GameLevel four; four.Load("levels/four.lvl", this->Width, this->Height / 2);
    this->Levels.push_back(one);
    this->Levels.push_back(two);
    this->Levels.push_back(three);
    this->Levels.push_back(four);
    this->Level = 0;
}  
```

Все, что осталось сделать, это создать уровень. Для этого вызываем функцию текущего активного уровня *Draw* которая в свою очередь вызывает функции *Gameobject* *Draw* с использованием данного спрайтового рендера. Рядом с уровнем, мы также добавим сцену с красивым [фоновым изображением](background.jpg) \(courtesy of Tenha\):

```cpp
void Game::Render()
{
    if(this->State == GAME_ACTIVE)
    {
        // draw background
        Renderer->DrawSprite(ResourceManager::GetTexture("background"), 
            glm::vec2(0.0f, 0.0f), glm::vec2(this->Width, this->Height), 0.0f
        );
        // draw level
        this->Levels[this->Level].Draw(*Renderer);
    }
}
```

В результате получается хорошо сконструированный уровень, который действительно делает игру более живой:

![](2.png)

### Платформа игрока

Далее мы создаем платформу в нижней части сцены, которая контролируется игроком. Платформа может двигаться только горизонтально, и всякий раз, когда она касается какого-либо края сцены, ее движение должно останавливаться. В качестве платформы мы будем использовать [следующую](paddle.png) текстуру:

![](3.png)

Объект игровой платформы будет иметь положение, размер и спрайтовую текстуру, так что имеет смысл определить этот объект как *Gameobject*:

```cpp
// Initial size of the player paddle
const glm::vec2 PLAYER_SIZE(100.0f, 20.0f);
// Initial velocity of the player paddle
const float PLAYER_VELOCITY(500.0f);

GameObject *Player;
  
void Game::Init()
{
    [...]    
    ResourceManager::LoadTexture("textures/paddle.png", true, "paddle");
    [...]
    glm::vec2 playerPos = glm::vec2(
        this->Width / 2.0f - PLAYER_SIZE.x / 2.0f, 
        this->Height - PLAYER_SIZE.y
    );
    Player = new GameObject(playerPos, PLAYER_SIZE, ResourceManager::GetTexture("paddle"));
}
```

Здесь мы определили несколько константных значений, которые определяют размер и скорость платформы. В рамках функции *Init* мы вычисляем стартовое положение ракетки внутри сцены. Мы убедимся, что центр ракетки соответствует горизонтальному центру сцены.

Вместе с инициализацией игровой платформы, нам также нужно добавить ссылку на функцию *Render* :

```cpp
Player->Draw(*Renderer);
```

Если бы вы начали игру сейчас, вы бы увидели не только уровень, но и модную платформу игрока, расположенную на нижнем крае сцены. Покачто она ничего не делает, поэтому мы собираемся доработать функцию *Processinput*, чтобы горизонтально перемещать платформу всякий раз, когда пользователь нажимает клавишу A или D:

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
                Player->Position.x -= velocity;
        }
        if (this->Keys[GLFW_KEY_D])
        {
            if (Player->Position.x <= this->Width - Player->Size.x)
                Player->Position.x += velocity;
        }
    }
} 
```

Здесь мы перемещаем игровую платформу либо в левом, либо в правом направлении, в зависимости от того, какую клавишу нажал пользователь \(обратите внимание, как мы умножаем скорость с переменной deltatime\). Если бы значение платформы было меньше 0, она бы переместилась за левый край, так что мы перемещаем платформу только влево, если значение х больше, чем положение левого ребра х \(0,0\). Мы делаем то же самое в случае, когда платформа выходит за границы правого края, но мы должны сравнить положение правого края с правым ребром платформы \(вычитая ширину платформы из позиции х правого ребра\).

Теперь при запуске игры у нас есть игровая платформа, которую мы можем перемещать по всему нижнему краю игрового поля:

![](4.png)

Вы можете найти обновленный исходный код класса Game здесь:

- Game: [header](game.h), [code](game.cpp)
