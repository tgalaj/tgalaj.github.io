---
layout: post
title: Renderowanie sprite'ów
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: in-practice-2dgame
---

{% include learnopengl.md link="In-Practice/2D-Game/Rendering-Sprites" %}

Aby wprowadzić życie w obecnie czarne otchłanie naszego świata gry, stworzymy sprite'y, które wypełnią pustkę. <def>Sprite</def> ma wiele definicji, ale w zasadzie jest to obraz 2D użyty razem z niektórymi danymi do umieszczenia go w większym świecie, jak pozycja, kąt obrotu i dwuwymiarowy rozmiar. Zasadniczo, spritey są renderowanymi obiektami tekstury używanymi w grze 2D.

Możemy, tak jak w przypadku większości tutoriali, utworzyć kształt 2D z danych wierzchołkowych, przekazać wszystkie dane do procesora graficznego i przekształcić wszystko ręcznie. Jednak dla większej aplikacji takiej jak ta raczej mamy pewne abstrakcje dotyczące renderowania kształtów 2D. Gdybyśmy mieli ręcznie definiować te kształty i transformacje dla każdego obiektu, kod szybko stanie się nieczytelny.

W tym samouczku zdefiniujemy klasę renderowania, która pozwoli nam renderować dużą liczbę sprite'ów z minimalną ilością kodu. W ten sposób oddzielamy kodu rozgrywki od grubego kodu renderującego, jak to zwykle robi się w większych projektach. Najpierw musimy jednak ustawić właściwą macierz projekcji.

## Macierz projekcji 2D

Wiemy z tutoriala [układy współrzędnych]({% post_url /learnopengl/1_getting_started/2017-09-25-uklady-wspolrzednych %}), że macierz projekcji przekształca wszystkie współrzędne przestrzeni widoku na znormalizowane współrzędne urządzenia. Poprzez wygenerowanie odpowiedniej macierzy projekcji możemy pracować z różnymi współrzędnymi, które są prawdopodobnie łatwiejsze do pracy w porównaniu do bezpośredniego określania wszystkich współrzędnych jako znormalizowanych współrzędnych urządzenia.

Nie potrzebujemy żadnej perspektywy dla współrzędnych, ponieważ gra jest całkowicie dwuwymiarowa, więc macierz rzutu prostokątnego dobrze nadawałaby się do renderowania 2D. Ponieważ macierz rzutowania prostokątnego prawie bezpośrednio przekształca wszystkie współrzędne do przestrzeni NDC, dlatego możemy wybrać podawanie współrzędnych w przestrzeni świata jako współrzędne ekranu, definiując macierz rzutowania w następujący sposób:

```cpp
    glm::mat4 projection = glm::ortho(0.0f, 800.0f, 600.0f, 0.0f, -1.0f, 1.0f);  
```

Pierwsze cztery argumenty określają w kolejności lewą, prawą, dolną i górną część frustum. Ta macierz przekształca wszystkie współrzędne `x` między `0` i `800` na `-1` i `1` oraz wszystkie współrzędne `y` między `0` a `600` na `-1` i `1`. Tutaj określiliśmy, że górna część frustum ma współrzędną `y` wynoszącą `0`, natomiast dół ma współrzędną `y` wynoszącą `600`. Rezultat jest taki, że górna lewa współrzędna sceny znajduje się w (`0,0`), a dolna prawa część ekranu ma współrzędne (`800,600`), podobnie jak współrzędne ekranu; Współrzędne przestrzeni widoku odpowiadają bezpośrednio wynikowym współrzędnym na ekranie.

![Rzutowanie ortograficzne w OpenGL](/img/learnopengl/projection.png){: .center-image }

Dzięki temu możemy określić wszystkie współrzędne wierzchołków równe współrzędnym pikselowym, które pojawiają się na ekranie, co jest dość intuicyjne w przypadku gier 2D.

## Renderowanie sprite'ów

Renderowanie sprite'ów nie powinno być zbyt skomplikowane. Tworzymy oteksturowany kwadrat, który możemy przekształcać za pomocą macierzy modelu, po czym rzutujemy go przy użyciu wcześniej zdefiniowanej macierzy rzutowania prostokątnego.

{: .box-note }
Ponieważ Breakout jest grą statyczną, nie ma potrzeby tworzenia macierzy widoku/kamery, więc za pomocą macierzy projekcji możemy bezpośrednio przekształcać współrzędne w przestrzeni świata we współrzędne w przestrzeni NDC.

Aby przekształcić sprite używamy następującego Vertex Shadera:

```glsl
    #version 330 core
    layout (location = 0) in vec4 vertex; // <vec2 position, vec2 texCoords>

    out vec2 TexCoords;

    uniform mat4 model;
    uniform mat4 projection;

    void main()
    {
        TexCoords = vertex.zw;
        gl_Position = projection * model * vec4(vertex.xy, 0.0, 1.0);
    }
```

