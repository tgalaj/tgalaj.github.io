---
layout: post
title: Programy cieniujące
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: pierwsze-kroki
---

{% include learnopengl.md link="Getting-started/Shaders" %}

Jak wspomniano w samouczku [Witaj Trójkącie]({% post_url learnopengl/1_getting_started/2017-08-28-witaj-trojkacie %}), shadery są to małe programy, które działają na GPU. Programy te są uruchamiane dla każdego konkretnego etapu potoku graficznego. W podstawowym znaczeniu, shadery to nic więcej jak programy przekształcające dane wejściowe w dane wyjściowe. Shadery są również bardzo odizolowanymi programami, ponieważ nie mogą komunikować się ze sobą; Jedyną ich komunikacją są ich wejścia i wyjścia.  

W poprzednim samouczku powierzchownie dotknęliśmy shaderów i tego jak ich właściwie używać. W tej części kursu, shadery zostaną wyjaśnione bardziej szczegółowo, a szczególnie skupimy się na języku w jakim są one pisane - OpenGL Shading Language (GLSL).

## GLSL

Shadery są pisane w języku GLSL, który jest podobny do C. GLSL jest dostosowany do współpracy z programowaniem grafiki i zawiera przydatne funkcje, które są ukierunkowane na manipulację wektorami i macierzami.

Shadery zawsze zaczynają się deklaracją wersji, a następnie listą zmiennych wejściowych i wyjściowych, uniform'ów i funkcji <span class="fun">main</span>. Każdy punkt wejścia (ang. _entry point_) programu cieniującego znajduje się w funkcji <span class="fun">main</span>, gdzie przetwarzamy dowolne dane wejściowe i zapisujemy wyniki w odpowiednich zmiennych wyjściowych. Nie przejmuj się, jeśli nie wiesz, co to są uniform'y, dojdziemy do nich wkrótce.

Shader ma zazwyczaj następującą strukturę:

```glsl
#version version_number

in type in_variable_name;  
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

void main()  
{  
    // Przetwórz dane wejściowe  
    ...  
    // Zapisz przetworzone dane do zmiennych wyjściowych  
    out_variable_name = weird_stuff_we_processed;  
}
```

Kiedy mówimy konkretnie o Vertex Shader każda zmienna wejściowa jest również znana jako <span class="def">atrybut wierzchołka</span> (ang. vertex attribute). Istnieje maksymalna liczba atrybutów wierzchołkowych, które możemy zadeklarować i ta liczba jest ograniczona przez sprzęt. OpenGL gwarantuje zawsze istnienie co najmniej szesnastu <span class="var">4</span>-komponentowych atrybutów wierzchołków, ale niektóre karty graficzne mogą zezwalać na więcej. Tą liczbę można pobrać, poprzez zapytanie OpenGL o wartość <span class="var">GL_MAX_VERTEX_ATTRIBS</span>:

```cpp
GLint nrAttributes;  
glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &nrAttributes);  
std::cout << "Maximum nr of vertex attributes supported: " << nrAttributes << std::endl;
```

Ten kod często zwraca minimum, które wynosi 16. Pomimo tego, ta wartość powinna być więcej niż wystarczająca dla większości celów.

## Typy

GLSL, podobnie jak inne języki programowania, posiada typy danych do określania na jakiego typu zmiennej chcemy pracować. GLSL zawiera większość podstawowych typów znanych z innych języków takich jak C: <span class="var">int, float, double, uint</span> i <span class="var">bool</span>. GLSL zawiera również dwa typy kontenerów, których będziemy dużo używali podczas ćwiczeń, mianowicie <span class="var">wektory</span> i <span class="var">macierze</span>. Macierze omówimy w późniejszym samouczku.

### Wektory

Wektor w GLSL to pojemnik zawierający 1, 2, 3 lub 4 komponenty dla dowolnego z wcześniej wspomnianych typów podstawowych. Mogą przyjmować następującą postać (n oznacza liczbę elementów):

*   <span class="var">vecn</span>: podstawowy wektor zawierający <span class="var">n</span> komponentów typu <span class="var">float</span>.
*   <span class="var">bvecn</span>: wektor zawierający <span class="var">n</span> komponentów typu <span class="var">bool</span>.
*   <span class="var">ivecn</span>: wektor zawierający <span class="var">n</span> komponentów typu <span class="var">int</span>.
*   <span class="var">uvecn</span>: wektor zawierający <span class="var">n</span> komponentów typu <span class="var">unsigned int</span>.
*   <span class="var">dvecn</span>: wektor zawierający <span class="var">n</span> komponentów typu <span class="var">double</span>.

