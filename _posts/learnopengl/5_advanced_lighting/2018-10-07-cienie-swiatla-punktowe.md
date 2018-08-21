---
layout: post
title: Cienie - światła punktowe
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: advanced-lighting-shadows
mathjax: true
---

{% include learnopengl.md link="Advanced-Lighting/Shadows/Point-Shadows" %}

W ostatnim tutorialu nauczyliśmy się tworzyć dynamiczne cienie za pomocą shadow mappingu. Działa to świetnie, ale nadaje się tylko dla kierunkowych świateł, ponieważ cienie są generowane tylko w jednym kierunku źródła światła. Dlatego jest to również znane jako <def>kierunkowe mapowanie cieni</def>, ponieważ mapa głębi (lub cienia) jest generowana tylko z jednego kierunku, z którego "patrzy" światło.

Na czym skupi się ten samouczek, to generowanie dynamicznych cieni we wszystkich otaczających kierunkach. Technika, której używamy, jest idealna dla świateł punktowych, ponieważ prawdziwe światło punktowe rzucałoby cienie we wszystkich kierunkach. Ta technika znana jest jako cienie punktowych świateł lub bardziej formalnie jako <def>omnidirectional shadow maps</def>.

{: .box-note }
Ten samouczek opiera się na poprzednim [tutorialu]({% post_url /learnopengl/5_advanced_lighting/2018-10-05-shadow-mapping %}), więc jeśli nie znasz tradycyjnego shadow mappingu, zaleca się przeczytanie najpierw poprzedniego samouczka.

Algorytm pozostaje w większości taki sam jak dla kierunkowego źródła światła: generujemy mapę głębokości z perspektywy światła, próbkujemy mapę głębokości na podstawie aktualnej pozycji fragmentu i porównujemy każdy fragment z zapisaną wartością głębokości, aby zobaczyć, czy jest on w cieniu. Główną różnicą między mapowaniem cieni dla punktowych źródeł światła a mapowaniem cieni dla kierunkowych świateł jest map głębokości.

Potrzebna nam mapa głębi wymaga renderowania sceny ze wszystkich otaczających kierunków światła punktowego i jako taka normalna mapa głębi 2D nie będzie działać; co jeśli zamiast tego użyjemy [cubemapy]({% post_url /learnopengl/4_advanced_opengl/2018-09-03-cubemaps %})? Ponieważ cubemapa może przechowywać dane środowiskowe z 6 powierzchniami, można wyrenderować całą scenę do każdej z powierzchni mapy w kształcie sześcianu i próbkować ją jako otaczające wartości głębokości światła punktowego.

![Obraz działania shadow mappingu dla punktowych świateł](/img/learnopengl/point_shadows_diagram.png){: .center-image }

Wygenerowana cubemapa głębokości jest następnie przekazywana do shadera oświetlenia, który pobiera próbkę cubemapy za pomocą wektora kierunkowego, aby pobrać głębię (z perspektywy światła) dla tego fragmentu. Większość skomplikowanych rzeczy omówiliśmy już w poprzednim samouczku o shadow mappingu. Tym, co czyni ten algorytm nieco trudniejszym, jest generowanie cubemapy głębokości.

## Generowanie cubemapy głęobokości

Aby utworzyć cubemapę głębokości otoczenia światła, musimy renderować scenę 6 razy: raz dla każdej ścianki. Jednym (dość oczywistym) sposobem na zrobienie tego jest renderowanie sceny 6 razy z 6 różnymi macierzami widoku, za każdym razem dołączając inną ściankę do obiektu bufora ramki. Wyglądałoby to mniej więcej tak:

```cpp
    for(unsigned int i = 0; i < 6; i++)
    {
        GLenum face = GL_TEXTURE_CUBE_MAP_POSITIVE_X + i;
        glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, face, depthCubemap, 0);
        BindViewMatrix(lightViewMatrices[i]);
        RenderScene();  
    }
```

Może to być dość kosztowne, ponieważ wiele wywołań renderowania jest potrzebnych tylko dla jednej mapy głębokości. W tym samouczku zastosujemy alternatywne (bardziej zorganizowane) podejście, wykorzystując małą sztuczkę w Geometry Shader, która pozwala nam zbudować cubemapę głębokości podczas jednego wywołania rysowania (ang. *draw call*).

