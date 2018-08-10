---
layout: post
title: Test głębokości
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: zaawansowany-opengl
mathjax: true
---

{% include learnopengl.md link="Advanced-OpenGL/Depth-testing" %}

W samouczku [Układy współrzędnych]({% post_url /learnopengl/1_getting_started/2017-09-25-uklady-wspolrzednych %}) wyrenderowaliśmy kontener 3D i skorzystaliśmy z <span class = "def">bufora głębi</span> (ang. *depth buffer*), aby zapobiec renderowaniu ścianek z przodu, gdy w rzeczywistości znajdują się one za innymi ściankami. W tym samouczku dowiemy się trochę więcej na temat <span class = "def">wartości głębi</span>, które są zapisywane do bufora głębi (lub bufora-z/z-buffer) i jak one faktycznie określają, czy dany fragment jest przesłonięty przez inne fragmenty.

Bufor głębi jest buforem, który podobnie jak <span class = "def">bufor koloru</span> (który przechowuje wszystkie kolory fragmentów: wizualne wyjście), przechowuje informacje dla każdego fragmentu i (zazwyczaj) ma tą samą szerokość i wysokość jako bufor koloru. Bufor głębi jest automatycznie tworzony przez system okienkowy i przechowuje jego wartości głębokości jako wartości `16`, `24` lub `32` bitowe. W większości systemów zobaczysz bufor głębi z precyzją `24` bitów.

Po włączeniu testu głębokości (ang. *depth test*), OpenGL testuje wartość głębokości fragmentu względem zawartości bufora głębi. OpenGL przeprowadza test głębokości i jeśli ten test się powiedzie, bufor głębi zostanie zaktualizowany o nową wartość głębokości. Jeśli test głębi się nie powiedzie, fragment zostanie odrzucony.

Test głębi odbywa się w przestrzeni ekranu po uruchomieniu Fragment Shader'a (i po teście szablonu (ang. *stencil test*), który omówimy w następnym samouczku). Współrzędne ekranowe odnoszą się bezpośrednio do widoku zdefiniowanego przez funkcję <span class = "fun">glViewport</span> i mogą być pobrane za pośrednictwem wbudowanej funkcji GLSL <span class="var">gl_FragCoord</span> w Fragment Shaderze. Składniki `x` i `y` <span class = "var">gl_FragCoord</span> reprezentują współrzędne fragmentu w przestrzeni ekranu (gdzie (0,0) jest lewym dolnym rogiem). Funkcja <span class = "var">gl_FragCoord</span> zawiera również składnik `z`, który zawiera faktyczną wartość głębokości fragmentu. Ta wartość `z` jest wartością porównywalną z zawartością bufora głębi.

<div class="box-note">Obecnie większość procesorów graficznych obsługuje funkcję sprzętową o nazwie <span class = "def">wczesny test głębokości</span> (ang. *early depth testing*). Wczesny test głębokości umożliwiaja uruchomienie testu głębokości przed uruchomieniem Fragment Shadera. Jeżeli jesteśmy pewni, że fragment nigdy nie będzie widoczny (znajduje się za innymi obiektami) możemy przedwcześnie odrzucić ten fragment.

Fragment Shadery są zwykle dość drogie, więc powinniśmy szukać obszarów, gdzie możemy zminimalizować ich pracę. Ograniczeniem dla Fragment Shaderów dla wczesnego testu głębokości jest to, że nie możesz zapisywać danych do głębi fragmentu z poziomu shadera. Jeśli Fragment Shader zapisuje cokolwiek do wartości głębokości, wczesne testy głębokości są niemożliwe do wykonania; OpenGL nie będzie wcześniej w stanie ustalić wartości głębokości fragmentu.
</div>

Test głębokości jest domyślnie wyłączony, więc aby włączyć test głębokości, korzystamy z opcji <span class = "var">GL_DEPTH_TEST</span>:

```cpp
    glEnable(GL_DEPTH_TEST);  
```

