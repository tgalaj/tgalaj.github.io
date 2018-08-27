---
layout: post
title: Debugowanie
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: in-practice
---

{% include learnopengl.md link="In-Practice/Debugging" %}

Programowanie grafiki może sprawiać wiele radości, ale może być także dużym źródłem frustracji, gdy coś nie renderuje się poprawnie, a nawet nie renderuje się wcale! Ponieważ większość naszych działań polega na manipulowaniu pikselami, może być trudno znaleźć przyczynę błędu, gdy coś nie działa tak, jak powinno. Debugowanie tego rodzaju _wizualnych_ błędów jest inne niż zwykłe debugowanie błędów w normalnych programach. Nie mamy konsoli do wyprowadzania tekstu, żadnych punktów przerwania, które można ustawić w naszym kodzie GLSL, i nie można łatwo sprawdzić stanu wykonania GPU.

W tym samouczku zajmiemy się kilkoma technikami i sztuczkami debugowania twojego programu OpenGL. Debugowanie w OpenGL nie jest zbyt trudne, a zrozumienie tych technik zdecydowanie opłaca się na dłuższą metę.

## glGetError()

W momencie, gdy niepoprawnie użyjesz OpenGL (jak konfiguracja bufora bez wcześniejszego powiązania (ang. *binding*)), zostanie to zauważone i wygenerowana zostanie jedna lub więcej flag błędów użytkownika za kulisami. Możemy zapytać o te flagi błędów za pomocą funkcji o nazwie <fun>glGetError</fun>, która po prostu sprawdza ustawioną flagę błędu i zwraca wartość błędu, jeśli OpenGL został źle użyty.

```cpp
    GLenum glGetError();  
```

W momencie wywołania funkcji <fun>glGetError</fun> zwraca ona albo flagę błędu, albo nic. Kody błędów, które <fun>glGetError</fun> może zwrócić, są wymienione poniżej:

<table align="center">
  <tbody><tr>
    <th style="text-align:center;">Flaga</th>
    <th style="text-align:center;">Kod</th>
    <th style="text-align:center;">Opis</th>
  </tr>
  <tr>
    <td style="text-align:center;"><var>GL_NO_ERROR</var></td>
    <td style="text-align:center;">0</td>
    <td style="text-align:center;">Nie zgłoszono żadnego błędu użytkownika od ostatniego wywołania <fun>glGetError</fun>.</td>
  </tr>  
  <tr>
    <td style="text-align:center;"><var>GL_INVALID_ENUM</var></td>
    <td style="text-align:center;">1280</td>
    <td style="text-align:center;">Ustawiona, gdy parametr wyliczeniowy nie jest poprawny.</td>
  </tr>  
  <tr>
    <td style="text-align:center;"><var>GL_INVALID_VALUE</var></td>
    <td style="text-align:center;">1281</td>
    <td style="text-align:center;">Ustawiona, gdy parametr wartości nie jest poprawny.</td>
  </tr>  
  <tr>
    <td style="text-align:center;"><var>GL_INVALID_OPERATION</var></td>
    <td style="text-align:center;">1282</td>
    <td style="text-align:center;">Ustawiona, gdy stan polecenia nie jest poprawny dla podanych parametrów.</td>
  </tr>  
  <tr>
    <td style="text-align:center;"><var>GL_STACK_OVERFLOW</var></td>
    <td style="text-align:center;">1283</td>
    <td style="text-align:center;">Ustawiona, gdy operacja wypychania stosu powoduje przepełnienie stosu.</td>
  </tr>  
  <tr>
    <td style="text-align:center;"><var>GL_STACK_UNDERFLOW</var></td>
    <td style="text-align:center;">1284</td>
    <td style="text-align:center;">Ustawiona, gdy nastąpi operacja pobierania wartości stosu, gdy stos znajduje się w najniższym punkcie.</td>
  </tr>  
  <tr>
    <td style="text-align:center;"><var>GL_OUT_OF_MEMORY</var></td>
    <td style="text-align:center;">1285</td>
    <td style="text-align:center;">Ustawiona, gdy operacja przydzielania pamięci nie może przydzielić (wystarczającej) ilości pamięci.</td>
  </tr>  
  <tr>
    <td style="text-align:center;"><var>GL_INVALID_FRAMEBUFFER_OPERATION</var></td>
    <td style="text-align:center;">1286</td>
    <td style="text-align:center;">Ustawiana podczas odczytu lub zapisu do bufora ramki, który nie jest kompletny.</td>
  </tr>  
