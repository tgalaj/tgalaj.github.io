---
layout: post
title: Cząsteczki
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: in-practice-2dgame-p2
---

{% include learnopengl.md link="In-Practice/2D-Game/Particles" %}

<span class="def">Cząsteczka</span> (ang. *particle*), widziana z perspektywy OpenGL, to maleńki kwadrat, który zawsze jest ustawiony przodem do kamery (billboarding) i (zwykle) zawiera teksturę z dużą ilością przezroczystości. Cząsteczka sama w sobie jest w zasadzie tylko sprite'em, których do tej pory tak intensywnie używamy, ale kiedy zbierzecie razem setki, a nawet tysiące tych cząsteczek, możecie stworzyć niesamowite efekty.

Podczas pracy z cząsteczkami zwykle występuje obiekt zwany <def>emiterem cząstek</def> (ang. *particle emitter*) lub <def>generatorem cząstek</def> (ang. *particle generator*), który z jego położenia stale <def>emituje/generuje</def> (ang. *spawns*) nowe cząstki, które rozpadają się z biegiem czasu. Jeśli taki emiter cząsteczek będzie na przykład generować drobne cząstki o strukturze dymo-podobnej, barwić je na mniej jasny kolor wraz ze wzrostem odległości od emitera i nada im efekt poświaty, otrzymamy efekt podobny do ognia:

![Przykład cząsteczek jako ognia](/img/learnopengl/particles_example.jpg){: .center-image }

Pojedyncza cząstka często ma zmienną czas życia (ang. *life*), która powoli spada po wygenerowaniu cząstki. Kiedy jej czas życia jest mniejszy niż pewien próg (zwykle `0`), <def>zabijamy</def> (ang. *kill*) cząstkę, aby można było zastąpić ją nowym obiektem cząstki. Emiter cząstek kontroluje wszystkie wygenerowane cząstki i zmienia ich zachowanie na podstawie ich atrybutów. Cząstka ma zazwyczaj następujące atrybuty:

```cpp
    struct Particle {
        glm::vec2 Position, Velocity;
        glm::vec4 Color;
        GLfloat Life;

        Particle() 
          : Position(0.0f), Velocity(0.0f), Color(1.0f), Life(0.0f) { }
    };    
```

Patrząc na przykład ognia, emiter cząsteczek prawdopodobnie generuje każdą cząstkę z pozycją bliską emiterowi i z prędkością w górę tak, aby każda cząstka poruszała się w pozytywnym kierunku `y`. Wydaje się, że ma 3 różne regiony, więc prawdopodobnie daje cząstkom większą prędkość niż inne. Widzimy także, że im wyższa pozycja cząstki `y`, tym mniej jasny/żółty staje się jej kolor. Po osiągnięciu przez cząsteczki określonej wysokości ich życie zostaje wyczerpane, a cząsteczki zostają zabite; nigdy nie docierają do gwiazd.

Możesz sobie wyobrazić, że dzięki takim systemom możemy tworzyć ciekawe efekty, takie jak ogień, dym, mgła, efekty magiczne, efekt wystrzału. W Breakout dodamy prosty generator cząstek dla piłki, aby wszystko wyglądało bardziej interesująco. Wyglądać to będzie mniej więcej tak:

<div align="center"><video width="600" loop="" controls="">  
<source src="https://learnopengl.com/video/in-practice/breakout/particles.mp4" type="video/mp4">  
</video></div>

Tutaj generator cząsteczek emituje każdą cząsteczkę z pozycji piłki, daje jej prędkość równą ułamkowi prędkości piłki i zmienia kolor cząstki w oparciu o czas jej życia.

Do renderowania cząstek użyjemy innego zestawu shaderów:

```glsl
    #version 330 core
    layout (location = 0) in vec4 vertex; // <vec2 position, vec2 texCoords>

    out vec2 TexCoords;
    out vec4 ParticleColor;

    uniform mat4 projection;
    uniform vec2 offset;
    uniform vec4 color;

    void main()
    {
        float scale = 10.0f;
        TexCoords = vertex.zw;
        ParticleColor = color;
        gl_Position = projection * vec4((vertex.xy * scale) + offset, 0.0, 1.0);
    }
```

Fragment Shader:

```glsl
    #version 330 core
    in vec2 TexCoords;
    in vec4 ParticleColor;
    out vec4 color;

    uniform sampler2D sprite;

    void main()
    {
        color = (texture(sprite, TexCoords) * ParticleColor);
    }  
```

Przyjmujemy standardowe atrybuty pozycji i tekstury na cząstkę, a także akceptujemy uniform <var>offset</var> i <var>color</var>, aby zmienić wynik wizualny per cząstka. Zauważ, że w Vertex Shader przeskalujemy kwadrat cząsteczek o `10.0f`; możesz również ustawić skalę jako uniform i kontrolować ją indywidualnie dla każdej cząstki.

