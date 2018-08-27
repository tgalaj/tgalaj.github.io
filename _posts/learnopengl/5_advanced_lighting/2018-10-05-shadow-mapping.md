---
layout: post
title: Shadow mapping
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: advanced-lighting-shadows
mathjax: true
---

{% include learnopengl.md link="Advanced-Lighting/Shadows/Shadow-Mapping" %}

Cienie są wynikiem braku światła z powodu okluzji; kiedy promienie źródła światła nie uderzają w obiekt, ponieważ zostają one zasłonięte przez jakiś inny obiekt, obiekt jest w cieniu. Cienie dodają dużo realizmu do sceny i ułatwiają widzowi obserwowanie relacji przestrzennych między obiektami. Dają większą głębię naszej scenie i przedmiotom. Spójrz na następujący obraz sceny z cieniami i bez cieni:

![porównanie sceny bez i z cieniami w OpenGL](/img/learnopengl/shadow_mapping_with_without.png){: .center-image }

Widać, że z cieniami staje się o wiele bardziej oczywiste, w jaki sposób obiekty odnoszą się do siebie. Na przykład fakt, że jedna z kostek unosi się nad innymi, jest znacznie bardziej zauważalny, gdy mamy cienie.

Cienie są jednak nieco trudne do zaimplementowania, szczególnie dlatego, że w obecnych badaniach grafiki w czasie rzeczywistym nie opracowano jeszcze idealnego algorytmu cieni. Istnieje kilka dobrych technik aproksymacji cieni, ale wszystkie mają swoje małe wady, które musimy wziąć pod uwagę.

Jedną z technik używanych w większości gier wideo, która zapewnia przyzwoite rezultaty i jest stosunkowo łatwa do wdrożenia, jest technika <def>shadow mapping</def>. Shadow mapping nie jest zbyt trudne do zrozumienia, nie kosztuje zbyt dużo wydajności i jest dość łatwo rozszerzalne na bardziej zaawansowane algorytmy (np. Mapy cieni świateł punktowych (ang. *Omnidirectional Shadow Maps*) i Kaskadowe Mapy Cieni (ang. *Cascaded Shadow Maps*)).

## Shadow mapping

Idea shadow mappingu jest dość prosta: renderujemy scenę z punktu widzenia światła i wszystko, co widzimy z perspektywy światła, jest oświetlone, a wszystko, czego nie widzimy, musi być w cieniu. Wyobraź sobie podłogę z dużym pudełkiem między podłogą a źródłem światła. Ponieważ źródło światła będzie widzieć to pudełko, a nie skrawek podłogi, to ten konkretny skrawek podłogi powinien znajdować się w cieniu.

![Zilustrowany Shadow mapping.](/img/learnopengl/shadow_mapping_theory.png){: .center-image }

Tutaj wszystkie niebieskie linie reprezentują fragmenty, które może zobaczyć źródło światła. Zasłonięte fragmenty są pokazane jako czarne linie: są one renderowane jako zacienione. Gdybyśmy narysowali linię lub <def>promień</def> ze źródła światła do fragmentu po prawej stronie, widzimy, jak promień uderza najpierw w unoszący się kontener, zanim trafi w najbardziej prawy kontener. W rezultacie fragment unoszącego się kontenera jest oświetlony, a fragment po prawej stronie kontenera nie jest oświetlony, a więc jest w cieniu.

Chcemy uzyskać punkt na promieniu, w którym po raz pierwszy trafił obiekt i porównać ten _najbliższy punkt_ z innymi punktami na tym promieniu. Następnie wykonujemy podstawowy test, aby sprawdzić, czy położenie punktu testowego znajduje się dalej na tym promieniu niż najbliższy punkt, jeśli tak, to ten punkt testowy musi być w cieniu. Iteracja po tysiącu promieni światła z takiego źródła światła jest niezwykle nieskutecznym podejściem i nie nadaje się zbyt dobrze do renderowania w czasie rzeczywistym. Możemy zrobić coś podobnego, ale bez rzucania promieni światła. Zamiast tego używamy czegoś, co jest nam dobrze znane: bufora głębi.

Prawdopodobnie pamiętasz z tutoriala [test głębokości]({% post_url /learnopengl/4_advanced_opengl/2018-08-22-test-glebokosci %}), że wartość w buforze głębi odpowiada głębokości fragmentu w zakresie `[0, 1]` z punktu widzenia kamery. Co by było, gdybyśmy wyrenderowali scenę z perspektywy światła i zachowali wynikowe wartości głębokości w teksturze? W ten sposób możemy próbkować najbliższe wartości głębokości widziane z perspektywy światła. W końcu wartości głębokości pokazują pierwszy fragment widoczny z perspektywy światła. Wszystkie te wartości głębokości przechowujemy w teksturze, którą nazywamy <def>mapą głębi</def> (ang. *depth map*) lub <def>mapą cieni</def> (ang. *shadow map*).

![Różne transformacje współrzędnych/przestrzeni dla shadow mappingu.](/img/learnopengl/shadow_mapping_theory_spaces.png){: .center-image }

Lewy obraz pokazuje kierunkowe źródło światła (wszystkie promienie światła są do siebie równoległe) rzucając cień na powierzchnię pod kostką. Używając wartości głębokości zapisanych w mapie głębokości, znajdujemy najbliższy punkt i używamy go do określenia, czy fragmenty są w cieniu. Tworzymy mapę głębokości poprzez renderowanie sceny (z perspektywy światła) przy użyciu macierzy widoku i projekcji charakterystycznej dla tego źródła światła. Macierze projekcji i widoku razem tworzą transformację $T$, która przekształca dowolną pozycję 3D na przestrzeń współrzędnych światła.

