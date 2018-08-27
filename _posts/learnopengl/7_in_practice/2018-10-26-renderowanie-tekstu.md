---
layout: post
title: Renderowanie tekstu
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: in-practice
---

{% include learnopengl.md link="In-Practice/Text-Rendering" %}

Na pewnym etapie swoich przygód graficznych będziesz chciał narysować tekst w OpenGL. W przeciwieństwie do tego, czego można się spodziewać, uzyskanie prostego ciągu do renderowania na ekranie jest dość trudne w przypadku biblioteki niskiego poziomu, takiej jak OpenGL. Jeśli nie zależy Ci na renderowaniu ponad 128 różnych znaków, to prawdopodobnie nie będzie to zbyt trudne. Sprawy stają się coraz trudniejsze, gdy tylko każdy znak ma inną szerokość, wysokość i margines. Na podstawie tego, gdzie mieszkasz, możesz potrzebować więcej niż 128 znaków, a co jeśli chcesz wyrazić specjalne symbole dla podobnych wyrażeń matematycznych lub symboli nutowych, a co z renderowaniem tekstu od góry do dołu? Gdy pomyślisz o tych wszystkich skomplikowanych sprawach z tekstem, nie zdziwi Cię to, że prawdopodobnie nie należy to do zadań interfejsu API niskiego poziomu, takiego jak OpenGL.

Ponieważ w OpenGL nie ma wsparcia dla jakichkolwiek funkcji tekstowych, to od nas zależy, czy zdefiniujemy system do renderowania tekstu na ekranie. Ponieważ nie ma graficznych prymitywów dla znaków tekstowych, musimy coś wymyślić. Niektóre przykładowe techniki to: rysowanie kształtów liter za pośrednictwem <var>GL_LINES</var>, tworzenie siatek 3D liter lub renderowanie tekstur znaków do kwadratów w środowisku 3D.

Najczęściej programiści decydują się na renderowanie tekstur znaków na kwadratach. Samo renderowanie tych oteksturowanych kwadratów nie powinno być zbyt trudne, ale uzyskanie odpowiednich znaków na teksturach może okazać się trudne. W tym samouczku omówimy kilka metod i zaimplementujemy bardziej zaawansowaną, ale bardziej elastyczną technikę renderowania tekstu za pomocą biblioteki FreeType.

## Klasyczne renderowanie tekstu: czcionki bitmapowe

Na początku, renderowanie tekstu polegało na wybraniu czcionki (lub samodzielnego utworzenia), jakiej chciałbyś użyć dla swojej aplikacji, a następnie trzeba było wyodrębnić wszystkie istotne znaki z tej czcionki, aby wkleić je do jednej dużej tekstury. Taka tekstura, którą od teraz nazywamy <def>czcionką bitmapową</def>, zawiera wszystkie symbole znaków, które chcemy użyć w predefiniowanych regionach tekstury. Te symbole znaków czcionki są znane jako <def>glify</def>. Każdy glif ma powiązany z nim region współrzędnych tekstury. Za każdym razem, gdy chcesz renderować znak, wybierasz odpowiedni glif, renderując tę ​​część czcionki bitmapowej do kwadratu.

![Arkusz znaków](/img/learnopengl/bitmapfont.png){: .center-image }

