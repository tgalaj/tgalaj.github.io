---
layout: post
title: Przygotowania
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: in-practice-2dgame
---

{% include learnopengl.md link="In-Practice/2D-Game/Setting-up" %}

Zanim zaczniemy od implementacji rzeczywistej mechaniki gry, musimy najpierw stworzyć prosty framework gry. Gra będzie korzystać z kilku bibliotek zewnętrznych, z których większość została wprowadzona we wcześniejszych samouczkach. Wszędzie tam, gdzie wymagana jest nowa biblioteka, zostanie ona prawidłowo wprowadzona.

Najpierw definiujemy tak zwaną <def>uber</def> klasę gry, która zawiera cały istotny kod renderowania i rozgrywki. Ideą takiej klasy gry jest to, że organizuje ona twój kod, jednocześnie oddzielając cały kod okienkowy od gry. W ten sposób możesz użyć tej samej klasy w zupełnie innej bibliotece okienkowej (na przykład SDL lub SFML) bez większego wysiłku.

{: .box-note }
Istnieją tysiące sposobów na próbę abstrakcji i generalizacji kodu gry/grafiki na klasy i obiekty. To, co zobaczysz w tych samouczkach, to tylko jedno podejście do rozwiązania tego problemu. Jeśli uważasz, że istnieje lepsze podejście, spróbuj wymyślić własną implementację.

Klasa gry obsługuje funkcję inicjalizacji (init), funkcję aktualizacji (update), funkcję przetwarzania I/O (process input) i funkcję renderowania (render):

```cpp
    class Game
    {
        public:
            // Game state
            GameState  State;	
            GLboolean  Keys[1024];
            GLuint	   Width, Height;
            // Constructor/Destructor
            Game(GLuint width, GLuint height);
            ~Game();
            // Initialize game state (load all shaders/textures/levels)
            void Init();
            // GameLoop
            void ProcessInput(GLfloat dt);
            void Update(GLfloat dt);
            void Render();
    };
```

Klasa jest gospodarzem tego, czego możesz oczekiwać od klasy gry. Inicjujemy grę, podając szerokość i wysokość okna (odpowiadającą rozdzielczości, w której chcemy zagrać w grę) i używamy funkcji <fun>Init</fun>, aby załadować shadery, tekstury i zainicjować cały stan rozgrywki. Możemy przetwarzać dane wejściowe przechowywane w tablicy <var>Keys</var>, wywołując <fun>ProcessInput</fun> i aktualizować wszystkie zdarzenia związane z rozgrywką (takie jak ruch gracza/piłki) w <fun>Update</fun>. Na koniec możemy renderować grę, wywołując <fun>Render</fun>. Zauważ, że oddzieliliśmy logikę poruszania się od logiki renderowania.

Klasa <fun>Game</fun> udostępnia również zmienną o nazwie <var>State</var>, która jest typu <def>GameState</def> zgodnie z poniższą definicją:

```cpp
    // Represents the current state of the game
    enum GameState {
        GAME_ACTIVE,
        GAME_MENU,
        GAME_WIN
    }; 
```

To pozwala nam śledzić, w jakim stanie jest obecnie gra. W ten sposób możemy zdecydować czy renderować i/lub przetwarzać różne rzeczy w oparciu o bieżący stan gry (prawdopodobnie renderujemy i przetwarzamy różne rzeczy, gdy jesteśmy w menu gry).

W tej chwili funkcje klasy gry są całkowicie puste, ponieważ musimy jeszcze napisać kod gry, ale tutaj znajduje się [nagłówek](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/game_setting-up.h) klasy Game i plik z [kodem](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/game_setting-up).

## Narzędzia

Ponieważ tworzymy dużą aplikację, często będziemy musieli ponownie użyć kilku obiektów OpenGL, takich jak tekstury i shadery. To ma sens, aby stworzyć łatwiejszy w użyciu interfejs dla tych dwóch elementów, podobnie jak w jednym z wcześniejszych samouczków, w których stworzyliśmy klasę Shader.

Zdefiniowana klasa Shader, generuje skompilowany obiekt shader'a (lub generuje komunikaty o błędach, jeśli kompilacja/linkowanie się nie powiedzie) z dwóch lub trzech ciągów znaków (jeśli obecny jest Geometry Shader). Klasa Shader zawiera również wiele użytecznych funkcji pomocniczych do szybkiego ustawiania wartości uniformów. Została również zdefiniowana klasa tekstury, która generuje obraz tekstury 2D (w oparciu o jej właściwości) z tablicy bajtów i danej szerokości i wysokości. Ponownie, klasa tekstury zawiera również funkcje pomocnicze.