{: .box-note }
Kierunkowe światło nie ma pozycji, ponieważ jest modelowane jako bardzo (nieskończenie) odległe źródło światła. Jednak ze względu na shadow mapping, musimy renderować scenę z perspektywy światła, a tym samym oddać scenę z miejsca gdzieś wzdłuż linii kierunku światła.

Obrazie po prawej stronie, widzimy to samo światło kierunkowe i widza. Renderujemy fragment w punkcie $\bar{\color{red}{P}}$, dla którego musimy określić, czy jest on w cieniu. Aby to zrobić, najpierw przekształcamy punkt $\bar{\color{red}{P}}$ w przestrzeń współrzędnych światła za pomocą $T$. Ponieważ punkt $\bar{\color{red}{P}}$ jest teraz widziany z perspektywy światła , jego współrzędna `z` odpowiada jej głębokości, która w tym przykładzie wynosi `0.9`. Używając punktu $\bar{\color{red}{P}}$ możemy także indeksować mapę głębi, aby uzyskać najbliższą widoczną głębię z perspektywy światła, która znajduje się w punkcie $\bar{\color{green}{C}}$ o głębokości `0.4`. Ponieważ indeksowanie mapy głębokości zwróciło głębokość mniejszą niż głębokość w punkcie $\bar{\color{red}{P}}$ możemy wywnioskować, że punkt $\bar{\color{red}{P}}$ jest przesłonięty, a więc jest w cieniu.

Shadow mapping składa się z dwóch etapów: najpierw renderujemy mapę głębi, a w drugim etapie renderujemy scenę i używamy wygenerowanej mapy głębokości do obliczenia, czy fragmenty są w cieniu. To może wydawać się nieco skomplikowane, ale gdy tylko przejdziemy przez tę technikę krok po kroku, prawdopodobnie zacznie to mieć sens.

## Mapa głębokości

Pierwszy etap wymaga wygenerowania mapy głębokości. Mapa głębi jest teksturą głębi renderowaną z perspektywy światła, której będziemy używać do obliczania cieni. Ponieważ musimy zapisać wyrenderowany wynik sceny w teksturach, potrzebujemy ponownie [framebufferów]({% post_url /learnopengl/4_advanced_opengl/2018-08-31-framebuffers %}).

Najpierw utworzymy obiekt framebuffer do renderowania mapy głębokości:

```cpp
    unsigned int depthMapFBO;
    glGenFramebuffers(1, &depthMapFBO);  
```

Następnie tworzymy teksturę 2D, której użyjemy jako bufora głębi dla bufora ramki:

```cpp
    const unsigned int SHADOW_WIDTH = 1024, SHADOW_HEIGHT = 1024;

    unsigned int depthMap;
    glGenTextures(1, &depthMap);
    glBindTexture(GL_TEXTURE_2D, depthMap);
    glTexImage2D(GL_TEXTURE_2D, 0, GL_DEPTH_COMPONENT, 
                 SHADOW_WIDTH, SHADOW_HEIGHT, 0, GL_DEPTH_COMPONENT, GL_FLOAT, NULL);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT); 
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);  
```

Generowanie mapy głębokości nie powinno wydawać się zbyt skomplikowane. Ponieważ interesują nas tylko o wartości głębokości, określamy formaty tekstury jako <var>GL_DEPTH_COMPONENT</var>. Podajemy również szerokość i wysokość tekstury równą `1024`: jest to rozdzielczość mapy głębi.

Z wygenerowaną teksturą głębi możemy ją dołączyć jako bufor głębi dla bufora ramki:

```cpp
    glBindFramebuffer(GL_FRAMEBUFFER, depthMapFBO);
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_TEXTURE_2D, depthMap, 0);
    glDrawBuffer(GL_NONE);
    glReadBuffer(GL_NONE);
    glBindFramebuffer(GL_FRAMEBUFFER, 0);  
```

Potrzebujemy tylko informacji o głębokości podczas renderowania sceny z perspektywy światła, więc nie ma potrzeby stosowania bufora kolorów. Obiekt bufora ramki nie jest jednak kompletny bez bufora kolorów, więc musimy wyraźnie powiedzieć OpenGL, że nie będziemy renderować żadnych kolorów. Robimy to poprzez ustawienie bufora odczytu i rysowania na <var>GL_NONE</var> za pomocą <fun>glDrawBuffer</fun> i <fun>glReadbuffer</fun>.

Przy poprawnie skonfigurowanym buforze ramki, który renderuje wartości głębokości do tekstury, możemy rozpocząć pierwszy etap: generowanie mapy głębokości. Cały etap renderowania obu etapów wygląda mniej więcej tak:

```cpp
    // 1. najpierw renderuj do mapy głębi
    glViewport(0, 0, SHADOW_WIDTH, SHADOW_HEIGHT);
    glBindFramebuffer(GL_FRAMEBUFFER, depthMapFBO);
        glClear(GL_DEPTH_BUFFER_BIT);
        ConfigureShaderAndMatrices();
        RenderScene();
    glBindFramebuffer(GL_FRAMEBUFFER, 0);
    // 2. następnie wyrenderuj scenę z shadow mappingiem (korzystając z mapy głębi)
    glViewport(0, 0, SCR_WIDTH, SCR_HEIGHT);
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    ConfigureShaderAndMatrices();
    glBindTexture(GL_TEXTURE_2D, depthMap);
    RenderScene();
```

Ten kod pominął pewne szczegóły, ale daje ogólne pojęcie o shadow mappingu. Należy tutaj zwrócić uwagę na wywołania <fun>glViewport</fun>. Ponieważ mapy cieni często mają inną rozdzielczość w porównaniu do tego, w jakiej rozdzielczości renderowaliśmy pierwotnie scenę (zazwyczaj jest to rozdzielczość okna), musimy zmienić parametry viewportu, aby uwzględnić rozmiar mapy cienia. Jeśli zapomnimy zaktualizować parametry viewportu, uzyskana mapa głębi będzie niekompletna lub zbyt mała.

