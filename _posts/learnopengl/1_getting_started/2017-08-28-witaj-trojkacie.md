---
layout: post
title: Witaj Trójkącie
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: pierwsze-kroki
---

{% include learnopengl.md link="Getting-started/Hello-Triangle" %}

W OpenGL wszystko znajduje się w przestrzeni 3D, ale ekran i okno aplikacji są dwuwymiarowymi tablicami pikseli, więc duża część pracy OpenGL polega na przekształcaniu wszystkich współrzędnych 3D we współrzędne 2D, które pasują do Twojego ekranu. Proces przekształcania współrzędnych 3D na współrzędne 2D jest zarządzany przez <span class="def">potok renderujący</span> OpenGL. Potok renderujący można podzielić na dwie duże części: pierwsza transformuje współrzędne 3D na współrzędne 2D, a druga część przekształca współrzędne 2D w już pokolorowane piksele. W tym samouczku krótko omówimy potok renderowania oraz to, jak możemy go wykorzystać na naszą korzyść, aby utworzyć kilka fantazyjnych efektów graficznych.  

{: .box-note }
Istnieje różnica między współrzędną 2D a pikselem. Współrzędna 2D jest bardzo precyzyjną reprezentacją punktu, informującą o tym gdzie znajduje się punkt w przestrzeni 2D, natomiast piksel 2D jest aproksymacją tego punktu ograniczonego rozdzielczością ekranu/okna.

Potok renderujący przyjmuje zbiór współrzędnych 3D i przekształca je w kolorowe piksele 2D na ekranie. Potok renderujący można podzielić na kilka etapów, gdzie każdy krok wymaga danych wyjściowych z poprzedniego etapu jako jego dane wejściowe. Wszystkie te kroki są wysoce wyspecjalizowane (posiadają jedną konkretną funkcję) i mogą być wykonywane równolegle. Ze względu na ich równoległość, większość dzisiejszych kart graficznych posiada tysiące małych rdzeni przetwarzania, które umożliwiają szybkie przetwarzanie danych w potoku renderującym, uruchamiając małe programy na GPU dla każdego etapu potoku. Te małe programy nazywane są <span class="def">programami cieniującymi</span> (ang. _shaders_).

Niektóre z tych shaderów są konfigurowane przez programistę, co pozwala nam na napisanie własnych shaderów w celu zastąpienia domyślnych. Daje nam to dużo większą kontrolę nad konkretnymi częściami potoku, a ponieważ działają na GPU, mogą zaoszczędzić nam cenny czas CPU. Shadery są pisane w języku <span class="def">OpenGL Shading Language (GLSL)</span> i będziemy go dokładniej analizować w następnym samouczku.

Poniżej znajdziesz abstrakcyjną reprezentację wszystkich etapów potoku graficznego. Należy zauważyć, że niebieskie sekcje przedstawiają sekcje, w których można "wstrzykiwać" własne shadery.

![The OpenGL graphics pipeline with shader stages]({{ site.baseurl }}/img/learnopengl/pipeline.png){: .center-image}

Jak widać, potok graficzny zawiera dużą liczbę sekcji, które obsługują jedną konkretną część konwersji danych wierzchołkowych na w pełni wyrenderowany piksel. Krótko wyjaśnimy każdą część potoku w uproszczony sposób, aby uzyskać dobry pogląd tego, jak działa potok renderujący.

Jako dane wejściowe, do potoku graficznego przekazujemy, tablicę trzech współrzędnych 3D, która powinna utworzyć trójkąt. Tą tablicę nazwijmy Vertex Data; Ta tablica to zbiór wszystkich wierzchołków. <span class="def">Wierzchołek</span> (ang. _vertex_) jest zasadniczo zbiórem współrzędnych 3D. Dane tego wierzchołka są reprezentowane za pomocą <span class="def">atrybutów wierzchołków</span> (ang. _vertex attributes_), które mogą zawierać dowolne dane, ale dla uproszczenia, załóżmy, że każdy wierzchołek składa się tylko z pozycji 3D i pewnej wartości koloru.

{: .box-note }
Aby OpenGL mógł wiedzieć, co zrobić ze zbioru współrzędnych i wartości kolorów, OpenGL wymaga podpowiedzi, jakiego typu geometrię chcesz utworzyć za pomocą danych. Czy chcemy, aby dane były renderowane jako zbiór punktów, zbiór trójkątów czy może tylko jako jedna długa prosta? Te wskazówki nazywane są prymitywami i są podawane do OpenGL podczas wywołania dowolnego z poleceń rysowania (ang. _draw call_). Niektóre z podanych wskazówek to <span class="var">GL_POINTS, GL_TRIANGLES</span> i <span class="var">GL_LINE_STRIP</span>.

Pierwszą częścią potoku jest <span class="def">Vertex Shader</span>, który pobiera jako dane wejściowe jeden wierzchołek. Głównym celem Vertex Shader'a jest przekształcenie współrzędnych 3D w inne współrzędne 3D (więcej o tym później). Dodatkowo pozwala nam również na pewne podstawowe przetwarzanie na wierzchołków.

Etap <span class="def">składania prymitywów</span> (ang. _primitive assembly_) przyjmuje jako wejście wszystkie wierzchołki (lub wierzchołek, jeśli wybrano <span class="var">GL_POINTS</span>) z Vertex Shader'a, który tworzy prymitywy i montuje wszystkie punkty w podany kształt; w tym przypadku trójkąt.

Wyjście z etapu składania prymitywów jest przekazywane do <span class="def">Geometry Shader</span>. Shader geometrii przyjmuje jako dane wejściowe kolekcję wierzchołków, które tworzą prymityw i mają zdolność generowania innych kształtów, emitując nowe wierzchołki w celu stworzenia nowych (lub innych) prymitywów. W tym przypadku zostanie wygenerowany drugi trójkąt z podanego kształtu.

Wyjście Geometry Shader'a zostaje następnie przekazane do <span class="def">etapu rasteryzacji</span> (ang. _rasterization stage_), w którym mapuje uzyskane prymitywy z odpowiadającymi im pikselami na finalnym ekranie, dając w rezultacie fragmenty dla Fragment Shader'a. Zanim Fragment Shader zostanie uruchomiony, wykonywane jest <span class="def">obcinanie</span> (ang. _clipping_). Obcinanie odrzuca wszystkie fragmenty, które są poza obszarem renderowania, zwiększając tym samym wydajność rysowania.

{: .box-note }
Fragment w OpenGL to wszystkie dane, które są wymagane przez OpenGL, do wyrenderowania pojedynczego piksela.