Najpierw potrzebujemy listy cząstek, które następnie tworzymy z domyślnym konstruktorem struktury <fun>Particle</fun>.

```cpp
    GLuint nr_particles = 500;
    std::vector<Particle> particles;

    for (GLuint i = 0; i < nr_particles; ++i)
        particles.push_back(Particle());
```

Następnie w każdej klatce tworzymy kilka nowych cząstek z wartościami początkowymi, a następnie dla każdej cząstki, która jest (nadal) żywa, aktualizujemy jej wartości.

```cpp
    GLuint nr_new_particles = 2;
    // Add new particles
    for (GLuint i = 0; i < nr_new_particles; ++i)
    {
        int unusedParticle = FirstUnusedParticle();
        RespawnParticle(particles[unusedParticle], object, offset);
    }
    // Update all particles
    for (GLuint i = 0; i < nr_particles; ++i)
    {
        Particle &p = particles[i];
        p.Life -= dt; // reduce life
        if (p.Life > 0.0f)
        {	// particle is alive, thus update
            p.Position -= p.Velocity * dt;
            p.Color.a -= dt * 2.5;
        }
    }  
```

Pierwsza pętla może wyglądać trochę zniechęcająco. Ponieważ cząstki giną z upływem czasu, chcemy odrodzić cząstki <var>nr_new_particles</var> w każdej klatce, ale ponieważ od początku zdecydowaliśmy, że całkowita ilość cząstek, które będziemy wykorzystywać, to <var>nr_particles</var> nie możemy po prostu wstawić nowych cząstek na koniec listy. W ten sposób szybko otrzymamy listę wypełnioną tysiącami cząsteczek, która nie jest zbyt efektywna, biorąc pod uwagę, że tylko niewielka część tej listy zawiera żywe cząstki.

Chcemy znaleźć pierwszą martwą cząstkę (życie < `0.0f`) i zaktualizować tę cząstkę jako nową odrodzoną cząstkę.

Funkcja <fun>FirstUnusedParticle</fun> próbuje znaleźć pierwszą martwą cząstkę i zwraca jej indeks.

```cpp
    GLuint lastUsedParticle = 0;
    GLuint FirstUnusedParticle()
    {
        // Search from last used particle, this will usually return almost instantly
        for (GLuint i = lastUsedParticle; i < nr_particles; ++i){
            if (particles[i].Life <= 0.0f){
                lastUsedParticle = i;
                return i;
            }
        }
        // Otherwise, do a linear search
        for (GLuint i = 0; i < lastUsedParticle; ++i){
            if (particles[i].Life <= 0.0f){
                lastUsedParticle = i;
                return i;
            }
        }
        // Override first particle if all others are alive
        lastUsedParticle = 0;
        return 0;
    }  
```

Funkcja przechowuje indeks ostatnio znalezionej martwej cząstki, ponieważ następna martwa cząstka będzie najprawdopodobniej tuż po tym ostatnim indeksie cząstek, więc najpierw szukamy począwszy od tego zapisanego indeksu. Jeśli nie znaleźliśmy żadnych martwych cząstek, po prostu wykonamy wolniejsze wyszukiwanie liniowe. Jeśli żadne cząstki nie są martwe, zwróć indeks `0`, co spowoduje nadpisanie pierwszej cząstki. Zauważ, że jeśli dojdzie do tego ostatniego przypadku, oznacza to, że cząstki są żywe zbyt długo, musisz odradzać mniej cząsteczek na klatkę i/lub po prostu nie masz wystarczającej ilości zarezerwowanych cząstek.

Następnie, po znalezieniu pierwszej martwej cząstki na liście, aktualizujemy jej wartości, wywołując <fun>RespawnParticle</fun>, która przyjmuje cząsteczkę, obiekt <fun>GameObject</fun> i wektor przesunięcia:

```cpp
    void RespawnParticle(Particle &particle, GameObject &object, glm::vec2 offset)
    {
        GLfloat random = ((rand() % 100) - 50) / 10.0f;
        GLfloat rColor = 0.5 + ((rand() % 100) / 100.0f);
        particle.Position = object.Position + random + offset;
        particle.Color = glm::vec4(rColor, rColor, rColor, 1.0f);
        particle.Life = 1.0f;
        particle.Velocity = object.Velocity * 0.1f;
    }  
```

Ta funkcja po prostu resetuje życie cząstki do `1.0f`, losowo nadaje jej jasność (poprzez wektor koloru) zaczynając od `0.5` i przypisuje (nieco losową) pozycję i prędkość w oparciu o <fun>GameObject</fun>.

