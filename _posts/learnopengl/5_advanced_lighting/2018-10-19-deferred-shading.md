---
layout: post
title: Deferred shading
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: advanced-lighting
mathjax: true
---

{% include learnopengl.md link="Advanced-Lighting/Deferred-Shading" %}

Sposób, w jaki dotychczas wykonywaliśmy oświetlenie, nazywa się <def>forward renderingiem/shadingiem</def> (renderowanie/cieniowanie naprzód), jest to proste podejście, w którym renderujemy obiekt, oświetlamy go zgodnie z ustawieniami wszystkich źródeł światła w scenie, a następnie renderujemy następny obiekt i tak dalej dla każdego obiektu w scenie. Choć jest to dość łatwe do zrozumienia i implementacji, to jest również dość ciężkie obliczeniowo, ponieważ każdy renderowany obiekt musi powtarzać obliczenia każdego źródła światła dla każdego renderowanego fragmentu, co jest dużym obciążeniem! Forward rendering powoduje również marnowanie wielu inwokacji Fragment Shadera w scenach o dużej złożoności (wiele obiektów obejmuje ten sam piksel na ekranie), ponieważ większość wyników Fragment Shadera jest nadpisywanych.

<span class ="def">Deferred rendering/shading</span> (odroczony rendering/cieniowanie) próbuje przezwyciężyć te problemy i drastycznie zmienia sposób renderowania obiektów. Daje nam to kilka nowych opcji, które znacznie optymalizują sceny z dużą liczbą świateł, co pozwala nam renderować setki, a nawet tysiące świateł z akceptowalną częstotliwością. Poniżej znajduje się zdjęcie sceny z 1847 punktowymi światłami wyrenderowane za pomocą deferred shadingu (zdjęcie dzięki uprzejmości Hannesa Nevalainena); coś, co nie byłoby możliwe przy forward renderingu.

![Przykład mocy deferred shadingu w OpenGL, ponieważ możemy z łatwością renderować 1000 świateł z akceptowalną szybkością klatek na sekundę](/img/learnopengl/deferred_example.png){: .center-image }

Deferred shading opiera się na pomyśle, że my _odraczamy_ lub _opóźniamy_ większość czasochłonnych obliczeń (jak oświetlenie) do późniejszego etapu. Deferred shading składa się z dwóch przebiegów: w pierwszym przebiegu zwanym <def>przejściem geometrii </def> renderujemy scenę raz i pobieramy wszelkiego rodzaju informacje geometryczne o obiektach, które przechowujemy w zbiorze tekstur zwanych <def>G-buffer</def>; pomyśl o wektorach pozycji, wektorach kolorów, wektorach normalnych i/lub wartościach specular. Informacje geometryczne sceny zapisanej w <def>G-buffer</def> są następnie wykorzystywane do (bardziej złożonych) obliczeń oświetlenia. Poniżej znajduje się zawartość G-buffera pojedynczej klatki:

![Przykład G-buffer wypełnionego danymi geometrycznymi sceny w OpenGL](/img/learnopengl/deferred_g_buffer.png){: .center-image }

Korzystamy z tekstur G-buffera w drugim przebiegu zwanym <def>przejściem oświetlenia</def>, w którym renderijemy ekran wypełniony kwadratem i obliczamy oświetlenie sceny dla każdego fragmentu przy użyciu informacji geometrycznych przechowywanych w G-buffer; piksel po pikselu iterujemy po G-buffer. Zamiast przenosić każdy obiekt z Vertex Shadera do Fragment Shadera, odraczamy zaawansowane procesy FS na późniejszy etap. Obliczenia oświetlenia pozostają dokładnie takie same, jak robiliśmy do tej pory, ale tym razem pobierzemy wszystkie wymagane zmienne wejściowe z odpowiednich tekstur G-buffer zamiast z Vertex Shadera (plus niektóre zmienne uniform).

Poniższy obrazek dobrze ilustruje całkowity proces odroczonego cieniowania.

![Omówienie techniki odroczonego cieniowania w OpenGL](/img/learnopengl/deferred_overview.png){: .center-image }

Główną zaletą tego podejścia jest to, że jakikolwiek fragment, który znajdzie się w G-buffer, to faktyczna informacja o fragmencie, która znajdzie się na ekranie, ponieważ test głębi już zakończył przetwarzanie wszystkich fragmentów. Gwarantuje to, że dla każdego piksela przetwarzanego w przejściu oświetlenia robimy to tylko raz; oszczędzając nam wielu niewykorzystanych wywołań renderujących. Co więcej, odroczone renderowanie otwiera możliwości dalszej optymalizacji, która pozwala nam renderować znacznie większą ilość źródeł światła, niż moglibyśmy narysować z forward renderingiem.

