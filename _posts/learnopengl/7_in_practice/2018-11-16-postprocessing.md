---
layout: post
title: Postprocessing
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: in-practice-2dgame-p2
---

{% include learnopengl.md link="In-Practice/2D-Game/Postprocessing" %}

Czy nie byłoby fajnie, gdybyśmy mogli całkowicie urozmaicić grę Breakout za pomocą kilku efektów postprocessingowych? Możemy stworzyć rozmyty efekty shake'a, odwrócić wszystkie kolory sceny, wykonać szalony ruch ekranu i/lub skorzystać z innych interesujących efektów z względną łatwością dzięki framebufferom OpenGL.

{: .box-note }
Ten samouczek w dużym stopniu wykorzystuje pojęcia z samouczków [framebuffers]({% post_url /learnopengl/4_advanced_opengl/2018-08-31-framebuffers %}) i [anty-aliasing]({% post_url /learnopengl/4_advanced_opengl/2018-09-14-antyaliasing %}).

W samouczku framebuffers zademonstrowaliśmy, w jaki sposób można wykorzystać efekty postprocesingu, aby uzyskać interesujące efekty przy użyciu tylko jednej tekstury. W Breakout zamierzamy zrobić coś podobnego: stworzymy obiekt framebuffer z multisamplingowym renderbuffer'em dołączonym jako załącznik koloru framebuffera. Cały kod renderujący gry powinien być renderowany do tego multisamplingowego framebuffera, który następnie kopiuje swoją zawartość do innego bufora ramki z załącznikiem tekstury jako jego buforem koloru. Ta tekstura zawiera wyrenderowany wygładzony obraz gry, który będziemy renderować do dużego kwadratu bez lub z dodatkowym efektem postprocessingu.

Aby podsumować te kroki renderowania:

1. Powiąż multisampled framebuffer
2. Renderuj grę jak zwykle
3. Skopiuj multisampled framebuffer do normalnego framebuffera
4. Odepnij framebuffery (użyj domyślnego framebuffera)
5. Użyj tekstury bufora koloru z normalnego bufora ramki w shaderze postprocessingu
6. Renderuj quad o rozmiarze ekranu jako wyjściowy postprocessing

Shader postprocessingu pozwala na trzy rodzaje efektów: shake, confuse i chaos.

*   **shake**: delikatnie porusza sceną z małym rozmyciem.
*   **confuse**: odwraca kolory sceny, ale także oś `x` i `y`.
*   **chaos**: wykorzystuje jądro (kernel) do wykrywania krawędzi do tworzenia interesujących wizualizacji, a także porusza oteksturowany obraz po okręgu, aby uzyskać ciekawy efekt chaosu.

Poniżej znajduje się obraz tego, jak te efekty będą wyglądać:

![Efekty postprocessingu w grze OpenGL Breakout](/img/learnopengl/postprocessing_effects.png){: .center-image }

Działając na kwadracie, Vertex Shader wygląda następująco:

```cpp
    #version 330 core
    layout (location = 0) in vec4 vertex; // <vec2 position, vec2 texCoords>

    out vec2 TexCoords;

    uniform bool  chaos;
    uniform bool  confuse;
    uniform bool  shake;
    uniform float time;

    void main()
    {
        gl_Position = vec4(vertex.xy, 0.0f, 1.0f); 
        vec2 texture = vertex.zw;
        if(chaos)
        {
            float strength = 0.3;
            vec2 pos = vec2(texture.x + sin(time) * strength, texture.y + cos(time) * strength);        
            TexCoords = pos;
        }
        else if(confuse)
        {
            TexCoords = vec2(1.0 - texture.x, 1.0 - texture.y);
        }
        else
        {
            TexCoords = texture;
        }
        if (shake)
        {
            float strength = 0.01;
            gl_Position.x += cos(time * 10) * strength;        
            gl_Position.y += cos(time * 15) * strength;        
        }
    }  
```

