---
layout: post
title: Framebuffers
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: zaawansowany-opengl
mathjax: true
---

{% include learnopengl.md link="Advanced-OpenGL/Framebuffers" %}

Do tej pory używaliśmy kilku typów buforów ekranowych: bufora kolorów do zapisywania wartości kolorów, bufora głębi do zapisu informacji o głębokości i wreszcie bufora szablonu, który pozwalał nam odrzucić pewne fragmenty na podstawie pewnych warunków. Kombinacja tych buforów nazywa się <def>buforami ramki</def> (ang. *framebuffer*) i jest przechowywana gdzieś w pamięci GPU. OpenGL daje nam elastyczność w definiowaniu własnych buforów ramek, a tym samym definiowania własnego koloru i opcjonalnie bufora głębi i szablonu.

Operacje renderowania, które do tej pory wykonywaliśmy, zostały wykonywane na buforach renderowania (ang. *render buffers*) dołączonych do <def> domyślnego framebuffera</def>. Domyślny bufor ramki jest tworzony i konfigurowany podczas tworzenia okna (GLFW robi to za nas). Tworząc własny framebuffer możemy uzyskać dodatkowe "okna" do renderowania.

Zastosowanie framebufferów może nie mieć natychmiastowego sensu, ale renderowanie sceny do innego framebuffera pozwala nam tworzyć na przykład lustra w scenie lub robić fajne efekty post-processingu. Najpierw omówimy, jak działają bufory ramki, a następnie wykorzystamy je, implementując różne efekty post-processingu.

## Tworzenie bufora ramki

Podobnie jak każdy inny obiekt w OpenGL, możemy stworzyć obiekt bufora ramki (w skrócie FBO) za pomocą funkcji o nazwie <fun>glGenFramebuffers</fun>:

```cpp
    unsigned int fbo;
    glGenFramebuffers(1, &fbo);
```

Ten schemat tworzenia i używania obiektów jest czymś, co widzieliśmy dziesiątki razy, więc ich funkcje są podobne do wszystkich innych obiektów, które widzieliśmy; najpierw tworzymy obiekt framebuffer, ustawiamy go jako aktywny framebuffer, wykonujemy niektóre operacje i usuwamy framebuffer. Aby powiązać framebuffer z kontekstem (ustawić go jako domyślny framebuffer) używamy <fun>glBindFramebuffer</fun>:

```cpp
    glBindFramebuffer(GL_FRAMEBUFFER, fbo);  
```

Przez powiązanie framebuffera z opcją <var>GL_FRAMEBUFFER</var> wszystkie następne operacje _odczytu_ (ang. *read*) i _zapisu_ (ang. *write*) będą miały wpływ na aktualnie powiązany framebuffer. Możliwe jest również powiązanie bufora ramki tylko do odczytu lub zapisu, przez powiązanie z opcjami <var>GL_READ_FRAMEBUFFER</var> lub <var>GL_DRAW_FRAMEBUFFER</var>. Bufor ramki powiązany z <var>GL_READ_FRAMEBUFFER</var> jest następnie używany do wszystkich operacji odczytu, takich jak <fun>glReadPixels</fun>. Natomiast framebuffer powiązany z <var>GL_DRAW_FRAMEBUFFER</var> jest używany jako miejsce docelowe dla renderowania, usuwania i innych operacji zapisu. W większości przypadków nie trzeba wprowadzać tego rozróżnienia i zazwyczaj wiąże się framebuffer z opcją <var>GL_FRAMEBUFFER</var>, która ustawia nasz bufor ramki jednocześnie do odczytu i zapisu.

Niestety nie możemy użyć naszego framebuffera, ponieważ nie jest on <def>kompletny</def>. Aby bufor ramki był kompletny, muszą być spełnione następujące warunki:

*   Musimy dołączyć co najmniej jeden bufor (koloru, głębokości lub bufor szablonu).
*   Powinnien być przynajmniej jeden załącznik koloru (ang. *color attachment*).
*   Wszystkie załączniki powinny być również kompletne (muszą mieć zarezerwowaną pamięć).
*   Każdy bufor powinien mieć taką samą liczbę próbek.

Nie martw się, jeśli nie wiesz, co to są próbki, omówimy je w późniejszym samouczku.

