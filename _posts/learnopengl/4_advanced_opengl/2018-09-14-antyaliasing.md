---
layout: post
title: Antyaliasing
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: zaawansowany-opengl
---

{% include learnopengl.md link="Advanced-OpenGL/Anti-Aliasing" %}

Gdzieś podczas przygody z renderowaniem pojawiły się poszarpane wzory przypominające piłę wzdłuż krawędzi modeli. Powód pojawiania się <def>postrzępionych krawędzi</def> wynika z tego, jak rasteryzator przekształca dane wierzchołków w rzeczywiste fragmenty. Przykład tego, jak wyglądają postrzępione krawędzie, można już zobaczyć podczas rysowania prostego sześcianu:

![Pojemnik z widocznym aliasingiem](/img/learnopengl/anti_aliasing_aliasing.png){: .center-image }

Choć nie jest to od razu widoczne, jeśli przyjrzysz się bliżej krawędziom sześcianu, zobaczysz postrzępiony wzór:

![Powiększany pojemnik z widocznym aliasingiem](/img/learnopengl/anti_aliasing_zoomed.png){: .center-image }

To zdecydowanie nie jest coś, co chcemy oglądać w ostatecznej wersji aplikacji. Efekt ten, polegający na wyraźnym widzeniu pikseli, które tworzą krawędź, jest nazywany <def>aliasingiem</def>. Istnieje wiele technik <def>antyaliasingu</def>, które starają się zwalczać efekt poszarpanych krawędzi, poprzez ich _wygładzanie_.

Na początku mieliśmy technikę zwaną <def>super sample anti-aliasing</def> (SSAA), która tymczasowo używała znacznie wyższej rozdzielczości do renderowania sceny (super sampling/super próbkowanie) i kiedy wizualne wyjście jest aktualizowane w buforze ramki, rozdzielczość była zmniejszana z powrotem do normalnej rozdzielczości. Ta _wyższa_ rozdzielczość została użyta, aby zapobiec tym postrzępionym krawędziom. Mimo że dostarczyliśmy rozwiązanie problemu aliasingu, przyszedł on z poważną wadą wydajności, ponieważ musieliśmy narysować o wiele więcej fragmentów niż zwykle. Technika ta miała zatem tylko krótki moment chwały.

Technika ta dała początek bardziej nowoczesnej technice zwanej <def>multisample anti-aliasing</def> lub MSAA, która zapożycza koncepcje leżące u podstaw SSAA, jednocześnie wdrażając znacznie bardziej efektywne podejście. W tym samouczku będziemy szeroko omawiać technikę MSAA, która jest "wbudowana" w OpenGL.

## Multisampling

Aby zrozumieć, czym jest multisampling i jak on działa, aby rozwiązać problem aliasingu, najpierw musimy zagłębić się nieco we wewnętrzne działanie rasteryzatora OpenGL.

Rasteryzator jest kombinacją wszystkich algorytmów i procesów, które znajdują się pomiędzy końcowymi przetworzonymi wierzchołkami a Fragment Shaderem. Rasteryzator przyjmuje wszystkie wierzchołki należące do pojedynczego prymitywu i przekształca je w zestaw fragmentów. Współrzędne wierzchołków mogą teoretycznie mieć dowolną współrzędną, ale fragmenty nie mogą być, ponieważ są powiązane z rozdzielczością twojego okna. Niemal nigdy nie będzie mapowania jeden do jednego między współrzędnymi wierzchołka i fragmentami, więc rasteryzer musi w jakiś sposób określić, na którym fragmencie/współrzednej ekranu znajduje się każdy konkretny wierzchołek.

![Rasteryzowanie trójkąta w OpenGL](/img/learnopengl/anti_aliasing_rasterization.png){: .center-image }