### Transformacja przestrzeni światła

Niewiadomą w poprzednim fragmencie kodu jest funkcja <fun>ConfigureShaderAndMatrices</fun>. W drugim etapie jest to normalne: upewnij się, że ustawiono odpowiednie macierze projekcji i widoku oraz odpowiednie macierze modelu dla każdego obiektu. Jednak w pierwszym etapie używamy innej macierzy projekcji i widoku, aby renderować scenę z punktu widzenia światła.

Ponieważ modelujemy kierunkowe źródło światła, wszystkie jego promienie światła są równoległe. Z tego powodu użyjemy macierzy rzutowania ortograficznego dla źródła światła, w której nie ma deformacji spowodowanej perspektywą:

```cpp
    float near_plane = 1.0f, far_plane = 7.5f;
    glm::mat4 lightProjection = glm::ortho(-10.0f, 10.0f, -10.0f, 10.0f, near_plane, far_plane);  
```

Oto przykładowa macierz rzutowania ortograficznego użyta w tej demonstracyjnej scenie tego samouczka. Ponieważ macierz projekcji pośrednio określa zakres widoczności, np. chcesz mieć pewność, że frustum zawiera obiekty, które mają się znaleźć na mapie głębi. Gdy obiekty lub fragmenty nie znajdują się w mapie głębi, nie będą tworzyć cieni.

Aby utworzyć macierz widoku do transformacji każdego obiektu, aby były widoczne z punktu widzenia światła, użyjemy niesławnej funkcji <fun>glm::lookAt</fun>; tym razem z pozycją źródła światła, które patrzy na środek sceny.

```cpp
    glm::mat4 lightView = glm::lookAt(glm::vec3(-2.0f, 4.0f, -1.0f), 
                                      glm::vec3( 0.0f, 0.0f,  0.0f), 
                                      glm::vec3( 0.0f, 1.0f,  0.0f));  
```

Połączenie tych dwóch macierzy daje nam macierz transformacji światła, która przekształca każdy wektor w przestrzeni świata w przestrzeń widoku źródła światła; dokładnie to, czego potrzebujemy, aby wyrenderować mapę głębi.

```cpp
    glm::mat4 lightSpaceMatrix = lightProjection * lightView; 
```

Macierz <var>lightSpaceMatrix</var> jest macierzą transformacji, którą wcześniej oznaczaliśmy jako $T$. Za pomocą <var>lightSpaceMatrix</var> możemy renderować scenę tak, jak zwykle, pod warunkiem, że damy shaderowi równoważniki macierzy projekcji i widoku w przestrzeni światła. Jednak interesują nas tylko wartości głębokości, a nie wszystkie kosztowne obliczenia fragmentów w naszym głównym shaderze. Aby zaoszczędzić na wydajności, użyjemy innego, ale znacznie prostszego shadera do renderowania mapy głębi.

### Renderowanie do mapy głębokości

Kiedy renderujemy scenę z perspektywy światła, wolimy raczej używać prostego shadera, który tylko przekształca wierzchołki w przestrzeń światła i nic więcej. W przypadku tak prostego shadera o nazwie <var>simpleDepthShader</var> użyjemy następującego Vertex Shadera:

```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;

    uniform mat4 lightSpaceMatrix;
    uniform mat4 model;

    void main()
    {
        gl_Position = lightSpaceMatrix * model * vec4(aPos, 1.0);
    }  
```

Ten Vertex Shader przyjmuje macierz modelu na każdy obiekt i przekształca wszystkie wierzchołki do przestrzeni światła za pomocą <var>lightSpaceMatrix</var>.

Ponieważ nie mamy bufora kolorów, powstałe fragmenty nie wymagają żadnego przetwarzania, więc możemy po prostu użyć pustego Fragment Shadera:

```glsl
    #version 330 core

    void main()
    {             
        // gl_FragDepth = gl_FragCoord.z;
    }  
```

Pusty Fragment Shader nie wykonuje żadnego przetwarzania, a po zakończeniu działania bufor głębi jest aktualizowany. Możemy wyraźnie ustawić głębokość poprzez od komentowanie tej jednej linii, ale tak naprawdę dzieje się to za kulisami.

Teraz renderowanie bufora głębokości wygląda tak:

```cpp
    simpleDepthShader.use();
    glUniformMatrix4fv(lightSpaceMatrixLocation, 1, GL_FALSE, glm::value_ptr(lightSpaceMatrix));

    glViewport(0, 0, SHADOW_WIDTH, SHADOW_HEIGHT);
    glBindFramebuffer(GL_FRAMEBUFFER, depthMapFBO);
        glClear(GL_DEPTH_BUFFER_BIT);
        RenderScene(simpleDepthShader);
    glBindFramebuffer(GL_FRAMEBUFFER, 0);  
```

W tym przypadku funkcja <fun>RenderScene</fun> przyjmuje program shadera, wywołuje wszystkie odpowiednie funkcje rysowania i w razie potrzeby ustawia odpowiednie macierze modelu.

Rezultatem jest ładnie wypełniony bufor głębi, utrzymujący najbliższą głębokość każdego widocznego fragmentu z perspektywy światła. Wyświetlając tę ​​teksturę na kwadracie 2D, który wypełnia cały ekran (podobnie do tego, co zrobiliśmy w sekcji post-processingu na końcu tutoriala [framebuffers]({% post_url /learnopengl/4_advanced_opengl/2018-08-31-framebuffers %})) otrzymujemy coś takiego:

![Mapa głębi (lub cieni) techniki mapowania cieni](/img/learnopengl/shadow_mapping_depth_map.png){: .center-image }

Aby wyświetlić mapę głębi na kwadracie, użyliśmy następującego Fragment Shadera:

```glsl
    #version 330 core
    out vec4 FragColor;

    in vec2 TexCoords;

    uniform sampler2D depthMap;

    void main()
    {             
        float depthValue = texture(depthMap, TexCoords).r;
        FragColor = vec4(vec3(depthValue), 1.0);
    }  
```

Zauważ, że istnieją pewne subtelne różnice podczas wyświetlania głębokości za pomocą macierzy rzutowania perspektywicznego zamiast ortograficznej macierzy projekcji, ponieważ głębokość jest nieliniowa w przypadku rzutowania perspektywicznego. Pod koniec tego samouczka omówimy niektóre z tych subtelnych różnic.

Możesz znaleźć kod źródłowy renderowania sceny do mapy głębi [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/5.advanced_lighting/3.1.1.shadow_mapping_depth/shadow_mapping_depth.cpp).

## Renderowanie cieni

Przy poprawnie wygenerowanej mapie głębokości możemy zacząć generować rzeczywiste cienie. Kod służący do sprawdzenia, czy fragment jest w cieniu, jest (oczywiście) zaimplementowany w Fragment Shaderze, ale wykonujemy transformację światła w Vertex Shaderze:

```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aNormal;
    layout (location = 2) in vec2 aTexCoords;

    out VS_OUT {
        vec3 FragPos;
        vec3 Normal;
        vec2 TexCoords;
        vec4 FragPosLightSpace;
    } vs_out;

    uniform mat4 projection;
    uniform mat4 view;
    uniform mat4 model;
    uniform mat4 lightSpaceMatrix;

    void main()
    {    
        vs_out.FragPos = vec3(model * vec4(aPos, 1.0));
        vs_out.Normal = transpose(inverse(mat3(model))) * aNormal;
        vs_out.TexCoords = aTexCoords;
        vs_out.FragPosLightSpace = lightSpaceMatrix * vec4(vs_out.FragPos, 1.0);
        gl_Position = projection * view * model * vec4(aPos, 1.0);
    }
```

Nowością jest dodatkowy wektor wyjściowy <var>FragPosLightSpace</var>. Używamy tej samej macierzy <var>lightSpaceMatrix</var> (używanej do przekształcania wierzchołków w przestrzeń światła w etapie generowania mapy głębi) i przekształca położenia wierzchołków z przestrzeni świata do przestrzeni światła. Vertex Shader przekazuje normalnie przekształconą pozycję wierzchołka w przestrzeni świata <var>vs_out.FragPos</var> oraz przekształconą pozycję do przestrzeni światła <var>vs_out.FragPosLightSpace</var> do Fragment Shader.

Fragment Shader, którego użyjemy do renderowania sceny, wykorzystuje model oświetlenia Blinna-Phonga. W Fragment Shader obliczamy wartość <var>shadow</var>, która wynosi albo `1.0`, gdy fragment jest w cieniu, albo `0.0`, gdy nie jest w cieniu. Otrzymane kolory <var>diffuse</var> i <var>specular</var> są następnie mnożone przez ten składnik. Ponieważ cienie rzadko są całkowicie ciemne z powodu rozpraszania światła, pozostawiamy kolor <var>ambient</var> w spokoju.

```glsl
    #version 330 core
    out vec4 FragColor;

    in VS_OUT {
        vec3 FragPos;
        vec3 Normal;
        vec2 TexCoords;
        vec4 FragPosLightSpace;
    } fs_in;

    uniform sampler2D diffuseTexture;
    uniform sampler2D shadowMap;

    uniform vec3 lightPos;
    uniform vec3 viewPos;

    float ShadowCalculation(vec4 fragPosLightSpace)
    {
        [...]
    }

    void main()
    {           
        vec3 color = texture(diffuseTexture, fs_in.TexCoords).rgb;
        vec3 normal = normalize(fs_in.Normal);
        vec3 lightColor = vec3(1.0);
        // ambient
        vec3 ambient = 0.15 * color;
        // diffuse
        vec3 lightDir = normalize(lightPos - fs_in.FragPos);
        float diff = max(dot(lightDir, normal), 0.0);
        vec3 diffuse = diff * lightColor;
        // specular
        vec3 viewDir = normalize(viewPos - fs_in.FragPos);
        float spec = 0.0;
        vec3 halfwayDir = normalize(lightDir + viewDir);  
        spec = pow(max(dot(normal, halfwayDir), 0.0), 64.0);
        vec3 specular = spec * lightColor;    
        // oblicz cień
        float shadow = ShadowCalculation(fs_in.FragPosLightSpace);       
        vec3 lighting = (ambient + (1.0 - shadow) * (diffuse + specular)) * color;    

        FragColor = vec4(lighting, 1.0);
    }
```

Fragment Shader jest w dużej mierze kopią tego, co robiliśmy w tutorialu [zaawansowane oświetlenie]({% post_url /learnopengl/5_advanced_lighting/2018-10-01-zaawansowane-oswietlenie %}), ale z dodatkowym obliczeniem cienia. Zadeklarowaliśmy funkcję <fun>ShadowCalculation</fun>, która wykonuje większość pracy związanej z cieniami. Na końcu Fragment Shadera mnożymy komponenty diffuse i specular przez odwrotność komponentu <var>shadow</var> - jak bardzo fragment _nie jest_ w cieniu. Ten Fragment Shader przyjmuje jako dodatkowe wejście położenie fragmentu w przestrzeni światła i mapę głębokości wygenerowaną podczas pierwszego etapu renderowania.