Ta technika ma także kilka wad, ponieważ G-buffer wymaga od nas przechowywania względnie dużej ilości danych sceny w swoich buforach kolorów tekstury, które zjadają pamięć, zwłaszcza, że ​​dane sceny, takie jak wektory pozycji, wymagają dużej precyzji. Inną wadą jest to, że nie obsługuje blendingu (ponieważ mamy tylko informację o najbardziej widocznym fragmencie), a MSAA już nie działa. Istnieje kilka sposobów obejścia tych niedogodności, które omówimy na końcu samouczka.

Wypełnianie G-buffera w przejściu geometrii jest dość wydajne, ponieważ bezpośrednio przechowujemy informacje o obiekcie, takie jak położenie, kolor lub wektor normalny w buforze ramki z małą lub zerową ilością przetwarzania. Korzystając również z <def>Multiple Render Targets</def> (MRT), możemy to zrobić nawet w jednym przejściu renderowania (ang. *render pass*).

## G-buffer

<span class="def">G-buffer</span> to zbiorcze określenie wszystkich tekstur używanych do przechowywania istotnych dla oświetlenia danych dla ostatecznego przejścia oświetlenia. Przejrzyjmy wszystkie dane potrzebne do oświetlenia fragmentu za pomocą forward renderingu:

* Pozycja 3D **position** do obliczenia (interpolowanej) zmiennej pozycji fragmentu używanej dla obliczenia <var>lightDir</var> i <var>viewDir</var>.
* Wektor diffuse RGB **color** znany również jako <def>albedo</def>.
* Wektor **normal** do wyznaczania nachylenia powierzchni.
* **Intensywność lustrzana** (ang. *specular intensity*) typu float.
* Wszystkie pozycje i kolory źródeł światła.
* Wektor pozycji gracza lub widza.

Dysponując tymi zmiennymi (per-fragment), jesteśmy w stanie obliczyć oświetlenie (Blinn-)Phonga, do którego jesteśmy przyzwyczajeni. Pozycje i kolory źródła światła oraz pozycja widoku gracza mogą być konfigurowane za pomocą zmiennych uniform, ale pozostałe zmienne są specyficzne dla każdego fragmentu obiektu. Jeśli uda nam się przekazać dokładnie te same dane do końcowego odroczonego przejścia oświetlenia, możemy obliczyć te same efekty świetlne, nawet jeśli renderujemy fragmenty na powierzchni 2D.

W OpenGL nie ma ograniczeń co do tego, co możemy przechowywać w teksturach, więc sensowne jest przechowywanie wszystkich danych per-fragment w jednej lub wielu całoekranowych teksturach, nazywanych G-bufferem i używanie ich później w przejściu obliczania oświetlenia. Ponieważ tekstury G-buffera będą miały te same dane fragmentów, które mieliśmy w ustawieniach forward renderingu, ale tym razem w przejściu oświetlenia; istnieje mapowanie jeden na jeden.

W pseudokodzie cały proces będzie wyglądał tak:

```cpp
    while(...) // render loop
    {
        // 1\. geometry pass: render all geometric/color data to g-buffer 
        glBindFramebuffer(GL_FRAMEBUFFER, gBuffer);
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
        gBufferShader.use();
        for(Object obj : Objects)
        {
            ConfigureShaderTransformsAndUniforms();
            obj.Draw();
        }  
        // 2\. lighting pass: use g-buffer to calculate the scene's lighting
        glBindFramebuffer(GL_FRAMEBUFFER, 0);
        glClear(GL_COLOR_BUFFER_BIT);
        lightingPassShader.use();
        BindAllGBufferTextures();
        SetLightingUniforms();
        RenderQuad();
    }
```

Dane, których będziemy potrzebować dla każdego fragmentu to wektor **position**, **normal**, **color** i **specular intensity**. W przejściu geometrii musimy zatem wyrenderować wszystkie obiekty sceny i zapisać te dane w G-buffer. Możemy ponownie użyć <def>multiple render targets</def>, aby renderować do wielu buforów kolorów w pojedynczym przebiegu renderowania; zostało to krótko omówione w samouczku [bloom]({% post_url /learnopengl/5_advanced_lighting/2018-10-17-bloom %}).

W przypadku przejścia geometrii musimy zainicjować obiekt bufora ramki, który nazwiemy <var>gBuffer</var>, który ma wiele dołączonych buforów kolorów i pojedynczy obiekt bufora głębi. W przypadku tekstur położenia i normalnych najlepiej użyć tekstury o wysokiej dokładności (16 lub 32-bitów na komponent), a dla tekstur albedo i specular można użyć domyślnej precyzji (8-bitów na komponent).