Najpierw musimy utworzyć cubemapę:

```cpp
    unsigned int depthCubemap;
    glGenTextures(1, &depthCubemap);
```

I wygenerować każdą z pojedynczych ścianek cubemapy jako obrazy głębokości 2D:

```cpp
    const unsigned int SHADOW_WIDTH = 1024, SHADOW_HEIGHT = 1024;
    glBindTexture(GL_TEXTURE_CUBE_MAP, depthCubemap);
    for (unsigned int i = 0; i < 6; ++i)
            glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_DEPTH_COMPONENT, 
                         SHADOW_WIDTH, SHADOW_HEIGHT, 0, GL_DEPTH_COMPONENT, GL_FLOAT, NULL);  
```

Nie zapomnij również ustawić odpowiednich parametrów tekstury:

```cpp
    glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
    glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
    glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);  
```

Normalnie do obiektu bufora ramki zostałaby dołączona pojedyncza ścianka cubemapy jako tekstura dla framebuffer'a i 6-krotnie wyrenderowana scena, za każdym razem przełączając bufor głębi bufora ramki na inną ściankę cubemapy. Ponieważ zamierzamy użyć Geometry Shadera, który pozwala nam renderować do wszystkich ścianek na raz w jednym przebiegu renderowania, możemy bezpośrednio dołączyć cubemapę jako załącznik głębokości bufora ramki za pomocą <fun>glFramebufferTexture</fun>:

```cpp
    glBindFramebuffer(GL_FRAMEBUFFER, depthMapFBO);
    glFramebufferTexture(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, depthCubemap, 0);
    glDrawBuffer(GL_NONE);
    glReadBuffer(GL_NONE);
    glBindFramebuffer(GL_FRAMEBUFFER, 0);  
```

Ponownie wywołujemy <fun>glDrawBuffer</fun> i <fun>glReadBuffer</fun>: dbamy tylko o wartości głębi podczas generowania cubemapy głębokości, więc musimy jawnie powiedzieć OpenGL, że obiekt bufora ramki nie renderuje do bufor koloru.

Przy mapach cieni świateł punktowych mamy dwa przebiegi renderowania: najpierw generujemy mapę głębi, a potem używamy tej mapy głębokości, aby tworzyć cienie w scenie. W przypadku obiektu framebuffer i cubemapy ten proces wygląda trochę tak:

```
    // 1. wygeneruj mapę głębokości
    glViewport(0, 0, SHADOW_WIDTH, SHADOW_HEIGHT);
    glBindFramebuffer(GL_FRAMEBUFFER, depthMapFBO);
        glClear(GL_DEPTH_BUFFER_BIT);
        ConfigureShaderAndMatrices();
        RenderScene();
    glBindFramebuffer(GL_FRAMEBUFFER, 0);
    // 2. normalnie wyrenderuj scenę korzystając z mapy głębokości (cubemap)
    glViewport(0, 0, SCR_WIDTH, SCR_HEIGHT);
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    ConfigureShaderAndMatrices();
    glBindTexture(GL_TEXTURE_CUBE_MAP, depthCubemap);
    RenderScene();
```

Proces jest dokładnie taki sam, jak w przypadku wcześniejszego shadow mappingu, chociaż tym razem renderujemy i używamy cubemap głębokości w porównaniu do tekstury głębi 2D. Zanim wyrenderujemy scenę ze wszystkich kierunków światła, musimy najpierw obliczyć odpowiednie macierze transformacji.

### Transformacje przestrzeni światła

Po ustawieniu framebuffera i cubemapy potrzebujemy jakiegoś sposobu, aby przekształcić całą geometrię sceny w odpowiednie przestrzenie światła we wszystkich 6 kierunkach światła. Podobnie do tutoriala [shadow mapping]({% post_url /learnopengl/5_advanced_lighting/2018-10-05-shadow-mapping %}) potrzebujemy macierzy transformacji światła $T$, ale tym razem po jednej na każdą ściankę cubemapy.