Głównym celem <span class="def">Fragment Shader'a</span> jest obliczenie końcowego koloru piksela i jest to zazwyczaj etap, w którym tworzy się wszystkie zaawansowane efekty OpenGL. Zazwyczaj Fragment Shader zawiera dane o scenie 3D, których można użyć do obliczania końcowego koloru piksela (takiego jak światła, cienie, kolor światła itp.).

Po ustaleniu wszystkich odpowiednich wartości koloru, obiekt końcowy przechodzi przez jeszcze jeden etap, który nazywamy jest <span class="def">testem głębokości</span> (ang. depth test), <span class="def">testem alfy</span> (ang. alpha test) i <span class="def">testem mieszania</span> (ang. blending test). Etap ten sprawdza odpowiednią wartość głębokości (i szablonu; dojdziemy do tego później) fragmentu potrzebnej do sprawdzenia, czy powstały fragment znajduje się przed lub za innymi obiektami i czy powinien zostać odrzucony. Etap sprawdza także wartości <span class="def">alfa</span> (wartości alfa definiują krycie/przezroczystość obiektu) i <span class="def">mieszania</span> (ang. blending) dla obiektów. Tak więc, nawet jeśli w Fragment Shader jest obliczany kolor piksela, to ostateczny kolor może być zupełnie inny w przypadku renderowania wielu trójkątów.

Jak widać, potok graficzny jest całkiem złożoną całością i zawiera wiele konfigurowalnych części. Jednak w niemal wszystkich przypadkach musimy pracować tylko z Vertex i Fragment Shaderem. Geometry Shader jest opcjonalny i zazwyczaj pozostaje domyślnym programem cieniującym.

Nowoczesny OpenGL **wymaga**, abyśmy sami zdefiniowali co najmniej Vertex i Fragment Shader (nie ma domyślnych Vertex / Fragment Shaderów na GPU). Z tego powodu bardzo często trudno jest rozpocząć naukę nowoczesnego OpenGL, ponieważ wymagana jest duża wiedza, zanim będzie można wyrenderować pierwszy trójkąt. Gdy dojdziesz do końca tego rozdziału i wyrenderujesz swój pierwszy trójkąt, to będziesz posiadał znacznie większą wiedzę na temat programowania grafiki.

## Vertex input

Aby rozpocząć rysowanie, musimy najpierw podać OpenGL dane wierzchołków. OpenGL jest biblioteką grafiki 3D, więc wszystkie współrzędne, które będziemy definiować będą w 3D układzie współrzędnych (współrzędne x, y i z). OpenGL nie przekształca **wszystkich** Twoich współrzędnych 3D na piksele 2D na ekranie; OpenGL przetwarza tylko współrzędne 3D, które znajdują się w określonym przedziale między<span class="var">-1.0</span> a <span class="var">1.0</span> na wszystkich 3 osiach (x, y i z). Wszystkie współrzędne, w tym tak zwanym <span class="def">znormalizowanym układzie współrzędnych</span> (ang. _normalized device coordinates_, _NDC_) będą widoczne na ekranie (a wszystkie współrzędne poza tym regionem nie będą).

Ponieważ chcemy renderować pojedynczy trójkąt, to musimy podać łącznie trzy wierzchołki, gdzie każdy wierzchołek posiada pozycję 3D. Zdefiniujemy je w znormalizowanym układzie współrzędnych (widocznym obszarze OpenGL) w tablicy <span class="var">GLfloat</span>:

```cpp
GLfloat vertices[] = {  
                        -0.5f, -0.5f, 0.0f,  
                         0.5f, -0.5f, 0.0f,  
                         0.0f,  0.5f, 0.0f  
                     };
```

Ponieważ OpenGL pracuje w przestrzeni 3D, to dlatego tworzymy trójkąt 2D z każdym wierzchołkiem o współrzędnej z równej <span class="var">0.0</span>. W ten sposób _głębokość_ (ang. _depth_) trójkąta pozostaje taka sama, co sprawia, że wygląda jakby był w 2D.

{: .box-note }
**Normalized Device Coordinates (NDC)**  
Kiedy współrzędne wierzchołków zostały przetworzone w Vertex Shader, powinny znajdować się w <span class="def">układzie współrzędnych znormalizowanych (NDC)</span>, czyli małej przestrzeni, gdzie wartości x, y i z mieszczą się w przedziale od <span class="var">-1.0</span> do <span class="var">1.0</span>. Wszystkie współrzędne poza tym zakresem zostaną odrzucone/przycięte i nie będą widoczne na ekranie. Poniżej widać trójkąt określony w NDC (ignorując oś z):  
![2D Normalized Device Coordinates as shown in a graph]({{ site.baseurl }}/img/learnopengl/ndc.png){: .center-image }
W przeciwieństwie do zwykłych współrzędnych ekranu dodatnia oś y wskazuje w górę, a współrzędne <span class="var">(0,0)</span> znajdują się w centrum wykresu, zamiast w górnym lewym. W końcu chcesz, aby wszystkie (przekształcone) współrzędne znalazły się w tej przestrzeni współrzędnych, bo w przeciwnym razie nie będą widoczne.  
Współrzędne NDC zostaną przekształcone we <span class="def">współrzędne ekranu</span> (ang. _screen space_) za pomocą <span class="def">transformacji obszaru renderowania</span> (ang. _viewport transform_) przy użyciu danych podanych w <span class="fun">glViewport</span>. Powstałe współrzędne ekranu są następnie przekształcane w fragmenty jako wejście do Fragment Shader'a.

Mając określone dane wierzchołków chcemy wysłać je jako dane wejściowe do pierwszego etapu potoku graficznego: Vertex Shader. Odbywa się to przez utworzenie pamięci na GPU, w której przechowujemy dane wierzchołków, ustawienie sposobu, w jaki OpenGL powinien interpretować pamięć i określić sposób wysłania danych do karty graficznej. Vertex Shader przetwarza tyle wierzchołków, ile mu powiemy podczas wywoływania operacji rysowania.

Możemy zarządzać tą pamięcią za pomocą tak zwanych <span class="def">Vertex Buffer Objects (VBO)</span>, które mogą przechowywać dużą liczbę wierzchołków w pamięci GPU. Zaletą korzystania z tych obiektów buforowych jest możliwość wysyłania dużych partii danych na kartę graficzną bez konieczności wysyłania danych pojedynczego wierzchołka za każdym razem. Wysyłanie danych na kartę graficzną z CPU jest stosunkowo wolne, więc gdziekolwiek możemy to wysłajmy jak najwięcej danych na raz. Gdy dane znajdują się w pamięci karty graficznej, to Vertex Shader ma prawie natychmiastowy dostęp do wierzchołków, co czyni go bardzo szybkim.