Na podstawie tego, jaki uniform jest ustawiony na `true`, Vertex Shader może obierać różne ścieżki. Jeśli <var>chaos</var> lub <var>confuse</var> jest ustawione na `true`, to Vertex Shader manipuluje współrzędnymi tekstur, aby przesunąć scenę (albo przesunąć współrzędne tekstury w sposób podobny do ruchu po okręgu, albo odwrócić współrzędne tekstury). Ponieważ ustawimy metody zawijania tekstur na `GL_REPEAT`, efekt chaosu spowoduje, że scena powtórzy się w różnych częściach kwadratu. Dodatkowo, jeśli <var>shake</var> jest ustawiona na `true`, to przesunie pozycje wierzchołków tylko o niewielką ilość. Zauważ, że <var>chaos</var> i <var>confuse</var> nie powinny mieć wartości `true` w tym samym czasie, podczas gdy <var>shake</var> może działać z dowolnym innym efektem.

Oprócz przesunięcia pozycji wierzchołków lub współrzędnych tekstury, chcielibyśmy również stworzyć znaczący efekt wizualny, gdy tylko którykolwiek z efektów będzie aktywny. Możemy to zrobić w Fragment Shader:

```glsl
    #version 330 core
    in  vec2  TexCoords;
    out vec4  color;

    uniform sampler2D scene;
    uniform vec2      offsets[9];
    uniform int       edge_kernel[9];
    uniform float     blur_kernel[9];

    uniform bool chaos;
    uniform bool confuse;
    uniform bool shake;

    void main()
    {
        color = vec4(0.0f);
        vec3 sample[9];
        // sample from texture offsets if using convolution matrix
        if(chaos || shake)
            for(int i = 0; i < 9; i++)
                sample[i] = vec3(texture(scene, TexCoords.st + offsets[i]));

        // process effects
        if(chaos)
        {           
            for(int i = 0; i < 9; i++)
                color += vec4(sample[i] * edge_kernel[i], 0.0f);
            color.a = 1.0f;
        }
        else if(confuse)
        {
            color = vec4(1.0 - texture(scene, TexCoords).rgb, 1.0);
        }
        else if(shake)
        {
            for(int i = 0; i < 9; i++)
                color += vec4(sample[i] * blur_kernel[i], 0.0f);
            color.a = 1.0f;
        }
        else
        {
            color =  texture(scene, TexCoords);
        }
    }
```

Ten długi shader prawie bezpośrednio opiera się na Fragment Shader z samouczka framebuffers i przetwarza efekty postprocesingu w zależności od aktywowanego typu efektu. Tym razem jednak macierze offset i jądra splotu są zdefiniowane jako uniformy, które ustawiamy z kodu aplikacji. Zaletą jest to, że musimy ustawić je tylko raz, zamiast przeliczać te macierze w każdym przebiegu Fragment Shadera. Na przykład macierz <var>offset</var> jest skonfigurowana w następujący sposób:

```cpp
    GLfloat offset = 1.0f / 300.0f;
    GLfloat offsets[9][2] = {
        { -offset,  offset  },  // top-left
        {  0.0f,    offset  },  // top-center
        {  offset,  offset  },  // top-right
        { -offset,  0.0f    },  // center-left
        {  0.0f,    0.0f    },  // center-center
        {  offset,  0.0f    },  // center - right
        { -offset, -offset  },  // bottom-left
        {  0.0f,   -offset  },  // bottom-center
        {  offset, -offset  }   // bottom-right    
    };
    glUniform2fv(glGetUniformLocation(shader.ID, "offsets"), 9, (GLfloat*)offsets);  
```