Po włączeniu, OpenGL automatycznie zapisuje wartości głębokości fragmentów w buforze głębi, jeśli przeszły test głębokości i odrzuca fragmenty, jeśli nie przeszły pomyślnie testu głębokości. Jeśli masz włączony test głębokości, powinieneś wyczyścić bufor głębi przed każdą iteracją renderowania za pomocą flagi <span class = "var">GL_DEPTH_BUFFER_BIT</span>, w przeciwnym razie będziesz korzystał z wartościami głębokości z ostatniej iteracji renderowania:

```cpp
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);  
```

Istnieją pewne przypadki użycia, gdzie chcesz przeprowadzić test głębokości na wszystkich fragmentach i odpowiednio je odrzucić, ale **nie** aktualizujesz bufora głębi. Zasadniczo używasz bufora głębi w trybie <span class = "def">tylko do odczytu</span>. OpenGL pozwala nam wyłączyć zapisywanie do bufora głębi, ustawiając jego maskę głębi na `GL_FALSE`:

```cpp
    glDepthMask(GL_FALSE);  
```

Pamiętaj, że działa to tylko wtedy, gdy włączony jest test głębokości.

## Funkcje testu głębokości

OpenGL pozwala nam modyfikować operatory porównania, który jest używany do testu głębokości. To pozwala nam kontrolować, kiedy OpenGL powinien akceptować lub odrzucać fragmenty oraz kiedy aktualizować bufor głębi. Możemy ustawić operator porównania (lub funkcję głębi), wywołując <span class = "fun">glDepthFunc</span>:

```cpp
    glDepthFunc(GL_LESS);  
```

Funkcja akceptuje kilka operatorów porównania wymienionych w poniższej tabeli:

<table align="center">
  <tbody><tr>
  	<th style="text-align:center;">Funkcja</th>
  	<th style="text-align:center;">Opis</th>
  </tr>  
  <tr>
    <td style="text-align:center;">GL_ALWAYS</td>
 	<td style="text-align:center;">Test głębokości zawsze przechodzi.</td>
  </tr>
  <tr>
    <td style="text-align:center;">GL_NEVER</td>
 	<td style="text-align:center;">Test głębokości nigdy nie przechodzi.</td>
  </tr>
  <tr>
    <td style="text-align:center;">GL_LESS</td>
 	<td style="text-align:center;">Przechodzi, jeśli wartość głębi fragmentu jest mniejsza od zapisanej wartości głębokości.</td>
  </tr>
  <tr>
    <td style="text-align:center;">GL_EQUAL</td>
 	<td style="text-align:center;">Przechodzi, jeśli wartość głębi fragmentu jest równa zapisanej wartości głębokości.</td>
  </tr><tr>
    <td style="text-align:center;">GL_LEQUAL</td>
 	<td style="text-align:center;">Przechodzi, jeśli wartość głębi fragmentu jest mniejsza lub równa zapisanej wartości głębokości</td>
  </tr> 
  <tr>
    <td style="text-align:center;">GL_GREATER</td>
 	<td style="text-align:center;">Przechodzi, jeśli wartość głębi fragmentu jest większa niż zapisana wartość głębi.</td>
  </tr>
  <tr>
    <td style="text-align:center;">GL_NOTEQUAL</td>
 	<td style="text-align:center;">Przechodzi, jeśli wartość głębi fragmentu nie jest równa zapisanej wartości głębokości.</td>
  </tr>
  <tr>
    <td style="text-align:center;">GL_GEQUAL</td>
 	<td style="text-align:center;">Przechodzi, jeśli wartość głębi fragmentu jest większa lub równa zapisanej wartości głębokości</td>
  </tr>
</tbody></table>

Domyślnie używana jest funkcja głębokości <span class = "var">GL_LESS</span>, która odrzuca wszystkie fragmenty, których wartość głębi jest większa lub równa aktualnej wartości bufora głębi.