Przez większość czasu będziemy używać podstawowej wersji <span class="var">vecn</span>, ponieważ zmienne float są wystarczające do większości naszych celów.

Komponenty wektora można uzyskać za pośrednictwem <span class="var">vec.x</span>, gdzie <span class="var">x</span> jest pierwszym składnikiem wektora. Możesz użyć <span class="var">.x, .y, .z</span> i <span class="var">.w</span>, aby odpowiednio uzyskać dostęp do jego pierwszego, drugiego, trzeciego i czwartego składnika. GLSL umożliwia również używanie <span class="var">rgba</span> w kontekście kolorów lub <span class="var">stpq</span> dla współrzędnych tekstur, które uzyskują dostęp do tych samych komponentów.

Typ danych wektorowych umożliwia pewien interesujący i elastyczny sposób wyboru komponentów o nazwie <span class="def">swizzling</span>. Swizzling pozwala na następującą składnię:

```cpp
vec2 someVec;  
vec4 differentVec = someVec.xyxx;  
vec3 anotherVec = differentVec.zyw;  
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
```

Można użyć dowolnej kombinacji o maksymalnie 4 literach, aby utworzyć nowy wektor (tego samego typu), o ile oryginalny wektor zawiera te składniki; Nie można uzyskać dostępu do składnika <span class="var">.z</span> dla na przykład <span class="var">vec2</span>. Możemy również przekazywać wektory jako argumenty dla różnych konstruktorów wektora, zmniejszając liczbę wymaganych argumentów:

```cpp
vec2 vect = vec2(0.5f, 0.7f);  
vec4 result = vec4(vect, 0.0f, 0.0f);  
vec4 otherResult = vec4(result.xyz, 1.0f);
```

Wektory są elastycznym typem danych, które można wykorzystać do wszystkich rodzajów wejść i wyjść. W tym kursie zobaczysz wiele przykładów, w jaki sposób możemy twórczo zarządzać wektorami.

## Wejścia i wyjścia

Shadery to same w sobie ładne, małe, samodzielne programy, ale stanowią część pewnej całości i dlatego chcemy mieć możliwość definiowania wejść i wyjść dla poszczególnych shader'ów, dzięki czemu byśmy mogli przenosić nasze dane. GLSL zdefiniował specjalnie do tego celu słowa kluczowe in oraz out. Każdy moduł cieniujący może określić wejścia i wyjścia przy użyciu tych słów kluczowych i gdziekolwiek zmienna wyjściowa pasuje do zmiennej wejściowej następnego etapu cieniowania, to są one dalej przekazywane. Jednak Vertex i Fragment Shader są pod tym względem nieco inne.

Vertex Shader powinien otrzymać jakąś inną formę otrzymywania danych wejściowych, w przeciwnym wypadku byłby on nieefektywny. Vertex Shader otrzymuje dane wejściowe bezpośrednio z danych wierzchołkowych (Vertex Data). Aby określić jak zorganizowane są dane wierzchołkowe, określamy zmienne wejściowe z metadanymi lokalizacji, dzięki czemu możemy skonfigurować atrybuty wierzchołków na CPU. Widzieliśmy to w poprzednim tutorialu jako kwalifikator <span class="var">layout (location = 0)</span>. Vertex Shader wymaga więc dodatkowej specyfikacji dla wejść, dzięki czemu możemy je powiązać z danymi wierzchołków.

{: .box-note }
Można również pominąć specyfikację <span class="var">layout (location = 0)</span> i zamiast tego wysłać zapytanie do OpenGL o lokalizacje atrybutów w kodzie za pośrednictwem <span class="fun">glGetAttribLocation</span>. Jednak wolałabym ustawiać je w Vertex Shader. Jest to łatwiejsze do zrozumienia i oszczędza Tobie (i OpenGL) pracę.

Innym wyjątkiem jest to, że Fragment Shader wymaga wyjściowej zmiennej finalnego koloru typu <span class="var">vec4</span>, ponieważ Fragmenty Shader generują końcowy kolor fragmentu. Jeśli nie określiłbyś koloru wyjściowego w twoim Fragment Shader, OpenGL wyrenderuje Twój obiekt na czarno (lub biało).