```cpp
    unsigned int gBuffer;
    glGenFramebuffers(1, &gBuffer);
    glBindFramebuffer(GL_FRAMEBUFFER, gBuffer);
    unsigned int gPosition, gNormal, gColorSpec;

    // - position color buffer
    glGenTextures(1, &gPosition);
    glBindTexture(GL_TEXTURE_2D, gPosition);
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB16F, SCR_WIDTH, SCR_HEIGHT, 0, GL_RGB, GL_FLOAT, NULL);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, gPosition, 0);

    // - normal color buffer
    glGenTextures(1, &gNormal);
    glBindTexture(GL_TEXTURE_2D, gNormal);
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB16F, SCR_WIDTH, SCR_HEIGHT, 0, GL_RGB, GL_FLOAT, NULL);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT1, GL_TEXTURE_2D, gNormal, 0);

    // - color + specular color buffer
    glGenTextures(1, &gAlbedoSpec);
    glBindTexture(GL_TEXTURE_2D, gAlbedoSpec);
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, SCR_WIDTH, SCR_HEIGHT, 0, GL_RGBA, GL_UNSIGNED_BYTE, NULL);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT2, GL_TEXTURE_2D, gAlbedoSpec, 0);

    // - tell OpenGL which color attachments we'll use (of this framebuffer) for rendering 
    unsigned int attachments[3] = { GL_COLOR_ATTACHMENT0, GL_COLOR_ATTACHMENT1, GL_COLOR_ATTACHMENT2 };
    glDrawBuffers(3, attachments);

    // then also add render buffer object as depth buffer and check for completeness.
    [...]
```

Ponieważ używamy multiple render targets, musimy wyraźnie powiedzieć OpenGL, który z buforów kolorów jest powiązany z <var>gBuffer</var> do którego chcemy renderować za pomocą <fun>glDrawBuffers</fun>. Warto również zauważyć, że przechowujemy dane pozycji i normalnych w teksturze `RGB`, ponieważ mamy 3 komponenty, ale przechowujemy kolory i dane intensywności specular połączone w jedną teksturę `RGBA`; to oszczędza nam konieczności zadeklarowania dodatkowej tekstury bufora koloru. Ponieważ odroczony rendering staje się coraz bardziej złożony i wymaga więcej danych, szybko znajdziesz nowe sposoby łączenia danych w poszczególnych teksturach.

Następnie musimy wyrenderować scenę do G-buffera. Zakładając, że każdy obiekt ma tekstury diffuse, normalnych i intensowności specular, użyjemy następującego Fragment Shadera do renderowania do G-buffera:

```glsl
    #version 330 core
    layout (location = 0) out vec3 gPosition;
    layout (location = 1) out vec3 gNormal;
    layout (location = 2) out vec4 gAlbedoSpec;

    in vec2 TexCoords;
    in vec3 FragPos;
    in vec3 Normal;

    uniform sampler2D texture_diffuse1;
    uniform sampler2D texture_specular1;

    void main()
    {    
        // store the fragment position vector in the first gbuffer texture
        gPosition = FragPos;
        // also store the per-fragment normals into the gbuffer
        gNormal = normalize(Normal);
        // and the diffuse per-fragment color
        gAlbedoSpec.rgb = texture(texture_diffuse1, TexCoords).rgb;
        // store specular intensity in gAlbedoSpec's alpha component
        gAlbedoSpec.a = texture(texture_specular1, TexCoords).r;
    }  
```

Ponieważ korzystamy z multiple render targets, specyfikator układu (layout) informuje OpenGL, do którego bufora kolorów aktualnie aktywnego bufora ramki renderujemy. Zauważ, że nie przechowujemy intensywności zwierciadlanej w pojedynczej teksturze bufora kolorów, ponieważ możemy przechowywać jej wartość pojedynczego floata w elemencie alfa jednej z pozostałych tekstur bufora kolorów.

{: .box-error }
Należy pamiętać, że przy obliczeniach oświetlenia niezwykle ważne jest zachowanie wszystkich zmiennych w tej samej przestrzeni współrzędnych; w tym przypadku przechowujemy (i obliczamy) wszystkie zmienne w przestrzeni świata.

Gdybyśmy mieli teraz renderować dużą kolekcję obiektów nanokombinezonu do bufora ramki <var>gBuffer</var> i chcielibyśmy zwizualizować jego zawartość, wyświetlając bufory kolorów jeden po drugim na pełnoekranowym kwadracie, zobaczylibyśmy coś takiego:

![Obraz G-buffer w OpenGL z kilkoma nanokombinezonami](/img/learnopengl/deferred_g_buffer.png){: .center-image }