Każda macierz transformacji przestrzeni światła zawiera zarówno macierz projekcji, jak i widoku. Do macierzy projekcji wykorzystamy macierz rzutowania perspektywicznego; źródło światła reprezentuje punkt w przestrzeni, więc rzut perspektywiczny ma największy sens. Każda macierz transformacji przestrzeni światła używa tej samej macierzy projekcji:

```cpp
    float aspect = (float)SHADOW_WIDTH/(float)SHADOW_HEIGHT;
    float near = 1.0f;
    float far = 25.0f;
    glm::mat4 shadowProj = glm::perspective(glm::radians(90.0f), aspect, near, far); 
```

Należy tu zwrócić uwagę na parametr pola widzenia <fun>glm::perspective</fun>, który ustawiliśmy na 90 stopni. Ustawiając to na 90 stopni upewniamy się, że pole widzenia jest wystarczająco duże, aby prawidłowo wypełnić pojedynczą powierzchnię cubemapy tak, że wszystkie ściany są prawidłowo wyrównane na krawędziach.

Ponieważ macierz projekcji nie zmienia się w zależności od kierunku, możemy ją ponownie wykorzystać dla każdej z 6 macierzy transformacji. Potrzebujemy innej macierzy widoku na każdy kierunek. Za pomocą <fun>glm::lookAt</fun> tworzymy 6 macierzy widoku, gdzie każdy patrzy na inną ściankę cubemapy w kolejności: prawo, lewo, góra, dół, przód i tył.

```cpp
    std::vector<glm::mat4> shadowTransforms;
    shadowTransforms.push_back(shadowProj * 
                     glm::lookAt(lightPos, lightPos + glm::vec3( 1.0, 0.0, 0.0), glm::vec3(0.0,-1.0, 0.0));
    shadowTransforms.push_back(shadowProj * 
                     glm::lookAt(lightPos, lightPos + glm::vec3(-1.0, 0.0, 0.0), glm::vec3(0.0,-1.0, 0.0));
    shadowTransforms.push_back(shadowProj * 
                     glm::lookAt(lightPos, lightPos + glm::vec3( 0.0, 1.0, 0.0), glm::vec3(0.0, 0.0, 1.0));
    shadowTransforms.push_back(shadowProj * 
                     glm::lookAt(lightPos, lightPos + glm::vec3( 0.0,-1.0, 0.0), glm::vec3(0.0, 0.0,-1.0));
    shadowTransforms.push_back(shadowProj * 
                     glm::lookAt(lightPos, lightPos + glm::vec3( 0.0, 0.0, 1.0), glm::vec3(0.0,-1.0, 0.0));
    shadowTransforms.push_back(shadowProj * 
                     glm::lookAt(lightPos, lightPos + glm::vec3( 0.0, 0.0,-1.0), glm::vec3(0.0,-1.0, 0.0));
```

Tutaj tworzymy 6 macierzy widoków i mnożymy je za pomocą macierzy projekcji, aby uzyskać w sumie 6 różnych macierzy transformacji przestrzeni światła. Parametr `target` <fun>glm::lookAt</fun> skierowany jest w kierunku każdej pojedynczej ścianki cubemapy.

Te macierze transformacji są wysyłane do shaderów, które renderują cubemapę głębokości.

### Shadery głębi

Aby wyrenderować wartości głębokości do cubemapy głębokości, potrzebujemy w sumie trzech shaderów: Vertex Shadera i Fragment Shadera oraz [Geometry Shadera]({% post_url /learnopengl/4_advanced_opengl/2018-09-10-geometry-shader %}).

Geometry Shader będzie shaderem odpowiedzialnym za przekształcanie wszystkich wierzchołków przestrzeni świata do 6 różnych przestrzeni światła. Dlatego Vertex Shader po prostu przekształca wierzchołki do przestrzeni świata i kieruje je do Geometry Shadera:

```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;

    uniform mat4 model;

    void main()
    {
        gl_Position = model * vec4(aPos, 1.0);
    }  
```