Zwróć uwagę, że przechowujemy zarówno położenie, jak i dane współrzędnych tekstury w jednej zmiennej <fun>vec4</fun>. Ponieważ zarówno położenie, jak i współrzędne tekstury zawierają dwie zmienne, możemy je połączyć w jeden atrybut wierzchołka.

Fragment Shader jest również stosunkowo prosty. Pobieramy teksturę i wektor koloru, które wpływają na ostateczny kolor fragmentu. Mając również uniform wektora kolorów możemy łatwo zmienić kolor ikonek z kodu gry.

```glsl
    #version 330 core
    in vec2 TexCoords;
    out vec4 color;

    uniform sampler2D image;
    uniform vec3 spriteColor;

    void main()
    {    
        color = vec4(spriteColor, 1.0) * texture(image, TexCoords);
    }  
```

Aby uczynić rendering sprite'ów bardziej zorganizowanym, zdefiniowaliśmy klasę <fun>SpriteRenderer</fun>, która jest w stanie wyrenderować sprite'y za pomocą tylko jednej funkcji. Jej definicja jest następująca:

```cpp
    class SpriteRenderer
    {
        public:
            SpriteRenderer(Shader &shader);
            ~SpriteRenderer();

            void DrawSprite(Texture2D &texture, glm::vec2 position, 
                glm::vec2 size = glm::vec2(10, 10), GLfloat rotate = 0.0f, 
                glm::vec3 color = glm::vec3(1.0f));
        private:
            Shader shader; 
            GLuint quadVAO;

            void initRenderData();
    };
```

Klasa <def>SpriteRenderer</def> zawiera obiekt shadera, pojedynczy obiekt tablicy wierzchołków oraz funkcję renderowania i inicjalizacji. Jego konstruktor pobiera obiekt shadera, którego używa dla wszystkich przyszłych wywołań renderowania.

### Inicjalizacja

Najpierw zajrzyjmy do funkcji <fun>initRenderData</fun>, która konfiguruje <var>quadVAO</var>:

```cpp
    void SpriteRenderer::initRenderData()
    {
        // Configure VAO/VBO
        GLuint VBO;
        GLfloat vertices[] = { 
            // Pos      // Tex
            0.0f, 1.0f, 0.0f, 1.0f,
            1.0f, 0.0f, 1.0f, 0.0f,
            0.0f, 0.0f, 0.0f, 0.0f, 

            0.0f, 1.0f, 0.0f, 1.0f,
            1.0f, 1.0f, 1.0f, 1.0f,
            1.0f, 0.0f, 1.0f, 0.0f
        };

        glGenVertexArrays(1, &this->quadVAO);
        glGenBuffers(1, &VBO);

        glBindBuffer(GL_ARRAY_BUFFER, VBO);
        glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

        glBindVertexArray(this->quadVAO);
        glEnableVertexAttribArray(0);
        glVertexAttribPointer(0, 4, GL_FLOAT, GL_FALSE, 4 * sizeof(GLfloat), (GLvoid*)0);
        glBindBuffer(GL_ARRAY_BUFFER, 0);  
        glBindVertexArray(0);
    }
```

Tutaj najpierw definiujemy zbiór wierzchołków z współrzędną (`0,0`) będącą lewym górnym rogiem kwadratu. Oznacza to, że kiedy zastosujemy translację lub transformację skalowania na kwadracie, zostaną zastosowane względem lewego górnego położenia kwadratu. Jest to powszechnie akceptowane w grafice 2D i/lub systemach GUI, gdzie pozycje elementów odpowiadają lewemu górnemu rogowi elementów.

Następnie po prostu wysyłamy wierzchołki do GPU i konfigurujemy atrybuty wierzchołków, które w tym przypadku są pojedynczym atrybutem wierzchołków. Musimy zdefiniować pojedyncze VAO dla renderera sprite'ów, ponieważ wszystkie sprite'y mają te same dane wierzchołkowe.

### Renderowanie

Renderowanie sprite'ów nie jest zbyt trudne; używamy shadera do renderowania sprite'ów, konfigurujemy macierz modelu i ustawiamy odpowiednie uniformy. Ważna jest tutaj kolejność transformacji:

```cpp
    void SpriteRenderer::DrawSprite(Texture2D &texture, glm::vec2 position, 
      glm::vec2 size, GLfloat rotate, glm::vec3 color)
    {
        // Prepare transformations
        this->shader.Use();
        glm::mat4 model;
        model = glm::translate(model, glm::vec3(position, 0.0f));  

        model = glm::translate(model, glm::vec3(0.5f * size.x, 0.5f * size.y, 0.0f)); 
        model = glm::rotate(model, rotate, glm::vec3(0.0f, 0.0f, 1.0f)); 
        model = glm::translate(model, glm::vec3(-0.5f * size.x, -0.5f * size.y, 0.0f));

        model = glm::scale(model, glm::vec3(size, 1.0f)); 

        this->shader.SetMatrix4("model", model);
        this->shader.SetVector3f("spriteColor", color);

        glActiveTexture(GL_TEXTURE0);
        texture.Bind();

        glBindVertexArray(this->quadVAO);
        glDrawArrays(GL_TRIANGLES, 0, 6);
        glBindVertexArray(0);
    }  
```