Widzimy tutaj siatkę pikseli, gdzie środek każdego piksela zawiera <def>punkt próbkowania</def> (ang. *sample point*), który jest używany do określenia, czy piksel należy do trójkąta. Czerwone punkty próbkowania są objęte trójkątem i dla tych pokrytych próbek zostanie wygenerowany fragment. Mimo że niektóre części krawędzi trójkąta nadal pokrywają pewne piksele na ekranie, punkt próbkowania piksela nie jest pokryty przez wnętrze trójkąta, więc żaden piksel nie będzie przetwarzany przez Fragment Shader.

Prawdopodobnie już wiesz skąd bierze się aliasing. Wyrenderowana wersja trójkąta wyglądałaby tak:

![Wypełniony trójkąt w wyniku rasteryzacji w OpenGL](/img/learnopengl/anti_aliasing_rasterization_filled.png){: .center-image }

Ze względu na ograniczoną liczbę pikseli ekranu niektóre piksele będą renderowane wzdłuż krawędzi, a niektóre nie. Powoduje to, że renderujemy prymitywy z niegładkimi krawędziami, co powoduje powstanie postrzępionych krawędzi, które widzieliśmy wcześniej.

To, co robi multisampling, to nie wykorzystuje pojedynczego punktu próbkowania do określania pokrycia przez trójkąt, ale wykorzystuje wiele punktów próbkowania. Zamiast pojedynczego przykładowego punktu pośrodku każdego piksela, umieszczamy `4` <def>podpróbki</def> (ang. *subsample*) w ogólnym wzorze i wykorzystujemy je do określenia pokrycia pikseli. Oznacza to, że rozmiar bufora kolorów jest również zwiększany o liczbę podpróbek, których używamy w pikselach.

![Multisampling w OpenGL](/img/learnopengl/anti_aliasing_sample_points.png){: .center-image }

Lewa strona obrazu pokazuje, jak zwykle określilibyśmy pokrycie trójkąta. Ten konkretny piksel nie uruchomi Fragment Shadera (i dlatego pozostanie pusty), ponieważ jego punkt próbkowania nie jest objęty przez trójkąt. Prawa strona obrazu pokazuje wersję z wieloma próbkami, gdzie każdy piksel zawiera `4` punkty próbkowania. Tutaj widzimy, że tylko `2` z punktów próbkowania pokrywa trójkąt.

{: .box-note }
Ilość punktów próbkowania może być dowolną liczbą, którą chcemy uzyskać. Większa ilość próbek daje nam lepszą precyzję pokrycia.

Właśnie w tym miejscu multisampling staje się interesujący. Ustaliliśmy, że `2` podpróbki są objęte trójkątem, więc następnym krokiem jest określenie koloru dla tego konkretnego piksela. Naszym początkowym domysłem byłoby, że uruchomimy Fragment Shader dla każdej z objętych podpróbek, a później uśrednimy kolory podpróbek na każdy piksel. W tym przypadku dwukrotnie uruchomimy Fragment Shader na interpolowanych danych wierzchołkowych dla każdej podpróbki i zapiszemy wynikowy kolor w tych punktach próbkowania. Na szczęście **nie** tak to działa, ponieważ w zasadzie oznacza to, że musimy uruchomić o wiele więcej inwokacji Fragment Shadera niż bez multisamplingu, drastycznie zmniejszając wydajność.

Jak naprawdę działa MSAA?, Fragment Shader działa tylko raz na piksel (dla każdego prymitywu), niezależnie od tego, ile podpróbek obejmuje trójkąt. Fragment Shader jest uruchamiany z interpolowanymi danymi wierzchołków dla **centrum** piksela, a wynikowy kolor jest następnie przechowywany wewnątrz każdej z objętych podpróbek. Po tym, jak podpróbki bufora koloru zostaną wypełnione wszystkimi kolorami prymitywów, które wyrenderowaliśmy, wszystkie te kolory są następnie uśredniane na piksel, co daje jeden kolor na piksel. Ponieważ tylko dwie z 4 próbek zostały pokryte na poprzednim obrazie, kolor piksela został uśredniony z kolorem trójkąta i kolorem przechowywanym w pozostałych 2 punktach próbkowania (w tym przypadku: kolor czyszczenia), co daje jasnoniebieski kolor.