Następnie Geometry Shader przyjmuje jako dane wejściowe 3 wierzchołki trójkąta i tablicę uniform macierzy transformacji przestrzeni światła. Następnie Geometry Shader przekształca wierzchołki w przestrzenie światła; tutaj robi się interesująco.

Geometry Shader ma wbudowaną zmienną o nazwie <var>gl_Layer</var>, która określa, do której ścianki cubemapy ma zostać wyemitowany prymityw. W sam sobie Geometry Shader przesyła swoje prymitywy w dół potoku renderowania, ale kiedy aktualizujemy tę zmienną, możemy kontrolować, do której ścianki cubemapy wykonujemy renderowanie dla każdego prymitywu. To oczywiście działa tylko wtedy, gdy do aktywnego framebuffera dołączona jest tekstura z cubemapą.

```glsl
    #version 330 core
    layout (triangles) in;
    layout (triangle_strip, max_vertices=18) out;

    uniform mat4 shadowMatrices[6];

    out vec4 FragPos; // FragPos from GS (output per emitvertex)

    void main()
    {
        for(int face = 0; face < 6; ++face)
        {
            gl_Layer = face; // built-in variable that specifies to which face we render.
            for(int i = 0; i < 3; ++i) // for each triangle's vertices
            {
                FragPos = gl_in[i].gl_Position;
                gl_Position = shadowMatrices[face] * FragPos;
                EmitVertex();
            }    
            EndPrimitive();
        }
    }  
```

Ten Geometry Shader powinien być stosunkowo prosty. Jako dane wejściowe przyjmujemy trójkąt i wyprowadzamy w sumie 6 trójkątów (6 * 3 wierzchołki, co równa się 18 wierzchołkom). W funkcji <fun>main</fun> wykonujemy iteracje na 6 ściakach cubemapy, gdzie określamy każdą ściankę jako powierzchnię wyjściową, zapisując liczbę całkowitą powierzchni w <var>gl_Layer</var>. Następnie generujemy każdy trójkąt, przekształcając każdy wierzchołek z przestrzeni świata do odpowiedniej przestrzeni światła, mnożąc <var>FragPos</var> z macierzą transformacji światła danej ścianki cubemapy. Zauważ, że wysłaliśmy również wynikową zmienną <var>FragPos</var> do Fragment Shadera, którego będziemy potrzebować do obliczenia wartości głębokości.

W ostatnim tutorialu użyliśmy pustego Fragment Shadera i pozwoliliśmy OpenGL określić wartości głębokości mapy głębi. Tym razem będziemy obliczać naszą własną (liniową) głębokość jako odległość liniową między pozycją każdego fragmentu a pozycją źródła światła. Obliczanie własnych wartości głębokości sprawia, że ​​późniejsze obliczenia cieni są nieco bardziej intuicyjne.

```glsl
    #version 330 core
    in vec4 FragPos;

    uniform vec3 lightPos;
    uniform float far_plane;

    void main()
    {
        // get distance between fragment and light source
        float lightDistance = length(FragPos.xyz - lightPos);

        // map to [0;1] range by dividing by far_plane
        lightDistance = lightDistance / far_plane;

        // write this as modified depth
        gl_FragDepth = lightDistance;
    }  
```

Fragment Shader przyjmuje jako dane wejściowe <var>FragPos</var> z Geometry Shadera, wektora położenia światła i wartość dalekiej płaszczyzny frustum. Pobieramy odległość między fragmentem a źródłem światła, mapujemy je do zakresu [`0`, `1`] i zapisujemy jako wartość głębi fragmentu.

Renderowanie sceny za pomocą tych shaderów i aktywnego obiektu framebuffera, do którego podłączona jest cubemapa, powinno dać ci całkowicie wypełnioną mapę głębokości dla obliczeń cieni.

## Mapy cieni świateł punktowych

Po ustawieniu wszystkiego nadszedł czas, aby wyrenderować rzeczywiste cienie świateł punktowych. Procedura jest podobna do tej z samouczka o shadow mappingu dla świateł kierunkowych, chociaż tym razem wiążemy teksturę cubemapy zamiast tekstury 2D jako mapy głębi, a także przekazujemy dalszą płaszczyznę macierzy projekcji światła do shaderów.