Zatem jeśli chcemy wysyłać dane z jednego programu cieniującego do drugiego, musimy zadeklarować zmienne wyjściowe w shaderze, który je wysyła dalej i podobne dane wejściowe w shaderze, który te dane ma przyjąć. Kiedy typy i nazwy są takie same po obu stronach, OpenGL połączy ze sobą te zmienne, a następnie możliwa jest komunikacja między shaderami (odbywa się to podczas linkowania Program Object). Aby pokazać, jak to działa w praktyce, będziemy zmieniać shadery z poprzedniego samouczka, aby Vertex Shader decydował o kolorze zamiast Fragment Shader'a.

**Vertex shader**  
```glsl
#version 330 core  
layout (location = 0) in vec3 position; // Zmienna position ma atrybut lokalizacji równy 0

out vec4 vertexColor; // Zadeklaruj zmienną wyjściową koloru, która zostanie wysłana do FS

void main()  
{  
    gl_Position = vec4(position, 1.0); // Zauważ jak od razu przekazujemy vec3 do konstruktora vec4  
    vertexColor = vec4(0.5f, 0.0f, 0.0f, 1.0f); // Ustaw kolor na ciemno-czerwony  
}
```

**Fragment shader**  
```glsl
#version 330 core  
in vec4 vertexColor; // Zmienna wejściowa odebrana od VS (ten sam typ i nazwa)

out vec4 color;

void main()  
{  
    color = vertexColor;  
}
```

Możesz zauważyć, że zadeklarowaliśmy zmienną <span class="var">vertexColor</span> jako wyjście typu <span class="var">vec4</span>, które ustawiliśmy w Vertex Shader i zadeklarowaliśmy podobną zmienną <span class="var">vertexColor</span> w Fragment Shader jako daną wejściową. Ponieważ obie mają ten sam typ i nazwę, to zmienna <span class="var">vertexColor</span> w FS jest połączona ze zmienną <span class="var">vertexColor</span> w VS. Ponieważ w VS ustawiamy kolor na ciemno-czerwony, końcowe fragmenty powinny również być ciemno-czerwone. Poniższy obraz przedstawia wyjściowy obrazek:

![Shaders]({{ site.baseurl }}/img/learnopengl/shaders.png)

Właśnie przesłaliśmy wartość z VS do FS! Dodajmy temu trochę smaczku i zobaczmy czy jesteśmy w stanie przesłać wartość koloru prosto z naszej aplikacji do FS!

## Uniformy

<span class="def">Uniformy</span> są kolejnym sposobem przekazywania danych z naszej aplikacji do shaderów znajdujących się na GPU, z tym, że uniformy są nieco inne w porównaniu do atrybutów wierzchołków. Przede wszystkim uniformy są <span class="def">globalne</span>. Globalne oznacza, że zmienna uniform jest unikatowa dla każdego Program Object i można uzyskać do niej dostęp z dowolnego programu cieniującego. Po drugie, niezależnie od tego na jaką wartość ustawisz uniform, to zachowa tę wartości, dopóki nie zostanie ona zresetowana lub zaktualizowana.

{: .box-note }
Od tłumacza  
Uniformy są to specjalne zmienne, które dostępne są w programach cieniujących. Ich wartości możemy ustawiać z poziomu aplikacji na CPU. Od atrybutów wierzchołków różnią się tym, że uniformy zachowują swoją wartość pomiędzy inwokacjami shader'ów - każda inwokacja np. Vertex Shader'a z reguły przeskakuje o jeden atrybut wierzchołka, co inwokację. Dzięki temu przetwarzany jest każdy wierzchołek z osobna; uniformy natomiast zachowują swoją wartość przez cały okres działania programu cieniującego w danej ramce. W kolejnej ramce może mieć już inną, nową wartość.

Aby zadeklarować uniform w GLSL, po prostu dodaj słowo kluczowe uniform przed typem zmiennej w programie cieniującym. Od tego momentu możemy użyć nowo utworzonego uniform'a w shaderze. Przyjrzyjmy się, czy tym razem możemy ustawić kolor trójkąta za pomocą tej specjalnej zmiennej:

```glsl
#version 330 core  
out vec4 color;

uniform vec4 ourColor; // ustawiamy tą wartość w kodzie aplikacji OpenGL

void main()  
{  
    color = ourColor;  
}
```

W FS zadeklarowaliśmy uniform o typie <span class="var">vec4</span> i nazwie <span class="var">ourColor</span> i ustawiliśmy finalny kolor fragmentu na wartość tego uniforma. Ponieważ uniformy są zmiennymi globalnymi, możemy je zdefiniować w dowolnym shaderze, więc nie wracać do VS, by dopisać tam jakiś kod, który przeniesie wartość uniforma do FS. Nie używamy uniforma w VS, więc nie ma potrzeby go tam definiować.

{: .box-error }
Jeśli zadeklarujesz uniform, który nie jest używany w żadnym miejscu kodu GLSL, kompilator automatycznie usunie zmienną z wersji skompilowanej, co jest często powodem kilku frustrujących błędów; pamiętaj o tym!

Uniform jest obecnie pusty; nie dodaliśmy żadnych danych do niego, więc spróbujmy to zrobić. Najpierw musimy znaleźć indeks/położenie uniforma w naszym programie cieniującym. Kiedy mamy indeks/lokalizację uniforma, możemy zaktualizować jego wartość. Zamiast przekazywać cały czas ten sam kolor dla FS, zaszalejemy i będziemy stopniowo zmieniać kolor w miarę upływu czasu:

```cpp
GLfloat timeValue = glfwGetTime();  
GLfloat greenValue = (sin(timeValue) / 2.0) + 0.5;  
GLint vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");  
glUseProgram(shaderProgram);  
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```

Najpierw pobieramy czas w sekundach za pomocą funkcji <span class="fun">glfwGetTime()</span>. Następnie zmieniamy kolor w zakresie <span class="var">0.0 - 1.0</span> przy użyciu funkcji <span class="fun">sin</span> i zapisujemy wynik w <span class="var">greenValue</span>.

Następnie pobieramy lokalizację uniforma <span class="var">ourColor</span> za pomocą <span class="fun">glGetUniformLocation</span>. Przekazujemy do tej funkcji Program Object i nazwę uniforma (którego lokalizację chcemy pobrać). Jeśli <span class="fun">glGetUniformLocation</span> zwróci wartość <span class="var">-1</span> to oznacza, że nie może znaleźć danej lokalizacji. Na koniec możemy ustawić wartość uniforma za pomocą funkcji <span class="fun">glUniform4f</span>. Zauważ, że znalezienie lokalizacji uniforma nie wymaga wcześniejszego aktywowania Program Object. Natomiast uaktualnienie uniforma **wymaga** wcześniejszego aktywowania programu (wywołując funkcję <span class="fun">glUseProgram</span>), ponieważ ustawia uniform na aktualnie aktywnym programie cieniującym.

<div class="box-note">Ponieważ OpenGL jest w swoim rdzeniu biblioteką C, to nie posiada on natywnej obsługi przeciążania typów, dlatego wszędzie tam, gdzie można wywołać funkcję z różnymi typami OpenGL definiuje nowe funkcje dla każdego typu; <span class="fun">glUniform</span> jest doskonałym tego przykładem. Funkcja wymaga określonego przyrostka dla typu uniformu, który ma zostać ustawiony. Kilka z możliwych przyrostków to:

*   <span class="var">f</span>: funkcja oczekuje zmiennej typu <span class="var">float</span> jako swoją wartość.
*   <span class="var">i</span>: funkcja oczekuje zmiennej typu <span class="var">int</span> jako swoją wartość.
*   <span class="var">ui</span>: funkcja oczekuje zmiennej typu <span class="var">unsigned int</span> jako swoją wartość.
*   <span class="var">3f</span>: funkcja oczekuje zmiennej <span class="var">3 float'ów</span> jako swoją wartość.
*   <span class="var">fv</span>: funkcja oczekuje zmiennej typu wektora/tablicy <span class="var">float</span> jako swoją wartość.

Zawsze, kiedy chcesz skonfigurować jakąś opcję OpenGL, wystarczy wybrać przeciążoną funkcję, która odpowiada Twojemu typowi. W naszym przypadku chcemy ustawiać <span class="var">4</span> wartości typu float z osobna dla uniforma, więc przekazujemy nasze dane za pośrednictwem <span class="fun">glUniform4f</span> (pamiętaj, że mogliśmy również użyć wersji z <span class="var">fv</span>).
</div>