Druga pętla w ramach funkcji aktualizacji iteruje po wszystkich cząstkach i dla każdej cząstki zmniejsza jej żywotność o zmienną czasową delta; w ten sposób życie każdej cząstki odpowiada sekundom. Następnie sprawdzamy, czy cząstka jest żywa, jeśli tak, zaktualizuj jej położenie i atrybuty koloru. Tutaj powoli zmniejszamy składową alfa każdej cząstki, aby wyglądało na to, że z czasem zanikają.

Pozostaje nam tylko wyrenderować cząstki:

```cpp
    glBlendFunc(GL_SRC_ALPHA, GL_ONE);
    particleShader.Use();
    for (Particle particle : particles)
    {
        if (particle.Life > 0.0f)
        {
            particleShader.SetVector2f("offset", particle.Position);
            particleShader.SetVector4f("color", particle.Color);
            particleTexture.Bind();
            glBindVertexArray(particleVAO);
            glDrawArrays(GL_TRIANGLES, 0, 6);
            glBindVertexArray(0);
        } 
    } 
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
```

Tutaj, dla każdej cząstki, ustalamy jej wartość równa przesunięciu i kolorze, wiążemy teksturę i generujemy kwadraty. Interesujące są tutaj dwa wywołania funkcji <fun>glBlendFunc</fun>. Podczas renderowania cząstek zamiast domyślnego trybu mieszania celu `GL_ONE_MINUS_SRC_ALPHA` używamy trybu mieszania `GL_ONE`, który nadaje cząstkom bardzo ładny efekt <def>blasku</def>, gdy są ułożone jedna na drugiej. Prawdopodobnie jest to także tryb mieszania używany podczas renderowania ognia z górnej części samouczka, ponieważ ogień "błyszczy" w środku, gdzie znajduje się większość jego cząstek.

Ponieważ (podobnie jak większość innych części serii samouczków) lubimy porządkować wszystko, stworzono inną klasę o nazwie <fun>ParticleGenerator</fun>, która zawiera wszystkie funkcje, o których właśnie mówiliśmy. Poniżej znajdziesz kod źródłowy:

*   **ParticleGenerator**: [nagłówek](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/particle_generator.h), [kod](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/particle_generator)

Następnie w kodzie gry tworzymy taki generator cząstek i inicjalizujemy go za pomocą [tej](https://learnopengl.com/img/in-practice/breakout/textures/particle.png) tekstury.

```cpp
    ParticleGenerator   *Particles; 

    void Game::Init()
    {
        [...]
        ResourceManager::LoadShader("shaders/particle.vs", "shaders/particle.frag", nullptr, "particle");
        [...]
        ResourceManager::LoadTexture("textures/particle.png", GL_TRUE, "particle"); 
        [...]
        Particles = new ParticleGenerator(
            ResourceManager::GetShader("particle"), 
            ResourceManager::GetTexture("particle"), 
            500
        );
    }
```

Następnie zmieniamy funkcję <fun>Update</fun> klasy gry, dodając instrukcję aktualizacji generatora cząstek:

```cpp
    void Game::Update(GLfloat dt)
    {
        [...]
        // Update particles
        Particles->Update(dt, *Ball, 2, glm::vec2(Ball->Radius / 2));
        [...]
    }
```

Każda z cząstek korzysta z  właściwości obiektu gry z obiektu piłki, tworząc 2 cząstki na każdą ramkę, a ich pozycje zostaną przesunięte w kierunku środka kuli. Na koniec renderujemy cząstki:

```cpp
    void Game::Render()
    {
        if (this->State == GAME_ACTIVE)
        {
            [...]
            // Draw player
            Player->Draw(*Renderer);
            // Draw particles	
            Particles->Draw();
            // Draw ball
            Ball->Draw(*Renderer);
        }
    }  
```

Zwróć uwagę, że renderujemy cząstki, zanim piłka zostanie wyrenderowana i po tym, jak inne elementy zostaną wyrenderowane, dzięki czemu cząstki znajdą się przed wszystkimi innymi elementami, ale pozostaną za piłką. Możesz znaleźć zaktualizowany kod klasy gry [tutaj](https://learnopengl.com/code_viewer.php?code=in-practice/breakout/game_particles).

Jeśli teraz skompilujesz i uruchomisz swoją aplikację, powinieneś zobaczyć ślad cząsteczek podążający za piłką, tak jak widzieliśmy to na początku tego samouczka, nadając grze bardziej nowoczesny wygląd. System można również łatwo rozszerzyć, aby obsługiwał bardziej zaawansowane efekty, więc możesz poeksperymentować z generowaniem cząstek i sprawdzić, czy możesz wymyślić własne kreatywne efekty.