Patrząc na warunki powinno być jasne, że musimy stworzyć pewien rodzaj załącznika dla bufora ramki i dołączyć go do tego bufora ramki. Po spełnieniu wszystkich wymagań możemy sprawdzić, czy faktycznie udało nam się stworzyć kompletny bufor ramki, wywołując <fun>glCheckFramebufferStatus</fun> z opcją <var>GL_FRAMEBUFFER</var>. Funkcja sprawdza aktualnie powiązany framebuffer i zwraca jedną z wartości, które opisane są w [specyfikacji](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/%67lCheckFramebufferStatus.xhtml) OpenGL. Jeśli zwracaną wartością jest <var>GL_FRAMEBUFFER_COMPLETE</var>, to wszystko zostało utworzone we właściwy sposób:

```cpp
    if(glCheckFramebufferStatus(GL_FRAMEBUFFER) == GL_FRAMEBUFFER_COMPLETE)
      // wykonaj taniec zwycięstwa
```

Wszystkie kolejne operacje renderowania będą teraz renderowane do załączników aktualnie powiązanego bufora ramki. Ponieważ nasz framebuffer nie jest domyślnym buforem ramki, polecenia renderowania nie będą miały wpływu na wizualne wyjście twojego okna. Z tego powodu proces renderowania do innego bufora ramki jest nazywany <def>renderowaniem pozaekranowym</def> (ang. *off-screen rendering*). Aby upewnić się, że wszystkie operacje renderowania będą miały wizualny wpływ na główne okno, musimy ponownie aktywować domyślny bufor ramki, wiążąc go z obiektem `0`:

```cpp
    glBindFramebuffer(GL_FRAMEBUFFER, 0);   
```

Kiedy skończyliśmy ze wszystkimi operacjami na buforze ramki, nie zapomnijmy usunąć tego obiektu:

```cpp
    glDeleteFramebuffers(1, &fbo);  
```

Teraz przed sprawdzeniem kompletności musimy dołączyć jeden lub więcej załączników do bufora ramki. <def>Załącznik</def> (ang. *attachment*) to miejsce w pamięci, które może działać jako bufor dla bufora ramki, można o nim myśleć jako o obrazie. Podczas tworzenia załącznika mamy do wyboru dwie opcje: tekstury lub obiekty <def>renderbuffer</def>.

### Załączniki tekstur

Po dołączeniu tekstury do bufora ramki wszystkie polecenia renderowania będą zapisywać dane do tej tekstury tak, jakby był to normalny bufor koloru, głębi lub szablonu. Zaletą korzystania z tekstur jest to, że wynik wszystkich operacji renderowania będzie przechowywany jako obraz tekstury, który możemy następnie łatwo wykorzystać w naszych shaderach.

Tworzenie tekstury dla bufora ramki jest w prawie takie samo jak tworzenie normalnej tekstury:

```cpp
    unsigned int texture;
    glGenTextures(1, &texture);
    glBindTexture(GL_TEXTURE_2D, texture);

    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);

    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);  
```

Główną różnicą jest to, że ustawiamy wymiary równe rozmiarowi ekranu (chociaż nie jest to wymagane) i przekazujemy `NULL` jako parametr `data` tekstury. W przypadku tej tekstury przydzielamy tylko pamięć, a nie faktycznie ją wypełniamy. Wypełnienie tekstury nastąpi zaraz po renderowaniu do bufora ramki. Zwróć też uwagę, że nie dbamy też o żadne metody zawijania tekstury ani mipmapping, ponieważ w większości przypadków nie będziemy ich potrzebować.

{: .box-note }
Jeśli chcesz renderować cały ekran do tekstury o mniejszym lub większym rozmiarze, musisz ponownie wywołać <fun>glViewport</fun> (przed renderowaniem do bufora ramki) z wymiarami twojej tekstury, w przeciwnym razie tylko mała część tekstury lub ekranu zostanie zapisana w teksturze.

Teraz, gdy stworzyliśmy teksturę, ostatnią rzeczą, którą musimy zrobić, jest podpięcie jej do bufora ramki:

```cpp
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);  
```

Funkcja <fun>glFrameBufferTexture2D</fun> przyjmuje następujące parametry:

*   `target`: typ trybu bufora ramki (do zapisu, odczytu lub do odczytu i zapisu).
*   `attachment`: typ załącznika, który chcemy dołączyć. W tej chwili dołączamy załącznik koloru. Zauważ, że `0` na końcu sugeruje, że możemy dołączyć więcej niż jeden załącznik koloru. Omówimy to w późniejszym samouczku.
*   `textarget`: typ tekstury, którą chcesz dołączyć.
*   `texture`: tekstura do dołączenia do framebuffera.
*   `level`: poziom mipmapy. W tym przypadku ustawiamy wartość `0`.

Oprócz załączników koloru możemy również dołączyć teksturę głębi i szablonu do obiektu bufora ramki. Aby dołączyć załącznik głębokości, określamy typ załącznika jako <var>GL_DEPTH_ATTACHMENT</var>. Zwróć uwagę, że <def>format</def> i <def>internal format</def> tekstury powinny zostać ustawione na <var>GL_DEPTH_COMPONENT</var>, aby odzwierciedlić format pamięci bufora głębi. Aby dołączyć bufor szablonu, użyj <var>GL_STENCIL_ATTACHMENT</var> jako drugiego argumentu i określ format tekstury jako <var>GL_STENCIL_INDEX</var>.

Możliwe jest również dołączenie zarówno bufora głębi, jak i bufora szablonu jako pojedynczej tekstury. Każda 32-bitowa wartość tekstury obejmuje 24 bity informacji o głębokości i 8 bitów informacji o szablonie. Aby dołączyć bufor głębi i szablonów jako jedną teksturę, używamy typu załącznika <var>GL_DEPTH_STENCIL_ATTACHMENT</var> i konfigurujemy formaty tekstur tak, aby zawierał połączone wartości głębi i szablonu. Przykład dołączenia bufora głębi i szablonu jako jednej tekstury do bufora ramki pokazano poniżej:

```cpp
    glTexImage2D(
      GL_TEXTURE_2D, 0, GL_DEPTH24_STENCIL8, 800, 600, 0, 
      GL_DEPTH_STENCIL, GL_UNSIGNED_INT_24_8, NULL
    );

    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_TEXTURE_2D, texture, 0);  
```

### Załączniki obiektu Renderbuffer

<span class="def">Obiekty renderbuffer</span> zostały wprowadzone do OpenGL po teksturach jako możliwy typ załączników bufora ramki. Tekstury były jedynymi załącznikami używanymi w starych czasach OpenGL. Podobnie jak obraz tekstury, obiekt renderbuffer jest rzeczywistym buforem, np. tablicą bajtów, liczb całkowitych, pikseli lub czegokolwiek innego. Obiekt renderbuffer ma tę dodatkową zaletę, że przechowuje dane w natywnym formacie OpenGL, dzięki czemu jest zoptymalizowany pod kątem renderowania pozaekranowym do bufora ramki.

Obiekty Renderbuffer przechowują wszystkie dane renderowania bezpośrednio w swoich buforach bez żadnych konwersji do formatów specyficznych dla tekstury, dzięki czemu dane są szybciej zapisywane. Jednak obiekty renderbuffer są obiektami tylko do zapisu, więc nie można ich odczytać (jak to można robić w przypadku tekstur). Można z nich odczytać dane za pomocą <fun>glReadPixels</fun>, ale zwraca ona określony obszar pikseli z aktualnie powiązanego framebuffera, a nie bezpośrednio z samego załącznika.

Ponieważ dane są już w oryginalnym formacie, to renderbuffery są szybkie przy zapisywaniu danych lub po prostu kopiowaniu danych do innych buforów. Operacje, takie jak przełączanie buforów, są dość szybkie podczas korzystania z obiektów renderbuffer. Funkcja <fun>glfwSwapBuffers</fun>, której używaliśmy na końcu każdej iteracji renderowania, może być również zaimplementowana z obiektami renderbuffer: po prostu piszemy do renderbuffer'a i na końcu zamieniamy go z drugim renderbufferem. Obiekty renderbuffer są idealne do tego rodzaju operacji.

Tworzenie obiektu renderbuffer wygląda podobnie do kodu tworzenia bufora ramki:

```cpp
    unsigned int rbo;
    glGenRenderbuffers(1, &rbo);
```