Pierwszą rzeczą do zrobienia w celu sprawdzenia, czy fragment jest w cieniu, jest transformacja położenia fragmentu w przestrzeni światła do przestrzeni NDC. Kiedy wyprowadzamy pozycję wierzchołka do <var>gl_Position</var> w Vertex Shader, OpenGL automatycznie stosuje dzielenie perspektywy - przekształca współrzędne obcinania w zakresie [`-w`,` w`] na [`-1`, `1`] dzieląc składowe `x`, `y` i `z` przez komponent `w`. Jako że pozycja <var>FragPosLightSpace</var> nie jest przekazywany do Fragment Shader przez <var>gl_Position</var>, musimy zrobić dzielenie perspektywy sami:

```glsl
    float ShadowCalculation(vec4 fragPosLightSpace)
    {
        // przeprowadź dzielenie perspektywy
        vec3 projCoords = fragPosLightSpace.xyz / fragPosLightSpace.w;
        [...]
    }
```

Zwraca to pozycję fragmentu w przestrzeni światła w zakresie [`-1`, `1`].

{: .box-note }
Podczas korzystania z ortograficznej macierzy projekcji składnik `w` wierzchołka pozostaje nietknięty, więc ten krok jest właściwie bez znaczenia. Konieczne jest jednak zrobienie tego podczas rzutowania perspektywicznego, więc zostawienie tej linii zapewnia, że algorytm będzie ​​działał z obydwoma macierzami projekcji.

Ponieważ głębokość z mapy głębokości mieści się w zakresie [`0`, `1`] i chcemy również użyć <var>projCoords</var> do próbkowania z mapy głębi, przekształcamy współrzędne NDC do zakresu [`0`, `1`]:

```glsl
    projCoords = projCoords * 0.5 + 0.5; 
```

Tymi zrzutowanymi współrzędnymi możemy próbkować mapę głębokości, ponieważ wynikowe współrzędne [`0`, `1`] z <var>projCoords</var> odpowiadają bezpośrednio przekształconym współrzędnym NDC z pierwszego etapu renderowania. To daje nam najbliższą głębię z punktu widzenia światła:

```glsl
    float closestDepth = texture(shadowMap, projCoords.xy).r;   
```

Aby uzyskać bieżącą głębię w tym fragmencie, pobieramy po prostu współrzędną `z` wektora, która jest równa głębi fragmentu z perspektywy światła.

```glsl
    float currentDepth = projCoords.z;  
```

Porównanie jest po prostu sprawdzeniem, czy <var>currentDepth</var> jest większe niż <var>closestDepth</var>, jeśli tak, to fragment jest w cieniu.

```glsl
    float shadow = currentDepth > closestDepth  ? 1.0 : 0.0;  
```

Pełna funkcja <fun>ShadowCalculation</fun> wygląda tak:

```glsl
    float ShadowCalculation(vec4 fragPosLightSpace)
    {
        // przeprowadź dzielenie perspektywy
        vec3 projCoords = fragPosLightSpace.xyz / fragPosLightSpace.w;
        // przekształcić na zakres [0,1]
        projCoords = projCoords * 0.5 + 0.5;
        // uzyskaj najbliższą wartość głębi z perspektywy światła (używając fragPosLight z zakresu [0,1] jako współrzędnych)
        float closestDepth = texture(shadowMap, projCoords.xy).r; 
        // uzyskaj głębię bieżącego fragmentu z perspektywy światła
        float currentDepth = projCoords.z;
        // sprawdź, czy bieżący fragment jest w cieniu
        float shadow = currentDepth > closestDepth  ? 1.0 : 0.0;

        return shadow;
    }  
```

Po aktywacji tego shadera, powiązaniu odpowiednich tekstur i aktywacja domyślnej macierzy projekcji i widoku w drugim etapie renderowania, powinieneś uzyskać wynik podobny do poniższego:

![Shadow mapping bez ulepszeń.](/img/learnopengl/shadow_mapping_shadows.png){: .center-image }

Jeśli zrobiłeś wszystko dobrze, powinieneś zobaczyć (choć z kilkoma artefaktami) cienie na podłodze i kostce. Możesz znaleźć kod źródłowy aplikacji demonstracyjnej [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/5.advanced_lighting/3.1.2.shadow_mapping_base/shadow_mapping_base.cpp).

## Poprawianie map cieni

Udało nam się uzyskać podstawy shadow mappingu, ale jak widać, wciąż jest kilka artefaktów związanych z tą techniką, które chcielibyśmy poprawić, aby uzyskać lepsze wyniki. Skupimy się na tym w następnych sekcjach.

### Shadow acne

Jest oczywiste, że coś jest nie tak w poprzednim obrazie. Zoom pokazuje nam bardzo oczywisty wzór przypominający wzór Moiré:

![Obraz shadow acne jako wzór Moiré w shadow mappingu](/img/learnopengl/shadow_mapping_acne.png){: .center-image }

Widzimy dużą część podłogi pokrytą czarnymi liniami. Ten artefakt shadow mappingu nazywa się <def>shadow acne</def> i można go wytłumaczyć prostym obrazem:

![Shadow acne](/img/learnopengl/shadow_mapping_acne_diagram.png){: .center-image }

Ponieważ mapa cieni jest ograniczona przez rozdzielczość, wiele fragmentów może próbkować tę samą wartość z mapy głębokości, gdy znajdują się one stosunkowo daleko od źródła światła. Obrazek pokazuje podłogę, gdzie każdy przesunięty panel reprezentuje pojedynczy texel mapy głębi. Jak widać, kilka fragmentów próbkuje tę samą próbkę głębi.