Spróbuj sobie wyobrazić, że pozycja w przestrzeni świata i wektory normalne są rzeczywiście poprawne. Na przykład wektory normalne wskazujące na prawo będą bardziej wyrównane do koloru czerwonego, podobnie jak w przypadku wektorów pozycji, które wskazują od punkt początkowy sceny w prawo. Jak tylko będziesz zadowolony z zawartości G-buffera, czas przejść do następnego kroku: przejścia oświetlenia.

## Odroczone przejście oświetlenia

Dysponując dużą kolekcją danych fragmentów w G-buffer, mamy możliwość obliczenia oświetlenia sceny poprzez przetwarzanie każdej z tekstur G-Buffer piksel po pikselu i wykorzystanie ich zawartości jako danych wejściowych do algorytmów oświetleniowych. Ponieważ wartości tekstury G-buffer reprezentują ostateczne wartości przekształconych fragmentów, musimy wykonać tylko drogie operacje oświetleniowe raz na piksel. To sprawia, że ​​odroczone cieniowanie jest dość wydajne, szczególnie w złożonych scenach, w których łatwo byłoby wywołać wiele kosztownych wywołań Fragment Shadera per piksel w forward renderingu.

Dla przejścia oświetlenia wyrenderujemy pełnoekranowy kwadrat (trochę jak efekt post-processingu) i uruchomimy Fragment Shader oświetlenia:

```cpp
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glActiveTexture(GL_TEXTURE0);
    glBindTexture(GL_TEXTURE_2D, gPosition);
    glActiveTexture(GL_TEXTURE1);
    glBindTexture(GL_TEXTURE_2D, gNormal);
    glActiveTexture(GL_TEXTURE2);
    glBindTexture(GL_TEXTURE_2D, gAlbedoSpec);
    // also send light relevant uniforms
    shaderLightingPass.use();
    SendAllLightUniformsToShader(shaderLightingPass);
    shaderLightingPass.setVec3("viewPos", camera.Position);
    RenderQuad();  
```

Przed renderowaniem wiążemy wszystkie istotne tekstury G-buffera, a także wysyłamy uniformy związane z oświetleniem do Fragment Shadera.

Fragment Shader przejścia oświetlenia jest w dużej mierze podobny do shaderów z samouczków, których używaliśmy do tej pory. Nowością jest fragment kodu, w którym pobieramy zmienne wejściowe oświetlenia, które teraz bezpośrednio próbkujemy z G-buffera:

```glsl
    #version 330 core
    out vec4 FragColor;

    in vec2 TexCoords;

    uniform sampler2D gPosition;
    uniform sampler2D gNormal;
    uniform sampler2D gAlbedoSpec;

    struct Light {
        vec3 Position;
        vec3 Color;
    };
    const int NR_LIGHTS = 32;
    uniform Light lights[NR_LIGHTS];
    uniform vec3 viewPos;

    void main()
    {             
        // retrieve data from G-buffer
        vec3 FragPos = texture(gPosition, TexCoords).rgb;
        vec3 Normal = texture(gNormal, TexCoords).rgb;
        vec3 Albedo = texture(gAlbedoSpec, TexCoords).rgb;
        float Specular = texture(gAlbedoSpec, TexCoords).a;

        // then calculate lighting as usual
        vec3 lighting = Albedo * 0.1; // hard-coded ambient component
        vec3 viewDir = normalize(viewPos - FragPos);
        for(int i = 0; i < NR_LIGHTS; ++i)
        {
            // diffuse
            vec3 lightDir = normalize(lights[i].Position - FragPos);
            vec3 diffuse = max(dot(Normal, lightDir), 0.0) * Albedo * lights[i].Color;
            lighting += diffuse;
        }

        FragColor = vec4(lighting, 1.0);
    }  
```

Shader przejścia oświetlenia przyjmuje 3 uniformy tekstur, które reprezentują G-buffer i przechowuje wszystkie dane, które zapisaliśmy w przejściu geometrii. Gdybyśmy je spróbkowali z bieżącymi współrzędnymi tekstury fragmentu, otrzymalibyśmy dokładnie takie same wartości fragmentów, jak gdybyśmy bezpośrednio renderowali geometrię. Na początku Fragment Shadera odczytujemy istotne dla oświetlenia zmienne z tekstur G-buffera za pomocą prostego próbkowania tekstury. Zwróć uwagę, że pobieramy zarówno kolor <var>Albedo</var>, jak i intensywność <var>Specular</var> z pojedynczej tekstury <var>gAlbedoSpec</var>.

Ponieważ mamy już potrzebne zmienne per-fragment (i odpowiednie uniformy) do obliczenia oświetlenia Blinn-Phonga, nie musimy wprowadzać żadnych zmian w kodzie oświetlenia. Jedyną zmianą w odroczonym cieniowaniu jest metoda pobierania zmiennych wejściowych oświetlenia.

Uruchomienie prostej wersji demonstracyjnej z `32` małymi lampkami wygląda tak:

![Przykład odroczonego cieniowania w OpenGL](/img/learnopengl/deferred_shading.png){: .center-image }

Jedną z wad odroczonego cieniowania jest to, że nie można wykonać [blendingu]({% post_url /learnopengl/4_advanced_opengl/2018-08-27-blending %}), ponieważ wszystkie wartości w G-buffer pochodzą z pojedynczych fragmentów, a blending działa na wielu fragmentach. Inną wadą jest to, że odroczone cieniowanie wymaga użycia tego samego algorytmu oświetlenia dla większości oświetlenia sceny; możesz w jakiś sposób temu zaradzić poprzez dołączenie większej ilości danych specyficznych dla materiału w G-buffer.

Aby przezwyciężyć te niedogodności (zwłaszcza blending), często dzielimy renderer na dwie części: odroczoną część renderującą, a drugą - część renderującą przeznaczoną specjalnie dla blendingu i/lub specjalnych efektów cieniowania nieodpowiednich dla odroczonego renderowania. Aby zilustrować, jak to działa, będziemy renderować źródła światła jako małe kostki przy użyciu forward renderingu, ponieważ kostki światła wymagają specjalnego Fragment Shadera (wystarczy renderować pojedynczy kolor światła).

## Łączenie deferred rendering z forward rendering

Powiedzmy, że chcemy renderować każde ze źródeł światła jako kostkę 3D umieszczoną w miejscu źródła światła, emitującą kolor światła obok odroczonego renderera. Pierwszą myślą, która przychodzi do głowy, jest po prostu użycie forward renderingu po deferred renderingu. Zasadniczo wyrenderuj kostki tak jak zwykle, ale dopiero po zakończeniu odroczonych operacji renderowania. W kodzie będzie to wyglądać tak:

```cpp
    // deferred lighting pass
    [...]
    RenderQuad();

    // now render all light cubes with forward rendering as we'd normally do
    shaderLightBox.use();
    shaderLightBox.setMat4("projection", projection);
    shaderLightBox.setMat4("view", view);
    for (unsigned int i = 0; i < lightPositions.size(); i++)
    {
        model = glm::mat4();
        model = glm::translate(model, lightPositions[i]);
        model = glm::scale(model, glm::vec3(0.25f));
        shaderLightBox.setMat4("model", model);
        shaderLightBox.setVec3("lightColor", lightColors[i]);
        RenderCube();
    }
```

Jednak te wyrenderowane kostki nie uwzględniają żadnej z głębokości geometrii odroczonego renderera i są w rezultacie zawsze renderowane na wierzchu wcześniej wyrenderowanych obiektów; to nie jest rezultat, którego szukaliśmy.

![Obraz odroczonego renderowania z forward renderingiem, w którym nie skopiowaliśmy danych i świateł bufora głębi, jest renderowany na wierzchu całej geometrii w OpenGL](/img/learnopengl/deferred_lights_no_depth.png){: .center-image }

Musimy najpierw skopiować informacje o głębokości do domyślnego bufora głębi bufora ramki, a dopiero potem renderować kostki światła. W ten sposób fragmenty kostek światła są tylko renderowane jezeli znajdują na wierzchu poprzednio wyrenderowanej geometrii.

Możemy skopiować zawartość bufora ramki do zawartości innego framebuffera za pomocą <fun>glBlitFramebuffer</fun>, funkcja używana również w samoczuku o [antyaliasingu]({% post_url /learnopengl/4_advanced_opengl/2018-09-14-antyaliasing %}). Funkcja <fun>glBlitFramebuffer</fun> pozwala nam skopiować zdefiniowany przez użytkownika region bufora ramki do zdefiniowanego przez użytkownika regionu innego bufora ramki.

Przechowywaliśmy głębokość wszystkich obiektów renderowanych w odroczonym przejściu cieniowania w <var>gBuffer</var> FBO. Gdybyśmy po prostu skopiowali zawartość jego bufora głębi do bufora głębi domyślnego bufora ramki, kostki światła renderowałyby się tak, jakby cała geometria sceny była renderowana z forward renderingiem. Jak wyjaśniłem pokrótce w samouczku o antyaliasingu, musimy określić bufor ramki <var>gBuffer</var> jako bufor ramki do odczytu i podobnie określić domyślny bufor ramki jako framebuffer do zapisu:

```cpp
    glBindFramebuffer(GL_READ_FRAMEBUFFER, gBuffer);
    glBindFramebuffer(GL_DRAW_FRAMEBUFFER, 0); // write to default framebuffer
    glBlitFramebuffer(
      0, 0, SCR_WIDTH, SCR_HEIGHT, 0, 0, SCR_WIDTH, SCR_HEIGHT, GL_DEPTH_BUFFER_BIT, GL_NEAREST
    );
    glBindFramebuffer(GL_FRAMEBUFFER, 0);
    // now render light cubes as before
    [...]  
```