</tbody></table>

W dokumentacji OpenGL dla danej funkcji zawsze można znaleźć kody błędów, które funkcja generuje w momencie, gdy jest niewłaściwie używana. Na przykład, jeśli przejrzysz dokumentację funkcji [glBindTexture](http://docs.gl/gl3/glBindTextur%65), możesz znaleźć (w sekcji _Errors_) wszystkie kody błędów użytkownika, które można wygenerować.

W momencie ustawienia flagi błędu nie będą zgłaszane żadne inne flagi błędów. Ponadto, w momencie wywołania funkcji <fun>glGetError</fun>, usuwa ona wszystkie flagi błędów (lub tylko jedną, jeśli jest to w systemie rozproszonym, patrz uwaga poniżej). Oznacza to, że jeśli wywołasz <fun>glGetError</fun> raz na końcu każdej klatki i zwróci ona błąd, nie możesz wywnioskować, że był to jedyny błąd, a źródłem tego błędu mogło być dowolne miejsce w ramce.

{: .box-note }
Zauważ, że gdy OpenGL działa w systemach X11, inne kody błędów użytkowników mogą być generowane, o ile mają różne kody błędów. Wywołanie funkcji <fun>glGetError</fun> powoduje wówczas zresetowanie tylko jednej z flag kodu błędu zamiast wszystkich. Z tego powodu zaleca się wywołanie funkcji <fun>glGetError</fun> w pętli.

```cpp
    glBindTexture(GL_TEXTURE_2D, tex);
    std::cout << glGetError() << std::endl; // returns 0 (no error)

    glTexImage2D(GL_TEXTURE_3D, 0, GL_RGB, 512, 512, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
    std::cout << glGetError() << std::endl; // returns 1280 (invalid enum)

    glGenTextures(-5, textures);
    std::cout << glGetError() << std::endl; // returns 1281 (invalid value)

    std::cout << glGetError() << std::endl; // returns 0 (no error)
```

Wspaniałą rzeczą w <fun>glGetError</fun> jest to, że stosunkowo łatwo jest wskazać, gdzie może wystąpić błąd i sprawdzić poprawność korzystania z OpenGL. Powiedzmy, że dostajesz czarny ekran i nie masz pojęcia, co go powoduje: czy bufor ramki nie jest prawidłowo ustawiony? Czy zapomniałem powiązać teksturę? Wywołując <fun>glGetError</fun> we wszystkich miejscach kodu aplikacji, możesz szybko złapać pierwszy błąd OpenGL, co oznacza, że ​​przed tym wywołaniem coś poszło nie tak.

Domyślnie <fun>glGetError</fun> zwraca tylko numery błędów, które nie są łatwe do zrozumienia, chyba że zapamiętasz kody błędów. Często ma sens napisanie małej funkcji pomocniczej, aby łatwo wydrukować do konsoli ciągi błędów wraz z miejscem, w którym wywołano funkcję sprawdzania błędów:

```cpp
    GLenum glCheckError_(const char *file, int line)
    {
        GLenum errorCode;
        while ((errorCode = glGetError()) != GL_NO_ERROR)
        {
            std::string error;
            switch (errorCode)
            {
                case GL_INVALID_ENUM:                  error = "INVALID_ENUM"; break;
                case GL_INVALID_VALUE:                 error = "INVALID_VALUE"; break;
                case GL_INVALID_OPERATION:             error = "INVALID_OPERATION"; break;
                case GL_STACK_OVERFLOW:                error = "STACK_OVERFLOW"; break;
                case GL_STACK_UNDERFLOW:               error = "STACK_UNDERFLOW"; break;
                case GL_OUT_OF_MEMORY:                 error = "OUT_OF_MEMORY"; break;
                case GL_INVALID_FRAMEBUFFER_OPERATION: error = "INVALID_FRAMEBUFFER_OPERATION"; break;
            }
            std::cout << error << " | " << file << " (" << line << ")" << std::endl;
        }
        return errorCode;
    }
    #define glCheckError() glCheckError_(__FILE__, __LINE__) 
```

W przypadku, gdy nie zdajesz sobie sprawy z tego, czym są dyrektywy preprocesora `__FILE__` i `__LINE__`: zmienne te są zastępowane podczas kompilacji odpowiednią nazwą pliku i linią, w której zostały skompilowane. Jeśli zdecydujemy się na umieszczenie dużej liczby wywołań <fun>glCheckError</fun> w naszym kodzie pomocne jest dokładniejsze poznanie, które wywołanie <fun>glCheckError</fun> zwróciło błąd.

```cpp
    glBindBuffer(GL_VERTEX_ARRAY, vbo);
    glCheckError(); 
```

Da nam to następujące wyniki:

![Wyjście glGetError w debugowaniu OpenGL.](/img/learnopengl/debugging_glgeterror.png){: .center-image }

Jedną **ważną** rzeczą, którą należy wymienić, jest to, że GLEW ma błąd (już długo istniejący), w którym wywołanie <fun>glewInit()</fun> zawsze ustawia flagę błędu <var>GL_INVALID_ENUM</var>, a zatem pierwsze wywołanie <fun>glGetError</fun> zawsze zwróci kod błędu, który może być dla Ciebie zagadkowy. Aby to naprawić, po prostu wywołaj <fun>glGetError</fun> po <fun>glewInit</fun>, aby wyczyścić flagę:

```cpp
    glewInit();
    glGetError();
```

Funkcja <fun>glGetError</fun> nie pomoże ci zbytnio, ponieważ informacje, które zwraca, są raczej proste, ale często pomagają wykryć literówki lub szybko wskazać, gdzie w kodzie coś poszło nie tak; proste, ale skuteczne narzędzie w zestawie narzędzi do debugowania.

## Debug output

Rzadziej używanym, ale bardziej użytecznym narzędziem niż <fun>glCheckError</fun> jest rozszerzenie OpenGL o nazwie <def>debug output</def>, które stało się częścią core OpenGL od wersji 4.3. Dzięki rozszerzeniu debug output, OpenGL będzie bezpośrednio wysyłać użytkownikowi komunikat o błędzie lub ostrzeżeniu ze znacznie większą ilością szczegółów w porównaniu do <fun>glCheckError</fun>. Zapewnia nie tylko więcej informacji, ale może również pomóc w złapaniu błędów dokładnie tam, gdzie one wystąpią, inteligentnie za pomocą debuggera.

{: .box-note }
Debug output jest w core od wersji OpenGL 4.3, co oznacza, że ​​znajdziesz tę funkcjonalność na dowolnym komputerze, na którym działa OpenGL 4.3 lub nowszy. Jeśli nie jest dostępna ta wersja OpenGL, jego funkcjonalność można uzyskać za pomocą rozszerzenia `ARB_debug_output` lub `AMD_debug_output`. Zauważ, że OS X wydaje się nie obsługiwać funkcjonalności debug output (jak piszą ludzie w Internecie, sam tego nie przetestowałem - daj mi znać, jeśli się mylę).

Aby rozpocząć korzystanie z debug output, musimy zażądać kontekstu debug output OpenGL podczas procesu inicjalizacji. Ten proces różni się w zależności od używanego systemu okienkowego; tutaj omówimy ustawienie go na GLFW, ale na końcu znajdziesz informacje o innych systemach w sekcji _Dodatkowe materiały_.

### Debug output w GLFW

Żądanie kontekstu debugowania w GLFW jest zaskakująco łatwe, ponieważ wszystko, co musimy zrobić, to przekazać GLFW wskazówkę (ang. *hint*), że chcielibyśmy mieć kontekst debug output. Musimy to zrobić przed wywołaniem <fun>glfwCreateWindow</fun>:

```cpp
    glfwWindowHint(GLFW_OPENGL_DEBUG_CONTEXT, GL_TRUE);  
```

Po zainicjowaniu GLFW powinniśmy mieć kontekst debugowania, jeśli używamy OpenGL w wersji 4.3 lub wyższej. W przeciwnym razie musimy poprosić o debug output za pomocą rozszerzeń OpenGL.

{: .box-note }
Używanie OpenGL w kontekście debugowania może być znacznie wolniejsze w porównaniu do kontekstu bez debugowania, więc podczas pracy nad optymalizacją aplikacji, chcesz usunąć lub za komentować wskazówkę dotyczącą kontekstu debugowania GLFW.

Aby sprawdzić, czy udało nam się zainicjować kontekst debugowania, możemy zapytać o to OpenGL:

```cpp
    GLint flags; glGetIntegerv(GL_CONTEXT_FLAGS, &flags);
    if (flags & GL_CONTEXT_FLAG_DEBUG_BIT)
    {
        // initialize debug output 
    }
```

Sposób działania debug output polega na tym, że przekazujemy OpenGL wywołanie zwrotne funkcji (ang. *callback*) rejestrowania błędów (podobne do wywołań I/O GLFW), a w funkcji wywołania zwrotnego możemy przetwarzać dane o błędach OpenGL zgodnie z naszymi oczekiwaniami; w naszym przypadku będziemy wyświetlać przydatne dane o błędach w konsoli. Poniżej znajduje się prototyp funkcji wywołania zwrotnego, którego OpenGL oczekuje dla debug output:

```cpp
    void APIENTRY glDebugOutput(GLenum source, GLenum type, GLuint id, GLenum severity, 
                                GLsizei length, const GLchar *message, void *userParam);
```

Zauważ, że w niektórych implementacjach OpenGL oczekuje, że ostatni parametr będzie typu `const void*` zamiast `void*`.

Biorąc pod uwagę duży zestaw danych, które mamy do użycia, możemy stworzyć przydatne narzędzie do wypisywania błędów, takie jak poniżej:

```cpp
    void APIENTRY glDebugOutput(GLenum source, 
                                GLenum type, 
                                GLuint id, 
                                GLenum severity, 
                                GLsizei length, 
                                const GLchar *message, 
                                void *userParam)
    {
        // ignore non-significant error/warning codes
        if(id == 131169 || id == 131185 || id == 131218 || id == 131204) return; 

        std::cout << "---------------" << std::endl;
        std::cout << "Debug message (" << id << "): " <<  message << std::endl;

        switch (source)
        {
            case GL_DEBUG_SOURCE_API:             std::cout << "Source: API"; break;
            case GL_DEBUG_SOURCE_WINDOW_SYSTEM:   std::cout << "Source: Window System"; break;
            case GL_DEBUG_SOURCE_SHADER_COMPILER: std::cout << "Source: Shader Compiler"; break;
            case GL_DEBUG_SOURCE_THIRD_PARTY:     std::cout << "Source: Third Party"; break;
            case GL_DEBUG_SOURCE_APPLICATION:     std::cout << "Source: Application"; break;
            case GL_DEBUG_SOURCE_OTHER:           std::cout << "Source: Other"; break;
        } std::cout << std::endl;

        switch (type)
        {
            case GL_DEBUG_TYPE_ERROR:               std::cout << "Type: Error"; break;
            case GL_DEBUG_TYPE_DEPRECATED_BEHAVIOR: std::cout << "Type: Deprecated Behaviour"; break;
            case GL_DEBUG_TYPE_UNDEFINED_BEHAVIOR:  std::cout << "Type: Undefined Behaviour"; break; 
            case GL_DEBUG_TYPE_PORTABILITY:         std::cout << "Type: Portability"; break;
            case GL_DEBUG_TYPE_PERFORMANCE:         std::cout << "Type: Performance"; break;
            case GL_DEBUG_TYPE_MARKER:              std::cout << "Type: Marker"; break;
            case GL_DEBUG_TYPE_PUSH_GROUP:          std::cout << "Type: Push Group"; break;
            case GL_DEBUG_TYPE_POP_GROUP:           std::cout << "Type: Pop Group"; break;
            case GL_DEBUG_TYPE_OTHER:               std::cout << "Type: Other"; break;
        } std::cout << std::endl;

        switch (severity)
        {
            case GL_DEBUG_SEVERITY_HIGH:         std::cout << "Severity: high"; break;
            case GL_DEBUG_SEVERITY_MEDIUM:       std::cout << "Severity: medium"; break;
            case GL_DEBUG_SEVERITY_LOW:          std::cout << "Severity: low"; break;
            case GL_DEBUG_SEVERITY_NOTIFICATION: std::cout << "Severity: notification"; break;
        } std::cout << std::endl;
        std::cout << std::endl;
    }
```

Kiedy debug output wykryje błąd OpenGL, wywołaja tę funkcję zwrotną i będziemy mogli wypisać dużą ilość informacji dotyczących błędu OpenGL. Zauważ, że zignorowaliśmy kilka kodów błędów, które zwykle nie wyświetlają niczego użytecznego (np. `131185` w sterownikach NVidia, które mówią nam, że bufor został pomyślnie utworzony).

Teraz, gdy mamy funkcję wywołania zwrotnego, nadszedł czas na zainicjowanie debug output:

```cpp
    if (flags & GL_CONTEXT_FLAG_DEBUG_BIT)
    {
        glEnable(GL_DEBUG_OUTPUT);
        glEnable(GL_DEBUG_OUTPUT_SYNCHRONOUS); 
        glDebugMessageCallback(glDebugOutput, nullptr);
        glDebugMessageControl(GL_DONT_CARE, GL_DONT_CARE, GL_DONT_CARE, 0, nullptr, GL_TRUE);
    } 
```

Tutaj mówimy OpenGL, aby włączył debug output. Wywołanie <fun>glEnable(GL_DEBUG_SYNCRHONOUS)</fun> mówi OpenGL, aby bezpośrednio wywoływał funkcję zwrotną w momencie wystąpienia błędu.

### Filtrowanie debug output

Dzięki <fun>glDebugMessageControl</fun> możesz potencjalnie filtrować typy błędów, o których chcesz otrzymać wiadomość. W naszym przypadku zdecydowaliśmy się nie filtrować żadnych źródeł, typów ani wskaźników surowości/wagę (ang. *severity*). Gdybyśmy chcieli wyświetlać tylko wiadomości z interfejsu OpenGL API, które są błędami i mają wysoką wagę, powinniśmy skonfigurować go w następujący sposób:

```cpp
    glDebugMessageControl(GL_DEBUG_SOURCE_API, 
                          GL_DEBUG_TYPE_ERROR, 
                          GL_DEBUG_SEVERITY_HIGH,
                          0, nullptr, GL_TRUE); 
```

Biorąc pod uwagę naszą konfigurację i zakładając, że masz kontekst obsługujący dane wyjściowe debugowania, każde niepoprawne polecenie OpenGL wydrukuje teraz duży pakiet przydatnych danych:

![Wyjście danych debugowania OpenGL na konsoli tekstowej.](/img/learnopengl/debugging_debug_output.png){: .center-image }

### Śledzenie źródła błędu

Kolejną świetną sztuczką z debug output jest to, że możesz stosunkowo łatwo pobrać dokładny numer linii lub wywołać błąd. Ustawiając punkt przerwania (ang. *breakpoint*) w <fun>DebugOutput</fun> przy określonym typie błędu (lub na górze funkcji, jeśli nie obchodzi cię typ błędu), debugger wychwyci błąd i możesz przenieść stos wywołania do dowolnej funkcji, która spowodowała wysłanie wiadomości:

![Ustawianie breakpointa i używanie stosu wywołań w OpenGL w celu wychwycenia linii błędu w debug output.](/img/learnopengl/debugging_debug_output_breakpoint.png){: .center-image }

Wymaga to ręcznej interwencji, ale jeśli z grubsza wiesz, czego szukasz, niezwykle przydatne jest szybkie ustalenie, które wywołanie powoduje błąd.

### Niestandardowe wyjście błędów

Oprócz odczytywania wiadomości, możemy również przekazać wiadomości do systemu debug output za pomocą <fun>glDebugMessageInsert</fun>:

```cpp
    glDebugMessageInsert(GL_DEBUG_SOURCE_APPLICATION, GL_DEBUG_TYPE_ERROR, 0,                       
                         GL_DEBUG_SEVERITY_MEDIUM, -1, "error message here"); 
```

Jest to szczególnie przydatne, jeśli podłączasz się do innej aplikacji lub kodu OpenGL, który wykorzystuje kontekst debug output. Inni programiści mogą szybko wykryć każdy zgłoszony błąd występujący w twoim niestandardowym kodzie OpenGL.

Podsumowując, debug output (jeśli można z niego korzystać) jest niezwykle przydatny do szybkiego wychwytywania błędów i jest wart wysiłku jeżeli chodzi o konfigurację, ponieważ oszczędza to czas podczas pisania programu. Możesz znaleźć kopię kodu źródłowego [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/7.in_practice/1.debugging/debugging.cpp) zarówno z <fun>glGetError</fun> i skonfigurowanym kontekstem debug output; sprawdź, czy możesz naprawić wszystkie błędy.

## Debugowanie wyjścia shaderów

Jeśli chodzi o GLSL, niestety nie mamy dostępu do funkcji takich jak <fun>glGetError</fun> ani możliwości przejrzenia krok po kroku kodu shadera. Kiedy dostaniesz czarny ekran lub całkowicie błędne obrazy, często trudno jest zorientować się, co jest nie tak z kodem shadera. Tak, mamy raporty błędów kompilacji, które zgłaszają błędy składniowe, ale złapanie błędów semantycznych to inna sprawa.

Często używaną sztuczką, aby dowiedzieć się, co jest nie tak z shaderem, jest ocena wszystkich istotnych zmiennych w programie cieniującym poprzez wysłanie ich bezpośrednio do kanału wyjściowego Fragment Shadera. Poprzez wysyłanie zmiennych shadera bezpośrednio do wyjściowych kanałów kolorów, często możemy przekazać interesujące informacje, sprawdzając wyniki wizualne. Na przykład, powiedzmy, że chcemy sprawdzić, czy model ma poprawne wektory normalne, możemy przekazać je (przekształcone lub nie) z Vertex Shadera do Fragment Shadera, w którym następnie wyprowadzilibyśmy normalne w następujący sposób:

```glsl
    #version 330 core
    out vec4 FragColor;
    in vec3 Normal;
    [...]

    void main()
    {
        [...]
        FragColor.rgb = Normal;
        FragColor.a = 1.0f;
    }
```

Wyprowadzając zmienną (nie opisującą koloru) do wyjściowego kanału koloru, możemy szybko sprawdzić, czy zmienna wyświetla prawidłowe wartości. Jeśli na przykład wynik wizualny jest całkowicie czarny, jasne jest, że wektory normalne nie są poprawnie przekazywane do shaderów; a kiedy są wyświetlane, względnie łatwo sprawdzić, czy są poprawne, czy nie:

![Obraz modelu 3D z jego wektorami normalnymi wyświetlanymi jako wyjście Fragment Shadera w OpenGL w celu debugowania](/img/learnopengl/debugging_glsl_output.png){: .center-image }

Z wyników wizualnych widzimy, że wektory normalne wydają się być poprawne, ponieważ prawa strona modelu nanokombinezonu ma głównie kolor czerwony (co oznaczałoby, że normalne są skierowane z grubsza (poprawnie) w kierunku dodatniej osi x) i podobnie przednia strona nanokombinezon jest zabarwiona w kierunku dodatniej osi z (niebieski).

Takie podejście można łatwo rozszerzyć na dowolny typ zmiennej, którą chcesz przetestować. Za każdym razem, gdy utkniesz i podejrzewasz, że coś jest nie tak z twoimi shaderami, spróbuj wyświetlić wiele zmiennych i/lub wyników pośrednich, aby zobaczyć, w której części algorytmu coś jest niepoprawne.

## Kompilator referencyjny GLSL OpenGL

Każdy sterownik ma swoje własne dziwactwa i ciekawostki; na przykład sterowniki NVIDIA są mniej restrykcyjne i przeoczają pewne ograniczenia specyfikacji, podczas gdy sterowniki ATI/AMD mają tendencję do lepszego egzekwowania specyfikacji OpenGL (co jest moim zdaniem lepszym rozwiązaniem). Problem polega na tym, że shadery na jednym komputerze mogą nie działać z powodu różnic między sterownikami.

Dzięki kilkunastoletniemu doświadczeniu w końcu poznasz drobne różnice między dostawcami GPU, ale jeśli chcesz mieć pewność, że twój kod shadera działa na wszystkich rodzajach maszyn, możesz bezpośrednio sprawdzić swój kod shadera względem oficjalnej specyfikacji używając [referencyjnego kompilatora](https://www.khronos.org/opengles/sdk/tools/Reference-Compiler/) GLSL OpenGL. Możesz pobrać tak zwane pliki binarne <def>GLSL lang validator</def> [tutaj](https://www.khronos.org/opengles/sdk/tools/Reference-Compiler/) lub jego pełen kod źródłowy [tutaj](https://github.com/KhronosGroup/glslang).

Biorąc pod uwagę binarny walidator języka GLSL, możesz łatwo sprawdzić kod shadera, przekazując go jako pierwszy argument. Należy pamiętać, że walidator GLSL lang określa typ shadera według listy stałych rozszerzeń:

*   `.vert`: vertex shader.
*   `.frag`: fragment shader.
*   `.geom`: geometry shader.
*   `.tesc`: tessellation control shader.
*   `.tese`: tessellation evaluation shader.
*   `.comp`: compute shader.

Uruchamianie kompilatora referencyjnego GLSL jest tak proste, jak:

```
    glsllangvalidator shaderFile.vert  
```

Zauważ, że jeśli nie wykryje błędu, nie zwraca on żadnych danych wyjściowych. Uruchomienie kompilatora referencyjnego GLSL na niepoprawnym kodzie Vertex Shadera daje następujące wyniki:

![Output of the GLSL reference compiler (GLSL lang validator) in OpenGL](/img/learnopengl/debugging_glsl_reference_compiler.png){: .center-image }

Nie pokaże ci subtelnych różnic pomiędzy kompilatorami GLSL AMD, NVidia lub Intela, ani nie pomoże ci całkowicie usunąć wszystkie błędy z twoich shaderów, ale przynajmniej pomoże ci sprawdzić twoje shadery względem specyfikacji GLSL.

## Wyjście Framebuffera

Inną użyteczną sztuczką do zestawu narzędzi do debugowania jest wyświetlanie zawartości bufora ramki w pewnym predefiniowanym regionie twojej aplikacji OpenGL. Prawdopodobnie będziesz często korzystał z [framebufferów]({% post_url /learnopengl/4_advanced_opengl/2018-08-31-framebuffers %}), a ponieważ większość ich magii dzieje się za kulisami, czasami trudno jest zorientować się, co się dzieje. Wyświetlanie zawartości ramki bufora ramki w aplikacji jest przydatną opcją pozwalającą szybko sprawdzić, czy wszystko wygląda poprawnie.

{: .box-note }
Zauważ, że wyświetlanie zawartości (załączników) bufora ramki, jak wyjaśniono tutaj, działa tylko na załącznikach tekstur, a nie obiektach bufora renderowania (ang. *renderbuffer*).

Za pomocą prostego shadera wyświetlającego tylko teksturę możemy łatwo napisać małą funkcję pomocniczą, aby szybko wyświetlić dowolną teksturę w prawym górnym rogu ekranu:

```glsl
    // vertex shader
    #version 330 core
    layout (location = 0) in vec2 position;
    layout (location = 1) in vec2 texCoords;

    out vec2 TexCoords;

    void main()
    {
        gl_Position = vec4(position, 0.0f, 1.0f);
        TexCoords = texCoords;
    }

    // fragment shader
    #version 330 core
    out vec4 FragColor;
    in  vec2 TexCoords;

    uniform sampler2D fboAttachment;

    void main()
    {
        FragColor = texture(fboAttachment, TexCoords);
    } 
```

```cpp
    void DisplayFramebufferTexture(GLuint textureID)
    {
        if(!notInitialized)
        {
            // initialize shader and vao w/ NDC vertex coordinates at top-right of the screen
            [...]
        }

        glActiveTexture(GL_TEXTURE0);  	
        glUseProgram(shaderDisplayFBOOutput);
            glBindTexture(GL_TEXTURE_2D, textureID);
            glBindVertexArray(vaoDebugTexturedRect);
                glDrawArrays(GL_TRIANGLES, 0, 6);
            glBindVertexArray(0);
        glUseProgram(0);
    }

    int main()
    {
        [...]
        while (!glfwWindowShouldClose(window))
        {
            [...]
            DisplayFramebufferTexture(fboAttachment0);

            glfwSwapBuffers(window);
        }
    }  
```

Daje to ładne małe okienko w rogu ekranu do debugowania wyjścia bufora ramki. Przydatne, na przykład, do określenia, czy wektory normalne przejścia geometrii w odroczonym rendererze wyglądają poprawnie:

![Dołączanie ramki bufora do tekstury w celu debugowania w OpenGL](/img/learnopengl/debugging_fbo_output.png){: .center-image }

Można oczywiście rozszerzyć taką funkcję pomocniczą, aby obsługiwała renderowanie więcej niż jednej tekstury. Jest to szybki i mało elegancki sposób na uzyskanie ciągłej informacji zwrotnej wszystkiego, co znajduje się w buforze ramki.

## Zewnętrzne oprogramowanie do debugowania

Kiedy wszystko inne zawiedzie, nadal istnieje możliwość skorzystania z narzędzia innej firmy, aby pomóc nam w naszych działaniach związanych z debugowaniem. Aplikacje innych firm często wstrzykują siebie do sterownika OpenGL i są w stanie przechwytywać wszystkie rodzaje wywołań OpenGL, aby zapewnić szeroki wachlarz interesujących danych dotyczących twojej aplikacji OpenGL. Narzędzia te mogą pomóc na wiele sposobów: profilowanie użycia funkcji OpenGL, znajdowanie wąskich gardeł, sprawdzanie pamięci bufora i wyświetlanie tekstur oraz załączników bufora ramki. Kiedy pracujesz nad (dużym) projektem, tego rodzaju narzędzia mogą stać się nieocenione w procesie tworzenia aplikacji.

Poniżej wymieniono niektóre z bardziej popularnych narzędzi do debugowania; wypróbuj kilka z nich, aby zobaczyć, które najlepiej pasuje do Twoich potrzeb.

### RenderDoc

[RenderDoc](https://github.com/baldurk/renderdoc) jest świetnym (całkowicie open source'owym) samodzielnym narzędziem do debugowania. Aby rozpocząć przechwytywanie, określ plik wykonywalny (.exe), który chcesz przechwycić, oraz katalog roboczy. Aplikacja działa tak jak zwykle, a gdy chcesz przejrzeć konkretną ramkę, RenderDoc przechwytuje jedną lub więcej klatek w bieżącym stanie pliku wykonywalnego. W przechwyconych klatkach możesz zobaczyć stan potoku, wszystkie polecenia OpenGL, pamięć bufora i używane tekstury.

![Obraz RenderDoc działający na aplikacji OpenGL.](/img/learnopengl/debugging_external_renderdoc.png){: .center-image }

### CodeXL

[CodeXL](https://gpuopen.com/compute-product/codexl/) to narzędzie do debugowania GPU wydane zarówno jako samodzielne narzędzie, jak i wtyczka do Visual Studio. CodeXL zapewnia dobry zestaw informacji i doskonale nadaje się do profilowania aplikacji graficznych. CodeXL działa również na kartach NVidia lub Intel, ale bez obsługi debugowania Open**CL**.

![Obraz kodu CodeXL działającego na aplikacji OpenGL.](/img/learnopengl/debugging_external_codexl.png){: .center-image }

Nie mam dużego doświadczenia z korzystaniem z CodeXL, ponieważ osobiście uznałem, że RenderDoc jest łatwiejszy w użyciu, ale dodałem go mimo wszystko, ponieważ wygląda na całkiem solidne narzędzie i został głównie opracowany przez jednego z większych producentów GPU.

### NVIDIA Nsight

Popularne narzędzie NVIDIA [Nsight](https://developer.nvidia.com/nvidia-nsight-visual-studio-edition) do debugowania GPU nie jest samodzielnym narzędziem, ale wtyczką do Visual Studio IDE lub Eclipse IDE. Wtyczka Nsight jest niesamowicie przydatnym narzędziem dla programistów grafiki, ponieważ zapewnia wiele statystyk dotyczących czasu pracy procesora oraz stanu GPU klatka po klatce.

W momencie uruchomienia aplikacji z poziomu Visual Studio (lub Eclipse) przy użyciu poleceń debugowania lub profilowania Nsight  będzie działać w samej aplikacji. Wspaniałą cechą programu NSight jest to, że renderuje ona nakładki GUI z poziomu aplikacji, które można wykorzystać do gromadzenia wszelkiego rodzaju interesujących informacji o aplikacji, zarówno w czasie wykonywania, jak i podczas analizy klatka po klatce.

![Obraz Nsight działający na aplikacji OpenGL.](/img/learnopengl/debugging_external_nsight.png){: .center-image }

Nsight to niezwykle użyteczne narzędzie, które moim zdaniem przewyższa inne narzędzia wymienione powyżej, ale ma jedną poważną wadę - działa tylko na kartach NVIDIA. Jeśli pracujesz na kartach NVIDIA (i korzystasz z Visual Studio), zdecydowanie warto dać Nsight szansę.

Jestem pewien, że jest jeszcze kilka innych narzędzi do debugowania (niektóre, które przychodzą mi na myśl, to narzędzie Valve'a [VOGL](https://github.com/ValveSoftware/vogl) i [APItrace](https://apitrace.github.io/)), ale uważam, że ta lista powinna już dawać ci mnóstwo narzędzi do wyboru. Nie jestem ekspertem w żadnym z wyżej wymienionych narzędzi, więc daj mi znać w komentarzach, jeśli podałem gdzieś błędne informacje, a ja w razie potrzeby z radością je poprawię.

## Dodatkowe materiały

*   [Why is your code producing a black window](http://retokoradi.com/2014/04/21/opengl-why-is-your-code-producing-a-black-window/): lista ogólnych przyczyn autorstwa Reto Koradi, dlaczego ekran może nie generować żadnych wyników.
*   [Debug Output](http://vallentinsource.com/opengl/debug-output): rozbudowany artykuł o debug output autorstwa Vallentin Source ze szczegółowymi informacjami na temat konfigurowania kontekstu debugowania w wielu systemach okienkowych.