Ponieważ wszystkie koncepcje zarządzania (multisampling) framebufferami zostały już obszernie omówione we wcześniejszych samouczkach, tym razem nie będę zagłębiał się w szczegóły. Poniżej znajdziesz kod klasy <fun>PostProcessor</fun>, która zarządza inicjowaniem, zapisywaniem/odczytywaniem framebuffer'ów i renderowaniem pełnoekranowego kwadratu. Powinieneś być w stanie całkowicie zrozumieć kod, jeśli zrozumiałeś samouczki framebuuffers i anty-aliasing.

*   **PostProcessor**: [nagłówek](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/post_processor.h), [kod](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/post_processor).

Warto zwrócić uwagę na funkcje <fun>BeginRender</fun> i <fun>EndRender</fun>. Ponieważ musimy renderować całą scenę gry do bufora ramki, możemy w sposób konwencjonalny wywoływać <fun>BeginRender()</fun> i <fun>EndRender()</fun> odpowiednio przed i po kodzie renderowania sceny. Klasa zajmie się zakulisowymi działaniami bufora ramki. Na przykład użycie klasy <fun>PostProcessor</fun> wygląda tak w funkcji <fun>Render</fun>:

```cpp
    PostProcessor   *Effects;

    void Game::Render()
    {
        if (this->State == GAME_ACTIVE)
        {
            Effects->BeginRender();
                // Draw background
                // Draw level
                // Draw player
                // Draw particles	
                // Draw ball
            Effects->EndRender();
            Effects->Render(glfwGetTime());
        }
    }
```

Gdziekolwiek chcemy, możemy teraz wygodnie ustawić wymaganą właściwość efektu klasy postprocessingu na `true`, a jej efekt będzie od razu widoczny.

### Shake it

Jako (praktyczna) demonstracja tych efektów będziemy naśladować wizualny wpływ piłki, gdy uderzy ona w niezniszczalny blok. Po włączeniu efektu potrząsania przez krótki okres czasu, w miejscu występowania kolizji, wygląda na to, że zderzenie miało silniejszy wpływ.

Chcemy włączyć efekt wstrząsania tylko przez krótki czas. Możemy to uruchomić, tworząc zmienną o nazwie <var>ShakeTime</var>, która zawiera czas działania efektu wstrząsania. Wszędzie tam, gdzie występuje kolizja, resetujemy tę zmienną do określonego czasu trwania:

```cpp
    GLfloat ShakeTime = 0.0f;  

    void Game::DoCollisions()
    {
        for (GameObject &box : this->Levels[this->Level].Bricks)
        {
            if (!box.Destroyed)
            {
                Collision collision = CheckCollision(*Ball, box);
                if (std::get<0>(collision)) // If collision is true
                {
                    // Destroy block if not solid
                    if (!box.IsSolid)
                        box.Destroyed = GL_TRUE;
                    else
                    {   // if block is solid, enable shake effect
                        ShakeTime = 0.05f;
                        Effects->Shake = true;
                    }
                    [...]
                }
            }    
        }
        [...]
    }  
```

Następnie w ramach funkcji <fun>Update</fun> zmniejszamy tę zmienną <var>ShakeTime</var> aż do `0.0`, po czym wyłączamy efekt wstrząsania:

```cpp
    void Game::Update(GLfloat dt)
    {
        [...]
        if (ShakeTime > 0.0f)
        {
            ShakeTime -= dt;
            if (ShakeTime <= 0.0f)
                Effects->Shake = false;
        }
    }  
```

Za każdym razem, gdy uderzymy w niezniszczalny blok, ekran na krótko zaczyna się trząść i rozmazywać, dając graczowi wizualną informację zwrotną, gdy piłka zderzyła się z niezniszczalnym przedmiotem.

<div align="center"><video width="600" loop="" controls="">  
<source src="https://learnopengl.com/video/in-practice/breakout/postprocessing_shake.mp4" type="video/mp4">  
</video></div>

Możesz znaleźć zaktualizowany kod źródłowy klasy gry [tutaj](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/game_postprocessing).

W następnym samouczku dotyczącym powerup'ów, wykorzystamy dwa pozostałe efekty postprocessingu.