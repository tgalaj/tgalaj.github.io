---
layout: post
title: Zaawansowany GLSL
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: zaawansowany-opengl
---

{% include learnopengl.md link="Advanced-OpenGL/Advanced-GLSL" %}

Ten samouczek nie pokaże Ci super zaawansowanych nowych funkcji, które znacznie poprawiają jakość obrazu Twojej sceny. Ten samouczek w mniej lub bardziej interesujący sposób opisuje GLSL i kilka fajnych sztuczek, które mogą ci pomóc w przyszłości. Zasadniczo będą to _rzeczy warte poznania_ i _funkcjonalności, które mogą ułatwić Ci życie_ podczas tworzenia aplikacji OpenGL w połączeniu z GLSL.

Omówimy interesujące <def>zmienne wbudowane</def>, nowe sposoby organizowania danych wejściowych i wyjściowych shaderów oraz bardzo przydatne narzędzie zwane <def>obiektem bufora uniformów</def> (ang. *uniform buffer object, UBO*).

# Wbudowane zmienne GLSL

Jeśli potrzebujemy danych z dowolnego innego źródła poza bieżącym shaderem, musimy przekazać te dane do innego shadera. Nauczyliśmy się, że można tego dokonać poprzez atrybuty wierzchołków, uniformy i samplery. Istnieje jednak kilka dodatkowych zmiennych zdefiniowanych przez GLSL z przedrostkiem `gl_`, które dają nam dodatkowe możliwości do pobierania i/lub zapisywania danych. W dotychczasowych tutorialach spotkaliśmy już dwa z nich: <var>gl_Position</var>, który jest wektorem wyjściowym Vertex Shadera i zmiennej Fragment Shadera <var>gl_FragCoord</var>.