Chcemy powiązać obiekt renderbuffer, aby wszystkie kolejne operacje renderbuffer'a miały wpływ na bieżący <var>rbo</var>:

```cpp
    glBindRenderbuffer(GL_RENDERBUFFER, rbo);  
```

Ponieważ obiekty renderbuffer są generalnie tylko do zapisu, często są używane jako załączniki głębi i szablonu, ponieważ zwykle nie musimy odczytywać wartości z buforów głębi i szablonu, ale wciąż dbają one o test głębi i szablonu. **Potrzebujemy** wartości głębokości i szablonu do testów, ale nie musimy _próbkować_ tych wartości, więc obiekt renderbuffer idealnie pasuje do tego zadania. Gdy nie pobieramy próbek z tych buforów, zazwyczaj preferowany jest obiekt renderbuffer, ponieważ jest on bardziej zoptymalizowany.

Tworzenie obiektu renderbuffer z głębią i szablonem odbywa się poprzez wywołanie funkcji <fun>glRenderbufferStorage</fun>:

```cpp
    glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, 800, 600);
```

Tworzenie obiektu renderbuffer jest podobne do tworzenia obiektów tekstur, z tą różnicą, że obiekt ten został specjalnie zaprojektowany do użycia jako obraz zamiast bufora ogólnego przeznaczenia, taki jak tekstura. Tutaj wybraliśmy <var>GL_DEPTH24_STENCIL8</var> jako wewnętrzny format (ang. *internal format*), który zawiera bufor głębokości i szablonu z odpowiednio 24 i 8 bitami.

Ostatnia rzecz do zrobienia to dołączenie obiektu renderbuffer:

```cpp
    glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);  
```

Obiekty renderbuffer mogą zapewnić pewne optymalizacje w buforze ramki, ale ważne jest, aby zdać sobie sprawę, kiedy używać obiektów renderbuffer i kiedy używać tekstur. Ogólna zasada jest taka, że ​​jeśli nigdy nie będziesz potrzebować próbkować/pobierać danych z określonego bufora, dobrze jest użyć obiektu renderbuffer dla tego konkretnego bufora. Jeśli potrzebujesz pobierać dane, takie jak kolory lub wartości głębokości, powinieneś zamiast tego użyć załącznika tekstury. Jednak pod względem wydajności nie ma to ogromnego wpływu.

## Rendering do tekstury

Teraz, gdy wiemy, jak działają framebuffery, nadszedł czas, aby je dobrze wykorzystać. Zamienimy naszą scenę w kolorową teksturę dołączoną do obiektu bufora ramki, który stworzyliśmy, a następnie narysujemy tę teksturę na kwadracie obejmującym cały ekran. Wizualne wyjście jest dokładnie takie samo jak bez bufora ramki, ale tym razem wszystko jest narysowane na pojedynczym kwadracie. Dlaczego to jest przydatne? W następnej sekcji zobaczymy, dlaczego.

Pierwszą rzeczą do zrobienia jest stworzenie rzeczywistego obiektu bufora ramki i powiązanie go z kontekstem, to wszystko jest stosunkowo proste:

```cpp
    unsigned int framebuffer;
    glGenFramebuffers(1, &framebuffer);
    glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);    
```

Następnie tworzymy obraz tekstury, który załączamy jako załącznik koloru do bufora ramki. Ustawiamy wymiary tekstury równe szerokości i wysokości okna i nie inicjalizujemy danych:

```cpp
    // wygeneruj teksturę
    unsigned int texColorBuffer;
    glGenTextures(1, &texColorBuffer);
    glBindTexture(GL_TEXTURE_2D, texColorBuffer);
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR );
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glBindTexture(GL_TEXTURE_2D, 0);

    // dołącz ją do aktualnego obiektu framebuffer
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texColorBuffer, 0);  
```

Chcemy również upewnić się, że OpenGL jest w stanie wykonać test głębi (i opcjonalnie test szablonu, jeśli tego potrzebujesz), więc musimy upewnić się, że dodajemy również głębię (i szablon) do bufora ramki. Ponieważ próbkujemy tylko bufora kolorów, a innych buforów nie, to możemy w tym celu utworzyć obiekt renderbuffer. Pamiętaj, że to dobry wybór, gdy nie chcesz pobierać danych z określonych buforów.