Obiekt VBO jest naszym pierwszym obiektem OpenGL (obiekty OpenGL omówiliśmy w części [OpenGL]({% post_url learnopengl/1_getting_started/2017-07-10-opengl %}). Podobnie jak każdy obiekt w OpenGL, ten bufor posiada unikatowy identyfikator. Możemy wygenerować bufor z identyfikatorem przy użyciu funkcji <span class="fun">glGenBuffers</span>:

```cpp
GLuint VBO;  
glGenBuffers(1, &VBO);
```

OpenGL ma wiele typów obiektów buforowych. Typem bufora wierzchołków jest <span class="var">GL_ARRAY_BUFFER</span>. OpenGL umożliwia nam powiązanie kilku buforów naraz, o ile mają inny ID. Możemy powiązać nowo utworzony bufor z docelowym typem <span class="var">GL_ARRAY_BUFFER</span> za pomocą funkcji <span class="fun">glBindBuffer</span>:

```cpp
glBindBuffer(GL_ARRAY_BUFFER, VBO);
```

Od tego momentu każde dowolne wywołanie buforowe (na typie <span class="var">GL_ARRAY_BUFFER</span>) zostanie użyte do skonfigurowania aktualnie powiązanego bufora, którym jest nasze <span class="var">VBO</span>. Następnie możemy wywołać funkcję <span class="fun">glBufferData</span>, która kopiuje poprzednio określone dane wierzchołkowe do pamięci bufora:

```cpp
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

<span class="fun">glBufferData</span> jest funkcją specjalnie przeznaczoną do kopiowania danych zdefiniowanych przez użytkownika do obecnie powiązanego bufora. Pierwszym argumentem jest typ bufora, do którego chcemy skopiować dane: bufor aktualnie powiązanego z typem <span class="var">GL_ARRAY_BUFFER</span>. Drugi argument określa rozmiar danych (w bajtach), które chcemy przekazać do bufora; wystarczy proste wywołanie <span class="var">sizeof</span> na tablicy danych wierzchołków. Trzecim parametrem są rzeczywiste dane, które chcemy wysłać.

Czwarty parametr określa, jak chcemy, aby karta graficzna zarządzała danymi. Może to robić na 3 sposoby:

*   <span class="var">GL_STATIC_DRAW</span>: dane najprawdopodobniej nie zmienią się w ogóle lub bardzo rzadko.
*   <span class="var">GL_DYNAMIC_DRAW</span>: dane prawdopodobnie zmienią się często.
*   <span class="var">GL_STREAM_DRAW</span>: dane będą zmieniać się za każdym razem, gdy będą używane do rysowania.

Dane pozycji naszego trójkąta nie zmieniają się i pozostają takie same dla każdego wywołania renderującego, więc jego typem użycia powinien być najlepiej typ <span class="var">GL_STATIC_DRAW</span>. Jeśli na przykład ktoś miałby bufor z danymi, które często się zmieniają, to typ użycia <span class="var">GL_DYNAMIC_DRAW</span> lub <span class="var">GL_STREAM_DRAW</span> **może** spowodować, że karta graficzna umieści dane w pamięci, która pozwala na szybszy zapis.

Od teraz przechowujemy dane wierzchołkowe w pamięci karty graficznej, zarządzanej przez obiekt buforu wierzchołkowego o nazwie VBO. Teraz chcemy utworzyć Vertex i Fragment Shader, które będą rzeczywiście przetwarzać te dane, więc zacznijmy je tworzyć.

## Vertex shader

Vertex Shader jest jednym z programów cieniujących, który jest programowany przez nas. Współczesny OpenGL wymaga, aby przynajmniej utworzyć Vertex i Fragment Shader, jeśli chcemy coś wyrenderować. Dlatego pokrótce wprowadzimy i skonfigurujemy dwa bardzo proste shadery do rysowania naszego pierwszego trójkąta. W następnym samouczku będziemy bardziej szczegółowo omawiać shadery.

Pierwszą rzeczą, którą musimy zrobić, to napisać Vertex Shader w języku cieniującym GLSL (OpenGL Shading Language), a następnie skompilować, abyśmy mogli go używać w naszej aplikacji. Poniżej znajdziesz kod źródłowy bardzo prostego Vertex Shader'a w GLSL:

```glsl
#version 330 core  
layout (location = 0) in vec3 position;

void main()  
{  
    gl_Position = vec4(position.x, position.y, position.z, 1.0);  
}
```

Jak widać, GLSL wygląda podobnie do C. Każdy program cieniujący zaczyna się od deklaracji jego wersji. Od wersji OpenGL 3.3 i wyższej numery wersji GLSL są zgodne z wersją OpenGL (na przykład wersja GLSL w wersji 420 odpowiada OpenGL w wersji 4.2). Jawnie mówimy w tym kodzie, że używamy funkcji profilu core.

Następnie deklarujemy wszystkie wejściowe atrybuty wierzchołków (ang. _input vertex attributes_) w Vertex Shaderze oznaczając je słowem kluczowym _in_. Teraz tylko interesują nas dane o pozycji, więc potrzebujemy tylko jednego atrybutu wierzchołka. GLSL posiada typy danych wektorowych, który zawiera od 1 do 4 komponentów typu <span class="var">float</span>. Ponieważ każdy wierzchołek posiada współrzędną 3D, tworzymy zmienną wejściową <span class="var">vec3</span> o nazwie <span class="var">position</span>. Dokładnie określamy również lokalizację tej zmiennej wejściowej za pomocą kwalifikatora <span class="var">layout (location = 0)</span>. Później zobaczysz, dlaczego potrzebujemy tej lokalizacji.

{: .box-note }
**Wektor**  
W programowaniu grafiki, dość często używamy matematycznej koncepcji wektora, ponieważ w prosty sposób reprezentuje pozycje/kierunki w dowolnej przestrzeni i ma użyteczne właściwości matematyczne. Wektor w GLSL ma maksymalny rozmiar 4, a każda z jego wartości może być pobierana za pomocą <span class="var">vec.x, vec.y, vec.z</span> i <span class="var">vec.w</span>, gdzie każda z nich reprezentuje współrzędną w przestrzeni. Zauważ, że składnik <span class="var">vec.w</span> nie jest używany jako pozycja w przestrzeni (mamy do czynienia z 3D, a nie 4D), ale jest używany do czegoś o nazwie <span class="def">dzielenie perspektywy</span> (ang. _perspective division_). Wektory zostaną bardziej szczegółowo omówione w późniejszym samouczku.

Aby ustawić wyjście Vertex Shader'a, musimy przypisać dane położenia do predefiniowanej zmiennej <span class="var">gl_Position</span>, która za kulisami jest typu <span class="var">vec4</span>. Na końcu funkcji <span class="fun">main</span>, niezależnie od tego, jak ustawiamy <span class="var">gl_Position</span> to będzie ona użyta jako wyjście Vertex Shader'a. Ponieważ nasza dana wejściowa jest wektorem o rozmiarze 3, musimy ją zrzutować na wektor o rozmiarze 4\. Możemy to zrobić, wstawiając wartości <span class="var">vec3</span> wewnątrz konstruktora <span class="var">vec4</span> i ustawić jego składnik <span class="var">w</span> na <span class="var">1.0f</span> (wyjaśnimy dlaczego jest tu taka wartość w późniejszym samouczku).

Aktualny Vertex Shader jest prawdopodobnie najprostszym programem cieniującym, który możemy sobie wyobrazić, ponieważ nie przetwarzaliśmy żadnych danych wejściowych i po prostu przekazaliśmy je do wyjścia programu cieniującego. W rzeczywistych aplikacjach, dane wejściowe zwykle nie są przekształcone do przestrzeni NDC, dlatego najpierw musimy przekształcić dane wejściowe we współrzędne leżące w widocznym obszarze OpenGL.

## Kompilacja shader'a

Napisaliśmy kod źródłowy dla Vertex Shader'a (przechowywany w ciągu znakowym C), ale aby OpenGL mógł go używać, to musi skompilować go w czasie wykonywania programu korzystając z jego kodu źródłowego.

Pierwszą rzeczą, którą musimy zrobić, jest utworzenie obiektu shader'a (ang. _shader object_), oznaczonego ID. Dlatego tworzymy uchwyt o typie <span class="var">GLuint</span> do tego obiektu i tworzymy go za pomocą funkcji <span class="fun">glCreateShader</span>:

```cpp
GLuint vertexShader;  
vertexShader = glCreateShader(GL_VERTEX_SHADER);
```

Jako argument funkcji <span class="fun">glCreateShader</span> przekazujemy typ shader'a jaki chcemy utworzyć. Jako, że tworzymy Vertex Shader, to przekazujemy wartość <span class="var">GL_VERTEX_SHADER</span>.

Następnie dołączamy kod źródłowy shader'a do shader object'a i go kompilujemy:

```cpp
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);  
glCompileShader(vertexShader);
```

Funkcja <span class="fun">glShaderSource</span> bierze jako pierwszy argument, obiekt shader'a, który ma skompilować. Drugi argument określa, ile łańcuchów znakowych przekazujemy jako kod źródłowy. W tym przypadku jest tylko jeden. Trzecim parametrem jest aktualny kod źródłowy shader'a. Czwarty parametr zostawmy póki co ustawiony na wartość <span class="var">NULL</span>.

<div class="box-note">Prawdopodobnie chcesz sprawdzić, czy kompilacja zakończyła się sukcesem po wywołaniu funkcji <span class="fun">glCompileShader</span> i jeśli kompilacja nie powiodła się, to jakie błędy zostały znalezione, które możemy naprawić. Sprawdzenie błędów kompilacji odbywa się w następujący sposób:  

```cpp
GLint success;  
GLchar infoLog[512];  
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
```

Najpierw definiujemy zmienną int przechowującą informację o tym czy operacja się powiodła i bufor dla komunikatów o błędach (jeśli istnieją). Następnie sprawdzamy, czy kompilacja zakończyła się sukcesem za pomocą funkcji <span class="fun">glGetShaderiv</span>. Jeżeli kompilacja nie powiodła się, to powinniśmy pobrać wiadomość informującą o błędach za pomocą funkcji <span class="fun">glGetShaderInfoLog</span> i wyświetlić ją na ekranie.  

```cpp
if(!success)  
{  
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);  
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::end;  
}
```

</div>

Jeżeli nie wystąpiły żadne błędy podczas kompilowania Vertex Shader'a to został on poprawnie skompilowany.

## Fragment Shader

Fragment Shader jest drugim i ostatnim programem cieniującym, który utworzymy w celu renderowania trójkąta. Fragment Shader zajmuje się obliczaniem kolorów dla pikseli. Aby nie komplikować, to Fragment Shader zawsze będzie wyświetlał kolor pomarańczowy.

{: .box-note }
Kolory w grafice komputerowej są reprezentowane jako tablica <span class="var">4</span> wartości: składnik czerwony, zielony, niebieski i alfa (przezroczystość), powszechnie określany skrótem RGBA. Określając kolor w OpenGL lub GLSL ustawiamy siłę każdego elementu na wartość pomiędzy <span class="var">0.0</span> a <span class="var">1.0</span>. Jeśli na przykład ustawimy kolor czerwony na <span class="var">1.0f</span>, a zielony na <span class="var">1.0f</span> otrzymamy mieszaninę obu kolorów i otrzymamy kolor żółty. Biorąc pod uwagę te 3 składniki kolorów, możemy wygenerować ponad 16 milionów kolorów!

```glsl
#version 330 core  
out vec4 color;