Nie będziemy zagłębiali się w szczegóły klas, ponieważ teraz powinniście z łatwością zrozumieć, jak one działają. Z tego powodu poniżej możesz znaleźć pliki nagłówków i kodów z komentarzami:

*   **Shader**: [nagłówek](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/shader.h), [kod](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/shader).
*   **Texture**: [nagłówek](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/texture.h), [kod](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/texture).

Zwróć uwagę, że bieżąca klasa tekstury jest przeznaczona wyłącznie dla tekstur 2D, ale można ją łatwo rozszerzyć o alternatywne typy tekstur.

## Zarządzanie zasobami

Podczas gdy klasy shader i texture działają same w sobie, wymagają one albo tablicy bajtów, albo kilku łańcuchów znaków, aby je zainicjować. Moglibyśmy z łatwością osadzić kod ładowania pliku w samych klasach, ale to lekko narusza <def>zasadę jednej odpowiedzialności</def> (ang. *single responsibility principle*) polegającą na tym, że klasy powinny skupiać się wyłącznie na teksturach lub shaderach, a niekoniecznie na mechanice ładowania plików.

Z tego powodu jest często tworzy się pojedynczy obiekt zaprojektowany do ładowania zasobów związanych z grami zwany <def>menedżerem zasobów</def> (ang. *resource manager*). Istnieje kilka podejść do tworzenia menedżera zasobów; w tym samouczku zdecydowaliśmy się użyć statycznego menedżera zasobów jako singleton, który (ze względu na jego statyczną naturę) jest zawsze dostępny w całym projekcie obsługującym wszystkie załadowane zasoby i odpowiednią funkcję ładowania.

Używanie klasy singleton z funkcjami statycznymi ma wiele wad i zalet, gdzie wadą jest przede wszystkim utrata właściwości OOP i utrata kontroli nad tworzeniem/niszczeniem obiektu. Jednak w przypadku stosunkowo małych projektów, takich jak te, łatwo jest z nimi pracować.

Podobnie jak inne pliki klas, menedżer zasobów jest wymieniony poniżej:

*   **Resource Manager**: [nagłówek](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/resource_manager.h), [kod](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/resource_manager).

Za pomocą menedżera zasobów możemy łatwo załadować shadery do programu:

```cpp
    Shader shader = ResourceManager::LoadShader("vertex.vs", "fragment.vs", nullptr, "test");
    // then use it
    shader.Use();
    // or
    ResourceManager::GetShader("test").Use();
```

Zdefiniowana klasa <fun>Game</fun> wraz z menedżerem zasobów i łatwymi w zarządzaniu klasami <fun>Shader</fun> i <fun>Texture2D</fun> stanowią podstawę do kolejnych tutoriali, gdzie będziemy intensywnie wykorzystywać te klasy do implementacji gry Breakout.

## Program

Nadal potrzebujemy okna dla gry i ustawiony początkowy stan OpenGL. Korzystamy z funkcji OpenGL [face-culling]({% post_url /learnopengl/4_advanced_opengl/2018-08-29-face-culling %}) i [blendingu]({% post_url /learnopengl/4_advanced_opengl/2018-08-27-blending %}). Nie używamy testów głębokości; ponieważ gra jest całkowicie dwuwymiarowa, wszystkie wierzchołki są zdefiniowane z tymi samymi wartościami `z`, więc włączenie testowania głębokości byłoby bezużyteczne i prawdopodobnie spowodowałoby z-fighting.

Kod startowy gry Breakout jest stosunkowo prosty: tworzymy okno GLFW, rejestrujemy kilka funkcji zwrotnych, tworzymy obiekt Game i propagujemy wszystkie odpowiednie funkcje do klasy gry. Kod jest podany poniżej:

*   **Program**: [kod](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/program).

Uruchomienie kodu powinno dać następujące wyniki:

![Pusty obraz początkowy gry Breakout w OpenGL](/img/learnopengl/setting-up.png){: .center-image }

Od tej pory mamy solidny framework dla nadchodzących samouczków; będziemy ciągle rozszerzać klasę gry o nowe funkcje. Kiedy będziesz gotowy, przejdź do następnego tutoriala.