Tutaj możesz zobaczyć, jak renderujemy tekst `OpenGL`, biorąc czcionkę bitmapową i próbkując odpowiednie glify z tekstury (przez ostrożny wybór współrzędnych tekstury), które wyświetlamy na kilku kwadratach. Włączając [blending]({% post_url /learnopengl/4_advanced_opengl/2018-08-27-blending %}) i utrzymując przezroczystość tła, otrzymamy tylko ciąg znaków wyświetlanych na ekranie. Ta konkretna czcionka bitmapowa została wygenerowana przy użyciu [Generator czcionek](http://www.codehead.co.uk/cbfg/) bitmapowych Codehead'a.

To podejście ma kilka zalet i wad. Po pierwsze, jest stosunkowo łatwe w implementacji, a ponieważ czcionki bitmapowe są wstępnie rasteryzowane, są dość wydajne. Jednak, to podejście nie jest szczególnie elastyczne. Jeśli chcesz użyć innej czcionki, musisz skompilować kompletną nową czcionkę bitmapową, a system jest ograniczony do jednej rozdzielczości; powiększanie szybko pokazuje piksele na krawędziach. Ponadto często ogranicza się do małego zestawu znaków, więc znaki rozszerzone lub znaki Unicode często nie wchodzą w grę.

Takie podejście było dość popularne, ponieważ było szybkie i działało na każdej platformie, ale dzisiaj istnieją bardziej elastyczne podejścia. Jednym z takich podejść jest ładowanie czcionek TrueType za pomocą biblioteki FreeType.

## Współczesne renderowanie tekstu: FreeType

FreeType to biblioteka programistyczna, która może ładować czcionki, renderować je do map bitowych i zapewniać wsparcie dla kilku operacji związanych z czcionkami. Jest to popularna biblioteka używana przez Mac OS X, Java, PlayStation Console, Linux i Android. To, co sprawia, że ​​FreeType jest szczególnie atrakcyjne, to możliwość ładowania czcionek TrueType.

Czcionka TrueType to zbiór glifów znakowych, które nie są zdefiniowane przez piksele ani żadne inne nieskalowane rozwiązanie, ale przez równania matematyczne (kombinacje splajnów/krzywych). Podobnie jak w przypadku obrazów wektorowych, zrasteryzowane obrazy czcionek mogą być generowane metodycznie na podstawie preferowanej wysokości czcionki, w której chcesz ją uzyskać. Używając czcionek TrueType możesz łatwo renderować glify znaków o różnych rozmiarach bez utraty jakości.

FreeType można pobrać z tej [strony internetowej](http://www.freetype.org/). Możesz wybrać samodzielne kompilowanie biblioteki z kodu źródłowego lub użyć jednej z ich skompilowanych bibliotek, jeśli Twoja platforma docelowa znajduje się na liście. Pamiętaj, aby zlinkować `freetype.lib` i upewnić się, że Twój kompilator wie, gdzie znaleźć pliki nagłówkowe.

Następnie dodaj odpowiednie nagłówki:

```cpp
    #include <ft2build.h>
    #include FT_FREETYPE_H  
```

{: .box-error }
Ze względu na to, jak rozwija się FreeType (przynajmniej w chwili pisania tego tekstu), nie możesz umieścić ich plików nagłówkowych w nowym katalogu; powinny znajdować się w katalogu głównym twojego katalogu include. Dołączenie nagłówków FreeType, jak np. `#include <FreeType/ft2build.h>` prawdopodobnie spowoduje kilka konfliktów nagłówków.

To co robi FreeType to ładuje czcionki TrueType i dla każdego glifu generuje obraz bitmapowy i oblicza kilka metryk. Możemy wyodrębnić te obrazy bitmapowe do generowania tekstur i odpowiednio rozmieścić każdy znak glifów za pomocą załadowanych metryk.

Aby załadować czcionkę, musimy jedynie zainicjować bibliotekę FreeType i załadować czcionkę jako <def>face</def>. Tutaj ładujemy plik czcionki TrueType `arial.ttf`, który został skopiowany z katalogu `Windows/Fonts`.

```cpp
    FT_Library ft;
    if (FT_Init_FreeType(&ft))
        std::cout << "ERROR::FREETYPE: Could not init FreeType Library" << std::endl;

    FT_Face face;
    if (FT_New_Face(ft, "fonts/arial.ttf", 0, &face))
        std::cout << "ERROR::FREETYPE: Failed to load font" << std::endl;  
```

Każda z tych funkcji FreeType zwraca niezerową liczbę całkowitą, gdy wystąpi błąd.

Po załadowaniu face'a, powinniśmy zdefiniować rozmiar czcionki, który chcemy wyodrębnić:

```cpp
    FT_Set_Pixel_Sizes(face, 0, 48);  
```

Funkcja ustawia parametry szerokości i wysokości czcionki. Ustawienie szerokości na `0` pozwala dynamicznie obliczyć szerokość na podstawie podanej wysokości.

Face FreeType'a zawiera kolekcję glifów. Możemy ustawić jeden z tych glifów jako aktywny glif, wywołując <fun>FT_Load_Char</fun>. Tutaj wybieramy załadowanie glifu znaku `X`:

```cpp
    if (FT_Load_Char(face, 'X', FT_LOAD_RENDER))
        std::cout << "ERROR::FREETYTPE: Failed to load Glyph" << std::endl;  
```

Ustawiając <var>FT_LOAD_RENDER</var> jako jedną z flag ładujących, mówimy FreeType, aby utworzył dla nas 8-bitowy obraz bitmapowy w skali szarości, do którego możemy uzyskać dostęp za pomocą `face->glyph->bitmap`.

Każdy z glifów, które ładujemy za pomocą FreeType, nie ma jednak tego samego rozmiaru (tak jak w przypadku czcionek bitmapowych). Obraz bitmapowy generowany przez FreeType jest na tyle duży, że zawiera widoczną część znaku. Na przykład obraz bitmapowy znaku kropki `.` jest znacznie mniejszy niż obraz bitmapowy znaku `X`. Z tego powodu FreeType ładuje również kilka danych, które określają, jak duży powinien być każdy znak i jak prawidłowo go pozycjonować. Poniżej znajduje się obraz z FreeType, który pokazuje wszystkie dane, które oblicza dla każdego glifu.

![Obraz miar glifów załadowanych przez FreeType](/img/learnopengl/glyph.png){: .center-image }

Każdy z glifów znajduje się na poziomej <def>linii bazowej</def> (ang. *baseline*) (jak pokazano strzałką poziomą), gdzie niektóre glify znajdują się dokładnie na szczycie tej linii bazowej (np. `X`) lub nieco poniżej linii podstawowej (np. `g` lub `p`). Te metryki definiują dokładne przesunięcia, aby właściwie ustawić każdy glif na linii bazowej, jak duży powinien być każdy glif i ile pikseli potrzebujemy, aby przejść do następnego glifu. Poniżej znajduje się niewielka lista tych właściwości, których będziemy potrzebować.

*   **width**: szerokość (w pikselach) bitmapy, do której można uzyskać dostęp poprzez `face->glyph->bitmap.width`.
*   **height**: wysokość (w pikselach) bitmapy, do której można uzyskać dostęp poprzez `face->glyph->bitmap.rows`.
*   **bearingX**: pozioma orientacja, np. położenie poziome (w pikselach) mapy bitowej względem początku (ang. *origin*) glifa, do którego można uzyskać dostęp poprzez `face->glyph->bitmap_left`. 
*   **bearingY**: pionowa orientacja, np. położenie pionowe (w pikselach) mapy bitowej względem linii bazowej, do którego można uzyskać dostęp poprzez `face->glyph->bitmap_top`.
*   **advance**: poziome przesunięcie, np. odległość w poziomie (w 1/64 piksela) od początku danego glifu do początku następnego glifa. Dostępny poprzez `face->glyph->advance.x`.

Moglibyśmy załadować glif, pobrać jego metryki i wygenerować teksturę za każdym razem, gdy chcemy renderować znak na ekranie, ale byłoby to nieefektywne, aby robić to za każdym razem. Wolimy przechowywać wygenerowane dane gdzieś w aplikacji i odpytać je, gdy chcemy renderować znak. Zdefiniujemy wygodną `struct`, którą będziemy przechowywać w <fun>map</fun>.

```cpp
    struct Character {
        GLuint     TextureID;  // ID handle of the glyph texture
        glm::ivec2 Size;       // Size of glyph
        glm::ivec2 Bearing;    // Offset from baseline to left/top of glyph
        GLuint     Advance;    // Offset to advance to next glyph
    };

    std::map<GLchar, Character> Characters;
```

W tym samouczku utrzymamy prostotę, ograniczając się do pierwszych 128 znaków zestawu znaków ASCII. Dla każdego znaku generujemy teksturę i przechowujemy odpowiednie dane w strukturze <fun>Character</fun>, którą dodajemy do mapy <var>Characters</var>. W ten sposób wszystkie dane wymagane do renderowania każdego znaku są przechowywane do późniejszego wykorzystania.

```cpp
    glPixelStorei(GL_UNPACK_ALIGNMENT, 1); // Disable byte-alignment restriction

    for (GLubyte c = 0; c < 128; c++)
    {
        // Load character glyph 
        if (FT_Load_Char(face, c, FT_LOAD_RENDER))
        {
            std::cout << "ERROR::FREETYTPE: Failed to load Glyph" << std::endl;
            continue;
        }
        // Generate texture
        GLuint texture;
        glGenTextures(1, &texture);
        glBindTexture(GL_TEXTURE_2D, texture);
        glTexImage2D(
            GL_TEXTURE_2D,
            0,
            GL_RED,
            face->glyph->bitmap.width,
            face->glyph->bitmap.rows,
            0,
            GL_RED,
            GL_UNSIGNED_BYTE,
            face->glyph->bitmap.buffer
        );
        // Set texture options
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
        // Now store character for later use
        Character character = {
            texture, 
            glm::ivec2(face->glyph->bitmap.width, face->glyph->bitmap.rows),
            glm::ivec2(face->glyph->bitmap_left, face->glyph->bitmap_top),
            face->glyph->advance.x
        };
        Characters.insert(std::pair<GLchar, Character>(c, character));
    }
```

W pętli for wyszczególniamy wszystkie 128 znaków zestawu ASCII i pobieramy ich odpowiednie znaki glifów. Dla każdego znaku generujemy teksturę, ustawiamy jej opcje i przechowujemy jej metryki. Warto zauważyć, że używamy <var>GL_RED</var> jako argumentów `internalFormat` i `format` tekstury. Mapa bitowa generowana z glifu to 8-bitowy obraz w skali szarości, w którym każdy kolor jest reprezentowany przez jeden bajt. Z tego powodu chcielibyśmy przechowywać każdy bajt bufora bitmapy jako wartość koloru tekstury. Osiągamy to, tworząc teksturę, w której każdy bajt odpowiada czerwonemu komponentowi koloru tekstury (pierwszy bajt jego wektora koloru). Jeśli użyjemy jednego bajtu do przedstawienia kolorów tekstury, musimy uważać na ograniczenia OpenGL:

```cpp
    glPixelStorei(GL_UNPACK_ALIGNMENT, 1);   
```

OpenGL wymaga, aby tekstury miały 4-bajtowe wyrównanie (ang. *alignment*), np. ich rozmiar jest zawsze wielokrotnością 4 bajtów. Zwykle nie stanowi to problemu, ponieważ większość tekstur ma szerokość, która jest wielokrotnością 4 i/lub używa 4 bajtów na piksel, ale ponieważ obecnie używamy tylko jednego bajta na piksel, mogą one mieć dowolną szerokość. Ustawiając wyrównanie rozpakowania na `1`, upewniamy się, że nie występują problemy z wyrównaniem (które mogą powodować błędy segmentacji).

Pamiętaj również, aby wyczyścić zasoby FreeType po zakończeniu przetwarzania glifów:

```cpp
    FT_Done_Face(face);
    FT_Done_FreeType(ft);
```

### Shadery

Aby wyrenderować rzeczywiste glify, użyjemy następującego Vertex Shadera:

```glsl
    #version 330 core
    layout (location = 0) in vec4 vertex; // <vec2 pos, vec2 tex>
    out vec2 TexCoords;

    uniform mat4 projection;

    void main()
    {
        gl_Position = projection * vec4(vertex.xy, 0.0, 1.0);
        TexCoords = vertex.zw;
    }  
```

Łączymy dane współrzędnych pozycji i tekstury w jeden <fun>vec4</fun>. Vertex Shader mnoży współrzędne z macierzą projekcji i przekazuje współrzędne tekstury do Fragment Shadera:

```glsl
    #version 330 core
    in vec2 TexCoords;
    out vec4 color;

    uniform sampler2D text;
    uniform vec3 textColor;

    void main()
    {    
        vec4 sampled = vec4(1.0, 1.0, 1.0, texture(text, TexCoords).r);
        color = vec4(textColor, 1.0) * sampled;
    }  
```

Fragment Shader ma dwa uniformy: jeden jest jednokolorowym obrazem bitmapowym glifu, a drugi jest uniformem koloru dla dostosowania ostatecznego koloru tekstu. Najpierw próbkujemy wartość koloru tekstury bitmapy. Ponieważ dane tekstury są przechowywane tylko w czerwonym składniku, próbkujemy składnik `r` tekstury jako próbkowaną wartość alfa. Zmieniając wartość alfa koloru, wynikowy kolor będzie przezroczysty dla wszystkich kolorów tła glifu i nieprzeźroczysty dla rzeczywistych pikseli znaków. Mnożymy także kolory RGB przez uniform <var>textColor</var>, aby zmienić kolor tekstu.

Musimy włączyć [blending]({% post_url /learnopengl/4_advanced_opengl/2018-08-27-blending %}), aby to działało:

```cpp
    glEnable(GL_BLEND);
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);  
```

Dla macierzy projekcji wykorzystamy macierz rzutowania prostokątnego. Do renderowania tekstu (zwykle) nie potrzebujemy perspektywy, a użycie macierzy rzutu prostokątnego pozwala nam również określić wszystkie współrzędne wierzchołka we współrzędnych ekranu, jeśli ustawimy je w następujący sposób:

```cpp
    glm::mat4 projection = glm::ortho(0.0f, 800.0f, 0.0f, 600.0f);
```

Ustawiamy dolny parametr macierzy rzutowania na `0.0f`, a jego górny parametr jest równy wysokości okna. Rezultatem jest to, że podajemy współrzędne z wartościami `y`, począwszy od dolnej części ekranu (`0.0f`) do górnej części ekranu (`600.0f`). Oznacza to, że punkt (`0.0`, `0.0`) odpowiada teraz lewemu dolnemu rogowi.

Na końcu tworzymy VBO i VAO do renderowania kwadratów. Na razie rezerwujemy wystarczającą ilość pamięci podczas inicjowania VBO, abyśmy mogli później aktualizować pamięć VBO podczas renderowania znaków.

```cpp
    GLuint VAO, VBO;
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);
    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(GLfloat) * 6 * 4, NULL, GL_DYNAMIC_DRAW);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 4, GL_FLOAT, GL_FALSE, 4 * sizeof(GLfloat), 0);
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindVertexArray(0);      
```

Kwadrat wymaga `6` wierzchołków `4` floatów, więc rezerwujemy `6 * 4` floaty pamięci. Ponieważ będziemy dość często aktualizować zawartość pamięci VBO, alokujemy pamięć za pomocą <var>GL_DYNAMIC_DRAW</var>.

### Renderowanie linii tekstu

Aby wyrenderować znak, wyodrębniamy odpowiedni znak <fun>Character</fun> z mapy <var>Characters</var> i obliczamy wymiary kwadratu, wykorzystując metryki znaku. Przy obliczonych wymiarach kwadratu dynamicznie generujemy zestaw 6 wierzchołków, których używamy do aktualizacji zawartości pamięci zarządzanej przez VBO za pomocą <fun>glBufferSubData</fun>.

Tworzymy funkcję o nazwie <fun>RenderText</fun>, która renderuje ciąg znaków:

```cpp
    void RenderText(Shader &s, std::string text, GLfloat x, GLfloat y, GLfloat scale, glm::vec3 color)
    {
        // Activate corresponding render state	
        s.Use();
        glUniform3f(glGetUniformLocation(s.Program, "textColor"), color.x, color.y, color.z);
        glActiveTexture(GL_TEXTURE0);
        glBindVertexArray(VAO);

        // Iterate through all characters
        std::string::const_iterator c;
        for (c = text.begin(); c != text.end(); c++)
        {
            Character ch = Characters[*c];

            GLfloat xpos = x + ch.Bearing.x * scale;
            GLfloat ypos = y - (ch.Size.y - ch.Bearing.y) * scale;

            GLfloat w = ch.Size.x * scale;
            GLfloat h = ch.Size.y * scale;
            // Update VBO for each character
            GLfloat vertices[6][4] = {
                { xpos,     ypos + h,   0.0, 0.0 },            
                { xpos,     ypos,       0.0, 1.0 },
                { xpos + w, ypos,       1.0, 1.0 },

                { xpos,     ypos + h,   0.0, 0.0 },
                { xpos + w, ypos,       1.0, 1.0 },
                { xpos + w, ypos + h,   1.0, 0.0 }           
            };
            // Render glyph texture over quad
            glBindTexture(GL_TEXTURE_2D, ch.textureID);
            // Update content of VBO memory
            glBindBuffer(GL_ARRAY_BUFFER, VBO);
            glBufferSubData(GL_ARRAY_BUFFER, 0, sizeof(vertices), vertices); 
            glBindBuffer(GL_ARRAY_BUFFER, 0);
            // Render quad
            glDrawArrays(GL_TRIANGLES, 0, 6);
            // Now advance cursors for next glyph (note that advance is number of 1/64 pixels)
            x += (ch.Advance >> 6) * scale; // Bitshift by 6 to get value in pixels (2^6 = 64)
        }
        glBindVertexArray(0);
        glBindTexture(GL_TEXTURE_2D, 0);
    }
```

Treść funkcji powinna być względnie prosta: najpierw obliczamy pozycję początkową kwadratu (jako <var>xpos</var> i <var>ypos</var>) oraz wielkość kwadratu (jako <var>w</var> i <var>h</var>) i generujemy zestaw 6 wierzchołków tworzących kwadrat; zauważ, że skalujemy każdą metrykę przez <var>scale</var>. Następnie aktualizujemy zawartość VBO i renderujemy kwadrat.

Poniższy wiersz kodu wymaga jednak dodatkowej uwagi:

```cpp
    GLfloat ypos = y - (ch.Size.y - ch.Bearing.y);   
```

Niektóre znaki (takie jak `p` lub `g`) są renderowane nieco poniżej linii bazowej, więc kwadrat powinien być umieszczony nieco poniżej wartości <var>y</var>. Dokładną ilość jaką potrzebujemy do przesunięcia <var>ypos</var> poniżej linii bazowej można ustalić na podstawie danych glifu:

![Przesunięcie poniżej linii bazowej glifu do pozycji 2D quad](/img/learnopengl/glyph_offset.png){: .center-image }

Aby obliczyć tę odległość, np. offset musimy określić odległość, na jaką glif rozciąga się poniżej linii bazowej; odległość ta jest oznaczona czerwoną strzałką. Jak widać na podstawie danych glifu, możemy obliczyć długość tego wektora przez odjęcie wartości `bearingY` od wysokości glifu (bitmapy). Ta wartość wynosi często `0.0` dla znaków lężących na linii bazowej (takich jak `X`) i jest liczbą dodatnią dla znaków, które znajdują się nieco poniżej linii bazowej (np. `g` lub `j`).

Jeśli zrobiłeś wszystko poprawnie, powinieneś być teraz w stanie renderować tekst za pomocą następujących instrukcji:

```cpp
    RenderText(shader, "This is sample text", 25.0f, 25.0f, 1.0f, glm::vec3(0.5, 0.8f, 0.2f));
    RenderText(shader, "(C) LearnOpenGL.com", 540.0f, 570.0f, 0.5f, glm::vec3(0.3, 0.7f, 0.9f));
```

Powinno to wyglądać podobnie do następującego obrazu:

![Obraz renderowania tekstu za pomocą OpenGL za pomocą FreeType](/img/learnopengl/text_rendering.png){: .center-image }

Możesz znaleźć kod tego przykładu [tutaj](https://learnopengl.com/code_viewer.php?code=in-practice/text_rendering).

Aby się dać Ci zrozumienie, jak obliczyliśmy wierzchołki kwadratu, możemy wyłączyć blending, aby zobaczyć, jak wyglądają faktycznie renderowane kwadraty:

![Obraz quadów bez przezroczystości do renderowania tekstu w OpenGL](/img/learnopengl/text_rendering_quads.png){: .center-image }

Tutaj wyraźnie widać większość kwadratów leżących na (wyobrażonej) linii bazowej, podczas gdy kwadraty odpowiadające glifom takim jak `p` lub `(` są przesunięte w dół.

## Co dalej?

W tym samouczku pokazano technikę renderowania tekstu z czcionkami TrueType przy użyciu biblioteki FreeType. Podejście jest elastyczne, skalowalne i działa z wieloma kodowaniami znaków. Jednak to podejście będzie prawdopodobnie przesadą dla twojej aplikacji, ponieważ generujemy i renderujemy tekstury dla każdego glifu. Dla wydajności najlepsza będzie jedna duża bitmapa, ponieważ potrzebujemy tylko jednej tekstury dla wszystkich naszych glifów. Najlepszym podejściem byłoby połączenie obu podejść poprzez dynamiczne generowanie tekstury bitmapowej czcionki zawierającej wszystkie glify znaków załadowane za pomocą FreeType. To oszczędza rendererowi przełączania dużej ilości tekstur na podstawie tego, jak mocno każdy glif jest upakowany, co może zaoszczędzić sporo wydajności.

Kolejną kwestią związaną z FreeType jest fakt, że tekstury glifów są przechowywane z ustalonym rozmiarem czcionki, więc może być wymagana znaczna ilość skalowania, która wprowadzi postrzępione krawędzie. Co więcej, rotacje zastosowane do glifów spowodują, że będą niewyraźne. Może to zostać złagodzone przez, zamiast przechowywania rzeczywistego zrasteryzowanego koloru piksela, przechowywanie odległości do najbliższego konturu glifu na piksel. Technika ta nosi nazwę <def>signed distance fields</def>, a Valve opublikował kilka lat temu [artykuł](http://www.valvesoftware.com/publications/2007/SIGGRAPH2007_AlphaTestedMagnification.pdf) na temat implementacji tej techniki, która działa zaskakująco dobrze w aplikacjach do renderowania 3D.