Rezultatem jest bufor kolorów, w którym wszystkie krawędzie prymitywu tworzą teraz gładszy wzór. Zobaczmy, jak wygląda multisampling, kiedy ponownie obliczymy pokrycie wcześniejszego trójkąta:

![Rasteryzacja trójkąta z multisamplingiem w OpenG](/img/learnopengl/anti_aliasing_rasterization_samples.png){: .center-image }

Tutaj każdy piksel zawiera 4 podpróbki (niepotrzebne próbki zostały ukryte), gdzie niebieskie podpróbki są pokryte trójkątem, a szare punkty próbkowania nie. Wewnątrz wewnętrznego obszaru trójkąta wszystkie piksele będą uruchamiały Fragment Shader, a jego kolor zostanie zapisany we wszystkich 4 podpróbkach. Na krawędziach trójkąta nie wszystkie podpróbki zostaną pokryte, więc wynik Fragment Shadera jest przechowywany tylko w niektórych podpróbkach. W zależności od ilości pokrytych podpróbek, wynikowy kolor piksela jest określany przez kolor trójkąta i kolory przechowywane w pozostałych podpróbkach.

Zasadniczo, im więcej punktów próbkowania pokrywa trójkąt, tym bardziej ostateczny kolor piksela jest kolorem trójkąta. Jeśli następnie wypełnimy piksele kolorami, tak jak wcześniej zrobiliśmy to za pomocą trójkąta bez multisamplingu, otrzymamy następujący obraz:

![Rasteryzowany trójkąt z multisamplingiem w OpenGL](/img/learnopengl/anti_aliasing_rasterization_samples_filled.png){: .center-image }

Dla każdego piksela, im mniej podpróbek jest częścią trójkąta, tym mniej przyjmuje on kolor trójkąta, jak to widać na obrazku. Twarde krawędzie trójkąta są teraz otoczone kolorami nieco jaśniejszymi niż rzeczywisty kolor krawędzi, co powoduje, że krawędź wydaje się gładka, gdy oglądana jest z daleka.

Nie tylko wartości kolorów mają wpływ na multisampling, ale także test głębi i szablonu wykorzystuje teraz wiele punktów próbkowania. W przypadku testowania głębokości wartość głębi wierzchołka jest interpolowana do każdej podpróbki przed uruchomieniem testu głębokości, a w przypadku testowania szablonu przechowujemy wartości szablonów na podpróbkę zamiast na piksel. Oznacza to, że rozmiar głębokości i bufor szablonu są teraz również zwiększone o ilość podpróbek na piksel.

To, o czym rozmawialiśmy, to podstawowy przegląd tego, jak działa multisampling za kulisami. Faktyczna logika stojąca za rasteryzatorem jest nieco bardziej skomplikowana, niż omówiliśmy to tutaj, ale teraz powinieneś być w stanie zrozumieć koncepcję i logikę stojącą za multisamplingiem.

## MSAA w OpenGL

Jeśli chcemy używać MSAA w OpenGL, musimy użyć bufora kolorów, który może przechowywać więcej niż jedną wartość koloru na piksel (ponieważ multisampling wymaga od nas przechowywania koloru na każdy punkt próbkowania). Potrzebujemy więc nowego typu bufora, który może przechowywać określoną ilość próbek i nazywa się to <def>multisample buffer</def> (bufor wielopróbkowy).

Większość systemów okienkowych jest w stanie dostarczyć nam bufor wielopróbkowy zamiast domyślnego bufora kolorów. GLFW zapewnia nam tę funkcję i wszystko, co musimy zrobić, to ustawić _hint_ GLFW, że chcemy użyć bufora wielopróbkowego z N próbek zamiast normalnego bufora kolorów, wywołując <fun>glfwWindowHint</fun> przed utworzeniem okna:

```cpp
    glfwWindowHint(GLFW_SAMPLES, 4);
```