Tworzenie obiektu renderbuffer nie jest zbyt trudne. Jedyne, o czym musimy pamiętać to to, że tworzymy go jako obiekt renderbuffer z głębią **i** szablonem. Ustawiliśmy jego wewnętrzny format na <var>GL_DEPTH24_STENCIL8</var>, który jest wystarczająco dokładny dla naszych celów.

```cpp
    unsigned int rbo;
    glGenRenderbuffers(1, &rbo);
    glBindRenderbuffer(GL_RENDERBUFFER, rbo); 
    glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, 800, 600);  
    glBindRenderbuffer(GL_RENDERBUFFER, 0);
```

Po przydzieleniu wystarczającej ilości pamięci dla obiektu renderbuffer możemy odpiąć renderbuffer od kontekstu.

Następnie, jako ostatni krok przed końcowym wypełnieniem bufora ramki, dołączamy obiekt renderbuffer do załączników głębokości **i** szablonu bufora ramki:

```cpp
    glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
```

Następnie jako końcowy krok. chcemy sprawdzić, czy framebuffer jest rzeczywiście kompletny, jeśli nie, napiszemy komunikat o błędzie.

```cpp
    if(glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE)
    	std::cout << "ERROR::FRAMEBUFFER:: Framebuffer is not complete!" << std::endl;
    glBindFramebuffer(GL_FRAMEBUFFER, 0);  
```

Następnie pamiętaj, aby odłączyć bufor ramki, aby upewnić się, że przypadkowo nie renderujemy do niewłaściwego bufora ramki.

Teraz, gdy framebuffer jest kompletny, wszystko, co musimy zrobić, aby renderować do naszego framebuffera zamiast do domyślnego framebuffera, to po prostu powiązanie naszego framebuffera z kontekstem. Wszystkie następne polecenia renderowania będą miały wpływ na aktualnie powiązany bufor ramki. Wszystkie operacje na wartościach głębi i szablonów będą również odczytywane z aktualnie powiązanych załączników głębokości i szablonu bufora ramki, jeśli są one dostępne. Jeśli na przykład pominąłeś bufor głębi, wszystkie operacje testowania głębi przestaną działać, ponieważ w aktualnie powiązanym buforze ramki nie ma bufora głębi.

Aby narysować scenę do pojedynczej tekstury, musimy wykonać następujące kroki:

1.  Wyrenderuj scenę jak zwykle z powiązanym nowym framebufferem jako aktywnym framebufferem.
2.  Powiąż domyślny bufor ramki.
3.  Narysuj kwadrat, który obejmuje cały ekran z teksturą z nowego załącznika koloru nowego bufora ramki.

Narysujemy tę samą scenę, której użyliśmy w tutorialu [test głębokości]({% post_url /learnopengl/4_advanced_opengl/2018-08-22-test-glebokosci %}), ale tym razem z starą znajomą teksturą [kontenera]({{ site.url }}/img/learnopengl/container.jpg).