Tutaj kopiujemy całą zawartość bufora głębi do domyślnego bufora głębi bufora ramki; podobnie można to zrobić dla buforów kolorów i buforów szablonów. Teraz, jeśli następnie wyrenderujemy kostki światła, kostki rzeczywiście zachowują się tak, jakby geometria sceny była prawdziwa, a nie po prostu wklejona na wierzch kwadratu:

![Obraz odroczonego renderowania z renderowaniem w przód, w którym skopiowaliśmy dane i światła bufora głębi, jest renderowany poprawnie z całą geometrią w OpenGL](/img/learnopengl/deferred_lights_depth.png){: .center-image }

Możesz znaleźć pełny kod źródłowy dema [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/5.advanced_lighting/8.1.deferred_shading/deferred_shading.cpp).

Dzięki takiemu podejściu możemy łatwo połączyć opóźnione cieniowanie z forward shadingiem. Jest to świetne, ponieważ możemy nadal stosować mieszanie i renderować obiekty wymagające specjalnych efektów cieniowania, co nie jest możliwe w kontekście samego odroczonego renderowania.

## Większa liczba świateł

To, za co często jest chwalony odroczony rendering, to zdolność do renderowania olbrzymiej ilości źródeł światła bez ponoszenia wysokich kosztów. Odroczone renderowanie samo w sobie nie pozwala na użycie bardzo dużej ilości źródeł świateł, ponieważ musielibyśmy obliczyć komponent oświetlenia każdego fragmentu dla każdego ze źródeł światła sceny. To, co sprawia, że ​​duża ilość źródeł światła jest możliwa, to bardzo zgrabna optymalizacja, którą możemy zastosować do odroczonego renderowania: <def>light volumes</def> (światła objętościowe).

Normalnie, gdy renderujemy fragmenty w dużej oświetlonej scenie, obliczymy wkład **każdego** źródła światła w scenie, niezależnie od ich odległości od fragmentu. Duża część tych źródeł światła nigdy nie dotrze do fragmentu, więc po co marnować wszystkie obliczenia oświetlenia?

Ideą świateł objętościowych jest obliczenie promienia lub objętości źródła światła, tj. obszaru, w którym jego światło jest w stanie dotrzeć do fragmentów. Ponieważ większość źródeł światła wykorzystuje pewną formę tłumienia, możemy użyć tego do obliczenia maksymalnej odległości lub promienia, na jaką światło jest w stanie dotrzeć. Wykonujemy wtedy tylko kosztowne obliczenia oświetlenia, jeśli fragment znajduje się w jednym lub kilku z tych świateł objętościowych. To może zaoszczędzić nam znacznej ilości obliczeń, ponieważ obliczamy oświetlenie tylko tam, gdzie jest to konieczne.

Sztuczka w tym podejściu polega głównie na określeniu rozmiaru lub promienia źródła światła.

### Obliczanie promienia (objętości) światła

Aby uzyskać promień światła objętościowego, musielibyśmy w zasadzie rozwiązać równanie tłumienia dla jasności, którą uważamy za _ciemną_, może to być `0.0` lub coś nieco bardziej oświetlonego, ale wciąż uważane za ciemne jak np. `0.03`. Aby zademonstrować, w jaki sposób możemy obliczyć promień światła objętościowego, użyjemy jednej z trudniejszych, ale bardziej rozbudowanej funkcji tłumienia, którą wprowadziliśmy w samouczku [typy świateł]({% post_url /learnopengl/2_lighting/2018-08-08-typy-swiatel %}):

$$F_{light} = \frac{I}{K_c + K_l * d + K_q * d^2}$$

Chcemy rozwiązać to równanie, dla $F_{light}$ równego `0.0`, kiedy światło jest całkowicie ciemne dla tej odległości. Jednak to równanie nigdy nie osiągnie dokładnie wartości `0.0`, więc nie będzie rozwiązania. To, co możemy zrobić, to nie rozwiązywać równania dla `0.0`, ale rozwiązać je dla wartości jasności bliskiej `0.0`, ale nadal postrzeganej jako ciemna. Wartość jasności, którą wybieramy jako akceptowalną dla sceny demonstracyjnej tego samouczka, wynosi i $5/256$; podzielone przez `256`, ponieważ domyślny 8-bitowy bufor ramki może wyświetlać wiele intensywności na komponent.