Kiedy teraz wywołamy <fun>glfwCreateWindow</fun> stworzone zostanie okno renderowania, tym razem z buforem kolorów zawierającym 4 podpróbki na piksel ekranu. GLFW automatycznie stworzy również bufory głębi i szablonu z 4 podpróbkami na piksel. Oznacza to, że rozmiar wszystkich buforów zwiększy się 4 razy.

Teraz, gdy poprosiliśmy GLFW o wielopróbkowy bufor, musimy włączyć multisampling, wywołując <fun>glEnable</fun> z opcją <var>GL_MULTISAMPLE</var>. W większości sterowników OpenGL, multisampling jest domyślnie włączony, więc to wywołanie jest nieco zbędne, ale zazwyczaj dobrym pomysłem jest jego włączenie. W ten sposób wszystkie implementacje OpenGL mają włączony multisampling.

```cpp
    glEnable(GL_MULTISAMPLE);  
```

Gdy domyślny bufor ramki ma multisamplingowe załączniki, wystarczy, że uruchomimy multisampling <fun>glEnable</fun> i gotowe. Ponieważ rzeczywiste algorytmy multisamplingu są zaimplementowane w sterownikach OpenGL, nie musimy już wiele robić. Gdybyśmy teraz renderowali zieloną kostkę z początku tego samouczka, powinniśmy widzieć znacznie gładsze krawędzie:

![Kostka z wygładzonymi krawędziami w OpenGL](/img/learnopengl/anti_aliasing_multisampled.png){: .center-image }