Teraz, jak już wiemy, jak ustawić wartości uniformów, możemy użyć ich do renderowania. Jeśli chcemy, aby kolor stopniowo się zmieniał, to musimy zaktualizować tego uniforma dla każdej iteracji pętli gry (więc zmienia się raz na jedną klatkę), w przeciwnym razie trójkąt utrzymywałby jednolity kolor, jeśli tylko ustawimy tego uniforma. Więc obliczamy wartość <span class="var">greenValue</span> i aktualizujemy uniform co jedną iterację:

```cpp
while(!glfwWindowShouldClose(window))  
{  
    // input  
    processInput(window);

    // render  
    // wyczyść bufor koloru  
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);  
    glClear(GL_COLOR_BUFFER_BIT);

    // upewnij się, że aktywowałeś program object  
    glUseProgram(shaderProgram);

    // uaktualnij wartość uniforma  
    float timeValue = glfwGetTime();  
    float greenValue = sin(timeValue) / 2.0f + 0.5f;  
    int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");  
    glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);

    // narysuj trójkąt  
    glBindVertexArray(VAO);  
    glDrawArrays(GL_TRIANGLES, 0, 3);

    // zamień bufory i sprawdź zdarzenia I/O  
    glfwSwapBuffers(window);  
    glfwPollEvents();  
}
```

Kod jest stosunkowo prostym uaktualnieniem poprzedniego kodu. Tym razem uaktualnimy wartość uniforma dla każdej iteracji przed rysowaniem trójkąta. Jeśli uaktualniasz uniform poprawnie, powinieneś zobaczyć trójkąt, który stopniowo zmienia się z zielonego na czarny i z powrotem na zielony.

<div align="center"><video width="600" height="450" loop="" controls="">  
<source src="https://learnopengl.com/video/getting-started/shaders.mp4" type="video/mp4">  
![]({{ site.baseurl }}/img/learnopengl/shaders2.png){: .center-image}
</video></div>

Jeżeli utknąłeś, sprawdź kod źródłowy [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/3.1.shaders_uniform/shaders_uniform.cpp).

Jak widać, uniformy są użytecznym narzędziem do ustawiania zmiennych, które mogą się zmieniać pomiędzy iteracjami renderowania, lub do wymiany danych między aplikacją a shaderami, ale co zrobić, jeśli chcemy ustawić kolor dla każdego wierzchołka? W takim przypadku musimy zadeklarować tyle uniformów, ile mamy wierzchołków. Lepszym rozwiązaniem byłoby dodanie większej liczby danych do atrybutów wierzchołków, co właśnie zrobimy.

## Więcej atrybutów!

W poprzednim samouczku widzieliśmy, jak możemy wypełnić VBO, skonfigurować wskaźniki atrybutów wierzchołków i przechowywać je w VAO. Tym razem chcemy dodać dane o kolorach do danych wierzchołkowych. Zamierzamy dodawać dane o kolorach jako 3 wartości typu <span class="var">float</span> do tablicy <span class="var">vertices</span>. Do każdego z wierzchołków naszego trójkąta przypisujemy odpowiednio kolory czerwony, zielony i niebieski:

```cpp
float vertices[] = {
     // pozycje          // kolory
     0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // dolny prawy
    -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // dolny lewy
     0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // górny 
};  
```

Ponieważ teraz mamy więcej danych do wysyłania do VS, konieczne jest jego dostosowanie tak, aby również otrzymywał wartość koloru dla każego wierzchołka. Należy zauważyć, że lokalizacja atrybutu <span class="var">color</span> została ustawiona na <span class="var">1</span> przy użyciu kwalifikatora layout:

```glsl
#version 330 core  
layout (location = 0) in vec3 position; // zmienna position ma lokalizację 0  
layout (location = 1) in vec3 color; // zmienna color ma lokalizację 1

out vec3 ourColor; // przekaż kolor do FS

void main()  
{  
    gl_Position = vec4(position, 1.0);  
    ourColor = color; // ustaw ourColor na kolor wejściowy z atrybutu wierzchołka  
}
```

Ponieważ używamy zmiennej wyjściowej <span class="var">ourColor</span> zamiast uniforma, musimy również dostosować FS:

```glsl
#version 330 core  
in vec3 ourColor;  
out vec4 color;

void main()  
{  
    color = vec4(ourColor, 1.0f);  
}
```

Ponieważ dodaliśmy kolejny atrybut wierzchołka i zaktualizowaliśmy pamięć VBO musimy ponownie skonfigurować wskaźniki atrybutu wierzchołka. Zaktualizowane dane w pamięci VBO wyglądają teraz tak:

![Przeplot danych dotyczących położenia i koloru w VBO, które mają być skonfigurowane za pomocą glVertexAttribPointer]({{ site.baseurl }}/img/learnopengl/vertex_attribute_pointer_interleaved.png){: .center-image }

Wiedząc obecny układ możemy zaktualizować format wierzchołków za pomocą <span class="fun">glVertexAttribPointer</span>:

```cpp
// Atrybut pozycji  
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*)0);  
glEnableVertexAttribArray(0);  
// Atrybut koloru  
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*)(3* sizeof(GLfloat)));  
glEnableVertexAttribArray(1);
```

Pierwsze kilka argumentów funkcji <span class="fun">glVertexAttribPointer</span> powinny być zrozumiałe. Tym razem konfigurujemy atrybut wierzchołka w lokalizacji <span class="var">1</span>. Wartości kolorów mają rozmiar <span class="var">3 float'ów</span> i nie normalizujemy ich wartości.

Ponieważ mamy teraz dwa atrybuty wierzchołka, musimy ponownie obliczyć wartość _stride_. Aby uzyskać następną wartość atrybutu (np. następny składnik <span class="var">x</span> wektora pozycji) w tablicy danych, należy przesunąć o <span class="var">6 float'ów</span> w prawo - trzy dla wartości pozycji i trzy dla wartości kolorów. Daje nam to wartość stride 6 razy większa niż <span class="var">float</span> w bajtach (= <span class="var">24</span> bajtów).  
Również tym razem musimy określić offset. Dla każdego wierzchołka, atrybut pozycji jest pierwszy, dlatego deklarujemy przesunięcie <span class="var">0</span>. Atrybut koloru rozpoczyna zaraz za danymi położenia, dlatego wynosi <span class="var">3 * sizeof(GLfloat)</span> w bajtach (= <span class="var">12</span> bajtów).

Uruchomienie aplikacji powinno pokazać następujący obraz:

![]({{ site.baseurl }}/img/learnopengl/shaders3.png){: .center-image }

Jeżeli utknąłeś, sprawdź kod źródłowy [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/3.2.shaders_interpolation/shaders_interpolation.cpp).

Obraz może nie być dokładnie taki, jakiego byś oczekiwał, ponieważ dostarczyliśmy tylko 3 kolory, a nie całą paletę kolorów, którą widzimy teraz. Jest to wynik czegoś, co nazywa się <span class="def">interpolacją fragmentu</span> w Fragment Shader. Podczas renderowania trójkąta, etap rasteryzacji zazwyczaj generuje więcej fragmentów niż wierzchołków, które zostały wcześniej zdefiniowane. Następnie rasteryzer określa położenie każdego z tych fragmentów w oparciu o miejsce, w którym znajdują się na trójkącie.  
Opierając się na tych pozycjach, <span class="def">interpoluje</span> wszystkie zmienne wejściowe FS. Powiedzmy na przykład, że mamy linię, gdzie górny punkt ma zielony kolor, a dolny punkt niebieski kolor. Jeśli fragment shader jest uruchamiany na fragmencie, który znajduje się koło pozycji w <span class="var">70%</span> długości linii, to jego atrybutem wejściowym koloru, byłaby liniowa kombinacja koloru zielonego i niebieskiego; żeby być bardziej precyzyjnym: <span class="var">30%</span> niebieskiego i <span class="var">70%</span> zielonego.

To jest dokładnie to, co się stało w naszym trójkącie. Mamy 3 wierzchołki, a więc 3 kolory i prawdopodobnie trójkąt zawiera około 50000 fragmentów, gdzie fragment shader interpolował kolory wśród tych pikseli. Jeśli dobrze przyjrzymy się kolorom, które widzisz, wszystko ma sens: idąc od czerwonego do niebieskiego, kolor najpierw staje się fioletowy, a następnie na niebieski. Interpolacja fragmentów jest stosowana do wszystkich atrybutów wejściowych Fragment Shader'a.