```cpp
    glViewport(0, 0, SCR_WIDTH, SCR_HEIGHT);
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    shader.use();  
    // ... send uniforms to shader (including light's far_plane value)
    glActiveTexture(GL_TEXTURE0);
    glBindTexture(GL_TEXTURE_CUBE_MAP, depthCubemap);
    // ... bind other textures
    RenderScene();
```

Tutaj funkcja <fun>renderScene</fun> renderuje kostki w dużym pokoju, które są rozproszonee wokół źródła światła znajdującego się na środku sceny.

Vertex Shader i Fragment Shader są w dużej mierze podobne do oryginalnych shaderów shadow mappingu: różnice polegają na tym, że Fragment Shader nie wymaga już położenia fragmentu w przestrzeni światła, ponieważ możemy teraz próbkować wartości głębokości za pomocą wektora kierunkowego.

Z tego powodu Vertex Shader nie musi już przekształcać swoich wektorów pozycji do przestrzeni światła, abyśmy mogli wykluczyć zmienną <var>FragPosLightSpace</var>:

```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aNormal;
    layout (location = 2) in vec2 aTexCoords;

    out vec2 TexCoords;

    out VS_OUT {
        vec3 FragPos;
        vec3 Normal;
        vec2 TexCoords;
    } vs_out;

    uniform mat4 projection;
    uniform mat4 view;
    uniform mat4 model;

    void main()
    {
        vs_out.FragPos = vec3(model * vec4(aPos, 1.0));
        vs_out.Normal = transpose(inverse(mat3(model))) * aNormal;
        vs_out.TexCoords = aTexCoords;
        gl_Position = projection * view * model * vec4(aPos, 1.0);
    }  
```

Kod cieniowania Blinna-Phonga Fragment Shadera jest dokładnie taki sam jak wcześniej z mnożeniem cieni na końcu:

```glsl
    #version 330 core
    out vec4 FragColor;

    in VS_OUT {
        vec3 FragPos;
        vec3 Normal;
        vec2 TexCoords;
    } fs_in;

    uniform sampler2D diffuseTexture;
    uniform samplerCube depthMap;

    uniform vec3 lightPos;
    uniform vec3 viewPos;

    uniform float far_plane;

    float ShadowCalculation(vec3 fragPos)
    {
        [...]
    }

    void main()
    {           
        vec3 color = texture(diffuseTexture, fs_in.TexCoords).rgb;
        vec3 normal = normalize(fs_in.Normal);
        vec3 lightColor = vec3(0.3);
        // ambient
        vec3 ambient = 0.3 * color;
        // diffuse
        vec3 lightDir = normalize(lightPos - fs_in.FragPos);
        float diff = max(dot(lightDir, normal), 0.0);
        vec3 diffuse = diff * lightColor;
        // specular
        vec3 viewDir = normalize(viewPos - fs_in.FragPos);
        vec3 reflectDir = reflect(-lightDir, normal);
        float spec = 0.0;
        vec3 halfwayDir = normalize(lightDir + viewDir);  
        spec = pow(max(dot(normal, halfwayDir), 0.0), 64.0);
        vec3 specular = spec * lightColor;    
        // calculate shadow
        float shadow = ShadowCalculation(fs_in.FragPos);                      
        vec3 lighting = (ambient + (1.0 - shadow) * (diffuse + specular)) * color;    

        FragColor = vec4(lighting, 1.0);
    }  
```

Istnieje kilka subtelnych różnic: kod oświetlenia jest taki sam, ale mamy teraz uniform `samplerCube`, a funkcja <fun>ShadowCalculation</fun> przyjmuje położenie fragmentu jako parametr zamiast pozycji fragmentu w przestrzeni światła. Teraz dodajemy również zmienną <var>far_plane</var> frustum światła, którą będziemy później potrzebować. Na końcu Fragment Shadera obliczamy element cienia, który jest `1.0`, gdy fragment jest w cieniu lub `0.0`, gdy nie jest. Używamy obliczonego składnika cienia, aby wpłynąć na rozproszone i lustrzane elementy oświetlenia.