Ten pojemnik rzeczywiście wygląda na bardziej gładki i to samo dotyczy każdego innego obiektu, który narysujesz w scenie. Możesz znaleźć kod źródłowy tego prostego przykładu [tutaj](https://learnopengl.com/code_viewer.php?code=advanced/anti_aliasing_multisampling).

## Poza ekranowe MSAA

Ponieważ GLFW dba o tworzenie wielopróbkowych buforów, włączenie MSAA jest dość łatwe. Jeśli jednak chcemy używać własnych buforów ramek, dla niektórych obrazów renderowanych poza głównym ekranem, musimy sami wygenerować multisamplingowe bufory; teraz musimy sami zadbać o stworzenie wielopróbkowych buforów.

Istnieją dwa sposoby stworzenia multisamplingowych buforów do działania jako załączniki dla buforów ramki: tekstury i renderbuffery, podobne do tych, które zostały omówione w samoczuku Framebuffers.

### Wielopróbkowe załączniki tekstur

Aby utworzyć teksturę, która obsługuje przechowywanie wielu punktów próbkowania, używamy <fun>glTexImage2DMultisample</fun> zamiast <fun>glTexImage2D</fun>, która akceptuje opcję <var>GL_TEXTURE_2D_MULTISAPLE</var> jako typ tekstury:

```cpp
    glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, tex);
    glTexImage2DMultisample(GL_TEXTURE_2D_MULTISAMPLE, samples, GL_RGB, width, height, GL_TRUE);
    glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, 0);  
```

Drugi argument określa teraz liczbę próbek, które chcemy mieć. Jeśli ostatni argument jest równy <var>GL_TRUE</var>, obraz użyje identycznych lokalizacji próbek i tej samej liczby podpróbek dla każdego texela.

Aby dołączyć teksturę wielopróbkową do bufora ramki używamy <fun>glFramebufferTexture2D</fun>, ale tym razem z opcją <var>GL_TEXTURE_2D_MULTISAMPLE</var> jako typem tekstury:

```cpp
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D_MULTISAMPLE, tex, 0); 
```

Obecnie powiązany bufor ramki ma teraz wielopróbkowy bufor kolorów w postaci obrazu tekstury.

### Wielopróbkowy obiekt renderbuffer

Podobnie jak w przypadku tekstur, tworzenie wielopróbkowego obiektu renderbuffer nie jest trudne. Jest to nawet całkiem łatwe, ponieważ wszystko, co musimy zmienić, to wywołanie <fun>glRenderbufferStorage</fun> na wywołanie <fun>glRenderbufferStorageMultisample</fun> kiedy określamy pamięć dla (aktualnie powiązanego) obiektu renderbuffera:

```cpp
    glRenderbufferStorageMultisample(GL_RENDERBUFFER, 4, GL_DEPTH24_STENCIL8, width, height);  
```

Jedyną rzeczą, która się tutaj zmieniła, jest dodatkowy parametr po przeznaczeniu bufora renderowania, w którym ustawiamy ilość próbek, które chcielibyśmy mieć, czyli 4 w tym konkretnym przypadku.

### Renderowanie do wielopróbkowego bufora ramki

Renderowanie do wielopróbkowego obiektu framebuffer dzieje się automatycznie. Za każdym razem, gdy rysujemy cokolwiek, gdy obiekt bufora ramki jest powiązany, rasteryzer zajmie się wszystkimi operacjami multisamplingu. Następnie otrzymujemy wielopróbkowe bufory kolorów i/lub bufora głębi i szablonu. Ponieważ bufor wielopróbkowy jest nieco wyjątkowy, nie możemy bezpośrednio wykorzystać jego obrazów do innych operacji, takich jak próbkowanie ich Fragment Shaderze.

Obraz wielopróbkowy zawiera znacznie więcej informacji niż zwykły obraz, więc to co musimy zrobić to zmniejszyć lub <def>rozwiązać</def> obraz. Rozwiązanie wielopróbkowego bufora ramki wykonuje się zwykle za pomocą <fun>glBlitFramebuffer</fun>, który kopiuje region z jednego bufora ramki do drugiego, jednocześnie rozdzielając wielopróbkowe bufory.

<span class="fun">glBlitFramebuffer</span> przenosi dany region <def>source</def> (źródła) zdefiniowany przez 4 współrzędne ekranowe do danego regionu <def>target</def> (celu) również zdefiniowanego przez 4 współrzędne ekranowe. Możesz sobie przypomnieć z tutoriala Framebuffers, że jeśli powiążemy FBO z <var>GL_FRAMEBUFFER</var>, będziemy wiązać bufor zarówno do odczytu, jak i do zapisu. Możemy również powiązać te cele indywidualnie, wiążąc framebuffery odpowiednio z <var>GL_READ_FRAMEBUFFER</var> i <var>GL_DRAW_FRAMEBUFFER</var>. Funkcja <fun>glBlitFramebuffer</fun> odczytuje te dwa cele, aby określić, które źródło jest docelowym buforem ramki. Następnie możemy przesłać wielopróbkowe wyjście bufora ramki do rzeczywistego ekranu przez <def>blitting</def> obrazu do domyślnego bufora ramki, tak jak zaprezentowano to poniżej:

```cpp
    glBindFramebuffer(GL_READ_FRAMEBUFFER, multisampledFBO);
    glBindFramebuffer(GL_DRAW_FRAMEBUFFER, 0);
    glBlitFramebuffer(0, 0, width, height, 0, 0, width, height, GL_COLOR_BUFFER_BIT, GL_NEAREST); 
```

Gdybyśmy mieli następnie renderować obraz, otrzymalibyśmy takie same wyniki jak bez bufora ramki: kostkę o limonkowym kolorze, która jest wyświetlana za pomocą MSAA, a więc pokazuje znacznie mniej poszarpanych krawędzi:

![Obraz kostki bez poszarpanych krawędzi](/img/learnopengl/anti_aliasing_multisampled.png){: .center-image }

Możesz znaleźć kod źródłowy [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/4.advanced_opengl/11.anti_aliasing_offscreen/anti_aliasing_offscreen.cpp).

Ale co by było, gdybyśmy chcieli użyć wyniku tekstury z wielopróbkowego framebuffera do robienia rzeczy takich jak post-processing? Nie możemy bezpośrednio użyć wielopróbkowej tekstury w Fragment Shader. To, co moglibyśmy zrobić, to blitting wielopróbkowego bufora do innego FBO z normalnym załącznikiem tekstury. Następnie używamy tej zwykłej tekstury kolorów do post-processingu, skutecznie przetwarzając obraz wyrenderowany za pomocą multisamplingu. Oznacza to, że musimy wygenerować nowy obiekt FBO, który działa wyłącznie jako pośredni obiekt bufora ramki, aby rozwiązać wielopróbkowy bufor do normalnej tekstury 2D, której możemy użyć w Fragment Shaderze. Ten proces wygląda trochę tak:

```cpp
    unsigned int msFBO = CreateFBOWithMultiSampledAttachments();
    // then create another FBO with a normal texture color attachment
    ...
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, screenTexture, 0);
    ...
    while(!glfwWindowShouldClose(window))
    {
        ...

        glBindFramebuffer(msFBO);
        ClearFrameBuffer();
        DrawScene();
        // teraz rozdziel bufor(y) wielopróbkowe na pośrednie FBO
        glBindFramebuffer(GL_READ_FRAMEBUFFER, msFBO);
        glBindFramebuffer(GL_DRAW_FRAMEBUFFER, intermediateFBO);
        glBlitFramebuffer(0, 0, width, height, 0, 0, width, height, GL_COLOR_BUFFER_BIT, GL_NEAREST);
        // teraz scena jest przechowywana jako obraz tekstury 2D, więc użyj tego obrazu do postprocessingu
        glBindFramebuffer(GL_FRAMEBUFFER, 0);
        ClearFramebuffer();
        glBindTexture(GL_TEXTURE_2D, screenTexture);
        DrawPostProcessingQuad();  

        ... 
    }
```

Gdybyśmy następnie zaimplementowali to w kodzie post-processingu z tutoriala Framebuffers, bylobyśmy w stanie stworzyć wszystkie fajne efekty postprocessingu na teksturze sceny bez postrzępionych krawędzi. Po zastosowaniu filtru z rozmyciem będzie wyglądać to mniej więcej tak:

![Obraz postprocessingu na scenie narysowanej za pomocą MSAA w OpenGL](/img/learnopengl/anti_aliasing_post_processing.png){: .center-image } 

{: .box-note} 
Ponieważ tekstura ekranu jest ponownie normalną teksturą z tylko jednym punktem próbkowania, niektóre filtry przetwarzania końcowego, takie jak _wykrywanie krawędzi_, ponownie wprowadzą postrzępione krawędzie. Aby to uwzględnić, można później rozmazać teksturę lub utworzyć własny algorytm antyaliasingu.

Widać, że gdy chcemy połączyć multisampling z renderowaniem poza ekranowym, musimy zadbać o dodatkowe szczegóły. Wszystkie szczegóły są warte tego dodatkowego wysiłku, ponieważ multisampling znacznie poprawia jakość wizualną Twojej sceny. Zwróć uwagę, że włączenie multisamplingu może znacząco zmniejszyć wydajność twojej aplikacji, dlatego, że używasz więcej próbek. W czasie pisania tego artykułu, powszechnie preferowane jest używanie MSAA z `4` próbkami.

## Niestandardowy (własny) algorytm antyaliasingu

Możliwe jest również bezpośrednie przekazanie obrazu z wieloma próbkami do shaderów zamiast je rozwiązywać. GLSL daje nam wtedy możliwość próbkowania obrazów tekstur na każdą podpróbkę, abyśmy mogli stworzyć własne algorytmy antyaliasingu, które są często wykorzystywane przez duże aplikacje graficzne.

Aby pobrać wartość koloru na podpróbkę, musisz zdefiniować sampler tekstury jako <fun>sampler2DMS</fun> zamiast zwykłego <fun>sampler2D</fun>:

```glsl
    uniform sampler2DMS screenTextureMS;    
```

Korzystając z funkcji <fun>texelFetch</fun>, można pobrać wartość koloru na próbkę:

```glsl
    vec4 colorSample = texelFetch(screenTextureMS, TexCoords, 3);  // Czwarta podpróbka
```

Nie będziemy wchodzić w szczegóły tworzenia niestandardowych algorytmów antyaliasingu, ale zapewniamy tylko kilka wskazówek, w jaki sposób można zaimplementować taką funkcję.