Aby narysować kwadrat pełno-ekranowy, stworzymy nowy zestaw prostych shaderów. Nie uwzględnimy żadnych fantazyjnych przekształceń macierzy, ponieważ będziemy dostarczać współrzędne wierzchołków jako [znormalizowane współrzędne urządzenia](https://learnopengl.com/code_viewer.php?code=advanced/framebuffers_quad_vertices), abyśmy mogli bezpośrednio określić je jako dane wyjściowe Vertex Shader. VS wygląda następująco:

```glsl
    #version 330 core
    layout (location = 0) in vec2 aPos;
    layout (location = 1) in vec2 aTexCoords;

    out vec2 TexCoords;

    void main()
    {
        gl_Position = vec4(aPos.x, aPos.y, 0.0, 1.0); 
        TexCoords = aTexCoords;
    }  
```

Nic nadzwyczajnego. Fragment Shader będzie jeszcze prostszy, ponieważ jedyną rzeczą, którą musimy zrobić, to spróbkować teksturę:

```glsl
    #version 330 core
    out vec4 FragColor;

    in vec2 TexCoords;

    uniform sampler2D screenTexture;

    void main()
    { 
        FragColor = texture(screenTexture, TexCoords);
    }
```

Teraz to do Ciebie należy stworzenie i skonfigurowanie VAO dla pełno-ekarnowego kwadratu. Iteracja renderowania za pomocą tekstury bufora ramki wygląda następująco:

```cpp
    // pierwsze przejście
    glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);
    glClearColor(0.1f, 0.1f, 0.1f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT); // nie używamy teraz bufora szablonu
    glEnable(GL_DEPTH_TEST);
    DrawScene();	

    // drugie przejście
    glBindFramebuffer(GL_FRAMEBUFFER, 0); // powrót do domyślnego FBO
    glClearColor(1.0f, 1.0f, 1.0f, 1.0f); 
    glClear(GL_COLOR_BUFFER_BIT);

    screenShader.use();  
    glBindVertexArray(quadVAO);
    glDisable(GL_DEPTH_TEST);
    glBindTexture(GL_TEXTURE_2D, textureColorbuffer);
    glDrawArrays(GL_TRIANGLES, 0, 6);  
```

Jest kilka rzeczy do zauważenia. Po pierwsze, ponieważ każdy bufor ramek, z którego korzystamy ma własny zestaw buforów, chcemy wyczyścić każdy z tych buforów za pomocą odpowiednich masek bitowych, wywołując <fun>glClear</fun>. Po drugie, podczas rysowania pełno-ekranowego kwadratu, wyłączamy testowanie głębokości, ponieważ nie bardzo zależy nam wtedy na testowaniu głębokości, ponieważ rysujemy prosty kwadrat; będziemy musieli ponownie włączyć testowanie głębokości, gdy będziemy rysować normalną scenę.

Jest kilka rzeczy, które mogą pójść nie tak, więc jeśli nic się nie pojawia, spróbuj zdebugować kod, gdzie to tylko możliwe, i ponownie przeczytaj odpowiednie sekcje samouczka. Jeśli wszystko przebiegło pomyślnie, uzyskasz efekt podobny do tego:

![Obraz sceny 3D renderowanej do tekstury za pomocą bufora ramki](/img/learnopengl/framebuffers_screen_texture.png){: .center-image }

Lewa strona pokazuje efekt wizualny dokładnie taki, jaki widzieliśmy w samouczku [test głębokości]({% post_url /learnopengl/4_advanced_opengl/2018-08-22-test-glebokosci %}), ale tym razem wyrenderowany na prostym kwadracie. Jeśli renderujemy scenę z widoczną siatką (ang. *wireframe mode*), staje się oczywiste, że narysowaliśmy tylko jeden kwadrat w domyślnym buforze ramki.

Możesz znaleźć kod źródłowy aplikacji [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/4.advanced_opengl/5.1.framebuffers/framebuffers.cpp).

Więc jaki był z tego pożytek? Cóż, ponieważ teraz możemy swobodnie uzyskać dostęp do każdego z pikseli wyrenderowanej sceny jako pojedynczego obrazu tekstury, możemy stworzyć interesujące efekty w Fragment Shaderze. Zbiór tych wszystkich interesujących efektów nazywa się <def>post-processingiem</def>.

# Post-processing

Teraz, gdy cała scena jest renderowana do pojedynczej tekstury, możemy stworzyć interesujące efekty, po prostu manipulując danymi tekstury. W tej sekcji pokażemy niektóre z bardziej popularnych efektów post-processingu i tego, jak możesz tworzyć własne efekty z odrobiną kreatywności.

Zacznijmy od jednego z najprostszych efektów post-processingu.

### Inwersja

Mamy dostęp do każdego z wyjściowych kolorów renderowania, więc nie jest tak trudno odwrócić te kolorów w Fragment Shaderze. Pobieramy kolor tekstury ekranowej i odwracamy go przez odjęcie tego koloru od `1.0`:

```glsl
    void main()
    {
        FragColor = vec4(vec3(1.0 - texture(screenTexture, TexCoords)), 1.0);
    }  
```

Inwersja jest stosunkowo prostym efektem post-processingu, ale jest dosyć ciekawy:

![Obraz postprocesowy sceny 3D w OpenGL z odwróconymi kolorami](/img/learnopengl/framebuffers_inverse.png){: .center-image }

Cała scena ma teraz wszystkie odwrócone kolory za pomocą pojedynczej linii kodu w Fragment Shaderze. Całkiem fajnie, nie?

### Skala szarości

Kolejnym interesującym efektem jest usunięcie wszystkich kolorów ze sceny, z wyjątkiem kolorów białego, szarego i czarnego, tworząc obraz w skali szarości. Prostym sposobem na zrobienie tego jest po prostu pobranie wszystkich składników koloru i uśrednienie ich wartości:

```glsl
    void main()
    {
        FragColor = texture(screenTexture, TexCoords);
        float average = (FragColor.r + FragColor.g + FragColor.b) / 3.0;
        FragColor = vec4(average, average, average, 1.0);
    }   
```

To już daje całkiem dobre wyniki, ale ludzkie oko jest bardziej wrażliwe na zielone kolory, a najmniej na niebieskie, więc aby uzyskać jak najbardziej dokładne wyniki, musimy użyć średniej ważonej kanałów:

```glsl
    void main()
    {
        FragColor = texture(screenTexture, TexCoords);
        float average = 0.2126 * FragColor.r + 0.7152 * FragColor.g + 0.0722 * FragColor.b;
        FragColor = vec4(average, average, average, 1.0);
    }   
```

![Obraz postprocesowy sceny 3D w OpenGL z kolorami w skali szarości](/img/learnopengl/framebuffers_grayscale.png){: .center-image }

Prawdopodobnie od razu nie zauważysz różnicy, ale przy bardziej skomplikowanych scenach, ważona skali szarości będzie bardziej realistyczna.

## Efekty jądra (ang. *kernel effects*)

Kolejną zaletą post-processingu jest to, że możemy próbkować wartości kolorów z innych części tekstury. Możemy na przykład zdefiniować mały obszar wokół aktualnego teksela i próbkować wiele wartości tekstur wokół aktualnej wartości tekstury. Możemy wtedy tworzyć ciekawe efekty, łącząc je na różne sposoby.

<span class="def">Jądro</span> (lub macierz splotu) to mała macierzopodobna tablica wartości, gdzie środkowa wartość to wartość bieżącego piksela, który mnoży otaczające go wartości pikseli przez swoją wartość jądra i sumuje je razem w celu utworzenia pojedynczej wartości. Zasadniczo dodajemy małe przesunięcie do współrzędnych tekstury w otoczeniu bieżącego piksela i łączymy wyniki w oparciu o jądro. Przykład jądra podano poniżej:

$$\begin{bmatrix}2 & 2 & 2 \\ 2 & -15 & 2 \\ 2 & 2 & 2 \end{bmatrix}$$

To jądro pobiera 8 otaczających wartości piksela i mnoży je przez `2`, a bieżący piksel przez `-15`. To przykładowe jądro zasadniczo mnoży otaczające piksele przez wagę określoną w jądrze i równoważy wynik przez pomnożenie bieżącego piksela przez dużą ujemną wagę.

{: .box-note }
Większość jąder, które znajdziesz w Internecie, sumuje się do `1`, jeśli dodasz wszystkie wagi do siebie. Jeśli nie sumują się do `1`, oznacza to, że wynikowy kolor tekstury staje się jaśniejszy lub ciemniejszy niż oryginalna wartość tekstury.

Jądra są niezwykle przydatnym narzędziem post-processingu, ponieważ są dość łatwe w użyciu, eksperymentowaniu i wiele ich przykładów można znaleźć w Internecie. Musimy nieco zmodyfikować Fragment Shader, aby faktycznie obsługiwać jądra. Zakładamy, że każde jądro, z którego będziemy korzystać, jest jądrem 3x3 (większość jąder ma taki rozmiar):

```glsl
    const float offset = 1.0 / 300.0;  

    void main()
    {
        vec2 offsets[9] = vec2[](
            vec2(-offset,  offset), // lewy górny
            vec2( 0.0f,    offset), // górny środek
            vec2( offset,  offset), // prawy górny
            vec2(-offset,  0.0f),   // lewy po środku
            vec2( 0.0f,    0.0f),   // środkowy
            vec2( offset,  0.0f),   // prawy po środku
            vec2(-offset, -offset), // lewy dolny
            vec2( 0.0f,   -offset), // dolny środkowy
            vec2( offset, -offset)  // prawy dolny  
        );

        float kernel[9] = float[](
            -1, -1, -1,
            -1,  9, -1,
            -1, -1, -1
        );

        vec3 sampleTex[9];
        for(int i = 0; i < 9; i++)
        {
            sampleTex[i] = vec3(texture(screenTexture, TexCoords.st + offsets[i]));
        }
        vec3 col = vec3(0.0);
        for(int i = 0; i < 9; i++)
            col += sampleTex[i] * kernel[i];

        FragColor = vec4(col, 1.0);
    }  
```
W Fragment Shader najpierw tworzymy tablicę 9 przesunięć `vec2` dla każdej otaczającej współrzędnej tekstury. Przesunięcie to po prostu stała wartość, którą można dostosować do własnych upodobań. Następnie definiujemy jądro, które w tym przypadku jest jądrem <def>sharpen</def>, które wyostrza każdą wartość koloru, próbkując wszystkie otaczające piksele. Na koniec dodajemy każde przesunięcie do bieżącej współrzędnej tekstury podczas próbkowania, a następnie mnożymy te wartości tekstury za pomocą ważonych wartości jądra, które sumujemy.

Efekt wyostrzenia wygląda następująco:

![Wyostrzony obraz](/img/learnopengl/framebuffers_sharpen.png){: .center-image }

Może to stworzyć interesujące efekty, gdzie gracz znajduje się w jakiejś narkotycznej przygodzie.

### Rozmycie

Jądro tworzące efekt <def>rozmycia</def> (ang. *blur*) jest zdefiniowane w następujący sposób:

$$\begin{bmatrix} 1 & 2 & 1 \\ 2 & 4 & 2 \\ 1 & 2 & 1 \end{bmatrix} / 16$$

Ponieważ wszystkie wartości sumują się do 16, po prostu łączenie próbkowanych kolorów wygeneruje bardzo jasny kolor, dlatego musimy podzielić każdą wartość jądra przez `16`. Powstała tablica jądra staje się:

```cpp
    float kernel[9] = float[](
        1.0 / 16, 2.0 / 16, 1.0 / 16,
        2.0 / 16, 4.0 / 16, 2.0 / 16,
        1.0 / 16, 2.0 / 16, 1.0 / 16  
    );
```

Zmieniając tablicę kernel na typ <fun>float</fun> w Fragment Shader, całkowicie zmieniamy efekt post-processing. Teraz wygląda to mniej więcej tak:

![Rozmycie w OpenGL](/img/learnopengl/framebuffers_blur.png){: .center-image }

Taki efekt rozmycia tworzy interesujące możliwości. Możemy zmieniać wielkość rozmycia w czasie, aby na przykład stworzyć efekt upijania się lub zwiększyć rozmycie, gdy główna postać nie nosi okularów. Rozmycie daje nam również przydatne narzędzie do wygładzania wartości kolorów, które wykorzystamy w późniejszych samouczkach.

Widać, że gdy mamy już tak małą implementację jądra, łatwo jest stworzyć fajne efekty post-processingu. Pokażmy ostatni popularny efekt, aby zakończyć ten artykuł.

### Wykrywanie krawędzi

Poniżej znajduje się jądro <def>wykrywania krawędzi</def> podobne do jądra wyostrzenia:

$$\begin{bmatrix} 1 & 1 & 1 \\ 1 & -8 & 1 \\ 1 & 1 & 1 \end{bmatrix}$$

To jądro podświetla wszystkie krawędzie i przyciemnia resztę, co jest bardzo przydatne, gdy zależy nam tylko na krawędziach w obrazie.

![Wykrywanie krawędzi w OpenGL](/img/learnopengl/framebuffers_edge_detection.png){: .center-image }

Prawdopodobnie nie jest zaskoczeniem, że takie jądra są używane jako narzędzia/filtry do obróbki obrazów w narzędziach takich jak Photoshop. Ze względu na zdolność karty graficznej do przetwarzania fragmentów o ekstremalnie równoległych możliwościach, możemy z łatwością manipulować obrazami w ujęciu pikselowym w czasie rzeczywistym. Narzędzia do edycji obrazu mają tendencję do częstszego korzystania z kart graficznych do przetwarzania obrazów.