W dużym stopniu różni się zawartość funkcji <fun>ShadowCalculation</fun>, która teraz pobiera wartości głębi z cubemapy zamiast tekstury 2D. Omówmy jej zawartość krok po kroku.

Pierwszą rzeczą, którą musimy zrobić, to pobrać głębokość z cubemapy. Jak możesz sobie przypomnieć z części tutoriala o cubemapach, to zapisaliśmy głębokość jako liniową odległość między fragmentem a pozycją światła; podejmiemy podobne podejście:

```glsl
    float ShadowCalculation(vec3 fragPos)
    {
        vec3 fragToLight = fragPos - lightPos; 
        float closestDepth = texture(depthMap, fragToLight).r;
    }  
```

Tutaj bierzemy różnicę między pozycją fragmentu a pozycją światła i wykorzystujemy ten wektor jako wektor kierunkowy do próbkowania cubemapy. Wektor kierunkowy nie musi być wektorem jednostkowym, aby pobierać próbki z cubemapy, więc nie ma potrzeby normalizowania go. Wynikowa wartość <var>closestDepth</var> jest znormalizowaną wartością głębokości między źródłem światła i jego najbliższym widocznym fragmentem.

Wartość <var>closestDepth</var> jest obecnie w zakresie [`0`, `1`], więc najpierw przekształcamy ją z powrotem do zakresu [`0`, `far_plane`] przez pomnożenie jej przez <var>far_plane</var>.

```glsl
    closestDepth *= far_plane;  
```

Następnie pobieramy wartość głębokości między bieżącym fragmentem a źródłem światła, które możemy łatwo uzyskać, pobierając długość <var>fragToLight</var> z dzięki obliczeniu wartości głębokości w cubemapie:

```glsl
    float currentDepth = length(fragToLight);  
```

Zwraca to wartość głębokości w tym samym (lub większym) zakresie, co <var>closestDepth</var>.

Teraz możemy porównać obie wartości głębokości, aby zobaczyć, która jest bliżej i określić, czy bieżący fragment jest w cieniu. Uwzględniamy również bias cienia, więc nie dostaniemy artefaktu shadow acne, co omówiono w poprzednim [samouczku]({% post_url /learnopengl/5_advanced_lighting/2018-10-05-shadow-mapping %}).

```glsl
    float bias = 0.05; 
    float shadow = currentDepth -  bias > closestDepth ? 1.0 : 0.0; 
```

Pełna funkcja <fun>ShadowCalculation</fun> wygląda następująco:

```glsl
    float ShadowCalculation(vec3 fragPos)
    {
        // get vector between fragment position and light position
        vec3 fragToLight = fragPos - lightPos;
        // use the light to fragment vector to sample from the depth map    
        float closestDepth = texture(depthMap, fragToLight).r;
        // it is currently in linear range between [0,1]. Re-transform back to original value
        closestDepth *= far_plane;
        // now get current linear depth as the length between the fragment and light position
        float currentDepth = length(fragToLight);
        // now test for shadows
        float bias = 0.05; 
        float shadow = currentDepth -  bias > closestDepth ? 1.0 : 0.0;

        return shadow;
    }  
```

Dzięki tym shaderom możemy uzyskać całkiem dobre cienie i tym razem we wszystkich kierunkach punktowego źródła światła. Scena ze światłem punktowym umieszczonym pośrodku prostej sceny będzie wyglądało mniej więcej tak:

![Mapy cieni punktowych źródeł światła w OpenGL](/img/learnopengl/point_shadows.png){: .center-image }

Możesz znaleźć kod źródłowy tego demo [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/5.advanced_lighting/3.2.1.point_shadows/point_shadows.cpp).

### Wizualizacja bufora głębi cubemapy

Jeśli jesteś trochę podobny do mnie, prawdopodobnie nie uzyskałeś powyższego efektu za pierwszym razem, więc warto przeprowadzić pewne debugowanie za pomocą jednego sprawdzenia, czy mapa głębi została poprawnie zbudowana. Ponieważ nie mamy już tekstury mapy głębi 2D, wizualizacja mapy głębi staje się nieco mniej oczywista.

