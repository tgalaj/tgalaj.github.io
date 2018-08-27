---
layout: post
title: Renderowanie tekstu
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: in-practice-2dgame-p2
---

{% include learnopengl.md link="In-Practice/2D-Game/Render-text" %}

W tym samouczku dodamy końcowe ulepszenia do gry, dodając system życia, warunek wygranej i informację zwrotną w postaci renderowanego tekstu. Ten samouczek w dużej mierze opiera się na wcześniejszym tutorialu [Renderowanie tekstu]({% post_url /learnopengl/7_in_practice/2018-10-26-renderowanie-tekstu %}), dlatego zaleca się, aby najpierw przeczytać przez ten samouczek, jeśli jeszcze tego nie zrobiłeś.

W Breakout cały kod renderujący tekst jest zamknięty w klasie o nazwie <fun>TextRenderer</fun>, która zawiera inicjalizację biblioteki FreeType, konfigurację renderowania i rzeczywisty kod renderowania. Poniżej znajdziesz kod klasy <fun>TextRenderer</fun>:

*   **TextRenderer**: [nagłówek](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/text_renderer.h), [kod](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/text_renderer).
*   **Text shaders**: [Vertex Shader](https://learnopengl.com/code_viewer.php?code=in-practice/text_rendering&type=vertex), [Fragment Shader](https://learnopengl.com/code_viewer.php?code=in-practice/text_rendering&type=fragment).

Zawartość funkcji renderowania tekstu jest prawie identyczna z kodem z samouczka renderowania tekstu. Jednak kod renderowania glifów na ekranie jest nieco inny:

```cpp
    void TextRenderer::RenderText(std::string text, GLfloat x, GLfloat y, GLfloat scale,glm::vec3 color)
    {
        [...]
        for (c = text.begin(); c != text.end(); c++)
        {
            GLfloat xpos = x + ch.Bearing.x * scale;
            GLfloat ypos = y + (this->Characters['H'].Bearing.y - ch.Bearing.y) * scale;

            GLfloat w = ch.Size.x * scale;
            GLfloat h = ch.Size.y * scale;
            // Update VBO for each character
            GLfloat vertices[6][4] = {
                { xpos,     ypos + h,   0.0, 1.0 },
                { xpos + w, ypos,       1.0, 0.0 },
                { xpos,     ypos,       0.0, 0.0 },

                { xpos,     ypos + h,   0.0, 1.0 },
                { xpos + w, ypos + h,   1.0, 1.0 },
                { xpos + w, ypos,       1.0, 0.0 }
            };
            [...]
        }
    }    
```

Powodem tego, że jest on nieco inny, jest to, że używamy innej macierzy projekcji niż ta, której użyliśmy w samouczku do renderowania tekstu. W samouczku do renderowania tekstu, wszystkie wartości `y` wahały się od dołu do góry, podczas gdy w grze Breakout wszystkie wartości `y` wahają się od góry do dołu z współrzędną `y` wynoszącą `0.0` odpowiadającą górnej krawędzi ekranu. Oznacza to, że musimy nieco zmodyfikować sposób obliczania przesunięcia pionowego.

Ponieważ teraz renderujemy w dół, od parametru <var>y</var> funkcji <fun>RenderText</fun>, obliczamy przesunięcie pionowe jako odległość, o którą glif jest przesunięty w dół od górnej krawędzi glifu. Patrząc wstecz na obraz danych glifów z FreeType, jest to oznaczone czerwoną strzałką:

![Przesunięcie pionowe glifu FreeType od wierzchołka jego glifowej przestrzeni dla pionowo odwróconej ortograficznej macierzy projekcyjnej w OpenGL](/img/learnopengl/glyph_offset.png){: .center-image }

Aby obliczyć to przesunięcie w pionie, musimy uzyskać górną powierzchnię glifu (w zasadzie długość czarnej pionowej strzałki od początku). Niestety, FreeType nie ma takich danych dla nas. Wiemy tylko, że niektóre glify zawsze dotykają tej górnej krawędzi; znaki takie jak `H`, `T` lub `X`. A co jeśli obliczymy długość tego czerwonego wektora przez odjęcie `bearingY` od któregokolwiek wartości `bearingY` tych, dotykającej górnej krawędzi glifów. W ten sposób przesuwamy glif w dół w zależności od tego, jak daleko jego górny punkt różni się od górnej krawędzi.

```cpp
    GLfloat ypos = y + (this->Characters['H'].Bearing.y - ch.Bearing.y) * scale;  
```

Oprócz aktualizacji obliczeń `ypos` zmieniliśmy nieco kolejność wierzchołków, aby upewnić się, że wszystkie wierzchołki są nadal zwrócone przodem do kamery, gdy są pomnożone przez bieżącą macierz rzutowania (jak omówiono w samouczku [face culling]({% post_url /learnopengl/4_advanced_opengl/2018-08-29-face-culling %})).

Dodanie do klasy Game <fun>TextRenderer</fun>'a jest proste:

```cpp
    TextRenderer  *Text;

    void Game::Init()
    {
        [...]
        Text = new TextRenderer(this->Width, this->Height);
        Text->Load("fonts/ocraext.TTF", 24);
    }
```

Renderer tekstu jest inicjowany czcionką OCR A Extended, którą można pobrać [tutaj](http://fontzone.net/font-details/ocr-a-extended). Jeśli czcionka nie odpowiada Twoim potrzebom, możesz użyć innej czcionki.

Teraz, gdy mamy renderer tekstu, zakończmy mechanikę rozgrywki.

## Punkty życia gracza

Zamiast natychmiast resetować grę, gdy tylko piłka osiągnie dolną krawędź, chcielibyśmy dać graczowi kilka dodatkowych szans. Robimy to w formie punktów życia gracza, gdzie gracz zaczyna od początkowej liczby żyć (powiedzmy `3`) i za każdym razem, gdy piłka dotyka dolnej krawędzi, całkowita suma życia gracza zmniejsza się o 1. Dopiero gdy suma punktów życia gracza zmieni się na `0`, resetujemy grę. Ułatwia to graczowi ukończenie poziomu, a jednocześnie buduje napięcie.

Przechowujemy punkty życia gracza, dodając je do klasy gry (zainicjowanej w konstruktorze wartością `3`):

```cpp
    class Game
    {
        [...]
        public:  
            GLuint Lives;
    }
```

Następnie modyfikujemy funkcję <fun>Update</fun>, aby zamiast resetować grę, zmniejszyć całkowite życie gracza i zresetować grę, gdy łączna wartość życia osiągnie `0`:

```cpp
    void Game::Update(GLfloat dt)
    {
        [...]
        if (Ball->Position.y >= this->Height) // Did ball reach bottom edge?
        {
            --this->Lives;
            // Did the player lose all his lives? : Game over
            if (this->Lives == 0)
            {
                this->ResetLevel();
                this->State = GAME_MENU;
            }
            this->ResetPlayer();
        }
    }
```

Jak tylko gracz przegra (<var>lives</var> jest równe `0`), resetujemy poziom i zmieniamy stan gry na <var>GAME_MENU</var>, który zrobimy później.

Nie zapomnij zresetować łącznej liczby punktów życia gracza, gdy tylko zresetujemy grę/poziom:

```cpp
    void Game::ResetLevel()
    {
        [...]
        this->Lives = 3;
    }  
```

Gracz ma teraz działające punkty życia, ale nie ma możliwości sprawdzenia, ile ma obecnie żyć w trakcie gry. Tutaj właśnie pojawia się renderer tekstu.

```cpp
    void Game::Render()
    {
        if (this->State == GAME_ACTIVE)
        {
            [...]
            std::stringstream ss; ss << this->Lives;
            Text->RenderText("Lives:" + ss.str(), 5.0f, 5.0f, 1.0f);
        }
    }  
```

Tutaj konwertujemy liczbę żyć na ciąg i wyświetlamy go w lewym górnym rogu ekranu. Teraz będzie wyglądać to tak:

![Renderowany tekst z FreeType w OpenGL wyświetlający całkowitą liczbę żyć gracza](/img/learnopengl/render_text_lives.png){: .center-image }

Gdy tylko piłka dotknie dolnej krawędzi, całkowita liczba żyć gracza zostaje zmniejszona, co jest natychmiast widoczne w lewym górnym rogu ekranu.

## Wybór poziomu

Ilekroć użytkownik znajdzie się w stanie gry <var>GAME_MENU</var>, chcemy dać graczowi kontrolę, aby mógł wybrać poziom, na którym chciałby grać. Za pomocą klawisza `w` lub `s` gracz powinien móc przewijać wszystkie załadowane poziomy. Ilekroć gracz czuje, że wybrany poziom jest rzeczywiście poziomem, na którym chciałby grać, może nacisnąć klawisz `Enter`, aby przejść ze stanu <var>GAME_MENU</var> do stanu <var>GAME_ACTIVE</var> .

Umożliwienie graczowi wybrania poziomu nie jest zbyt trudne. Wszystko, co musimy zrobić, to zwiększyć lub zmniejszyć zmienną <var>Level</var> klasy gry w zależności od tego, czy naciśnięty został odpowiednio klawisz `w` czy `s`:

```cpp
    if (this->State == GAME_MENU)
    {
        if (this->Keys[GLFW_KEY_ENTER])
            this->State = GAME_ACTIVE;
        if (this->Keys[GLFW_KEY_W])
            this->Level = (this->Level + 1) % 4;
        if (this->Keys[GLFW_KEY_S])
        {
            if (this->Level > 0)
                --this->Level;
            else
                this->Level = 3;   
        }
    }  
```

Korzystamy z operatora modulo (`%`), aby upewnić się, że zmienna <var>Level</var> pozostaje w akceptowalnym przedziale (między `0` i `3`). Oprócz przełączania poziomów chcemy także zdefiniować, co chcemy renderować, gdy jesteśmy w menu. Chcemy dać graczowi instrukcje w postaci tekstu, a także wyświetlić wybrany poziom w tle.

```cpp
    void Game::Render()
    {
        if (this->State == GAME_ACTIVE || this->State == GAME_MENU)
        {
            [...] // Game state's rendering code
        }
        if (this->State == GAME_MENU)
        {
            Text->RenderText("Press ENTER to start", 250.0f, Height / 2, 1.0f);
            Text->RenderText("Press W or S to select level", 245.0f, Height / 2 + 20.0f, 0.75f);
        }
    }  
```

Renderujemy grę za każdym razem, gdy jesteśmy w stanie <var>GAME_ACTIVE</var> lub <var>GAME_MENU</var> i gdy jesteśmy w stanie <var>GAME_MENU</var> również renderujemy dwa wiersze tekstu, aby poinformować gracza, aby wybrał poziom i/lub zaakceptował jego wybór. Pamiętaj, że aby to działało podczas uruchamiania gry, musisz domyślnie ustawić stan gry jako <var>GAME_MENU</var>.

![Wybieranie poziomów za pomocą FreeType renderuje tekst w OpenGL](/img/learnopengl/render_text_select.png){: .center-image }

Wygląda świetnie, ale gdy spróbujesz uruchomić kod, prawdopodobnie zauważysz, że zaraz po naciśnięciu klawisza `w` lub `s` gra szybko przewija poziomy, co utrudnia wybór żądanego poziomu do gry. Dzieje się tak, ponieważ gra zapisuje naciśnięcie klawisza dla wielu klatek, dopóki nie zwolnimy klawisza. Powoduje to, że funkcja <fun>ProcessInput</fun> przetwarza naciśnięty klawisz więcej niż jeden raz.

Możemy rozwiązać ten problem za pomocą małego triku powszechnie spotykanego w systemach GUI. Sztuką jest nie tylko zapisywanie naciśniętych klawiszy, ale również przechowywanie klawiszy, które zostały przetworzone raz, aż do ich zwolnienia. Następnie sprawdzamy (przed przetworzeniem), czy klawisz nie został jeszcze przetworzony, a jeśli tak, przetworzymy ten klawisz, po czym oznaczamy ten klawisz jako przetwarzany. Gdy chcemy przetworzyć ten sam klawisz ponownie bez zwalniania go, nie przetwarzamy klawisza. To prawdopodobnie brzmi nieco myląco, ale jak tylko zobaczysz to w praktyce (prawdopodobnie) zaczyna to mieć sens.

Najpierw musimy utworzyć kolejną tablicę wartości bool, aby wskazać, które klawisze zostały przetworzone. Definiujemy to w ramach klasy gry:

```cpp
    class Game
    {
        [...]
        public:  
            GLboolean KeysProcessed[1024];
    } 
```

Następnie ustawiamy odpowiednie klawisze na `true`, gdy tylko są przetwarzane i upewnij się, że przetwarzasz klawisz tylko wtedy, gdy nie został przetworzony wcześniej (dopóki nie zostanie zwolniony):

```cpp
    void Game::ProcessInput(GLfloat dt)
    {
        if (this->State == GAME_MENU)
        {
            if (this->Keys[GLFW_KEY_ENTER] && !this->KeysProcessed[GLFW_KEY_ENTER])
            {
                this->State = GAME_ACTIVE;
                this->KeysProcessed[GLFW_KEY_ENTER] = GL_TRUE;
            }
            if (this->Keys[GLFW_KEY_W] && !this->KeysProcessed[GLFW_KEY_W])
            {
                this->Level = (this->Level + 1) % 4;
                this->KeysProcessed[GLFW_KEY_W] = GL_TRUE;
            }
            if (this->Keys[GLFW_KEY_S] && !this->KeysProcessed[GLFW_KEY_S])
            {
                if (this->Level > 0)
                    --this->Level;
                else
                    this->Level = 3;
                this->KeysProcessed[GLFW_KEY_S] = GL_TRUE;
            }
        }
        [...]
    }  
```

Teraz, gdy tylko wartość klawisza w tablicy <var>KeysProcessed</var> nie została jeszcze ustawiona, przetwarzamy klawisz i ustawiamy jego wartość na `true`. Następnym razem, gdy osiągniemy warunek `if` tego samego klawisza, zostanie on przetworzony, więc będziemy udawać, że nigdy nie nacisnęliśmy przycisku, dopóki nie zostanie on ponownie zwolniony.

W ramach funkcji klawiszy wywołania zwrotnego GLFW musimy zresetować przetworzoną wartość klucza, gdy tylko zostanie on zwolniony, abyśmy mogli przetworzyć go ponownie przy następnym naciśnięciu:

```cpp
    void key_callback(GLFWwindow* window, int key, int scancode, int action, int mode)
    {
        [...]
        if (key >= 0 && key < 1024)
        {
            if (action == GLFW_PRESS)
                Breakout.Keys[key] = GL_TRUE;
            else if (action == GLFW_RELEASE)
            {
                Breakout.Keys[key] = GL_FALSE;
                Breakout.KeysProcessed[key] = GL_FALSE;
            }
        }
    }  
```

Uruchomienie gry daje nam ekran wyboru poziomu, który teraz precyzyjnie wybiera pojedynczy poziom przy każdym naciśnięciu klawisza bez względu na to, jak długo naciskamy klawisz.

## Zwycięstwo

Obecnie gracz jest w stanie wybrać poziom, zagrać w grę i przegrać. To trochę niefortunne, jeśli gracz dowie się po zniszczeniu wszystkich cegieł, że nie może wygrać. Naprawmy to.

Gracz wygrywa, gdy wszystkie bloki zostały zniszczone. Stworzyliśmy już funkcję sprawdzania tego stanu za pomocą klasy <fun>GameLevel</fun>:

```cpp
    GLboolean GameLevel::IsCompleted()
    {
        for (GameObject &tile : this->Bricks)
            if (!tile.IsSolid && !tile.Destroyed)
                return GL_FALSE;
        return GL_TRUE;
    }  
```

Sprawdzamy wszystkie cegły na poziomie gry i jeśli jakaś cegła nie jest jeszcze zniszczona, zwracamy `false`. Musimy tylko sprawdzić ten warunek w funkcji <fun>Update</fun> i gdy tylko zwróci `true`, zmieniamy stan gry na <var>GAME_WIN</var>:

```cpp
    void Game::Update(GLfloat dt)
    {
        [...]
        if (this->State == GAME_ACTIVE && this->Levels[this->Level].IsCompleted())
        {
            this->ResetLevel();
            this->ResetPlayer();
            Effects->Chaos = GL_TRUE;
            this->State = GAME_WIN;
        }
    }
```

Za każdym razem, gdy poziom jest ukończony, gdy gra jest aktywna, resetujemy grę i wyświetlamy mały komunikat zwycięstwa w stanie <var>GAME_WIN</var>. Dla zabawy włączymy efekt chaosu na ekranie <var>GAME_WIN</var>. W funkcji <fun>Render</fun> gratulujemy graczowi i poprosimy go o ponowne uruchomienie lub zakończenie gry:

```cpp
    void Game::Render()
    {
        [...]
        if (this->State == GAME_WIN)
        {
            Text->RenderText(
                "You WON!!!", 320.0, Height / 2 - 20.0, 1.0, glm::vec3(0.0, 1.0, 0.0)
            );
            Text->RenderText(
                "Press ENTER to retry or ESC to quit", 130.0, Height / 2, 1.0, glm::vec3(1.0, 1.0, 0.0)
            );
        }
    }  
```

Wtedy oczywiście musimy przetworzyć wymienione klawisze:

```cpp
    void Game::ProcessInput(GLfloat dt)
    {
        [...]
        if (this->State == GAME_WIN)
        {
            if (this->Keys[GLFW_KEY_ENTER])
            {
                this->KeysProcessed[GLFW_KEY_ENTER] = GL_TRUE;
                Effects->Chaos = GL_FALSE;
                this->State = GAME_MENU;
            }
        }
    }  
```

Jeśli uda Ci się wygrać, otrzymasz następujący obraz:

![Obraz wygranej w OpenGL Breakout z renderowanym tekstem FreeType](/img/learnopengl/render_text_win.png){: .center-image }

I to jest już koniec! Był to ostatni element gry Breakout, nad którą pracowaliśmy. Wypróbuj ją, dostosuj do swoich upodobań i pokaż go całej rodzinie i znajomym!

Poniżej znajdziesz ostateczną wersję kodu gry:

*   **Game**: [nagłówek](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/game.h), [kod](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/game).