Chociaż ogólnie jest to w porządku, staje się problemem, gdy źródło światła "patrzy" pod kątem w kierunku powierzchni, ponieważ w tym przypadku mapa głębokości jest również renderowana pod kątem. Kilka fragmentów ma wtedy dostęp do tej samej przesuniętej wartości głębokości, podczas gdy niektóre znajdują się powyżej, a niektóre poniżej podłogi; otrzymujemy rozbieżność cienia. Z tego powodu niektóre fragmenty są cieniu, a niektóre nie, dając paski.

Możemy rozwiązać ten problem za pomocą małego triku o nazwie <def>shadow bias</def>, w którym po prostu wyrównujemy głębokość powierzchni (lub mapę cienia) o niewielką wartością odchylenia, tak że fragmenty nie są błędnie rozpatrywane jako te, które znajdują się poniżej powierzchni.

![Shadow mapping, gdzie shadow acne jest naprawione używając shadow bias.](/img/learnopengl/shadow_mapping_acne_bias.png){: .center-image }

Przy zastosowanym biasie. wszystkie próbki mają głębokość mniejszą niż głębokość powierzchni, a zatem cała powierzchnia jest prawidłowo oświetlona bez żadnych pasków cieni. Możemy zaimplementować takie rozwiązanie w następujący sposób:

```glsl
    float bias = 0.005;
    float shadow = currentDepth - bias > closestDepth  ? 1.0 : 0.0;  
```

Shadow bias równy `0.005` rozwiązuje w znacznym stopniu problemy naszej sceny, ale niektóre powierzchnie, które mają stromy kąt względem źródła światła, mogą powodować shadow acne. Bardziej solidnym podejściem byłoby zmienianie wielkości biasu w oparciu o kąt powierzchni w kierunku światła: to jest coś, co możemy rozwiązać za pomocą iloczynu skalarnego:

```glsl
    float bias = max(0.05 * (1.0 - dot(normal, lightDir)), 0.005);  
```

Tutaj mamy maksymalny bias o wartości `0.05` i minimalny `0.005` w oparciu o wektor normalny i kierunek światła. W ten sposób powierzchnie, takie jak podłoga, która jest prawie prostopadła do źródła światła, mają mniejszą wartości biasu, podczas gdy powierzchnie takie jak powierzchnie boczne sześcianu uzyskują znacznie większy bias. Poniższy obrazek pokazuje tę samą scenę, ale teraz z zastosowanym shadow bias:

![Obrazy z shadow mappingiem z zastosowanym (pochyłym) shadow biasem.](/img/learnopengl/shadow_mapping_with_bias.png){: .center-image }

Wybór właściwej wartości biasu wymaga pewnych dostosowań, ponieważ będą one inne w każdej scenie, ale w większości przypadków jest to po prostu kwestia zwiększania biasu, aż do usunięcia całego shadow acne.

### Peter panning

Wadą stosowania shadow biasu jest to, że stosujesz przesunięcie względem rzeczywistej głębokości obiektów. W rezultacie odchylenie może stać się wystarczająco duże, aby zobaczyć widoczne przesunięcie cieni w porównaniu do rzeczywistych lokalizacji obiektów, jak widać poniżej (z wyolbrzymioną wartością odchylenia):

![Peter panning w implementacji shadow mapping](/img/learnopengl/shadow_mapping_peter_panning.png){: .center-image }

Ten artefakt cieni nazywa się <def>peter panning</def>, ponieważ obiekty wydają się być _odłączone_ od ich cieni. Możemy użyć sztuczki, aby rozwiązać większość problemu z peter panningiem, korzystając z funkcji usuwania przednich ścianek podczas renderowania mapy głębokości. Być może pamiętasz z tutoriala [face culling]({% post_url /learnopengl/4_advanced_opengl/2018-08-29-face-culling %}), że OpenGL domyślnie usuwa tylne ścianki.

Ponieważ potrzebujemy tylko wartości głębokości dla mapy głębokości, nie powinno mieć to znaczenia dla zamkniętych obiektów, niezależnie od tego, czy bierzemy głębokość ich przednich ścianek czy tylnych. Używanie głębi tylnej ścianki nie daje błędnych wyników, ponieważ nie ma znaczenia, czy mamy cienie wewnątrz obiektów; i tak nie możemy ich zobaczyć.

![Usuwanie peter panningu przy usuwaniu przednich ścianek](/img/learnopengl/shadow_mapping_culling.png){: .center-image }

Aby w większości naprawić peter panning, usuwamy przednie ścianki. Zauważ, że najpierw musisz włączyć <var>GL_CULL_FACE</var>.

```cpp
    glCullFace(GL_FRONT);
    RenderSceneToDepthMap();
    glCullFace(GL_BACK); // don't forget to reset original culling face
```

To skutecznie rozwiązuje problemy z peter panningiem, ale **tylko dla zamkniętych** obiektów, które nie mają otworów. Na przykład w naszej scenie działa to doskonale na kostkach, ale nie działa na podłodze, ponieważ usunięcie przedniej ścianki całkowicie usuwa podłogę podczas renderowania mapy głębi. Podłoga jest jednopłaszczyznowa i dlatego zostanie całkowicie usunięta. Jeśli ktoś chce rozwiązać problem z peter panningiem, należy zachować ostrożność, aby usuwać tylko przednie powierzchnie obiektów, kiedy ma to sens.

Innym zagadnieniem jest to, że obiekty znajdujące się w pobliżu odbiornika cieni (takie jak daleki sześcian) mogą nadal dawać nieprawidłowe wyniki. Należy zachować ostrożność, aby używać usuwania przednich ścianek na obiektach, na których ma to sens. Jednak przy normalnych wartościach biasu można ogólnie uniknąć artefaktu peter panningu.

### Nadpróbkowanie (ang. *oversampling*)