void main()  
{  
    color = vec4(1.0f, 0.5f, 0.2f, 1.0f);  
}
```

Fragment Shader wymaga tylko jednej zmiennej wyjściowej i jest to wektor o rozmiarze <span class="var">4</span>, który definiuje końcowy kolor, który musimy sami obliczyć. Możemy zadeklarować wartości wyjściowe za pomocą słowa kluczowego out, które pojawia się przed zmienną color. Następnie przypisujemy kolor pomarańczowy do zmiennej color jako <span class="var">vec4</span> z wartością alfa wynoszącą <span class="var">1.0</span> (<span class="var">1.0</span> oznacza całkowitą nieprzezroczystość).

Proces kompilacji Fragment Shader'a jest podobny do kompilacji Vertex Shader'a, ale tym razem używamy wartości <span class="var">GL_FRAGMENT_SHADER</span> jako typ programu cieniującego:

```cpp
GLuint fragmentShader;  
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);  
glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);  
glCompileShader(fragmentShader);
```

Obydwa shadery są teraz skompilowane, a jedyną rzeczą, jaką należy zrobić, to powiązać oba shader object z <span class="def">program object</span>, który można wykorzystać do renderowania.

### Program Object

Program Object łączy w sobie zlinkowane ze sobą typy shaderów. Aby użyć niedawno skompilowanych shaderów, musimy je <span class="def">zlinkować</span> dołączyć do Program Object, a następnie go aktywować przed renderowaniem obiektów. Aktywowany Program Object będzie użyty podczas wywoływania funkcji renderujących (ang. _render calls_).

Podczas linkowania shaderów w Program Object, wyjścia każdego shader'a są łączone z wejściami następnego programu cieniującego. Jest to również miejsce, w którym pojawią się błędy linkowania, jeśli dane wyjściowe i wejściowe nie pasują do siebie.

Tworzenie Program Object jest łatwe:

```cpp
GLuint shaderProgram;  
shaderProgram = glCreateProgram();
```

Funkcja <span class="fun">glCreateProgram</span> tworzy Program Object i zwraca referencję (ID) do nowo utworzonego obiektu. Teraz musimy dołączyć wcześniej wygenerowane shadery do Program Object, a następnie je powiązać za pomocą funkcji <span class="fun">glLinkProgram</span>:

```cpp
glAttachShader(shaderProgram, vertexShader);  
glAttachShader(shaderProgram, fragmentShader);  
glLinkProgram(shaderProgram);
```

Powyższy kod powinien być całkiem zrozumiały. Dołączamy shadery do programu i łączymy je za pomocą funkcji <span class="fun">glLinkProgram</span>.

<div class="box-note">Podobnie jak przy kompilacji programów cieniujących, możemy sprawdzić, czy linkowanie Program Object powiodło się czy nie i możemy pobrać odpowiednią wiadomość o błędach. Jednak zamiast używać funkcji <span class="fun">glGetShaderiv</span> i <span class="fun">glGetShaderInfoLog</span> używamy:

```cpp
glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);  
if(!success) {  
    glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);  
    ...  
}
```

</div>

Wynikiem jest Program Object, który możemy aktywować, wywołując funkcję <span class="fun">glUseProgram</span> z nowo utworzonym programem jako argumentem:

```cpp
glUseProgram(shaderProgram);
```

Każde wywołanie draw call'a po wywołaniu funkcji <span class="fun">glUseProgram</span> będzie korzystało z tego Program Object.

Nie zapomnij usunąć obiektów shaderów po powiązaniu ich z Program Object; już ich nie potrzebujemy:

```cpp
glDeleteShader(vertexShader);  
glDeleteShader(fragmentShader);
```

Teraz mamy już przesłane dane wierzchołków do GPU i poinstruowaliśmy GPU, w jaki sposób powinien przetwarzać te dane w Vertex i Fragment Shader. Jesteśmy prawie przy końcu, ale jeszcze nie całkiem. OpenGL nie wie jeszcze, jak powinien interpretować dane wierzchołków, które zapisaliśmy w pamięci i jak powinien podłączyć dane wierzchołków do atrybutów Vertex Shader'a. Będziemy mili i powiemy OpenGL jak to zrobić.

## Łączenie atrybutów wierzchołków (ang. _Vertex Attributes_)  
    
Vertex Shader pozwala nam określić dowolne dane wejściowe w postaci atrybutów wierzchołków, a to pozwala na dużą elastyczność. A to oznacza, że musimy ręcznie określić, jaką część naszych danych wejściowych ma trafić do konkretnego atrybutu w Vertex Shader. Oznacza to, że musimy określić, w jaki sposób OpenGL powinien interpretować dane wierzchołków przed renderowaniem.

Nasze dane w buforze wierzchołków są ustawione w następujący sposób:

![Vertex attribte pointer setup of OpenGL VBO]({{ site.baseurl }}/img/learnopengl/vertex_attribute_pointer.png)

*   Dane pozycji są zapisywane jako 32-bitowe (4 bajtowe) wartości zmiennoprzecinkowe.
*   Każda pozycja składa się z 3 takich wartości.
*   Nie ma miejsca (lub innych wartości) pomiędzy każdym zestawem 3 wartości. Wartości są <span class="def">ściśle upakowane</span> w tablicy.
*   Pierwsza wartość jest na początku buforu.

Dzięki tej wiedzy możemy powiedzieć OpenGL, jak należy interpretować dane wierzchołków (na każdy atrybut wierzchołka) używając funkcji <span class="fun">glVertexAttribPointer</span>:

```cpp
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);  
glEnableVertexAttribArray(0);
```

Funkcja <span class="fun">glVertexAttribPointer</span> ma kilka parametrów, więc prześledźmy je uważnie:

*   Pierwszy parametr określa atrybut wierzchołka, który chcemy skonfigurować. Pamiętaj, że określiliśmy lokalizację atrybutu <span class="var">position</span> w Vertex Shader za pomocą <span class="var">layout (location = 0)</span>. Ustawia to położenie atrybutu wierzchołka na pozycji <span class="var">0</span>, a ponieważ chcemy przekazać dane do tego atrybutu wierzchołka, to przekazujemy jako parametr wartość <span class="var">0</span>.
*   Następny argument określa rozmiar atrybutu wierzchołka. Atrybutem wierzchołka jest <span class="var">vec3</span> - składa się z 3 wartości.
*   Trzeci argument określa typ danych, którym jest <span class="var">GL_FLOAT</span> (typ <span class="var">vec*</span> w GLSL składa się z wartości zmiennoprzecinkowych).
*   Następny argument określa, czy chcemy, aby dane zostały znormalizowane. Jeśli ustawimy to na wartość <span class="var">GL_TRUE</span> to wszystkie dane, których wartość nie zawiera wartości w przedziale <span class="var">0</span> (lub <span class="var">-1</span> dla wartości ze znakiem), a <span class="var">1</span> to zostaną one zmapowane do tego przedziału. Ustawiamy to na wartość <span class="var">GL_FALSE</span>.
*   Piąty argument jest znany jako <span class="def">skok</span> (ang. stride) i mówi nam o tym, jaka jest przestrzeń pomiędzy kolejnymi zestawami atrybutów wierzchołków. Ponieważ następny zestaw danych dotyczących pozycji znajduje się dokładnie po 3 danych typu <span class="var">GLfloat</span>, to ustawiamy tę wartość jako skok. Zauważ, że skoro wiemy, że tablica jest ściśle upakowana (nie ma miejsca pomiędzy kolejnymi wartościami atrybutów wierzchołków) to możemy też określić skok jako <span class="var">0</span>, aby OpenGL mógł sam określić skok (to tylko działa gdy wartości są szczelnie upakowane). Kiedy mamy więcej atrybutów wierzchołków, musimy sami dokładnie określić odstęp między każdym atrybutem wierzchołka. Jak to zrobić zobaczymy w późniejszym przykładzie.
*   Ostatni parametr jest typu <span class="var">GLvoid*</span> i dlatego wymaga tego dziwnego rzutowania. Jest to <span class="def">offset</span> oznaczajacy gdzie dane pozycji zaczynają się w buforze. Ponieważ dane pozycji znajdują się na początku tablicy danych, wartość ta wynosi <span class="var">0</span>. Później zbadamy ten parametr bardziej szczegółowo.

{: .box-note }
Każdy atrybut wierzchołka pobiera swoje dane z pamięci zarządzanej przez VBO, i z którego VBO pobiera dane (może być wiele VBO). Jest to określane przez aktualnie powiązane VBO z <span class="var">GL_ARRAY_BUFFER</span> podczas wywołania funkcji <span class="fun">glVertexAttribPointer</span>. Ponieważ wcześniej zdefiniowany VBO był powiązany przed wywołaniem funkcji <span class="fun">glVertexAttribPointer</span> atrybut wierzchołka <span class="var">0</span> jest teraz skojarzony z jego danymi wierzchołków.

Teraz, gdy określiliśmy, jak OpenGL powinien interpretować dane wierzchołków, należy również włączyć dany atrybut wierzchołka za pomocą funkcji <span class="fun">glEnableVertexAttribArray</span> przekazując jej jako argument lokalizację atrybutu wierzchołka; atrybuty wierzchołków są domyślnie wyłączone. Od tego momentu mamy wszystko skonfigurowane: zainicjowaliśmy dane wierzchołków w buforze przy użyciu Vertex Buffer Object, ustawiliśmy Vertex i Fragment Shader i powiedzieliśmy OpenGL, jak połączyć dane wierzchołków z danymi atrybutami w Vertex Shader. Rysowanie obiektu za pomocą OpenGL wygląda teraz tak:

```cpp
// 0. Wypełnij VBO danymi wierzchołków  
glBindBuffer(GL_ARRAY_BUFFER, VBO);  
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  
// 1. Ustaw wskaźniki atrybutu wierzchołka  
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);  
glEnableVertexAttribArray(0);  
// 2. Włącz program cieniujący, którego chcemy użyć do renderowania  
glUseProgram(shaderProgram);  
// 3. Narysuj obiekt  
someOpenGLFunctionThatDrawsOurTriangle();
```

Musimy powtórzyć ten proces za każdym razem, gdy chcemy narysować obiekt. Może to nie wyglądać na tak długi proces, ale wyobraź sobie, że mamy ponad 5 atrybutów wierzchołków i około 100 różnych przedmiotów (co nie jest rzadko spotykane). Powiązanie odpowiednich obiektów bufora i konfigurowanie wszystkich atrybutów wierzchołków dla każdego z tych obiektów szybko staje się uciążliwym procesem. Co by było, gdybyśmy mogli przechowywać wszystkie te konfiguracje stanów w jednym obiekcie i po prostu podpiąć ten obiekt do kontekstu, aby przywrócić jego stan?

### Vertex Array Object

<span class="def">Vertex Array Object</span> (również znane jako <span class="def">VAO</span>) może być powiązany z kontekstem podobnie jak Vertex Buffer Object, a kolejne odwołania do atrybutu wierzchołka są przechowywane wewnątrz VAO. Ma to tę zaletę, że podczas konfigurowania wskaźników do atrybutów wierzchołków wystarczy tylko raz je ustawić i kiedy tylko chcemy narysować obiekt, możemy po prostu podpiąć do kontekstu odpowiednie VAO. To sprawia, że przełączanie pomiędzy różnymi danymi wierzchołkowymi i konfiguracjami atrybutów jest tak proste, jak podłączanie różnych VAO. Cały stan, który właśnie ustawimy, będzie przechowywany wewnątrz VAO.

{: .box-error }
Core OpenGL **wymaga** żebyśmy używali VAO, by wiedział co zrobić z naszymi wierzchołkami. Jeśli nie uda nam się podpiąć VAO, OpenGL najprawdopodobniej odmówi narysowania czegokolwiek.

VAO przechowuje następujące informacje:

*   Wywołania do funkcji <span class="fun">glEnableVertexAttribArray</span> lub <span class="fun">glDisableVertexAttribArray</span> - które atrybuty wierzchołków są włączone, a które nie.
*   Konfigurację atrybutów wierzchołków za pomocą funkcji <span class="fun">glVertexAttribPointer</span>.
*   Vertex Buffer Objects, które są skojarzone z odpowiednimi atrybutami wierzchołków, poprzez wywołanie fukcji <span class="fun">glVertexAttribPointer</span>.

![Image of how a VAO (Vertex Array Object) operates and what it stores in OpenGL]({{ site.baseurl }}/img/learnopengl/vertex_array_objects.png){: .center-image }

Proces generowania VAO wygląda podobnie do VBO:

```cpp
GLuint VAO;  
glGenVertexArrays(1, &VAO);
```

Aby korzystać z VAO, wszystko co musisz zrobić, to powiązać VAO używając funkcji<span class="fun">glBindVertexArray</span>. Od tego momentu powinniśmy powiązać/skonfigurować odpowiednie VBO i atrybuty wierzchołków. Jak tylko chcemy narysować obiekt, po prostu powiążemy VAO z kontekstem, które zawiera preferowane ustawienia, przed narysowaniem obiektu. W kodzie wyglądałoby to mniej więcej tak:

```cpp
// ..:: Inicjalizacja (robione raz (o ile Twoje obiekty nie zmieniają się często)) :: ..  
// 1. powiąż Vertex Array Object  
glBindVertexArray(VAO);  
// 2. skopiuj naszą tablicę wierzchołków do VBO  
glBindBuffer(GL_ARRAY_BUFFER, VBO);  
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  
// 3. ustaw wskaźniki do atrybutów wierzchołków  
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);  
glEnableVertexAttribArray(0);