{: .box-note }
Zastosowana funkcja tłumienia jest w większości ciemna w swoim widzialnym zakresie, więc gdybyśmy ograniczyli ją do jeszcze ciemniejszej jasności niż $5/256$, objętość światła stałaby się zbyt duża, a przez to mniej skuteczna. Dopóki użytkownik nie widzi nagłego odcięcia źródła światła na granicy jego objętości, nic się nie stanie. Oczywiście to zawsze zależy od rodzaju sceny; wyższy próg jasności powoduje mniejsze natężenie światła, a tym samym lepszą wydajność, ale może generować zauważalne artefakty, w których oświetlenie wydaje się psuć na granicy objętości.

Równanie tłumienia, które musimy rozwiązać, staje się:

$$\frac{5}{256} = \frac{I_{max}}{Attenuation}$$

Tutaj $I_{max}$ jest najjaśniejszym komponentem koloru źródła światła. Używamy najjaśniejszego komponentu koloru źródła światła, ponieważ rozwiązanie równania dla najjaśniejszej wartości natężenia światła najlepiej odzwierciedla idealny promień światła.

Teraz kontynuujemy rozwiązywanie równania:

$$\frac{5}{256} * Attenuation = I_{max}$$

$$5 * Attenuation = I_{max} * 256$$

$$Attenuation = I_{max} * \frac{256}{5}$$

$$K_c + K_l * d + K_q * d^2 = I_{max} * \frac{256}{5}$$

$$K_q * d^2 + K_l * d + K_c - I_{max} * \frac{256}{5} = 0$$

Ostatnie równanie jest równaniem postaci $ax^2 + bx + c = 0$, które jest równaniem kwadratowym:

$$x = \frac{-K_l + \sqrt{K_l^2 - 4 * K_q * (K_c - I_{max} * \frac{256}{5})}}{2 * K_q}$$

To daje nam ogólne równanie, które pozwala nam obliczyć $x$, tj. promień światła objętościowego dla źródła światła, z podaniem parametru stałego, liniowego i kwadratowego:

```cpp
    float constant  = 1.0; 
    float linear    = 0.7;
    float quadratic = 1.8;
    float lightMax  = std::fmaxf(std::fmaxf(lightColor.r, lightColor.g), lightColor.b);
    float radius    = 
      (-linear +  std::sqrtf(linear * linear - 4 * quadratic * (constant - (256.0 / 5.0) * lightMax))) 
      / (2 * quadratic);  
```

To zwraca promień pomiędzy około `1.0`, a `5.0` w zależności od maksymalnej intensywności światła.

Obliczamy ten promień dla każdego źródła światła sceny i używamy go tylko do obliczania oświetlenia dla tego źródła światła, jeśli fragment znajduje się wewnątrz źródła światła. Poniżej znajduje się zaktualizowany Fragment Shader, który uwzględnia obliczone objętości światła. Zauważ, że takie podejście jest jedynie wykonywane dla celów dydaktycznych i nie jest opłacalne w praktyce (wkrótce to omówimy):

```glsl
    struct Light {
        [...]
        float Radius;
    }; 

    void main()
    {
        [...]
        for(int i = 0; i < NR_LIGHTS; ++i)
        {
            // calculate distance between light source and current fragment
            float distance = length(lights[i].Position - FragPos);
            if(distance < lights[i].Radius)
            {
                // do expensive lighting
                [...]
            }
        }   
    }
```

Wyniki są dokładnie takie same jak poprzednio, ale tym razem każde światło oblicza jedynie oświetlenie dla źródeł światła, w których znajduje się fragment.

Możesz znaleźć ostateczny kod źródłowy dema [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/5.advanced_lighting/8.2.deferred_shading_volumes/deferred_shading_volumes.cpp).

### Jak naprawdę używamy świateł objętościowych

Pokazany powyżej Fragment Shader naprawdę nie działa w praktyce i tylko ilustruje, w jaki sposób możemy użyć świateł objętościowych, aby zmniejszyć koszt obliczeń oświetlenia. W rzeczywistości GPU i GLSL są naprawdę złe w optymalizacji pętli i warunków. Powodem tego jest to, że wykonywanie Fragment Shadera na GPU jest wysoce równoległe, a większość architektur wymaga, aby przy dużych zbiorach wątków działał dokładnie ten sam kod shadera, aby był wydajny. Często oznacza to, że uruchamiany jest Fragment Shader, który zawsze wykonuje wszystkie gałęzie instrukcji `if`, aby zapewnić, że przebiegi Fragment Shaderów są takie same, przez co nasza poprzednia funkcja sprawdzania objętości jest całkowicie bezużyteczna; wciąż obliczalibyśmy oświetlenie dla wszystkich źródeł światła!