Pokażmy, jaki wpływ ma zmiana funkcji głębi na efekt wizualny. Użyjemy nowego projektu, który wyświetli podstawową scenę z dwoma oteksturowanymi sześcianami na oteksturowanej podłodze bez oświetlenia. Możesz znaleźć kod źródłowy [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/4.advanced_opengl/1.1.depth_testing/depth_testing.cpp).

W kodzie źródłowym zmieniliśmy funkcję głębokości na <span class = "var">GL_ALWAYS</span>:

```cpp
    glEnable(GL_DEPTH_TEST);
    glDepthFunc(GL_ALWAYS); 
```

Symuluje to samo zachowanie, kiedy nie włączyliśmy testu głębokości. Test głębokości po prostu zawsze przechodzi, więc fragmenty, które są rysowane jako ostatnie, są renderowane przed fragmentami, które zostały narysowane wcześniej, mimo że powinny znajdować się z przodu. Ponieważ narysowaliśmy płaszczyznę podłogi jako ostatnią, to fragmenty podłogi nadpisują każdy z fragmentów kontenera:

![Test głębokości w OpenGL z GL_ALWAYS jako funkcją głębi](/img/learnopengl/depth_testing_func_always.png){: .center-image }

Ustawienie funkcji głębokości z powrotem na <span class = "var">GL_LESS</span> daje nam scenę, do której jesteśmy przyzwyczajeni:

![Test głębokości w OpenGL z GL_LESS jako funkcją głębi](/img/learnopengl/depth_testing_func_less.png){: .center-image }

## Precyzja wartości głębi

Bufor głębokości zawiera wartości głębokości z zakresu od `0.0` do `1.0` i porównuje jego zawartość z wartościami wszystkich obiektów na scenie, widzianych z perspektywy kamery. Te wartości `z` w przestrzeni widoku mogą być dowolną wartością pomiędzy wartością `near` i `far`. Potrzebujemy więc jakiegoś sposobu na przekształcenie tych wartości `z` przestrzeni widoku do zakresu `[0,1]`, a jednym ze sposobów jest liniowe przekształcenie ich do zakresu `[0,1]`. Następujące (liniowe) równanie przekształca wartość `z` na wartość głębokości pomiędzy `0.0` a `1.0`:

\begin{equation} F_{depth} = \frac{z - near}{far - near} \end{equation}

Tutaj płaszczyzny `near` i `far` są to wartości, które przekazaliśmy do utworzenia macierzy projekcji w celu ustawienia frustum (patrz Układy Współrzędnych). Równanie przyjmuje wartość głębokości `z` w frustum i przekształca go do zakresu `[0,1]`. Relację między wartością `z` a odpowiadającą jej wartością głębokości przedstawiono na poniższym wykresie:

![Wykres wartości głębokości w OpenGL jako funkcja liniowa](/img/learnopengl/depth_linear_graph.png){: .center-image }

{: .box-note }
Zauważ, że wszystkie równania podają wartość głębokości bliską `0.0`, gdy obiekt znajduje się w pobliżu, i wartość głębi bliską `1.0`, gdy obiekt znajduje się blisko dalszej (far) płaszczyzny.

W praktyce <span class = "def">liniowy bufor głębokości</span> (ang. *linear depth buffer*) jest prawie nigdy nieużywany. Dla poprawnych właściwości projekcji stosuje się nieliniowe równanie głębokości, które jest proporcjonalne do $\frac{1}{z}$. W zasadzie daje nam to ogromną precyzję, gdy `z` jest małe i znacznie mniej precyzyjne, gdy `z` jest daleko. Pomyślcie o tym przez chwilę: czy naprawdę chcemy, aby wartości głębokości odległe o `1000` miały taką samą precyzję, jak obiekty o wysokiej szczegółowości w odległości `1`? Równanie liniowe nie bierze tego pod uwagę.