[...]

// ..:: Kod renderujący (w pętli renderującej) :: ..  
// 4. narysuj obiekt  
glUseProgram(shaderProgram);  
glBindVertexArray(VAO);  
someOpenGLFunctionThatDrawsOurTriangle(); 
```

I to jest wszystko! Wszystko, co zrobiliśmy na ostatnich kilku milionach stron, prowadziło do tej chwili - VAO, które przechowuje naszą konfigurację atrybutów wierzchołków i których VBO ma użyć. Zwykle, gdy masz wiele obiektów, które chcesz narysować, najpierw generujesz/konfigurujesz wszystkie VAO (a więc wymagane VBO i atrybuty wierzchołków) i przechowujesz je do późniejszego użycia. W chwili, gdy chcemy narysować jeden z naszych obiektów, bierzemy odpowiednie VAO, wiążemy je z kontekstem, a następnie rysujemy obiekt.

### Trójkąt, na który wszyscy czekaliśmy

Aby narysować wybrane obiekty, OpenGL udostępnia nam funkcję <span class="fun">glDrawArrays</span>, która rysuje prymitywy używając obecnie aktywnego programy cieniującego, poprzednio zdefiniowaną konfigurację atrybutów wierzchołków oraz dane wierzchołkowe w VBO (pośrednio powiązane przez VAO).

```cpp
glUseProgram(shaderProgram);  
glBindVertexArray(VAO);  
glDrawArrays(GL_TRIANGLES, 0, 3);
```

Funkcja <span class="fun">glDrawArrays</span> przyjmuje jako pierwszy argument typ prymitywu OpenGL, który chcemy narysować. Od początku mówiliśmy, że chcemy narysować trójkąt (a nie lubię kłamać), więc przekazujemy wartość <span class="var">GL_TRIANGLES</span>. Drugi argument określa indeks początkowy tablicy wierzchołków, którą chcielibyśmy narysować; Po prostu zostawiamy tu wartość <span class="var">0</span>. Ostatni argument określa, ile wierzchołków chcemy narysować - 3 (renderujemy tylko jeden trójkąt z naszych danych, czyli dokładnie 3 wierzchołki).

Teraz spróbuj skompilować kod i jeśli pojawią się jakieś błędy to je napraw. Gdy skompilujesz aplikację, powinieneś zobaczyć następujący wynik:

![An image of a basic triangle rendered in modern OpenGL]({{ site.baseurl }}/img/learnopengl/hellotriangle.png){: .center-image }

Kod źródłowy całego programu można znaleźć [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/2.1.hello_triangle/hello_triangle.cpp).

Jeśli obraz końcowy nie wygląda tak samo, to prawdopodobnie zrobiłeś coś nie tak. Sprawdź więc cały kod źródłowy, zobacz, czy czegoś nie brakowało lub poproś o pomoc w sekcji komentarzy.

## Element Buffer Object

Jest jedna rzecz, którą chciałbym omówić dotyczącą renderowania wierzchołków i jest to <span class="def">obiekt bufora elementów</span> (ang. _Element Buffer Object_) skracany do EBO. Aby wyjaśnić, jak działają te obiekty, najlepiej jest podać przykład: przypuśćmy, że chcemy narysować prostokąt zamiast trójkąta. Możemy narysować prostokąt przy użyciu dwóch trójkątów (OpenGL działa głównie z trójkątami). Spowoduje to wygenerowanie następującego zestawu wierzchołków:

```cpp
GLfloat vertices[] = {  
    // Pierwszy trójkąt  
     0.5f,  0.5f, 0.0f, // Prawy górny  
     0.5f, -0.5f, 0.0f, // Prawy dolny  
    -0.5f,  0.5f, 0.0f, // Lewy górny  
    // Drugi trójkąt  
     0.5f, -0.5f, 0.0f, // Prawy dolny  
    -0.5f, -0.5f, 0.0f, // Lewy dolny  
    -0.5f,  0.5f, 0.0f // Lewy górny  
};
```

Jak widać, jest kilka powtarzających się pozycji wierzchołków. Ustawiamy dwa razy <span class="var">dolny prawy</span> i <span class="var">górny lewy</span>! Jest to narzut 50%, ponieważ ten sam prostokąt można również określić za pomocą tylko <span class="var">4</span> wierzchołków zamiast <span class="var">6</span>. To się pogorszy, gdy tylko będziemy mieli bardziej skomplikowane modele, które mają ponad 1000 trójkątów, gdzie będzie dużo więcej wierzchołków, które będą się dublować. Lepszym rozwiązaniem byłoby przechowywanie tylko tych wierzchołków, które się nie powtarzają, a następnie określenie kolejności, w jakiej chcemy narysować te wierzchołki. W tym przypadku musielibyśmy tylko zapisać 4 wierzchołki prostokąta, a następnie tylko określić, w jakim porządku chcielibyśmy je narysować. Czy nie byłoby wspaniale, gdyby OpenGL dostarczył nam taką funkcjonalność?

Na szczęście EBO działają dokładnie w ten wyżej opisany sposób. EBO to bufor, podobnie jak VBO, który przechowuje indeksy, których OpenGL używa do określenia, jakie wierzchołki ma narysować. Ten tak zwany <span class="def">indexed drawing</span> (rysowanie indeksowe) jest właśnie rozwiązaniem naszego problemu. Aby rozpocząć, musimy najpierw określić (unikalne) wierzchołki i indeksy, aby tworzyły prostokąt:

```cpp
GLfloat vertices[] = {  
     0.5f,  0.5f, 0.0f, // Prawy górny  
     0.5f, -0.5f, 0.0f, // Prawy dolny  
    -0.5f, -0.5f, 0.0f, // Lewy dolny  
    -0.5f,  0.5f, 0.0f  // Lewy górny  
};  