Prostą sztuczką do wizualizacji bufora głębi jest wzięcie znormalizowanej wartości (w zakresie [`0`, `1`]) zmiennej <var>closestDepth</var> w funkcji <fun>ShadowCalculation</fun> i wyświetlenie jej jako:

```glsl
    FragColor = vec4(vec3(closestDepth / far_plane), 1.0);  
```

Rezultatem jest wyszarzona scena, w której każdy kolor reprezentuje liniowe wartości głębokości sceny:

![Zwizualizowana mapa głębokości](/img/learnopengl/point_shadows_depth_cubemap.png){: .center-image }

Na zewnętrznej ścianie można również zobaczyć obszary, które mają być zacienione. Jeśli Twoja scena wygląda nieco podobnie, to wiesz, że głębokość cubemapy została poprawnie wygenerowana. W przeciwnym razie prawdopodobnie zrobiłeś coś złego lub użyłeś <var>closestDepth</var> w zakresie [`0`, `far_plane`].

## PCF

Ponieważ mapy cieni świateł punktowych oparte są na tych samych zasadach co tradycyjny shadow mapping, ma również te same artefakty zależne od rozdzielczości. Jeśli przybliżysz się do cieni, ponownie zobaczysz postrzępione krawędzie. <def>Percentage-closer filtering</def> lub PCF pozwala nam wygładzać te postrzępione krawędzie, filtrując wiele próbek wokół pozycji fragmentu i uśredniając wyniki.

Jeśli weźmiemy ten sam prosty filtr PCF z poprzedniego samouczka i dodamy trzeci wymiar (ponieważ potrzebujemy wektorów kierunkowych 3D do próbkowania z cubemapy) otrzymamy:

```glsl
    float shadow  = 0.0;
    float bias    = 0.05; 
    float samples = 4.0;
    float offset  = 0.1;
    for(float x = -offset; x < offset; x += offset / (samples * 0.5))
    {
        for(float y = -offset; y < offset; y += offset / (samples * 0.5))
        {
            for(float z = -offset; z < offset; z += offset / (samples * 0.5))
            {
                float closestDepth = texture(depthMap, fragToLight + vec3(x, y, z)).r; 
                closestDepth *= far_plane;   // Undo mapping [0;1]
                if(currentDepth - bias > closestDepth)
                    shadow += 1.0;
            }
        }
    }
    shadow /= (samples * samples * samples);
```

Kod nie różni się zbytnio od tego, co mieliśmy w tradycyjnym shadow mappingu. Tutaj obliczamy przesunięcia tekstur w sposób dynamiczny na podstawie liczby próbek, które chcielibyśmy zastosować w każdej osi i bierzemy 3 razy więcej <var>samples</var> ilość podpróbek, które następnie uśredniamy na końcu.

Cienie wyglądają teraz o wiele bardziej miękko i gładko i dają o wiele bardziej wiarygodne wyniki.

![PCF dla map cieni świateł punktowych](/img/learnopengl/point_shadows_soft.png){: .center-image }

Jednak przy <var>samples</var> ustawionym na `4.0` pobieramy w sumie `64` próbek z każdego fragmentu, co jest dużą ilością!

Ponieważ większość z tych próbek jest zbędna, ponieważ próbkują one blisko oryginalnego wektora kierunku, może być bardziej sensowne, aby próbkować tylko w prostopadłych kierunkach wektora kierunku próbkowania. Ponieważ jednak nie ma (łatwego) sposobu ustalenia, które pod-kierunki są zbędne, staje się to trudne. Jedną z sztuczek, którą możemy zastosować, jest wyznaczenie szeregu kierunków przesunięcia, które można z grubsza oddzielić, np. każdy z nich wskazuje w zupełnie innym kierunku, zmniejszając liczbę pod-kierunków, które są blisko siebie. Poniżej mamy tablicę maksymalnie `20` kierunków przesunięcia:

```glsl
    vec3 sampleOffsetDirections[20] = vec3[]
    (
       vec3( 1,  1,  1), vec3( 1, -1,  1), vec3(-1, -1,  1), vec3(-1,  1,  1), 
       vec3( 1,  1, -1), vec3( 1, -1, -1), vec3(-1, -1, -1), vec3(-1,  1, -1),
       vec3( 1,  1,  0), vec3( 1, -1,  0), vec3(-1, -1,  0), vec3(-1,  1,  0),
       vec3( 1,  0,  1), vec3(-1,  0,  1), vec3( 1,  0, -1), vec3(-1,  0, -1),
       vec3( 0,  1,  1), vec3( 0, -1,  1), vec3( 0, -1, -1), vec3( 0,  1, -1)
    );   
```

Następnie możemy zmodyfikować algorytm PCF do pobrania ustalonej ilości próbek z <var>sampleOffsetDirections</var> i użyć ich do spróbkowania cubemapy. Zaletą jest to, że potrzebujemy dużo mniej próbek, aby uzyskać wizualnie podobne wyniki do pierwszego algorytmu PCF.

```glsl
    float shadow = 0.0;
    float bias   = 0.15;
    int samples  = 20;
    float viewDistance = length(viewPos - fragPos);
    float diskRadius = 0.05;
    for(int i = 0; i < samples; ++i)
    {
        float closestDepth = texture(depthMap, fragToLight + sampleOffsetDirections[i] * diskRadius).r;
        closestDepth *= far_plane;   // Undo mapping [0;1]
        if(currentDepth - bias > closestDepth)
            shadow += 1.0;
    }
    shadow /= float(samples);  
```

Tutaj dodajemy przesunięcia do określonego <var>diskRadius</var> wokół oryginalnego wektora kierunkowego <var>fragToLight</var>, aby pobrać próbkę z cubemapy.

Inną ciekawą sztuczką, którą możemy tutaj zastosować jest to, że możemy zmienić <var>diskRadius</var> na podstawie odległości od fragmentu; w ten sposób możemy zwiększyć promień przesunięcia o odległość do widza, co powoduje, że cienie stają się bardziej miękkie w oddali i ostrzejsze w pobliżu.

```glsl
    float diskRadius = (1.0 + (viewDistance / far_plane)) / 25.0;  
```

Wyniki tego algorytmu PCF dają równie dobre, jeśli nie lepsze, wyniki miękkich cieni:

![Bardziej wydajny algorytm PCF](/img/learnopengl/point_shadows_soft_better.png){: .center-image }

Oczywiście, <var>bias</var>, który dodajemy do każdej próbki, jest wysoce oparty na kontekście i zawsze będzie wymagał ulepszenia w zależności od rodzaju sceny, z którą pracujesz. Pobaw się z wszystkimi wartościami i zobacz, jak wpływają one na scenę.

Finalny kod możesz znaleźć [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/5.advanced_lighting/3.2.2.point_shadows_soft/point_shadows_soft.cpp).

Powinienem wspomnieć, że używanie shaderów geometrii do generowania mapy głębi nie zawsze jest szybsze niż renderowanie sceny 6 razy dla każdej ścianki. Używanie takiego shadera geometrii ma własne kary wydajności, które mogą przewyższać wzrost wydajności korzystania z jednego z nich. Zależy to oczywiście od rodzaju środowiska, konkretnych sterowników karty graficznej itp., Jeśli naprawdę zależy Ci na wydajności, upewnij się, że profilujesz obie metody i wybierasz bardziej wydajną dla swojej sceny. Osobiście wolę używanie Geometry Shaderów do mapowania cieni, ponieważ uważam je za bardziej intuicyjne w użyciu.

## Dodatkowe materiały

*   [Shadow Mapping for point light sources in OpenGL](http://www.sunandblackcat.com/tipFullView.php?l=eng&topicid=36):  tutorial o mapach cieni świateł punktowych autorstwa sunandblackcat.
*   [Multipass Shadow Mapping With Point Lights](http://ogldev.atspace.co.uk/www/tutorial43/tutorial43.html): tutorial o mapach cieni świateł punktowych autorstwa ogldev.
*   [Omni-directional Shadows](http://www.cg.tuwien.ac.at/~husky/RTR/OmnidirShadows-whyCaps.pdf): zestaw slajdów o mapach cieni świateł punktowych autorstwa Petera Houski.