Ponieważ funkcja nieliniowa jest proporcjonalna do $\frac{1}{z}$, wartości `z` pomiędzy `1.0` a `2.0` skutkowałyby, na przykład, wartościami głębokości pomiędzy `1.0` a `0.5`, co stanowi połowę precyzji jaką zapewnia nam typ float, dając nam ogromną precyzję przy małych wartościach `z`. Wartości `z` między `50.0` a `100.0` będą stanowić tylko 2% precyzji typu float, to jest dokładnie to czego chcemy. Takie równanie, które również bierze pod uwagę płaszczyzny bliską i daleką, jest podane poniżej:

\begin{equation} F_{depth} = \frac{1/z - 1/near}{1/far - 1/near} \end{equation}

Nie martw się, jeśli nie wiesz dokładnie, co się dzieje w tym równaniu. Należy pamiętać, że wartości w buforze głębokości nie są liniowe w przestrzeni ekranu (są one liniowe w przestrzeni widoku przed zastosowaniem macierzy projekcji). Wartość `0.5` w buforze głębi nie oznacza, że ​​wartości `z` obiektu są w połowie odległości w frustum; wartość `z` wierzchołka jest w rzeczywistości dość blisko bliższej płaszczyzny! Możesz zobaczyć nieliniową relację między wartością `z` a wartością w buforze głębokości na poniższym wykresie:

![Wykres wartości głębokości w OpenGL jako funkcja nieliniowa](/img/learnopengl/depth_non_linear_graph.png){: .center-image }