Omówimy kilka interesujących wbudowanych zmiennych wejściowych i wyjściowych, które są wbudowane w GLSL i wyjaśnimy, jak mogą nam one pomóc. Zauważ, że nie omówimy wszystkich zmiennych wbudowanych, które istnieją w GLSL, więc jeśli chcesz zobaczyć je wszystkie, zobacz [wiki OpenGL](http://www.opengl.org/wiki/Built-in_Variable_(GLSL)).

## Zmienne Vertex Shadera

Spotkaliśmy już <var>gl_Position</var>, która to zmienna jest wektorem pozycji wyjściowej w przestrzeni obcinania w Vertex Shaderze. Ustawienie <var>gl_Position</var> w Vertex Shader jest wymogiem, jeśli chcesz renderować cokolwiek na ekranie. Nic, czego nie widzieliśmy wcześniej.

### gl_PointSize

Jednym z prymitywów renderowania, z których możemy skorzystać, jest <var>GL_POINTS</var>, w którym to przypadku każdy pojedynczy wierzchołek jest renderowany jako punkt. Możliwe jest ustawienie rozmiaru renderowanych punktów za pomocą funkcji <fun>glPointSize</fun> OpenGL, ale możemy także wpływać na tę wartość w Vertex Shaderze.

Zmienna wyjściowa zdefiniowana przez GLSL nosi nazwę <var>gl_PointSize</var>, która jest zmienną typu <fun>float</fun>, za pomocą której można ustawić szerokość i wysokość punktu w pikselach. Opisując rozmiar punktu w Vertex Shaderze, można wpływać na wartość tego punktu na każdy wierzchołek z osobna.

Wpływanie na rozmiary punktów w Vertex Shaderze jest domyślnie wyłączone, ale jeśli chcesz to włączyć, musisz włączyć opcję <var>GL_PROGRAM_POINT_SIZE</var> OpenGL:

```cpp
    glEnable(GL_PROGRAM_POINT_SIZE);  
```

Prostym przykładem wpłynięcia na rozmiar punktów jest ustawienie rozmiaru punktu równej wartości pozycji z przestrzeni obcinania, która jest równa odległości wierzchołka względem kamery. Rozmiar punktu powinien rosnąć, im dalej jesteśmy od wierzchołka.

```glsl
    void main()
    {
        gl_Position = projection * view * model * vec4(aPos, 1.0);    
        gl_PointSize = gl_Position.z;    
    }  
```

W rezultacie punkty, które narysujemy, stają się większe, im bardziej oddalamy się od nich:

![Punkty w OpenGL narysowane ze zmienioną zmienną gl_PointSize w Vertex Shader](/img/learnopengl/advanced_glsl_pointsize.png){: .center-image }

Można sobie wyobrazić, że zmiana wielkości punktu na każdy wierzchołek jest przydatna dla technik takich jak generowanie cząsteczek (ang. *particle generation*).

### gl_VertexID

Zmienne <var>gl_Position</var> i <var>gl_PointSize</var> są _zmiennymi wyjściowymi_, ponieważ ich wartości są odczytywane jako wyjście z Vertex Shadera; możemy wpłynąć na ich wartości, zapisując im własne wartości. Vertex Shader daje nam również interesującą zmienną _wejściową_ <var>gl_VertexID</var>, z której możemy tylko odczytać wartość.

Zmienna <var>gl_VertexID</var> jest typu integer i zawiera bieżący identyfikator wierzchołka, który rysujemy. Podczas wykonywania _indeksowanego renderowania_ (za pomoca funkcji <fun>glDrawElements</fun>) ta zmienna zawiera bieżący indeks rysowanego wierzchołka. Podczas rysowania bez indeksów (przez <fun>glDrawArrays</fun>) ta zmienna przechowuje numer aktualnie przetwarzanego wierzchołka od początku wywołania renderowania.

Chociaż nie jest to szczególnie przydatne w tej chwili, dobrze jest wiedzieć, że mamy dostęp do takich informacji.

## Zmienne Fragment Shadera

W Fragment Shader mamy również dostęp do interesujących zmiennych. GLSL daje nam dwie interesujące zmienne wejściowe o nazwach <var>gl_FragCoord</var> i <var>gl_FrontFacing</var>.

### gl_FragCoord

Kilkakrotnie widzieliśmy <var>gl_FragCoord</var> podczas dyskusji o teście głębokości, ponieważ składnik `z` wektora <var>gl_FragCoord</var> jest równy wartości głębi tego konkretnego fragmentu. Jednak możemy również użyć komponentu `x` i `y` wektora dla osiągnięcia interesujących efektów.

Składniki `x` i `y` <var>gl_FragCoord</var> są współrzędnymi fragmentu w przestrzeni okna, które początek mają w lewym dolnym rogu okna. Określiliśmy okno o wielkości 800x600 za pomoca funkcji <fun>glViewport</fun>, więc współrzędne przestrzeni okna fragmentu będą miały wartości `x` między 0 a 800, a wartości `y` między 0 i 600.

Za pomocą Fragment Shadera możemy obliczyć inną wartość koloru na podstawie współrzędnej fragmentu. Typowe użycie zmiennej <var>gl_FragCoord</var> służy do porównywania efektów wizualnych różnych obliczeń na fragmentach, jak to zwykle widać w demonstracjach technicznych. Możemy na przykład podzielić ekran na dwie części, wyświetlając jednen efekt z lewej strony okna, a drugi z prawej strony okna. Przykładowy Fragment Shader, który wyświetla inny kolor na podstawie współrzędnych okna fragmentu, jest podany poniżej:

```glsl
    void main()
    {             
        if(gl_FragCoord.x < 400)
            FragColor = vec4(1.0, 0.0, 0.0, 1.0);
        else
            FragColor = vec4(0.0, 1.0, 0.0, 1.0);        
    }  
```

Ponieważ szerokość okna wynosi 800, za każdym razem, gdy współrzędna `x` piksela jest mniejsza niż `400`, musi znajdować się po lewej stronie okna, i dlatego nadajemy obiektowi inny kolor.

![Kostka w OpenGL narysowana w 2 kolorach za pomocą gl_FragCoord](/img/learnopengl/advanced_glsl_fragcoord.png){: .center-image }

Możemy teraz obliczyć dwa zupełnie różne wyniki cieniowania fragmentów i wyświetlić każdy z nich po innych stronach okna. To świetnie nadaje się, na przykład, do testowania różnych technik oświetleniowych.

### gl_FrontFacing

Inną interesującą zmienną wejściową w Fragment Shader jest zmienna <var>gl_FrontFacing</var>. W tutorialu Face Culling wspomnieliśmy, że OpenGL jest w stanie dowiedzieć się, czy ścianka jest ścianką przednią czy tylną z powodu kolejności definiowania wierzchołków. Jeśli nie używamy usuwania ścianek (wyłączając <var>GL_FACE_CULL</var>), zmienna <var>gl_FrontFacing</var> poinformuje nas, czy bieżący fragment znajduje się na ściance skierowanej przodem lub tyłem. Moglibyśmy wtedy zdecydować się na przykład na obliczanie różnych kolorów powierzchni ścianek.

Zmienna <var>gl_FrontFacing</var> jest typu <fun>bool</fun>, która zwraca wartość `true`, jeśli fragment znajduje się na przedniej ściance, w przeciwnym wypadku zwraca `false`. Moglibyśmy na przykład stworzyć kostkę w ten sposób, by miała inną teksturą od wewnątrz niż na zewnątrz:

```glsl
    #version 330 core
    out vec4 FragColor;

    in vec2 TexCoords;

    uniform sampler2D frontTexture;
    uniform sampler2D backTexture;

    void main()
    {             
        if(gl_FrontFacing)
            FragColor = texture(frontTexture, TexCoords);
        else
            FragColor = texture(backTexture, TexCoords);
    }  
```

Jeśli więc wejdziemy do kontenera, zobaczymy inną teksturę.

![Kontener OpenGL używający dwóch różnych tekstur za pomocą gl_FrontFacing](/img/learnopengl/advanced_glsl_frontfacing.png){: .center-image }

Zauważ, że jeśli włączyłeś funkcję usuwania ścianek, nie będziesz widział żadnych ścianek wewnątrz kontenera i zastosowanie <var>gl_FrontFacing</var> staje się bezcelowe.

### gl_FragDepth

Zmienna wejściowa <var>gl_FragCoord</var> jest zmienną wejściową, która pozwala odczytać współrzędne przestrzeni okna i uzyskać wartość głębi bieżącego fragmentu, ale jest to zmienna <def>tylko do odczytu</def>. Nie możemy wpływać na współrzędne fragmentu w przestrzeni okna, ale możliwe jest ustawienie wartości głębi fragmentu. GLSL daje nam zmienną wyjściową o nazwie <var>gl_FragDepth</var>, której możemy użyć do ustawienia wartości głębi fragmentu w Fragment Shader.

Aby faktycznie ustawić wartość głębi w Fragment Shader, po prostu wpisujemy wartość <fun>float</fun> z zakresu między `0.0` a `1.0` do zmiennej wyjściowej:

```glsl
    gl_FragDepth = 0.0; // ten fragment ma teraz wartość głębokości 0.0
```

Jeśli shader nie zapisuje wartości <var>gl_FragDepth</var>, zmienna automatycznie pobierze wartość z `gl_FragCoord.z`.

Ustawienie wartości głębokości przez nas samych ma jednak poważną wadę, ponieważ OpenGL wyłącza wtedy wszystkie <def>wczesne testy głębokości</def> (jak zostało to omówione w tutorialu [test głębokości]({% post_url /learnopengl/4_advanced_opengl/2018-08-22-test-glebokosci %})), jak tylko zapiszemy wartość do <var>gl_FragDepth</var> w Fragment Shader. Zostaje on wyłączony, ponieważ OpenGL nie wie, jaką wartość głębi będzie mieć fragment, przed rozpoczęciem Fragment Shadera, ponieważ może on całkowicie zmienić tę wartość głębokości.

Zapisując wartość do <var>gl_FragDepth</var>, należy wziąć pod uwagę tę wadę. Od wersji OpenGL 4.2 możemy w pewnym sensie pośredniczyć między obiema stronami, redefiniując zmienną <var>gl_FragDepth</var> na samej górze Fragment Shadera z <def>warunkiem głębokość</def> (ang. *depth condition*):

```glsl
    layout (depth_<condition>) out float gl_FragDepth;
```

`condition` może przyjmować następujące wartości:

<table align="center">
  <tbody><tr>
  	<th style="text-align:center;">Condition</th>
  	<th style="text-align:center;">Opis</th>
  </tr>  
  <tr>
    <td style="text-align:center;">any</td>
 	<td style="text-align:center;">Wartość domyślna. Wczesne testy głębokości są wyłączone i tracisz największą wydajność.</td>
  </tr>
  <tr>
    <td style="text-align:center;">greater</td>
    <td style="text-align:center;">Możesz tylko zwiększyć wartość głębokości w porównaniu do `gl_FragCoord.z`.</td>
  </tr>
  <tr>
    <td style="text-align:center;">less</td>
 	<td style="text-align:center;">Możesz tylko zmniejszyć wartość głębi w porównaniu do `gl_FragCoord.z`.</td>
  </tr>
  <tr>
    <td style="text-align:center;">unchanged</td>
    <td style="text-align:center;">Jeśli zapiszesz wartość do `gl_FragDepth`, zapiszesz ją dokładnie w `gl_FragCoord.z`.</td>
  </tr>
</tbody></table>

Wybierając `greater` lub `less` jako warunek głębi OpenGL może przyjąć założenie, że zapiszesz tylko wartości głębokości większe lub mniejsze niż wartość głębi fragmentu. W ten sposób OpenGL nadal może wykonać wczesny test głębokości w przypadkach, w których wartość głębi jest mniejsza niż wartość głębi fragmentu.

Przykład, w którym zwiększamy wartość głębi w Fragment Shader, ale chcemy zachować niektóre z wczesnych testów głębokości, jest pokazany poniżej:

```glsl
    #version 420 core // zwróć uwagę na wersję GLSL!
    out vec4 FragColor;
    layout (depth_greater) out float gl_FragDepth;

    void main()
    {             
        FragColor = vec4(1.0);
        gl_FragDepth = gl_FragCoord.z + 0.1;
    }  
```

Zwróć uwagę, że ta funkcja jest dostępna tylko w wersji OpenGL 4.2 lub nowszej.

# Bloki interfejsów (ang. *Interface blocks*)

Do tej pory za każdym razem, gdy chcieliśmy przesyłać dane z Vertex Shader do Fragment Shader, deklarowaliśmy kilka pasujących do siebie zmiennych wejściowych/wyjściowych. Deklarowanie tych zmiennych jedna po drugiej jest najłatwiejszym sposobem przesyłania danych z jednego shadera do innego, ale ponieważ aplikacje stają się większe, prawdopodobnie będziesz chciał wysłać więcej niż kilka zmiennych, które mogą zawierać tablice i/lub struktury.

Aby pomóc nam uporządkować te zmienne, GLSL oferuje nam coś, co nazywa się <def>blokami interfejsów</def>, co pozwala nam zgrupować te zmienne. Deklaracja takiego bloku interfejsu wygląda podobnie jak deklaracja <fun>struct</fun>, z tym że jest teraz deklarowane za pomocą słowa kluczowego <fun>in</fun> lub <fun>out</fun> w zależności czy blok jest blokiem wejściowym czy wyjściowym.

```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec2 aTexCoords;

    uniform mat4 model;
    uniform mat4 view;
    uniform mat4 projection;

    out VS_OUT
    {
        vec2 TexCoords;
    } vs_out;

    void main()
    {
        gl_Position = projection * view * model * vec4(aPos, 1.0);    
        vs_out.TexCoords = aTexCoords;
    }  
```

Zadeklarowaliśmy blok interfejsu o nazwie <var>vs_out</var>, który grupuje wszystkie zmienne wyjściowe, które chcemy wysłać do następnego shadera. Jest to banalny przykład, ale możesz sobie wyobrazić, że pomaga to uporządkować zmienne wejścia/wyjścia shaderów. Jest to również przydatne, gdy chcemy pogrupować zmienne wejściowe/wyjściowe shadera w tablice, co zobaczymy w następnym samouczku dotyczącym Geometry Shaderów.

Następnie musimy zadeklarować blok interfejsu wejściowego w kolejnym shaderze, który jest Fragment Shaderem. Nazwa <def>bloku</def> (<fun>VS_OUT</fun>) powinna być taka sama w Fragment Shaderze, ale <def>nazwa instancji</def> (<var>vs_out</var> jak ta użyta w Vertex Shader) może być jakakolwiek nam się podoba - unikając mylących nazw, takich jak <var>vs_out</var>, które faktycznie zawierają zmienne wejściowe.

```glsl
    #version 330 core
    out vec4 FragColor;

    in VS_OUT
    {
        vec2 TexCoords;
    } fs_in;

    uniform sampler2D texture;

    void main()
    {             
        FragColor = texture(texture, fs_in.TexCoords);   
    } 
```

Dopóki obie nazwy bloków interfejsu są takie same, odpowiadające im wejścia i wyjścia są ze sobą połączone. Jest to kolejna przydatna funkcjonalność ułatwiająca organizowanie kodu i jest przydatna podczas przechodzenia między niektórymi etapami cieniowania, takimi jak Geometry Shader.

# Obiekt bufora uniformów (ang. *Uniform buffer objects*)

Używamy OpenGL już od jakiegoś czasu i nauczyliśmy się kilku fajnych sztuczek, ale także kilku rzeczy, które irytują. Na przykład, gdy używasz więcej niż 1 shadera, musimy ciągle ustawiać zmienne uniform, w których większość z nich jest dokładnie taka sama dla każdego shadera - więc po co zawracać sobie głowę ustawianiem ich jeszcze raz?

OpenGL daje nam narzędzie zwane <def>obiektem bufora uniformów</def>, który pozwala nam zadeklarować zestaw _globalnych_ zmiennych uniform, które pozostają takie same dla różnych shaderów. Używając obiektu bufora uniformów, musimy ustawić tylko odpowiednie uniformy **raz**. Wciąż musimy ręcznie ustawiać uniformy, które są unikalne dla każdego shadera. Tworzenie i konfigurowanie obiektu bufora uniformów wymaga jednak nieco pracy.

Ponieważ obiekt bufora uniformów jest buforem, jak każdy inny bufor, możemy go utworzyć za pomocą <fun>glGenBuffers</fun>, powiązać go z <var>GL_UNIFORM_BUFFER</var> i przechowywać w nim wszystkie odpowiednie dane uniformów. Istnieją pewne zasady dotyczące przechowywania danych dla obiektu bufora uniformów, ale przejdziemy do tego później. Najpierw zajmiemy się prostym Vertex Shaderem i zapiszemy macierze <var>projekcji</var> i <var>widoku</var> w tak zwanym <def>bloku uniformów</def>:

```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;

    layout (std140) uniform Matrices
    {
        mat4 projection;
        mat4 view;
    };

    uniform mat4 model;

    void main()
    {
        gl_Position = projection * view * model * vec4(aPos, 1.0);
    }  
```

W większości naszych przykładach ustawialiśmy macierze projekcji i widoku jako macierz uniform w każdej iteracji dla każdego shadera, którego używaliśmy. Jest to doskonały przykład tego, gdzie obiekty bufora uniformów stają się użyteczne, ponieważ będziemy mogli ustawić te macierze tylko jeden raz.

Zadeklarowaliśmy blok uniform o nazwie <var>Matrices</var>, który przechowuje dwie macierze 4x4. Zmienne w bloku uniform można uzyskać bezpośrednio, bez prefiksu jego bloku. Następnie przechowujemy te wartości macierzy w buforze, aby każdy shader, który zadeklarował ten blok uniformów, miał dostęp do tych macierzy.

Prawdopodobnie zastanawiasz się teraz, co oznacza `layout (std140)`. Oznacza to, że aktualnie zdefiniowany blok uniformów używa określonego układu pamięci dla jego zawartości; to polecenie ustawia <def>układ pamięci obiektu bufora uniformów</def>.

## Układ pamięci obiektu bufora uniformów

Zawartość bloku uniformów jest przechowywana w obiekcie bufora, który w zasadzie jest niczym więcej jak zarezerwowaną częścią pamięci. Ponieważ ta część pamięci nie ma informacji o tym, jakie dane są w niej przechowywane, musimy powiedzieć OpenGL, jakie części pamięci odpowiadają zmiennym w shaderze.

Wyobraź sobie następujący blok uniformów w Fragment Shader:

```glsl
    layout (std140) uniform ExampleBlock
    {
        float value;
        vec3  vector;
        mat4  matrix;
        float values[3];
        bool  boolean;
        int   integer;
    };  
```

To, co chcemy wiedzieć, to rozmiar (w bajtach) i przesunięcie (od początku bloku) każdej z tych zmiennych, abyśmy mogli umieścić je w buforze w odpowiedniej kolejności. Rozmiar każdego z elementów jest wyraźnie określony w OpenGL i bezpośrednio odpowiada typom danych C++; wektory i macierze będące (dużymi) tablicami floatów. To, czego OpenGL nie określa jasno, to <def>odstęp</def> (ang. *spacing*) między zmiennymi. Pozwala to sprzętowi na umieszczanie zmiennych według własnego uznania. Niektóre urządzenia mogą np. umieścić <fun>vec3</fun> obok <fun>float</fun>. Nie każdy sprzęt może sobie z tym poradzić i przekształca <fun>vec3</fun> na wektor 4-elementowy. Świetna funkcjonalność, ale dla nas niewygodna.

Domyślnie GLSL używa układu pamięci obiektów bufora uniformów zwanego układem <def>shared</def> - współdzielonego, ponieważ po zdefiniowaniu przesunięć przez sprzęt, są one konsekwentnie _współdzielone_ pomiędzy programami. W przypadku wspólnego układu GLSL może zmieniać położenie zmiennych uniform w celu optymalizacji, o ile kolejność zmiennych pozostanie nienaruszona. Ponieważ nie wiemy, z jakim offsetem skończy każdy uniform, nie wiemy, jak precyzyjnie wypełnić nasz bufor uniformów. Możemy zapytać o te informacje za pomocą funkcji takich jak <fun>glGetUniformIndices</fun>, ale to wykracza poza zakres tego samouczka.

Podczas gdy współdzielony układ daje nam trochę oszczędności pamięci, to musimy pobrać wartość każdego przesunięcia dla każdej zmiennej uniform, co przekłada się na wiele pracy. Ogólna praktyka nie polega jednak na korzystaniu z układu współdzielonego, ale na wykorzystaniu układu <def>std140</def>. Układ std140 **jawnie** określa układ pamięci dla każdego typu zmiennej, określając ich odpowiednie przesunięcia regulowane przez zestaw reguł. Ponieważ jest to wyraźnie określone, możemy ręcznie określić przesunięcia dla każdej zmiennej.

Każda zmienna ma <def>bazowe wyrównanie</def> (ang. *base alignment*), które jest równe zajętości miejsca, jakie zajmuje dany typ zmiennej (w tym dopełnienie (ang. *padding*)) w ramach bloku uniformów - to bazowe wyrównanie jest obliczane przy użyciu reguł układu std140. Następnie dla każdej zmiennej obliczamy jej <def>offset</def>, który jest przesunięciem bajtów zmiennej od początku bloku. Wyrównane przesunięcie bajtów zmiennej **musi** być równe wielokrotności jej bazowego wyrównania.

Dokładne reguły tego układu pamięci można znaleźć w specyfikacji bufora uniformów OpenGL znajdującego się [tutaj](http://www.opengl.org/registry/specs/ARB/uniform_buffer_object.txt), ale poniżej wymienimy najczęstsze reguły. Każdy typ zmiennej w GLSL, taki jak <fun>int</fun>, <fun>float</fun> i <fun>bool</fun> są zdefiniowane jako wielkości czterobajtowe, przy czym każda jednostka o 4 bajtach jest reprezentowana jako `N`.

<table align="center">
  <tbody><tr>
    <th style="text-align:center;">Typ</th>
    <th style="text-align:center;">Reguła układu pamięci</th>
  </tr>
  
  <tr>
    <td style="text-align:center;">Skalar np. <fun>int</fun> lub <fun>bool</fun></td>
    <td style="text-align:center;">Każdy skalar ma bazowe wyrównanie równe N.</td>
  </tr>
  <tr>
    <td style="text-align:center;">Wektor</td>
    <td style="text-align:center;">Albo 2N, albo 4N. Oznacza to, że <fun>vec3</fun> ma bazowe wyrównanie równe 4N.</td>
  </tr>
  <tr>
    <td style="text-align:center;">Tablica skalarów lub wektorów</td>
    <td style="text-align:center;">Każdy element ma bazowe wyrównanie równe <fun>vec4</fun>.</td>
  </tr>
  <tr>
    <td style="text-align:center;">Macierze</td>
    <td style="text-align:center;">Przechowywane jako duża liczba wektorów kolumnowych, w których każdy z tych wektorów ma podstawowe wyrównanie równe <fun>vec4</fun>.</td>
  </tr>
  <tr>
    <td style="text-align:center;">Struktury</td>
    <td style="text-align:center;">Równy obliczonemu rozmiarowi jego wszystkich elementów zgodnie z poprzednimi regułami, ale dopełniony do wielokrotności wielkości <fun>vec4</fun>.</td>
  </tr>    
</tbody></table>

Podobnie jak większość specyfikacji OpenGL, łatwiej jest to zrozumieć na przykładzie. Bierzemy blok uniform o nazwie <var>ExampleBlock</var>, który wprowadziliśmy wcześniej i obliczamy wyrównane przesunięcie (ang. *aligned offset*) dla każdego z jego zmiennych, używając układu std140:

```glsl
    layout (std140) uniform ExampleBlock
    {
                         // base alignment  // aligned offset
        float value;     // 4               // 0 
        vec3 vector;     // 16              // 16  (musi być wielokrotnością 16, więc 4->16)
        mat4 matrix;     // 16              // 32  (kolumna 0)
                         // 16              // 48  (kolumna 1)
                         // 16              // 64  (kolumna 2)
                         // 16              // 80  (kolumna 3)
        float values[3]; // 16              // 96  (values[0])
                         // 16              // 112 (values[1])
                         // 16              // 128 (values[2])
        bool boolean;    // 4               // 144
        int integer;     // 4               // 148
    }; 
```

Jako ćwiczenie spróbuj obliczyć wartości przesunięcia samodzielnie i porównaj je z tą tabelą. Dzięki obliczonym wartościom przesunięcia, opartym na regułach układu std140, możemy wypełnić bufor danymi dla każdego przesunięcia za pomocą funkcji takich jak <fun>glBufferSubData</fun>. Chociaż nie jest to najbardziej wydajny układ, to gwarantuje, że układ pamięci pozostaje taki sam w każdym programie, który zadeklarował ten blok uniformów.

Dodając instrukcję `layout (std140)` przed definicją bloku uniformów mówimy OpenGL, że ten blok używa układu std140. Istnieją dwa inne układy do wyboru, które wymagają od nas pobrania każdego przesunięcia przed wypełnieniem buforów. Widzieliśmy już układ `shared`, a drugi to `packed`. Używając układu packed, nie ma gwarancji, że układ pozostanie taki sam pomiędzy różnymi programami (nie jest współdzielony), ponieważ pozwala kompilatorowi zoptymalizować zmienne uniform.

## Używanie buforów uniformów

Omówiliśmy definiowanie bloków uniform w shaderach i określiliśmy ich układ pamięci, ale nie rozmawialiśmy jeszcze o tym, jak ich używać.

Najpierw musimy stworzyć obiekt bufora uniformów, który jest tworzony za pomocą <fun>glGenBuffers</fun>. Gdy mamy obiekt bufora, wiążemy go z przeznaczeniem <var>GL_UNIFORM_BUFFER</var> i przydzielamy wystarczającą ilość pamięci, wywołując <fun>glBufferData</fun>.

```cpp
    unsigned int uboExampleBlock;
    glGenBuffers(1, &uboExampleBlock);
    glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
    glBufferData(GL_UNIFORM_BUFFER, 152, NULL, GL_STATIC_DRAW); // przydziel 152 bajty pamięci
    glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

Teraz, gdy chcemy zaktualizować lub wstawić dane do bufora, wiążemy <var>uboExampleBlock</var> z kontekstem i używamy <fun>glBufferSubData</fun> do aktualizacji pamięci. Trzeba tylko raz zaktualizować ten bufor uniformów, a wszystkie shadery korzystające z tego bufora będą korzystać z jego zaktualizowanych danych. Ale jak OpenGL wie, jakie bufory uniformów odpowiadają blokom uniformów?

W kontekście OpenGL jest zdefiniowana liczba <def>binding points</def> (punkty wiązania), do których możemy podłączyć bufor uniformów. Po utworzeniu bufora uniformów łączymy go z jednym z tych punktów wiązania, a także łączymy blok uniformów w shaderze z tym samym punktem wiązania, skutecznie łącząc je ze sobą. Poniższy diagram to ilustruje:

![Schemat punktów wiązania w OpenGL](/img/learnopengl/advanced_glsl_binding_points.png){: .center-image }

Jak widać możemy powiązać wiele buforów uniformów z różnymi punktami wiązania. Ponieważ shader A i shader B mają blok uniformów połączony z tym samym punktem wiązania `0`, ich bloki uniformów mają takie same dane uniformów w <var>uboMatices</var>; wymaganym warunkiem jest, aby oba shadery definiowały ten sam blok <var>Matrices</var>.

Aby ustawić blok uniformów dla określonego punktu wiązania, wywołujemy <fun>glUniformBlockBinding</fun>, który pobiera obiekt programu shadera jako pierwszy argument, indeks bloku uniformów i punkt wiązania. <def>Indeks bloku uniformów</def> jest indeksem lokalizacji zdefiniowanego bloku uniformów w shaderze. Można go pobrać przez wywołanie <fun>glGetUniformBlockIndex</fun>, który przyjmuje obiekt programu shadera i nazwę bloku uniformów. Możemy ustawić blok uniformów <var>Lights</var> z diagramu na punkt wiązania `2` w następujący sposób:

```cpp
    unsigned int lights_index = glGetUniformBlockIndex(shaderA.ID, "Lights");   
    glUniformBlockBinding(shaderA.ID, lights_index, 2);
```

Zauważ, że musimy powtórzyć ten proces dla **każdego** shadera.

<div class="box-note">Począwszy od OpenGL w wersji 4.2 i późniejszych możliwe jest także jawne zapisanie punktu wiązania dla  bloku uniformów w shaderze poprzez dodanie innego specyfikatora układu, oszczędzając nam wywoływania funkcji <fun>glGetUniformBlockIndex</fun> i <fun>glUniformBlockBinding</fun>. Poniższy kod jawnie definiuje punkt wiązania dla bloku uniformów <var>Lights</var>:

```glsl
    layout(std140, binding = 2) uniform Lights { ... };
```
</div>

Następnie musimy powiązać obiekt bufora uniformów z tym samym punktem wiązania i można to osiągnąć za pomocą funkcji <fun>glBindBufferBase</fun> lub <fun>glBindBufferRange</fun>.

```cpp
    glBindBufferBase(GL_UNIFORM_BUFFER, 2, uboExampleBlock); 
    // lub
    glBindBufferRange(GL_UNIFORM_BUFFER, 2, uboExampleBlock, 0, 152);
```

Funkcja <fun>glBindbufferBase</fun> przyjmuje wartość przeznaczenia bufora, indeks punktu wiązania i obiekt bufora uniformów jako swoje argumenty. Ta funkcja łączy <var>uboExampleBlock</var> z punktem wiązania `2` i od tego momentu obie strony są ze sobą połączone. Możesz także użyć funkcji <fun>glBindBufferRange</fun>, która oczekuje dodatkowego parametru przesunięcia i rozmiaru - w ten sposób możesz powiązać tylko określony zakres bufora uniformów z punktem wiązania. Używając <fun>glBindBufferRange</fun> możesz mieć wiele różnych bloków uniformów połączonych z jednym obiektem bufora uniformów.

Po skonfigurowaniu wszystkiego możemy rozpocząć dodawanie danych do bufora uniformów. Moglibyśmy dodać wszystkie dane w postaci tablicy bajtów lub zaktualizować części bufora za pomocą <fun>glBufferSubData</fun>. Aby zaktualizować zmienną uniform typu <var>boolean</var>, możemy zaktualizować obiekt bufora uniformów w następujący sposób:

```cpp
    glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
    int b = true; // bool w GLSL jest reprezentowany przez 4 bajty, więc przechowujemy je w typie int
    glBufferSubData(GL_UNIFORM_BUFFER, 144, 4, &b); 
    glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

Ta sama procedura dotyczy wszystkich pozostałych zmiennych uniformów wewnątrz bloku uniformów, ale z różnymi zakresami.

## Prosty przykład

Warto więc zademonstrować prawdziwy użyteczny przykład wykorzystania obiekt bufora uniformów. Jeśli spojrzymy wstecz na wszystkie poprzednie przykłady kodu, ciągle używaliśmy 3 macierzy: macierzy projekcji, widoku i modelu. Ze wszystkich tych macierzy tylko macierz modelu zmienia się często. Jeśli mamy wiele shaderów używających tego samego zestawu macierzy, prawdopodobnie lepiej byłoby używać obiektów bufora uniformów.

Przechowujemy macierze projekcji i widoku w bloku uniformów o nazwie <var>Matrices</var>. Nie będziemy tam przechowywać macierzy modelu, ponieważ macierz modelu ma tendencję do częstej zmiany pomiędzy wywołaniami shaderów, więc nie skorzystalibyśmy wówczas z zalet obiektu bufora uniformów.

```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;

    layout (std140) uniform Matrices
    {
        mat4 projection;
        mat4 view;
    };
    uniform mat4 model;

    void main()
    {
        gl_Position = projection * view * model * vec4(aPos, 1.0);
    }  
```

Nie dzieje się tu nic specjalnego, tylko że teraz używamy bloku uniformów z układem std140. W naszej przykładowej aplikacji chcemy wyświetlić 4 kostki, w których każda kostka jest wyświetlana przy użyciu innego programu cieniującego. Każdy z 4 programów cieniujących używa tego samego Vertex Shadera, ale ma inny Fragment Shader, który wyprowadza tylko jeden kolor, który różni się w zależności od shadera.

Najpierw ustawiamy blok uniformów Vertex Shadera równy punktowi wiązania `0`. Zauważ, że musimy to zrobić dla każdego shadera.

```cpp
    unsigned int uniformBlockIndexRed    = glGetUniformBlockIndex(shaderRed.ID, "Matrices");
    unsigned int uniformBlockIndexGreen  = glGetUniformBlockIndex(shaderGreen.ID, "Matrices");
    unsigned int uniformBlockIndexBlue   = glGetUniformBlockIndex(shaderBlue.ID, "Matrices");
    unsigned int uniformBlockIndexYellow = glGetUniformBlockIndex(shaderYellow.ID, "Matrices");  

    glUniformBlockBinding(shaderRed.ID,    uniformBlockIndexRed, 0);
    glUniformBlockBinding(shaderGreen.ID,  uniformBlockIndexGreen, 0);
    glUniformBlockBinding(shaderBlue.ID,   uniformBlockIndexBlue, 0);
    glUniformBlockBinding(shaderYellow.ID, uniformBlockIndexYellow, 0);
```

Następnie tworzymy obiekt bufora uniformów i wiążemy bufor do punktem wiązania `0`:

```cpp
    unsigned int uboMatrices
    glGenBuffers(1, &uboMatrices);

    glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
    glBufferData(GL_UNIFORM_BUFFER, 2 * sizeof(glm::mat4), NULL, GL_STATIC_DRAW);
    glBindBuffer(GL_UNIFORM_BUFFER, 0);

    glBindBufferRange(GL_UNIFORM_BUFFER, 0, uboMatrices, 0, 2 * sizeof(glm::mat4));
```

Najpierw przydzielamy wystarczającą ilość pamięci dla naszego bufora, który jest równy 2-krotności wielkości <fun>glm::mat4</fun>. Rozmiar macierzy GLM odpowiada bezpośrednio <fun>mat4</fun> w GLSL. Następnie łączymy określony zakres bufora, który w tym przypadku jest całym buforem, z punktem wiązania `0`.

Teraz pozostaje tylko wypełnić bufor. Jeśli zmienna _field of view_ macierzy projekcji nie będzie się zmieniać (więc wyłączamy zoom kamery), to musimy ją zdefiniować tylko raz w naszej aplikacji - oznacza to, że musimy tylko wstawić ją do bufora tylko raz. Ponieważ już przydzieliliśmy wystarczającą ilość pamięci w obiekcie bufora, możemy użyć <fun>glBufferSubData</fun>, aby zapisać macierz projekcji przed wejściem do pętli gry:

```cpp
    glm::mat4 projection = glm::perspective(glm::radians(45.0f), (float)width/(float)height, 0.1f, 100.0f);
    glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
    glBufferSubData(GL_UNIFORM_BUFFER, 0, sizeof(glm::mat4), glm::value_ptr(projection));
    glBindBuffer(GL_UNIFORM_BUFFER, 0);  
```

Przechowujemy pierwszą połowę bufora uniformów - macierz projekcji. Zanim narysujemy obiekty, po każdej iteracji aktualizujemy drugą połówkę bufora za pomocą macierzy widoku:

```cpp
    glm::mat4 view = camera.GetViewMatrix();	       
    glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
    glBufferSubData(GL_UNIFORM_BUFFER, sizeof(glm::mat4), sizeof(glm::mat4), glm::value_ptr(view));
    glBindBuffer(GL_UNIFORM_BUFFER, 0);  
```

I to wszystko jeżeli chodzi o obiekty bufora uniformów. Każdy Vertex Shader, który zawiera blok uniformów <var>Matrices</var>, będzie teraz zawierał dane przechowywane w <var>uboMatices</var>. Gdybyśmy teraz mieli narysować 4 kostki za pomocą 4 różnych shaderów, ich macierz projekcji i widoku powinny pozostać takie same:

```cpp
    glBindVertexArray(cubeVAO);
    shaderRed.use();
    glm::mat4 model;
    model = glm::translate(model, glm::vec3(-0.75f, 0.75f, 0.0f));	// przesuń w lewo i w górę
    shaderRed.setMat4("model", model);
    glDrawArrays(GL_TRIANGLES, 0, 36);        
    // ... narysuj zieloną kostkę
    // ... narysuj niebieską kostkę
    // ... narysuj żółtą kostkę  
```

Jedynym uniformem, który musimy jeszcze ustawić, jest macierz <var>model</var>. Używanie obiektu bufora uniformów w takim kontekście pozwala nam zaoszczędzić sporo zwykłych wywołań funkcji dla każdego shadera. Wynik wygląda mniej więcej tak:

![Obraz 4 kostek z ich uniformami ustawionymi na obiekt bufora uniformów OpenGL](/img/learnopengl/advanced_glsl_uniform_buffer_objects.png){: .center-image }

Każda kostka jest przesuwana w inny róg okna poprzez zmianę macierzy modelu. Ze względu na różne Fragment Shadery kolory tych obiektów są różne. Jest to stosunkowo prosty scenariusz, w którym możemy używać obiektów bufora uniformów, ale każda duża aplikacja może mieć ponad setki aktywnych shaderów; to tam właśnie zaczynają błyszczeć obiekty bufora uniformów.

Możesz znaleźć pełny kod źródłowy aplikacji [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/4.advanced_opengl/8.advanced_glsl_ubo/advanced_glsl_ubo.cpp).

Obiekty bufora uniformów mają kilka zalet w stosunku do zwykłych uniformów. Po pierwsze, ustawienie wielu uniformów na raz jest szybsze niż ustawianie wielu uniformów jeden po drugim. Po drugie, jeśli chcesz zmienić ten sam uniform dla kilku shaderów, znacznie łatwiej jest zmienić uniform raz w buforze. Zaletą, która nie jest od razu widoczna, jest to, że możesz używać znacznie więcej uniformów w shaderach przy użyciu obiektów bufora uniformów. OpenGL ma limit ilości danych, które może obsłużyć, co może być sprawdzane przy użyciu <var>GL_MAX_VERTEX_UNIFORM_COMPONENTS</var>. W przypadku stosowania obiektów bufora uniformów ten limit jest znacznie wyższy. Kiedy więc osiągniesz maksymalną liczbę uniformów (na przykład podczas animacji szkieletowych), zawsze możesz użyć obiektu bufora uniformów.