## Nasza własna klasa Shader

Pisanie, kompilowanie i zarządzanie shaderami może być dość kłopotliwe. Ostatnią rzeczą jaką zrobimy w temacie shaderów to sprawienie, żeby nasze życie stało się łatwiejsze. Zbudujemy klasę Shader, która czyta programy cieniujące z dysku, kompiluje i łączy je, sprawdza błędy i jest łatwa w użyciu. To także daje Ci trochę pojęcia, jak możemy opakować część wiedzy, której nauczyliśmy się do tej pory, w użyteczne, abstrakcyjne obiekty.

Utworzymy klasę Shader całkowicie w pliku nagłówkowym, głównie w celach edukacyjnych i przenośności. Zacznijmy od dołączenia wymaganych plików nagłówkowych i zdefiniujmy strukturę klasy:

```cpp
#ifndef SHADER_H  
#define SHADER_H

#include <"glad/glad.h">// dołącz glad, by móc korzystać w wszystkich wymaganych przez OpenGL funkcji</glad>

#include <string>
#include <fstream>
#include <sstream>
#include <iostream>

class Shader  
{  
public:  
    // ID program object  
    unsigned int ID;

    // konstruktor czyta plik shadera z dysku i tworzy go  
    Shader(const GLchar* vertexPath, const GLchar* fragmentPath);  
    // aktywuj shader  
    void use();  
    // funkcje operujące na uniformach  
    void setBool(const std::string &name, bool value) const;  
    void setInt(const std::string &name, int value) const;  
    void setFloat(const std::string &name, float value) const;  
};

#endif
```

{: .box-note }
W górnej części pliku nagłówkowego użyliśmy kilku <span class="def">dyrektyw preprocesora</span>. Użycie tych kilku linijek kodu (ang. _include guard_) informuje kompilator, aby dołączył i skompilował ten plik nagłówkowy, o ile jeszcze to się nie stało. Zapobiega to konfliktom linkera.

Klasa Shader zawiera identyfikator Program Object. Jego konstruktor wymaga ścieżek do plików kodu źródłowego Vertex i Fragment Shader'a, które możemy przechowywać na dysku jako proste pliki tekstowe. Dodatkowo, dodajemy kilka funkcji użytkowych, które ułatwią nam życie: <span class="fun">use</span> aktywuje program cieniujący, a wszystkie funkcje <span class="fun">set...</span> pobierają lokalizację uniforma i ustawiają jego wartość.

### Czytanie z pliku

Używamy strumienia plików C ++ do odczytywania zawartości pliku i zapisania jej do kilku obiektów typu <span class="var">string</span>:

```cpp
Shader(const char* vertexPath, const char* fragmentPath)  
{  
    // 1. pobierz kod źródłowy Vertex/Fragment Shadera z filePath  
    std::string vertexCode;  
    std::string fragmentCode;  
    std::ifstream vShaderFile;  
    std::ifstream fShaderFile;  
    // zapewnij by obiekt ifstream mógł rzucać wyjątkami  
    vShaderFile.exceptions (std::ifstream::failbit | std::ifstream::badbit);  
    fShaderFile.exceptions (std::ifstream::failbit | std::ifstream::badbit);  
    try  
    {  
        // otwórz pliki  
        vShaderFile.open(vertexPath);  
        fShaderFile.open(fragmentPath);  
        std::stringstream vShaderStream, fShaderStream;  
        // zapisz zawartość bufora pliku do strumieni  
        vShaderStream << vShaderFile.rdbuf();  
        fShaderStream << fShaderFile.rdbuf();  
        // zamknij uchtywy do plików  
        vShaderFile.close();  
        fShaderFile.close();  
        // zamień strumień w łańcuch znaków  
        vertexCode = vShaderStream.str();  
        fragmentCode = fShaderStream.str();  
    }  
    catch(std::ifstream::failure e)  
    {  
        std::cout << "ERROR::SHADER::FILE_NOT_SUCCESFULLY_READ" << std::endl;  
    }  
    const char* vShaderCode = vertexCode.c_str();  
    const char* fShaderCode = fragmentCode.c_str();  
    [...]
```

Następnie musimy skompilować i zlinkować shadery. Warto zauważyć, że sprawdzamy także, czy kompilacja/linkowanie powiodła się. Jeśli nie, to wydrukujemy błędy podczas kompilacji, które są niezwykle użyteczne podczas debugowania (w końcu i tak będziesz potrzebował tych logów):