Podczas próby pozycjonowania obiektów gdzieś w scenie z rotacją i skalowaniem, zaleca się najpierw wykonać skalowanie, a następnie rotację i ostatecznie translację. Ponieważ mnożenie macierzy odbywa się od prawej do lewej, przekształcamy macierz w odwrotnej kolejności: translacja, rotacja, a następnie skalowanie.

Transformacja rotacji może początkowo wydawać się nieco zniechęcająca. Wiemy z tutoriala [transformacje]({% post_url /learnopengl/1_getting_started/2017-09-18-transformacje %}), że rotacja zawsze obracaja obiekt wokół punktu początkowego (`0,0`). Ponieważ określiliśmy wierzchołki kwadratu z (`0,0`) jako lewą górną współrzędną kwadratu, wszystkie rotacje obrócą obiekt wokół tego punktu (`0,0`). Zasadniczo, <def>źródło rotacji</def> znajduje się w lewym górnym rogu kwadratu, co powoduje niepożądane wyniki. To, co chcemy zrobić, to przesunąć początek obrotu do środka kwadratu, tak aby kwadrat starannie obracał się wokół tego punktu, zamiast obracać się wokół lewego górnego rogu kwadratu. Rozwiązujemy to poprzez translację kwadratu, aby jego środek był na współrzędnej (`0,0`) przed obróceniem.

![Prawidłowa rotacja kwadratu względem jego środka](/img/learnopengl/rotation-origin.png){: .center-image }

Ponieważ najpierw skalujemy kwadrat, musimy wziąć pod uwagę rozmiar sprite'a podczas translacji do środka sprite'a (dlatego mnożymy ją przez wektor <var>size</var>). Po zastosowaniu transformacji rotacji cofamy poprzednią translację.

Łącząc wszystkie te transformacje, możemy pozycjonować, skalować i obracać każdy sprite w dowolny sposób. Poniżej znajduje się kompletny kod źródłowy SpriteRenderer'a:

*   **SpriteRenderer**: [nagłówek](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/sprite_renderer.h), [kod](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/sprite_renderer)

## Witaj sprite

Z klasą <fun>SpriteRenderer</fun> mamy wreszcie możliwość renderowania rzeczywistych obrazów na ekranie! Zróbmy inicjalizację w kodzie gry i załadujmy naszą ulubioną [teksturę](https://learnopengl.com/img/textures/awesomeface.png):

```cpp
    SpriteRenderer  *Renderer;

    void Game::Init()
    {
        // Load shaders
        ResourceManager::LoadShader("shaders/sprite.vs", "shaders/sprite.frag", nullptr, "sprite");
        // Configure shaders
        glm::mat4 projection = glm::ortho(0.0f, static_cast<GLfloat>(this->Width), 
            static_cast<GLfloat>(this->Height), 0.0f, -1.0f, 1.0f);
        ResourceManager::GetShader("sprite").Use().SetInteger("image", 0);
        ResourceManager::GetShader("sprite").SetMatrix4("projection", projection);
        // Set render-specific controls
        Renderer = new SpriteRenderer(ResourceManager::GetShader("sprite"));
        // Load textures
        ResourceManager::LoadTexture("textures/awesomeface.png", GL_TRUE, "face");
    }
```

Następnie w funkcji renderowania możemy narysować naszą ukochaną maskotkę, aby sprawdzić, czy wszystko działa tak, jak powinno:

```cpp
    void Game::Render()
    {
        Renderer->DrawSprite(ResourceManager::GetTexture("face"), 
            glm::vec2(200, 200), glm::vec2(300, 400), 45.0f, glm::vec3(0.0f, 1.0f, 0.0f));
    }  
```

Tutaj umieszczamy sprite nieco bliżej środka ekranu, którego wysokość jest nieco większa od jego szerokości. Obracamy go również o 45 stopni i nadajemy mu zielony kolor. Zwróć uwagę, że pozycja, którą dajemy, jest równa lewemu górnemu wierzchołkowi kwadratu sprite'a.

Jeśli zrobiłeś wszystko dobrze, powinieneś otrzymać następujący wynik:

![Obraz renderowanego sprite'a za pomocą naszej niestandardowej klasy SpriteRenderer](/img/learnopengl/rendering-sprites.png){: .center-image }

Możesz znaleźć zaktualizowany kod źródłowy klasy gry [tutaj](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/game_rendering-sprites).

Teraz, gdy mamy działające systemy renderowania, możemy je dobrze wykorzystać w kolejnym tutorialu, w którym będziemy pracować nad budowaniem poziomów gry.