GLuint indices[] = { // Zauważ, że zaczynamy od 0!  
    0, 1, 3, // Pierwszy trójkąt  
    1, 2, 3 // Drugi trójkąt  
};
```

Możesz zauważyć, że przy używaniu indeksów, potrzebujemy tylko <span class="var">4</span> wierzchołków zamiast <span class="var">6</span>. Następnie musimy utworzyć EBO:

```cpp
GLuint EBO;  
glGenBuffers(1, &EBO);
```

Podobnie jak w przypadku VBO wiążemy EBO i kopiujemy do niego indeksy za pomocą <span class="fun">glBufferData</span>. Tak samo jak w przypadku VBO chcemy umieścić te wywołania po funkcji odpowiedzialnej za wiązanie. Tym razem jako typ buforu określamy <span class="var">GL_ELEMENT_ARRAY_BUFFER</span>.

```cpp
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);  
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
```

Zauważ, że teraz przekazujemy wartość <span class="var">GL_ELEMENT_ARRAY_BUFFER</span> jako typ bufora. Ostatnią rzeczą jaką trzeba zrobić jest zastąpienie funkcji <span class="fun">glDrawArrays</span> funkcją <span class="fun">glDrawElements</span> żeby zaznaczyć, że chcemy rysować obiekt przy użyciu EBO. Kiedy używamy funkcji <span class="fun">glDrawElements</span> to OpenGL będzie rysował obiekty za pomocą indeksów, które są skojarzone z obecnie powiązanym EBO.

```cpp
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);  
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