Kolejną wizualną rozbieżnością, która może ci się spodobać lub nie, jest to, że niektóre regiony poza frustum światła są uważane za bycie w cieniu, podczas gdy nie są. Dzieje się tak dlatego, że rzutowane współrzędne na zewnątrz frustum światła są większe niż `1.0`, a zatem będą próbkować teksturę głębi poza jej domyślnym zakresem [`0, 1`]. W oparciu o metodę zawijania tekstury otrzymamy niepoprawne wyniki głębokości nieoparte na rzeczywistych wartościach głębokości ze źródła światła.

![Shadow mapping z widocznymi krawędziami mapy głębi, zawijanie tekstur](/img/learnopengl/shadow_mapping_outside_frustum.png){: .center-image }

Na obrazie widać, że istnieje jakiś wyimaginowany obszar światła, i spora część poza tym obszarem jest w cieniu; ten obszar reprezentuje rozmiar mapy głębokości rzutowanej na podłogę. Dzieje się tak dlatego, że wcześniej ustawiliśmy opcje zawijania mapy głębokości na <var>GL_REPEAT</var>.

Chcielibyśmy raczej, aby wszystkie współrzędne poza zasięgiem mapy głębokości miały głębokość `1.0`, co w rezultacie oznacza, że ​​te współrzędne nigdy nie będą w cieniu (ponieważ żaden obiekt nie będzie miał większej głębokości niż `1.0`). Możemy to osiągnąć, przechowując kolor obramowania i ustawiając opcje zawijania tekstury mapy głębokości na <var>GL_CLAMP_TO_BORDER</var>:

```cpp
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_BORDER);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_BORDER);
    float borderColor[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);  
```

Teraz za każdym razem, gdy próbkujemy poza zakresem współrzędnych mapy głębokości [`0`, `1`], funkcja <fun>texture</fun> zawsze zwróci głębokość `1.0`, zwracając wartość <var>shadow</var> równą `0.0`. Wynik wygląda teraz znacznie bardziej wiarygodnie:

![Shadow mapping z opcją obcinania do obramowania zawijania tekstury](/img/learnopengl/shadow_mapping_clamp_edge.png){: .center-image }

Wciąż wydaje się, że jedna część wciąż jest w cieniu. Są to współrzędne poza daleką płaszczyzną ortograficznego frustum światła. Widać, że ten ciemny obszar zawsze pojawia się na drugim końcu frustum źródła światła, patrząc na kierunki cienia.

Rzutowana współrzędna jest większa niż daleka płaszczyzna obcinania frustum światła, gdy jej współrzędna `z` jest większa niż `1.0`. W takim przypadku opcja zawijania <var>GL_CLAMP_TO_BORDER</var> nie działa, ponieważ porównujemy składnik `z` współrzędnej z wartościami mapy głębi; to zawsze zwraca wartość `true` dla `z` większego niż `1.0`.

Poprawienie tego jest również stosunkowo proste, ponieważ po prostu wymuszamy wartość <var>shadow</var> ustawioną na `0.0`, gdy współrzędna `z` rzutowanego wektora jest większa niż `1.0`:

```glsl
    float ShadowCalculation(vec4 fragPosLightSpace)
    {
        [...]
        if(projCoords.z > 1.0)
            shadow = 0.0;

        return shadow;
    }  
```

Sprawdzenie dalekiej płaszczyzny i obcinanie wartości mapy głębi na określonym kolorze obramowania rozwiązuje nadmierne próbkowanie mapy głębi i ostatecznie daje nam wynik, którego szukamy:

![Shadow mapping z naprawionym nadpróbkowaniem](/img/learnopengl/shadow_mapping_over_sampling_fixed.png){: .center-image }

Rezultat tego wszystkiego oznacza, że ​​mamy tylko cienie, w których współrzędne rzutowanego fragmentu mieszczą się w zakresie mapy głębokości, więc cokolwiek znajdzie się poza tym zakresem nie będzie miało widocznych cieni. Ponieważ gry zazwyczaj zapewniają, że dzieje się to tylko w odległości, jest to bardziej prawdopodobny efekt niż czarne obszary, które mieliśmy wcześniej.

## PCF

Cienie w tej chwili są miłym dodatkiem do scenerii, ale wciąż nie jest dokładnie tym, czego chcemy. Jeśli przybliżysz cienie, szybko staje się widoczna zależność od rozdzielczości shadow mappingu.

![Poszarpane krawędzie mapy cieni](/img/learnopengl/shadow_mapping_zoom.png){: .center-image }

Ponieważ mapa głębi ma stałą rozdzielczość, głębokość często obejmuje więcej niż jeden fragment na teksel. W rezultacie wiele fragmentów pobiera tę samą wartość głębokości z mapy głębokości, które powodują powstawanie postrzępionych krawędzi.

Możesz zredukować te postrzępione cienie, zwiększając rozdzielczość mapy głębi lub próbując dopasować frustum światła jak najbliżej sceny.

Innym (częściowym) rozwiązaniem tych poszarpanych krawędzi jest PCF lub <def>percentage-closer filtering</def>, które jest zbiorem wielu różnych funkcji filtrowania, które wytwarzają _gładsze_ cienie, co sprawia, że ​​wydają się mniej poszarpane. Chodzi o to, aby próbkować więcej niż jeden raz z mapy głębokości, za każdym razem z nieco innymi współrzędnymi tekstury. Dla każdej pojedynczej próbki sprawdzamy, czy jest w cieniu, czy nie. Wszystkie pod-wyniki są następnie łączone i uśredniane, a my otrzymujemy ładny, miękki cień.

Jedną z prostych implementacji PCF jest po prostu spróbkowanie otaczających tekseli mapy głębi i uśrednienie wyników:

```glsl
    float shadow = 0.0;
    vec2 texelSize = 1.0 / textureSize(shadowMap, 0);
    for(int x = -1; x <= 1; ++x)
    {
        for(int y = -1; y <= 1; ++y)
        {
            float pcfDepth = texture(shadowMap, projCoords.xy + vec2(x, y) * texelSize).r; 
            shadow += currentDepth - bias > pcfDepth ? 1.0 : 0.0;        
        }    
    }
    shadow /= 9.0;
```

Tutaj <fun>textureSize</fun> zwraca `vec2` szerokości i wysokości podanego samplera tekstury na `0` poziomie mipmapy. Odwrotność zwraca rozmiar pojedynczego teksela, którego używamy do przesunięcia współrzędnych tekstury, upewniając się, że każda nowa próbka pobiera inną wartość głębokości. Tutaj próbkujemy 9 wartości wokół wartości przewidywanych współrzędnych `x` i `y`, testujemy zacienienie i ostatecznie uśredniamy wyniki dzieląc przez całkowitą liczbę pobranych próbek.

Poprzez użycie większej liczby próbek i/lub zmianę zmiennej <var>texelSize</var> można zwiększyć jakość cieni. Poniżej możesz zobaczyć cienie z zastosowanym prostym PCF:

![Shadow mapping z PCF](/img/learnopengl/shadow_mapping_soft_shadows.png){: .center-image }

Z daleka cienie wyglądają o wiele lepiej i są mniej twarde. Jeśli przybliżysz obraz, nadal możesz zobaczyć artefakty rozdzielczości shadow mappingu, ale generalnie daje to dobre wyniki dla większości aplikacji.

Możesz znaleźć pełny kod źródłowy przykładu [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/5.advanced_lighting/3.1.3.shadow_mapping/shadow_mapping.cpp).

PCF i wiele innych technik znacznie poprawia jakość miękkich cieni, ale ze względu na długość tego samouczka zostawimy to na późniejszą dyskusję.

## Projekcja prostokątna vs perspektywiczna

Istnieje różnica między renderowaniem mapy głębokości z macierzą prostokątną lub perspektywiczną. Macierz rzutowania prostokątnego nie deformuje sceny z perspektywą, więc wszystkie widoki/promienie światła są równoległe, co czyni ją doskonałą macierzą dla kierunkowych świateł. Macierz rzutowania perspektywicznego deformuje jednak wszystkie wierzchołki w oparciu o perspektywę, która daje różne wyniki. Poniższy obrazek przedstawia różne obszary cieni obu metod projekcji:

![Różnica pomiędzy rzutem prostokątnym i perspektywicznym](/img/learnopengl/shadow_mapping_projection.png){: .center-image }

Projekcja perspektywiczna ma większy sens dla źródeł światła, które mają rzeczywiste położenie w przeciwieństwie do kierunkowych świateł. Projekcja perspektywiczna jest więc najczęściej używane ze światłami punktowymi i reflektorowymi, podczas gdy rzuty prostokątne są używane do kierunkowych świateł.

Inną subtelną różnicą jest to, że wizualizacja bufora głębi często daje prawie całkowicie biały wynik. Dzieje się tak dlatego, że przy rzutowaniu perspektywicznym głębokość jest przekształcana na nieliniowe wartości głębokości z większością zauważalnego zakresu bliskiej płaszczyzny. Aby móc poprawnie wyświetlić wartości głębokości, podobnie jak w przypadku rzutowania prostokątnego, najpierw należy przekształcić nieliniowe wartości głębokości na liniowe, jak to omówiono w tutorialu [test głębokości]({% post_url /learnopengl/4_advanced_opengl/2018-08-22-test-glebokosci %}).

```glsl
    #version 330 core
    out vec4 FragColor;

    in vec2 TexCoords;

    uniform sampler2D depthMap;
    uniform float near_plane;
    uniform float far_plane;

    float LinearizeDepth(float depth)
    {
        float z = depth * 2.0 - 1.0; // Back to NDC 
        return (2.0 * near_plane * far_plane) / (far_plane + near_plane - z * (far_plane - near_plane));
    }

    void main()
    {             
        float depthValue = texture(depthMap, TexCoords).r;
        FragColor = vec4(vec3(LinearizeDepth(depthValue) / far_plane), 1.0); // perspektywa
        // FragColor = vec4(vec3(depthValue), 1.0); // orthographic
    }  
```

Pokazuje to wartości głębokości podobne do tego, co widzieliśmy z rzutowaniem prostokątnym. Zauważ, że jest to użyteczne tylko podczas debugowania; kontrole głębokości pozostają takie same w przypadku macierzy ortograficznych lub perspektywicznych, ponieważ głębokości względne się nie zmieniają.

## Dodatkowe materiały

*   [Tutorial 16 : Shadow mapping](http://www.opengl-tutorial.org/intermediate-tutorials/tutorial-16-shadow-mapping/): podobny tutorial shadow mappingu autorstwa opengl-tutorial.org z kilkoma dodatkowymi notatkami.
*   [Shadow Mapping - Part 1](http://ogldev.atspace.co.uk/www/tutorial23/tutorial23.html): kolejny tutorial do shadow mappingu autorstwa ogldev.
*   [How Shadow Mapping Works](https://www.youtube.com/watch?v=EsccgeUpdsM): 3-częściowy samouczek YouTube autorstwa TheBennyBox na temat mapowania cieni i jego implementacji.
*   [Common Techniques to Improve Shadow Depth Maps](https://msdn.microsoft.com/en-us/library/windows/desktop/ee416324%28v=vs.85%29.aspx): świetny artykuł Microsoftu wymieniający wiele technik poprawiających jakość map cieni.