Odpowiednim podejściem do wykorzystywania świateł objętościowych jest renderowanie rzeczywistych sfer, skalowanych przez promień światła. Środek tych sfer znajduje się w pozycji źródła światła, a ponieważ jest skalowany promieniem światła, sfera dokładnie obejmuje widzialną objętość światła. Tutaj pojawia się sztuczka: używamy w dużej mierze tego samego odroczonego shadera do renderowania sfer. Renderowana kula generuje wywołania Fragment Shadera, które dokładnie pasują do pikseli, na które wpływa źródło światła, renderujemy tylko odpowiednie piksele i pomijamy wszystkie pozostałe piksele. Poniższy obrazek to ilustruje:

![Obraz światła objętościowego renderowanego z shaderem odroczonym w OpenGL](/img/learnopengl/deferred_light_volume_rendered.png){: .center-image }

Odbywa się to dla każdego źródła światła w scenie, a powstałe fragmenty są dodatkowo łączone ze sobą. Rezultatem jest dokładnie ta sama scena co poprzednio, ale tym razem renderowana tylko dla odpowiednich fragmentów per źródło światła. To skutecznie redukuje obliczenia z `nr_objects * nr_lights` do `nr_objects + nr_lights`, co czyni je niesamowicie wydajnymi w scenach z dużą liczbą świateł. To podejście powoduje, że renderowanie odroczone jest odpowiednie do renderowania dużej liczby świateł.

Nadal występuje problem z tym podejściem: należy uaktywnić funkcję usuwania ścianek (w przeciwnym razie renderowalibyśmy efekt oświetlenia dwukrotnie) i gdy jest włączona, użytkownik może wejść w objętość źródła światła, po czym światło objętościowe nie jest już renderowane (z powodu usuwania tylnych ścianek), co usuwa wpływ źródła światła; można to rozwiązać za pomocą sztuczki z buforem szablonu.

Renderowanie świateł objetościowych niesie za sobą ogromne straty wydajności. Podczas gdy, generalnie jest szybsze niż normalne odroczone cieniowanie, ale nie jest ono najlepszą optymalizacją. Istnieją dwa inne popularne (i bardziej wydajne) rozszerzenia w stosunku do odroczonego cieniowania o nazwie <def>odroczone oświetlenie</def> (ang. *deferred lighting*) i <def>odroczone cieniowanie oparte na kafelkach</def> (ang. *tile-based deferred shading*). Są niewiarygodnie wydajne w renderowaniu dużych ilości światła, a także pozwalają na stosunkowo wydajne MSAA. Jednak ze względu na długość tego samouczka, te techniki nie zostaną omówione.

## Deferred rendering vs forward rendering

Odroczone cieniowanie (bez świateł objętościowych) jest już dużą optymalizacją, ponieważ na każdy piksel uruchamiany jest tylko jeden Fragment Shader, w porównaniu do forward renderingu, w którym często uruchamiamy Fragment Shader kilka razy na piksel. Odroczone renderowanie ma jednak kilka wad: duży nadmiar pamięci, brak MSAA i blendingu wciąż musi być wykonywane z wykorzystaniem forward renderingu.

Kiedy masz małą scenę i niezbyt wiele świateł, odroczone renderowanie niekoniecznie jest szybsze, a czasem nawet wolniejsze, ponieważ narzut przewyższa korzyści odroczonego renderowania. W bardziej złożonych scenach odroczone renderowanie szybko staje się znaczącą optymalizacją; zwłaszcza w przypadku zastosowania bardziej zaawansowanych rozszerzeń optymalizacyjnych.

Na koniec chciałbym również wspomnieć, że zasadniczo wszystkie efekty, które można osiągnąć przy forward renderingu, mogą być również implementowane w odroczonym rendererze. Na przykład, jeśli chcemy użyć normal mappingu w odroczonym rendererze, zmienilibyśmy Geometry Shader, aby wyprowadzał normalne w przestrzeni świata wyodrębnione z mapy normalnych (z wykorzystaniem macierzy TBN) zamiast wektora normalnego powierzchni; obliczenia oświetlenia w przejściu oświetlenia nie muszą się w ogóle zmieniać. A jeśli chcesz, aby działało parallax mapping, musisz najpierw zmodyfikować współrzędne tekstury w przejściu geometrii przed próbkowaniem tekstur rozproszonych, zwierciadlanych lub normalnych obiektu.

## Dodatkowe materiały

*   [Tutorial 35: Deferred Shading - Part 1](http://ogldev.atspace.co.uk/www/tutorial35/tutorial35.html): trzyczęściowy samouczek o odroczonym cieniowaniu autorstwa OGLDev. W części 2 i 3 omówiony jest temat renderowania świateł objętościowych.
*   [Deferred Rendering for Current and Future Rendering Pipelines](https://software.intel.com/sites/default/files/m/d/4/1/d/8/lauritzen_deferred_shading_siggraph_2010.pdf): slajdy Andrew Lauritzena omawiające odroczone cieniowanie oparte na tile'ach.