Jak widać, wartości głębokości są w dużej mierze zdeterminowane przez małe wartości `z`, dając nam ogromną dokładność głębi dla obiektów znajdujących się w pobliżu. Równanie do transformacji wartości `z` (z perspektywy widza) jest zawarte w macierzy projekcji, więc kiedy przekształcamy współrzędne wierzchołków z przestrzeni widoku do przestrzeni obcinania, a następnie na przestrzeń ekranu, stosowane jest równanie nieliniowe. Jeśli jesteś ciekawy, jak tak naprawdę wygląda macierz projekcji, proponuję następujący [świetny artykuł](http://www.songho.ca/opengl/gl_projectionmatrix.html).

Efekt tego równania nieliniowego szybko staje się widoczny, gdy próbujemy zwizualizować bufor głębi.

## Wizualizacja bufora głębi

Wiemy, że wartość `z` wbudowanej zmiennej GLSL <span class = "var">gl_FragCoord</span> w Fragment Shader zawiera wartość głębokości danego fragmentu. Gdybyśmy wyprowadzili tę wartość głębi fragmentu jako kolor, moglibyśmy wyświetlić wartości głębokości wszystkich fragmentów w scenie. Możemy to zrobić, zwracając wektor koloru na podstawie wartości głębi fragmentu:

```glsl
    void main()
    {             
        FragColor = vec4(vec3(gl_FragCoord.z), 1.0);
    }  
```

Jeśli ponownie uruchomisz program, prawdopodobnie zauważysz, że wszystko jest białe, jakby wyglądało na to, że wszystkie nasze wartości głębokości są równe `1.0`, które są maksymalną wartością głębi. Dlaczego więc żadna z wartości głębokości nie jest bliższa `0.0`, a przez to ciemniejsza?

Możesz pamiętać z poprzedniej sekcji, że wartości głębokości w przestrzeni ekranu są nieliniowe, np. mają bardzo wysoką precyzję dla małych wartości `z` i małą precyzję dla dużych wartości `z`. Wartość głębi fragmentu rośnie gwałtownie wraz z odległością, więc prawie wszystkie wierzchołki mają wartości zbliżone do `1.0`. Gdybyśmy powoli poruszali się bardzo blisko obiektów, moglibyśmy ostatecznie dostrzec ciemniejsze kolory, pokazując, że ich wartości `z` stają się mniejsze:

![Bufor głębokości zwizualizowany w OpenGL i GLSL](/img/learnopengl/depth_testing_visible_depth.png){: .center-image }

To wyraźnie pokazuje nieliniowość wartości głębokości. Obiekty znajdujące się w pobliżu mają znacznie większy wpływ na wartość głębi niż obiekty odległe. Tylko przesunięcie o kilka cali powoduje, że kolory przechodzą od ciemnego do całkowicie białego.

Możemy jednak przekształcić nieliniowe wartości głębokości fragmentu z powrotem na wartości liniowe. Aby to osiągnąć, musimy odwrócić proces projekcji tylko dla wartości głębokości. Oznacza to, że najpierw musimy ponownie przekształcić wartości głębokości z zakresu `[0,1]` na znormalizowane współrzędne urządzenia w zakresie `[-1,1]` (przestrzeń obcinania NDC). Następnie chcemy odwrócić nieliniowe równanie (równanie 2), jak to zrobiono w macierzy projekcji i zastosować to odwrócone równanie do uzyskanej wartości głębokości. Rezultatem jest liniowa wartość głębi. Brzmi nieźle, prawda?

Najpierw chcemy przekształcić wartość głębokości do NDC, co nie jest zbyt trudne:

```glsl
    float z = depth * 2.0 - 1.0; 
```

Następnie przyjmujemy wynikową wartość `z` i stosujemy odwrotną transformację, aby uzyskać wartość głębokości liniowej:

```glsl
    float linearDepth = (2.0 * near * far) / (far + near - z * (far - near));	
```

To równanie pochodzi z macierzy projekcji, która ponownie używa równania 2 do "odliniowania" wartości głębokości zwracanych z zakresu <span class = "var">near</span> i <span class = "var">far</span>. Ten [matematyczny artykuł](http://www.songho.ca/opengl/gl_projectionmatrix.html) wyjaśnia macierz projekcji w ogromnych szczegółach; pokazuje również, skąd pochodzą równania.

Pełny Fragment Shader przekształcający nieliniową głębokość w przestrzeni ekranu na liniową wartość głębokości jest następujący:

```glsl
    #version 330 core
    out vec4 FragColor;

    float near = 0.1; 
    float far  = 100.0; 

    float LinearizeDepth(float depth) 
    {
        float z = depth * 2.0 - 1.0; // z powrotem do NDC
        return (2.0 * near * far) / (far + near - z * (far - near));	
    }

    void main()
    {             
        float depth = LinearizeDepth(gl_FragCoord.z) / far; // podziel przez far dla demonstracji
        FragColor = vec4(vec3(depth), 1.0);
    }
```

Ponieważ zlinearyzowane wartości głębokości wahają się od <span class = "var">near</span> do <span class = "var">far</span> większość wartości będzie powyżej `1.0` i wyświetlana będzie całkowicie biała. Dzieląc liniową wartość głębokości przez <span class = "var">far</span> w funkcji <span class = "fun">main</span> przeliczamy liniową wartość głębokości na z grubsza zakres `[0, 1]`. W ten sposób możemy stopniowo dostrzec, że scena staje się jaśniejsza, im bliżej fragmenty znajdują się płaszczyzn frustum, co lepiej nadaje się do celów demonstracyjnych.

Gdybyśmy teraz uruchomili aplikację, otrzymamy wartości głębokości, które w rzeczywistości są liniowe wraz z odległością. Spróbuj poruszać się po scenie, aby zobaczyć zmiany wartości głębokości w sposób liniowy.

![Bufor głębokości zwizualizowany w OpenGL i GLSL jako wartości liniowe](/img/learnopengl/depth_testing_visible_linear.png){: .center-image }

Kolory są przeważnie czarne, ponieważ wartości głębokości rozciągają się liniowo od płaszczyzny `near`, która wynosi `0.1` do płaszczyzny `far`, która jest ustawiona na wartość `100`, która jest dość daleko od nas. Rezultat jest taki, że jesteśmy stosunkowo blisko bliższej płaszczyźnie i tym samym uzyskujemy niższe (ciemniejsze) wartości głębokości.

## Z-fighting

Powszechny artefakt wizualny może wystąpić, gdy dwie płaszczyzny lub trójkąty są tak blisko siebie, że bufor głębi nie ma wystarczającej dokładności, aby ustalić, który z dwóch kształtów znajduje się przed drugim. Powoduje to, że dwa kształty nieustannie zmieniają kolejność wyświetlania, co powoduje dziwne wzory. Nazywa się to <span class = "def">z-fighting</span>, ponieważ wygląda na to, że prymitywy walczą o to, który ma być z przodu.

W scenie, z której korzystaliśmy do tej pory, jest kilka miejsc, w których z-fighting jest dość zauważalny. Pojemniki umieszczono na tej samej wysokości, na której umieszczono podłogę, co oznacza, że ​​dolna płaszczyzna pojemnika jest współpłaszczyznowa z podłogą. Wartości głębokości obu płaszczyzn są wówczas takie same, dlatego wynikowy test głębi nie pozwala na ustalenie, która z nich jest mniejsza.

Jeśli umieścisz kamerę w jednym z pojemników, efekty są wyraźnie widoczne, dolna część pojemnika ciągle zamienia się miejscami z płaszczyzną podłogi, tworząc zygzakowaty wzór:

![Demonstration of Z-fighting in OpenGL](/img/learnopengl/depth_testing_z_fighting.png){: .center-image }

Z-fighting jest częstym problemem powiązanym z buforami głębi i generalnie jest tym silniejsze, im obiekty znajdują się w dalszej odległości (ponieważ bufor głębi ma mniejszą dokładność przy większych wartościach `z`). Z-fightingowi nie można całkowicie zapobiec, ale zazwyczaj istnieje kilka sztuczek, które pomogą złagodzić lub całkowicie zapobiec temu efektowi w twojej scenie.

### Zapobieganie z-fighting

Pierwszym i najważniejszym trikiem jest _nie umieszczanie obiektów zbyt blisko siebie, tak, że niektóre trójąty się pokrywają_. Tworząc małe przesunięcie między dwoma obiektami, które jest mało zauważalne przez użytkownika, całkowicie zapobiegnie z-fightingowi pomiędzy dwoma obiektami. W przypadku kontenerów i podłogi mogliśmy lekko przesunąć pojemniki w dodatnim kierunku `y`. Niewielka zmiana pozycji kontenera prawdopodobnie nie byłaby zauważalna i całkowicie ograniczyłaby z-fighting. Wymaga to jednak ręcznej interwencji dla każdego z obiektów i dokładnych testów, aby upewnić się, że żadne obiekty w scenie nie powodują z-fightingu.

Druga sztuczka polega na tym, aby _ustawić bliższą płaszczyznę tak daleko, jak to możliwe_. W jednym z poprzednich rozdziałów omawialiśmy, że precyzja jest bardzo duża, gdy jesteśmy blisko płaszczyzny `near`, więc jeśli przesuniemy płaszczyznę `near` dalej od widza, będziemy mieli znacznie większą precyzję w całym zakresie frustum. Jednak ustawienie płaszczyzny `near` zbyt daleko może spowodować obcinanie najbliższych obiektów, więc zwykle jest to kwestia eksperymentowania, aby znaleźć najlepszą wartość dla zmiennej `near`.

Kolejną świetną sztuczką kosztem wydajności jest _użycie wyższej dokładności bufora głębi_. Większość buforów głębi ma precyzję `24` bitów, ale większość kart obsługuje obecnie bufory głębi `32` bitowe, które znacznie zwiększają precyzję. Kosztem wydajności osiągniesz znacznie większą precyzję redukując z-fighting.

Trzy omówione przez nas techniki są najczęściej stosowanymi i łatwymi do wdrożenia technikami walki z-fightingiem. Istnieje kilka innych technik, które wymagają znacznie więcej pracy i nadal nie będą całkowicie wyłączały z-fightingu. Z-fighting jest częstym problemem, ale jeśli użyjesz właściwej kombinacji wymienionych technik, prawdopodobnie nie będziesz musiał zajmować problemem z-fightingu.