Pierwszy argument określa tryb, w jakim chcemy rysować, podobnie jak w przypadku funkcji <span class="fun">glDrawArrays</span>. Drugim argumentem jest liczba elementów, które chcielibyśmy narysować. Wyznaczyliśmy 6 indeksów, więc chcemy narysować łącznie 6 wierzchołków. Trzecim argumentem jest typ indeksów, które są typu <span class="var">GL_UNSIGNED_INT</span>. Ostatni argument pozwala nam określić przesunięcie (ang. offset) w EBO (lub przekazać wskaźnik do tablicy indeksowej, ale tylko wtedy gdy, nie używasz EBO), więc po prostu zostawiamy to na wartość 0.

Funkcja <span class="fun">glDrawElements</span> bierze indeksy z EBO, który jest aktualnie powiązanego z typem <span class="var">GL_ELEMENT_ARRAY_BUFFER</span>. Oznacza to, że musimy powiązać odpowiednie EBO za każdym razem, gdy chcemy renderować obiekt za pomocą indeksów, co wydaje się nieco kłopotliwe. Dzieje się tak dlatego, że VAO śledzi również EBO. EBO, który jest wiązany jest przechowywany jako obiekt VAO (o ile VAO zostało wcześniej powiązane z kontekstem). Wiązanie VAO automatycznie wiąże także EBO z kontekstem.