```cpp
// 2. skompiluj shadery  
unsigned int vertex, fragment;  
int success;  
char infoLog[512];

// Vertex Shader  
vertex = glCreateShader(GL_VERTEX_SHADER);  
glShaderSource(vertex, 1, &vShaderCode, NULL);  
glCompileShader(vertex);  
// wypisz błędy kompilacji, jeśli są jakieś  
glGetShaderiv(vertex, GL_COMPILE_STATUS, &success);  
if(!success)  
{  
    glGetShaderInfoLog(vertex, 512, NULL, infoLog);  
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;  
};

// podobnie dla Fragment Shader'a  
[...]

// Program Object  
ID = glCreateProgram();  
glAttachShader(ID, vertex);  
glAttachShader(ID, fragment);  
glLinkProgram(ID);  
// wypisz błędy linkowania, jeśli są jakieś  
glGetProgramiv(ID, GL_LINK_STATUS, &success);  
if(!success)  
{  
    glGetProgramInfoLog(ID, 512, NULL, infoLog);  
    std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;  
}

// usuń obiekty shader'ów, które są już powiązane  
// z Program Object - nie będą nam już potrzebne  
glDeleteShader(vertex);  
glDeleteShader(fragment);
```

Funkcja <span class="fun">use</span> jest bardzo prosta:

```cpp
void use()  
{  
    glUseProgram(ID);  
}
```

Podobnie jak funkcje <span class="fun">set...</span> dla uniformów:

```cpp
void setBool(const std::string &name, bool value) const  
{  
glUniform1i(glGetUniformLocation(ID, name.c_str()), (int)value);  
}  
void setInt(const std::string &name, int value) const  
{  
glUniform1i(glGetUniformLocation(ID, name.c_str()), value);  
}  
void setFloat(const std::string &name, float value) const  
{  
glUniform1f(glGetUniformLocation(ID, name.c_str()), value);  
} 
```

I tak mamy ukończoną [klasę Shader](https://learnopengl.com/code_viewer_gh.php?code=includes/learnopengl/shader_s.h). Użycie tej klasy jest dość łatwe; tworzymy obiekt typu Shader raz i od tego momentu po prostu możemy go używać:

```cpp
Shader ourShader("path/to/shaders/shader.vs", "path/to/shaders/shader.fs");  
...  
while(...)  
{  
    ourShader.use();  
    ourShader.setFloat("someUniform", 1.0f);  
    DrawStuff();  
}
```

Tutaj kod źródłowy VS i FS został zapisany w dwóch plikach o nazwach <span class="var">shader.vs</span> i <span class="var">shader.frag</span>. Możesz dowolnie nazwać pliki shaderów; dla mnie osobiście, rozszerzenia <span class="var">.vs</span> i <span class="var">.frag</span> są dość intuicyjne.

Kod źródłowy możesz znaleźć [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/3.3.shaders_class/shaders_class.cpp). Używa on, naszej nowo utworzonej [klasy Shader](https://learnopengl.com/code_viewer_gh.php?code=includes/learnopengl/shader_s.h). Zauważ, że możesz kliknąć ścieżki plików cieniujących, aby otworzyć ich kod źródłowy.

## Ćwiczenia

*   Zmień VS tak, by trójkąt wyświetlał się do góry nogami: [rozwiązanie](https://learnopengl.com/code_viewer.php?code=getting-started/shaders-exercise1 "undefined").
*   Określ przesunięcie poziome za pomocą uniforma i przesuń trójkąt w prawą stronę ekranu. W VS skorzystaj z wartości przesunięcia podanej w uniformie: [rozwiązanie](https://learnopengl.com/code_viewer.php?code=getting-started/shaders-exercise2 "undefined").
*   Prześlij pozycje wierzchołków z VS do FS za pomocą słowa kluczowego out i ustaw koloru fragmentu, tak by był on równy przekazanej pozycji wierzchołka (zobacz jak pozycje wierzchołka są interpolowane na trójkącie). Gdy udało się to zrobić, spróbuj odpowiedzieć na następujące pytanie: dlaczego lewy dolny róg naszego trójkąta jest czarny? [rozwiązanie](https://learnopengl.com/code_viewer.php?code=getting-started/shaders-exercise3 "undefined").