![Image of VAO's structure / what it stores now also with EBO bindings.]({{ site.baseurl }}/img/learnopengl/vertex_array_objects_ebo.png){: .center-image }

{: .box-error }
VAO śledzi wywołania <span class="fun">glBindBuffer</span> kiedy typem jest <span class="var">GL_ELEMENT_ARRAY_BUFFER</span>. Oznacza to również, że równieź śledzi funkcje, które odwiązują EBO od kontekstu. Musisz uważać, żeby nie odwiązać EBO kiedy podłączone jest w tym momencie VAO. W przeciwnym razie spowoduje to, że Twoje EBO nie zostanie skonfigurowane.

Wynikowy kod inicjalizacji i rysowania wygląda następująco:

```cpp
// ..:: Inicjalizacja :: ..  
// 1. powiąż Vertex Array Object  
glBindVertexArray(VAO);  
// 2. skopiuj naszą tablicę wierzchołków do VBO  
glBindBuffer(GL_ARRAY_BUFFER, VBO);  
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  
// 3. skopiuj naszą tablicę indeksów do EBO  
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);  
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);  
// 4. ustaw wskaźniki do atrybutów wierzchołków  
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);  
glEnableVertexAttribArray(0);

[...]

// ..:: Kod renderujący (w pętli renderującej) :: ..  
glUseProgram(shaderProgram);  
glBindVertexArray(VAO);  
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0)  
glBindVertexArray(0);
```

Uruchomienie programu powinno dawać obraz podobny do tego poniżej. Lewy obraz powinien wyglądać bardzo podobnie. Natomiast prawy obraz przedstawia prostokąt narysowany w <span class="def">trybie szkieletowym</span> (ang. _wireframe mode_). Prostokąt szkieletowy pokazuje, że faktycznie składa się on z dwóch trójkątów.

![A rectangle drawn using indexed rendering in OpenGL]({{ site.baseurl }}/img/learnopengl/hellotriangle2.png){: .center-image }

{: .box-note }
**Tryb szkieletowy**  
Aby narysować trójkąty w trybie szkieletowym, można skonfigurować sposób, w jaki OpenGL rysuje prymitywy za pomocą funkcji <span class="fun">glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)</span>. Pierwszy argument mówi, że chcemy zastosować ten tryb do przedniej i tylnej ścianki wszystkich trójkątów, a drugi parametr mówi, aby rysować je jako linie. Każde kolejne wywołanie rysowania spowoduje wyświetlenie trójkątów w trybie szkieletowym, dopóki nie przywrócimy go do wartości domyślnej, używając funkcji <span class="fun">glPolygonMode(GL_FRONT_AND_BACK, GL_FILL)</span>.

Jeśli masz jakieś błędy, prześledź swoją pracę w tył i sprawdź, czy czegoś nie brakuje. Pełny kod źródłowy możesz znaleźć [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/2.2.hello_triangle_indexed/hello_triangle_indexed.cpp). Możesz też zadać jakiekolwiek pytanie w sekcji komentarzy poniżej.

Jeśli udało Ci się narysować trójkąt lub prostokąt tak, jak to zrobiliśmy, gratulacje, udało Ci się przebrnąć przez najtrudniejszą część nowoczesnego OpenGL: narysowanie pierwszego trójkąta. Jest to trudna sprawa, ponieważ wymagany jest duży zasób wiedzy, zanim będzie można narysować swój pierwszy trójkąt. Na szczęście pokonaliśmy tę barierę, a kolejne lekcje będą (mam nadzieję) znacznie łatwiejsze do zrozumienia.

## Dodatkowe materiały

*   [antongerdelan.net/hellotriangle](http://antongerdelan.net/opengl/hellotriangle.html): próba rysowania pierwszego trójkąta przez Anton Gerdelan'a
*   [open.gl/drawing](https://open.gl/drawing): Alexander Overvoorde rysuje pierwszy trójkąt.
*   [antongerdelan.net/vertexbuffers](http://antongerdelan.net/opengl/vertexbuffers.html): kilka dodatkowych informacji na temat VBO.
*   [learnopengl.com/#!In-Practice/Debugging](https://learnopengl.com/#!In-Practice/Debugging): w tym samouczku było wiele kroków; jeżeli gdzieś stanąłeś, to być może lektura tego samoczuka pomoże Ci "zdebugować" aplikację OpenGL (tłumaczenie w przygotowaniu).
*   [Tutorial 03 – Pierwszy trójkąt]({% post_url beginner_opengl/2014-03-10-tutorial-03-pierwszy-trojkat %}): w tym samoczuku opisywałem proces rysowania pierwszego trójkąta - może się przydać.  

## Ćwiczenia

Aby uzyskać naprawdę dobre zrozumienie omawianych tutaj zagadnień, podaję kilka ćwiczeń. Zalecam, aby je zrobić, zanim przejdziesz do kolejnego tematu, aby upewnić się, że masz dobrą znajomość tego, co się dzieje.

*   Spróbuj narysować dwa trójkąty obok siebie przy użyciu <span class="fun">glDrawArrays</span> dodając więcej wierzchołków do danych: [rozwiązanie](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/2.3.hello_triangle_exercise1/hello_triangle_exercise1.cpp).
*   Teraz utwórz te same 2 trójkąty przy użyciu dwóch różnych VAO i VBO dla ich danych: [rozwiązanie](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/2.4.hello_triangle_exercise2/hello_triangle_exercise2.cpp).
*   Utwórz dwa programy cieniujące, gdzie drugi program używa innego Fragment Shader'a, który wyświetla kolor żółty; Narysuj dwa trójkąty ponownie, gdzie drugi jest koloru żółtego: [rozwiązanie](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/2.5.hello_triangle_exercise3/hello_triangle_exercise3.cpp).