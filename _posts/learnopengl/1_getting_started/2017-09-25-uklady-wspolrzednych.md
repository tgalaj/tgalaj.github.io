---
layout: post
title: Układy współrzędnych
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: pierwsze-kroki
mathjax: true
---

{% include learnopengl.md link="Getting-started/Coordinate-Systems" %}

W ostatnim tutorialu dowiedzieliśmy się, jak wykorzystać macierze transformacji do przekształcania wszystkich wierzchołków. OpenGL oczekuje, że wszystkie wierzchołki, które chcemy, aby stały się widoczne, znajdują się w znormalizowanym układzie współrzędnych (ang. _Normalized Device Coordinates (NDC)_) po każdym wywołaniu Vertex Shader'a. Oznacza to, że współrzędne <span class="var">x, y</span> i <span class="var">z</span> każdego wierzchołka powinny mieścić się pomiędzy <span class="var">-1.0</span> i <span class="var">1.0</span>; współrzędne poza tym zakresem nie będą widoczne. To, co zazwyczaj robimy, to określenie współrzędnych, a w Vertex Shader przekształcamy te współrzędne do NDC. Współrzędne NDC są następnie podawane do rasteryzera, aby przekształcić je na współrzędne 2D/piksele na ekranie.  

Przekształcanie współrzędnych do NDC, a następnie do współrzędnych ekranu, zazwyczaj następuje krok po kroku, gdzie przekształcamy wierzchołki obiektu do kilku układów współrzędnych, zanim ostatecznie przekształcimy je w współrzędne ekranu. Zaletą przekształcenia ich do kilku _pośrednich_ układów współrzędnych jest to, że niektóre operacje/obliczenia są łatwiejsze w innych układach współrzędnych, co wkrótce stanie się bardziej zrozumiałe. Istnieje pięć różnych układów współrzędnych, które nas najbardziej interesują:

*   Przestrzeń Lokalna / Przestrzeń Obiektu (ang. _Local/Object Space_)
*   Przestrzeń Świata (ang. _World Space_)
*   Przestrzeń Widoku / Przestrzeń Kamery/Oka (ang. _View/Eye Space_)
*   Przestrzeń Obcinania (ang. _Clip Space_)
*   Przestrzeń Ekranu (ang. _Screen Space_)

Są to różne stany, w których nasze wierzchołki się znajdą, zanim ostatecznie skończą jako fragmenty.

Prawdopodobnie jesteś teraz dość zdezorientowany tym, czym jest przestrzeń świata czy układ współrzędnych, więc teraz wyjaśnimy je w bardziej zrozumiały sposób, pokazując całkowity obraz tego, co konkretna przestrzeń rzeczywiście robi.

## Ogólny obraz

Aby przekształcić współrzędne z jednej przestrzeni do następnej, użyjemy kilku macierzy transformacji, z których najważniejsze są macierze <span class="def">model/world</span> (modelu/świata), <span class="def">view</span> (widoku) i <span class="def">projection</span> (projekcji). Nasze współrzędne wierzchołków znajdują się najpierw w <span class="def">przestrzeni lokalnej</span> jako <span class="def">współrzędne lokalne</span> i następnie są przetwarzane do <span class="def">współrzędnych globalnych</span>, <span class="def">współrzędnych widoku</span>, <span class="def">współrzędnych obcinania</span> i ostatecznie kończą jako <span class="def">współrzędne ekranu</span>. Poniższy obraz przedstawia cały proces i pokazuje, co każda transformacja robi:

![]({{ site.baseurl }}/img/learnopengl/coordinate_systems.png){: .center-image }

1.  Lokalne współrzędne są współrzędnymi Twojego obiektu względem jego lokalnego punktu początkowego; są to współrzędne, w których zaczyna Twój obiekt.
2.  Następnym krokiem jest przekształcenie lokalnych współrzędnych do współrzędnych przestrzeni świata, które są współrzędnymi względem większego świata. Te współrzędne odnoszą się do globalnego punktu początkowego świata, łącznie z wieloma innymi obiektami, które są również umieszczone w stosunku do tego punktu.
3.  Następnie przekształcamy współrzędne świata do przestrzeni widoku, w taki sposób, aby każda współrzędna była widoczna z punktu widzenia kamery/obserwatora.
4.  Po tym jak współrzędne znajdą się w przestrzeni widoku, chcemy rzutować je do przestrzeni obcinania. Współrzędne obcinania są przetwarzane do zakresu <span class="var">-1.0</span> i <span class="var">1.0</span> i określają, które wierzchołki będą pojawią się na ekranie.
5.  Na koniec przekształcamy współrzędne obcinania na współrzędne ekranu w procesie, który nazywamy <span class="def">transformacją obszaru renderowania</span> (ang. _viewport transform_), która zmienia współrzędne z zakresu <span class="var">-1.0</span> i <span class="var">1.0</span> do współrzędnych z zakresu zdefiniowanego przez <span class="fun">glViewport</span>. Otrzymane współrzędne są następnie wysyłane do rasteryzera, aby przekształcić je w fragmenty.li>

Prawdopodobnie masz niewielkie wyobrażenie, tego, co czego każda przestrzeń jest używana. Powodem, dla którego przekształcamy wierzchołki we wszystkie te różne przestrzenie, jest to, że niektóre operacje mają sens lub są łatwiejsze przy użyciu niektórych układów współrzędnych. Na przykład modyfikowanie obiektu najlepiej jest zrobić w przestrzeni lokalnej, podczas gdy obliczanie pewnych operacji na obiekcie w odniesieniu do pozycji innych obiektów ma największy sens we współrzędnych świata itd. Jeśli chcemy, możemy zdefiniować jedną macierz transformacji, która przekształca współrzędne z przestrzeni lokalnej do przestrzeni obcinania w jednej operacji, ale daje nam to mniej elastyczności.

Poniżej szczegółowo omówimy każdy układ współrzędnych.

## Przestrzeń Lokalna

Przestrzeń lokalna to układ współrzędnych, który jest lokalny dla Twojego obiektu, tzn. układ, w którym zaczyna się obiekt. Wyobraź sobie, że utworzono kostkę w programie do modelowania (np. Blender). Środek kostki jest prawdopodobnie ustawiony na punkt <span class="var">(0,0,0)</span>, mimo że kostka może trafić do innej aplikacji końcowej. Prawdopodobnie wszystkie modele, które stworzyłeś, mają punkt <span class="var">(0,0,0)</span> jako ich środek. Wszystkie wierzchołki Twojego modelu znajdują się w przestrzeni _lokalnej_: wszystkie są lokalne dla Twojego obiektu.

Wierzchołki kontenera, których używaliśmy, zostały określone jako współrzędne pomiędzy <span class="var">-0.5</span> i <span class="var">0.5</span> z punktem <span class="var">0.0</span> jako jego środkiem. Są to współrzędne lokalne.

## Przestrzeń Świata

Gdybyśmy chcieli zaimportować wszystkie nasze obiekty bezpośrednio do aplikacji, prawdopodobnie wszystkie znalazłyby się gdzieś ułożone jeden na drugim wokół środka całego wirtualnego świata <span class="var">(0,0,0)</span>, czego nie chcemy. Chcemy zdefiniować pozycję dla każdego obiektu z osobna, aby umieścić je w większym świecie. Współrzędne przestrzeni świata są dokładnie tym: współrzędne wszystkich wierzchołków są umieszczane względem świata (gry). Jest to przestrzeń współrzędnych, w której obiekty przekształcane są w taki sposób, że wszystkie są w jakiś sposób rozmieszczone (najlepiej w sposób realistyczny). Współrzędne obiektu są przekształcane z przestrzeni lokalnej do przestrzeni świata; odbywa się to za pomocą macierzy <span class="def">model/world</span>.

Macierz modelu/świata jest macierzą transformacji, która przesuwa, skaluje i/lub obraca obiekt, aby umieścić go w świecie w określonej lokalizacji/orientacji. Zastanów się nad tym, jak przekształcić dom, skalując go w dół (był nieco zbyt duży w przestrzeni lokalnej), ustawiając go na przedmieściach i obracając go nieco w lewo na osi y, tak aby pasował idealnie do sąsiednich domów. Możesz pomyśleć o macierzy, z poprzedniego samouczka, która umieszczała kontener w konkretnej lokalizacji na scenie jako o rodzaju macierzy modelu; przekształciliśmy lokalne współrzędne kontenera w inne miejsce na scenie/świecie.

## Przestrzeń Widoku

Przestrzeń widoku jest tym, co ludzie zwykle określają jako <span class="def">kamera</span> OpenGL (czasami też znana jest jako <span class="def">przestrzeń kamery</span> lub <span class="def">przestrzeń oka</span>). Przestrzeń widoku jest wynikiem przekształcania współrzędnych przestrzeni świata na współrzędne, które znajdują się przed widokiem użytkownikiem. Przestrzeń widoku jest zatem przestrzenią, dzięki której obserwujemy scenę z punktu widzenia kamery. Zwykle odbywa się to za pomocą kombinacji translacji i rotacji, aby przesunąć/obrócić scenę tak, aby niektóre elementy zostały przekształcone tak, aby były na przeciw kamery. Te połączone transformacje są zazwyczaj przechowywane wewnątrz <span class="def">macierzy widoku</span>, która przekształca współrzędne świata do przestrzeni widoku. W następnym ćwiczeniu będziemy szerzej mówić o tym, jak utworzyć taką macierz widoku w celu symulacji wirtualnej kamery.

## Przestrzeń Obcinania

Na końcu każdego wywołania Vertex Shader'a, OpenGL oczekuje, że współrzędne znajdą się w określonym przedziale i każda współrzędna poza tym zakresem zostanie <span class="def">obcięta</span>. Współrzędne, które są obcięte, są odrzucane, a pozostałe współrzędne kończą jako fragmenty widoczne na ekranie. Od tego zabiegu pochodzi nazwa <span class="def">przestrzeń obcinania</span>.

Ponieważ określenie wszystkich widocznych współrzędnych znajdujących się w zakresie <span class="var">-1.0</span> i <span class="var">1.0</span> nie jest zbyt intuicyjne, określmy własny zestaw współrzędnych do pracy i przekonwertujmy je do NDC, tak jak OpenGL tego oczekuje.

W celu przekształcenia współrzędnych wierzchołków z przestrzeni widoku do przestrzeni obcinania, definiujemy tzw. <span class="def">macierz projekcji</span>, która określa zakres współrzędnych np. <span class="var">-1000</span> i <span class="var">1000</span> w każdym wymiarze. Macierz projekcji przekształca współrzędne w tym określonym zakresie do znormalizowanych współrzędnych urządzenia (NDC) <span class="var">(- 1.0, 1.0)</span>. Wszystkie współrzędne poza tym zakresem nie będą mapowane do przedziału <span class="var">-1.0</span> i <span class="var">1.0</span>, a zatem powinny zostać obcięte. W tym określonym przedziale macierzy projekcji, współrzędna <span class="var">(1250, 500, 750)</span> nie byłaby widoczna, ponieważ współrzędna x jest poza zakresem, a zatem przekształca się w współrzędną większą niż <span class="var">1.0</span> w NDC, a zatem zostaje ona obcięta.

{: .box-note }
Zauważ, że jeśli tylko część prymitywu, np. trójkąta, znajduje się poza <span class="def">obszarem obcinania</span>, to OpenGL zrekonstruuje trójkąt jako jeden lub więcej trójkątów, aby dopasować go do obszaru obcinania.

To _pudełko widoku_ macierzy projekcji nazywa się <span class="def">frustum</span> i każda współrzędna, która znajduje się wewnątrz frustum, znajdzie się na ekranie użytkownika. Całkowity proces przekształcania współrzędnych z określonego zakresu do NDC, który może być łatwo odwzorowywany na współrzędne 2D przestrzeni widoku, nazywa się <span class="def">projekcją</span>, ponieważ macierze projekcji <span class="def">mapują/rzutują</span> współrzędne 3D na współrzędne 2D NDC.

Kiedy wszystkie wierzchołki zostaną przekształcone do przestrzeni obcinania, wykonana zostaje ostatnia operacja zwana <span class="def">dzieleniem perspektywicznym</span>, gdzie dzielimy komponenty <span class="var">x, y</span> i <span class="var">z</span> wektorów pozycji przez składnik <span class="var">w</span> tego wektora; dzielenie perspektywiczne jest tym, co przekształca współrzędne w przestrzeni obcinania 4D w współrzędnych 3D w NDC. Ten krok jest wykonywany automatycznie po zakończeniu każdego wywołania shadera.

Po tym etapie uzyskane współrzędne są odwzorowywane na współrzędne ekranu (używając ustawień funkcji <span class="fun">glViewport</span>) i zostają zamienione na fragmenty.

Macierz projekcji przekształcająca współrzędne widoku do współrzędnych obcinania może przybrać dwie różne formy, przy czym każda forma określa własną niepowtarzalną frustę. Możemy utworzyć macierz projekcji <span class="def">prostokątnej</span> lub macierz projekcji <span class="def">perspektywicznej</span>.

### Projekcja prostokątna

Macierz projekcji prostokątnej definiuje frustum podobne do sześcianu, który definiuje przestrzeń obcinania, w której każdy wierzchołek poza tym pudełkiem zostanie obcięty. Podczas tworzenia macierzy projekcji prostokątnej określamy szerokość, wysokość i długość widocznej frusty. Wszystkie współrzędne, które znajdą się wewnątrz tej frusty, po przekształceniu ich w przestrzeń obcinania, za pomocą macierzy projekcji prostokątnej, nie zostaną obcięte. Frusta wygląda trochę jak pojemnik:

![]({{ site.baseurl }}/img/learnopengl/orthographic_frustum.png){: .center-image }

Frustum określa widoczne współrzędne i jest określona za pomocą szerokości, wysokości i <span class="def">bliską</span> i <span class="def">daleką</span> płaszczyznę. Każda współrzędna znajdująca się przed bliską płaszczyzną zostaje obcięta i to samo dotyczy współrzędnych znajdujących się za daleką płaszczyzną. Frusta prostokątna bezpośrednio mapuje wszystkie współrzędne wewnątrz frustum do znormalizowanych współrzędnych urządzenia (NDC), ponieważ składnik w każdego z wektorów zostanie nienaruszony; jeśli składnik w jest równy <span class="var">1.0</span> dzielenie perspektywiczne nie zmienia jego współrzędnych.

Aby utworzyć macierz projekcji prostokątnej używamy wbudowanej funkcji GLM <span class="fun">glm::ortho</span>:

```cpp
glm::ortho(0.0f, 800.0f, 0.0f, 600.0f, 0.1f, 100.0f);
```

Pierwsze dwa parametry określają lewą i prawą współrzędną frustum, a trzeci i czwarty parametr określają dolną i górną część frustum. Za pomocą tych 4 punktów określiliśmy rozmiary bliskiej i dalekiej płaszczyzny, a parametr 5. i 6. następnie określają odległości pomiędzy bliską i daleką płaszczyzną. Ta specyficzna macierz projekcji przekształca wszystkie współrzędne z tego zakresu <span class="var">x, y</span> i <span class="var">z</span> na znormalizowane współrzędne urządzenia (NDC).

Macierz projekcji prostokątnej bezpośrednio mapuje współrzędne na płaszczyznę 2D, która znajduje się na Twoim ekranie, ale w rzeczywistości bezpośrednia projekcja daje nierealistyczne wyniki, ponieważ nie uwzględnia <span class="def">perspektywy</span>. To jest coś co <span class="def">projekcja perspektywiczna</span> naprawia za nas.

### Projekcja perspektywiczna

Jeśli kiedykolwiek cieszyłeś się grafiką jaką _prawdziwe życie_ ma do zaoferowania, to pewnie zauważyłeś, że obiekty, które są dalej wydają się znacznie mniejsze. Ten dziwny efekt to coś, co nazywamy <span class="def">perspektywą</span>. Perspektywa jest szczególnie zauważalna, gdy patrzy się w dół, na koniec nieskończonej autostrady lub torów kolejowy, tak jak to widać na poniższym obrazku:

![]({{ site.baseurl }}/img/learnopengl/perspective.png){: .center-image }

Jak widać, z uwagi na perspektywę, linie im są dalej to wydają się jakby miały się za chwilę połączyć. To jest właśnie efekt jaki projekcja perspektywiczna stara się naśladować i wykorzystuje do tego macierz <span class="def">projekcji perspektywicznej</span>. Macierz projekcji mapuje dany zakres frustum do przestrzeni obcinania, ale również manipuluje wartością <span class="var">w</span> każdej współrzędnej wierzchołka w taki sposób, że im dalej znajduje się wierzchołek, tym większy staje się ten składnik. Kiedy współrzędne zostaną przekształcone do przestrzeni obcinania to znajdują się one w zakresie <span class="var">-w</span> i <span class="var">w</span> (każdy wierzchołek poza tym zakresem zostanie obcięty). OpenGL wymaga, aby widoczne współrzędne znajdowały się w zakresie <span class="var">-1.0</span> i <span class="var">1.0</span> jako wyjściowy wierzchołek Vertex Shader'a. Gdy współrzędne znajdują się w przestrzeni obcinania, zostaje zastosowane dzielenie perspektywiczne do współrzędnych w przestrzeni obcinania:

$$ out = \begin{pmatrix} x /w \\ y / w \\ z / w \end{pmatrix} $$

Każdy składnik wierzchołka podzielony jest przez jego składnik <span class="var">w</span>, dający mniejsze współrzędne wierzchołka, im dalej znajduje się wierzchołek od obserwatora. Jest to kolejny powód, dla którego tak ważny jest składnik <span class="var">w</span>, ponieważ pomaga nam w projekcji perspektywicznej. Otrzymane współrzędne znajdują się następnie w znormalizowanej przestrzeni urządzenia (NDC). Jeśli jesteś zainteresowany, aby dowiedzieć się, jak w rzeczywistości obliczane są macierze projekcji prostokątnej i perspektywicznej (i nie boisz się matematyki) mogę polecić [ten świetny artykuł](http://www.songho.ca/opengl/gl_projectionmatrix.html "undefined") autorstwa Songho.

W GLM można utworzyć macierz projekcji perspektywicznej w następujący sposób:

```cpp
glm::mat4 proj = glm::perspective(glm::radians(45.0f), (float)width/(float)height, 0.1f, 100.0f);
```

Funkcja <span class="fun">glm::perspective</span> tworzy duże _frustum_, które definiuje widoczną przestrzeń, a cokolwiek znajdzie się poza tym frustum zostanie obcięte (nie będzie tego finalnie widać na ekranie użytkownika). Frustum perspektywiczne można zwizualizować jako nierówne pudełko, gdzie każda współrzędna wewnątrz tego pola zostanie później dopasowana do przestrzeni obcinania. Wygląd frustum perspektywicznego przedstawiono poniżej:

![]({{ site.baseurl }}/img/learnopengl/perspective_frustum.png){: .center-image }

Pierwszy parametr definiuje wartość <span class="def">fov</span>, która oznacza <span class="def">pole widzenia</span> (ang. _field of view_) i określa jak duży jest obszar widoku. Dla realistycznego widoku, ta wartość, jest zazwyczaj ustawiana na <span class="var">45.0f</span> stopni, ale w przypadku efektów rodem z Doom'a możesz ustawić ją na większą wartość. Drugi parametr określa współczynnik _aspect ratio_, który jest obliczany przez podzielenie szerokości przez wysokość ekranu. Trzeci i czwarty parametr ustawia bliską i daleką płaszczyznę frustum. Zwykle ustawiamy odległość do bliskiej płaszczyzny w pobliżu wartości <span class="var">0.1f</span> i odległość od dalszej płaszczyzny w okolicy <span class="var">100.0f</span>. Wszystkie wierzchołki między płaszczyzną _bliską_ a _daleką_ są wewnątrz frustum i zostaną wyrenderowane.

{: .box-note }
Kiedy wartość _bliskiej płaszczyzny_ Twojej macierzy perspektywicznej jest zbyt wysoka (na przykład <span class="var">10.0f</span>), OpenGL będzie blokował wszystkie wierzchołki znajdujące się w pobliżu wirtualnej kamery (między <span class="var">0.0f</span> i <span class="var">10.0f</span>), co daje w efekcie znajomy efekt wizualny z gier wideo, dzięki któremu można patrzeć przez obiekty jeśli jest się zbyt blisko tych obiektów.

Kiedy używamy projekcji prostokątnej, każda z współrzędnych wierzchołka jest bezpośrednio odwzorowywana w przestrzeni obcinania bez jakiegokolwiek, fantazyjnego dzielenia perspektywicznego (następuje dzielenie perspektywiczne, ale wartość <span class="var">w</span> nie jest zmieniana (pozostaje równe <span class="var">1</span>), a zatem nie ma ta operacja żadnego skutku). Ponieważ projekcja prostokątna nie wykorzystuje rzutów perspektywicznych, obiekty znajdujące się dalej nie wydają się mniejsze, co powoduje dziwny odbiór wizualny. Z tego powodu projekcja prostokątna jest wykorzystywana głównie do renderowania 2D oraz dla niektórych zastosowań architektonicznych lub inżynierskich, w których nie chcemy zniekształcać wierzchołków. Aplikacje, takie jak _Blender_, które są używane do modelowania 3D, czasem używa się w nich projekcji prostokątnej do modelowania, ponieważ dokładniej przedstawia ona wymiary każdego obiektu. Poniżej przedstawiono porównanie obu metod projekcji w programie Blender:

![]({{ site.baseurl }}/img/learnopengl/perspective_orthographic.png){: .center-image }

Widać że przy projekcji perspektywicznej, wierzchołki znajdujące się dalej oddalone są wydają się być znacznie mniejsze, podczas gdy przy projekcji prostokątnej każdy wierzchołek ma tę samą odległość od użytkownika.

## Łączymy wszystko w całość

Tworzymy macierz transformacji dla każdego z wyżej wymienionych etapów: macierz modelu, widoku i projekcji. Następnie współrzędna wierzchołka zostaje przekształcona do współrzędnych obcinania w następujący sposób:

$$ V_{clip} = M_{projection} \cdot M_{view} \cdot M_{model} \cdot V_{local} $$

Zauważ, że kolejność mnożenia macierzy jest odwrotna (pamiętaj, że musimy odczytywać mnożenie macierzy od prawej do lewej). Otrzymany wierzchołek powinien być następnie przypisany do zmiennej wbudowanej <span class="var">gl_Position</span> w Vertex Shader, a następnie OpenGL automatycznie wykona dzielenie perspektywiczne i obcinanie.

{: .box-note }
**A potem?**  
Wyjście Vertex Shader'a wymaga, aby współrzędne znajdowały się w przestrzeni obcinania, co właśnie zrobiliśmy za pomocą macierzy transformacji. Następnie OpenGL wykonuje _dzielenie perspektywiczne_ na _współrzędnych w przestrzeni obcinania_, aby przekształcić je w _znormalizowane współrzędne urządzenia (NDC)_. OpenGL używa parametrów z funkcji <span class="fun">glViewport</span>, aby odwzorować znormalizowane współrzędne urządzenia do _współrzędnych w przestrzeni ekranu_, gdzie każda współrzędna odpowiada punktowi na ekranie (w naszym przypadku ekran ma rozmiar <span class="var">800x600</span>). Proces ten nazywa się _transformacją obszaru renderowania_.

Jest to trudny temat do zrozumienia, więc jeśli nadal nie jesteś pewien do czego używana jest każda z przestrzeni to nie martw się. Poniżej zobaczysz, w jaki sposób możemy użyć tych przestrzeni współrzędnych.

## Uruchamianie 3D

Teraz, gdy wiemy, jak przekształcać współrzędne 3D na współrzędne 2D, możemy pokazać obiekty jako rzeczywiste obiekty 3D, zamiast pokazywać, jak do tej pory, zwykłą płaszczyznę 2D.

Aby rozpocząć rysowanie w 3D najpierw utworzymy macierz modelu. Macierz modelu składa się z translacji, skalowania i/lub rotacji, które chcielibyśmy zastosować do _przekształcenia_ wszystkich wierzchołków obiektu do przestrzeni świata. Zmieńmy nieco naszą płaszczyznę, obracając ją wokół osi x, tak, aby wygląda jakby, kładła się na podłogę. Macierz modelu wygląda zatem tak:

```cpp
glm::mat4 model;  
model = glm::rotate(model, glm::radians(-55.0f), glm::vec3(1.0f, 0.0f, 0.0f));
```

Mnożąc współrzędne wierzchołków z tą macierzą modelu, przekształcamy współrzędne wierzchołków na do przestrzeni świata. Nasza płaszczyzna, która jest lekko na podłodze, reprezentuje płaszczyznę w globalnym świecie.

Następnie musimy utworzyć macierz widoku. Chcemy poruszyć się lekko w tył sceny, aby obiekt stał się bardziej widoczny (gdy jesteśmy w przestrzeni świata to obserwator znajduje się w punkcie początkowym <span class="var">(0,0,0)</span>). Aby poruszać się po scenie, zastanów się nad następującymi kwestiami:

*   Przesuwanie kamery w tył, jest tym samym co przesuwanie całej sceny do przodu.

To jest dokładnie to, co robi macierz widoku, przesuwamy całą scenę odwrotnie do punktu, w którym chcemy, aby znalazła się kamera. Ponieważ chcemy przesuwać się do tyłu, a ponieważ OpenGL operuje prawoskrętnym układzie współrzędnych, musimy poruszać się po dodatniej osi z. Wykonujemy to, przesuwając scenę na negatywną oś z. To daje wrażenie, że idziemy do tyłu.

<div class="box-note">
**Prawoskrętny układ współrzędnych**  
OpenGL używa prawoskrętnego układu współrzędnych. To oznacza, że dodatnia oś x jest po prawej stronie, dodatnia oś y jest skierowana w górę, a dodatnia oś z wychodzi z ekranu w Twoją stronę. Wyobraź sobie, że Twój ekran jest centrum 3 osi, a dodatnia oś z jest skierowana w Twoim kierunku. Osie są rysowane w następujący sposób:

![]({{ site.baseurl }}/img/learnopengl/coordinate_systems_right_handed.png){: .center-image }

Aby zrozumieć, dlaczego nazywamy to prawoskrętnym układem współrzędnych, wykonaj następujące czynności:

*   Wyciągnij prawą rękę w górę, wzdłuż dodatniej osi y.
*   Niech Twój kciuk wskazuje prawą stronę.
*   Palec wskazujący niech będzie skierowany w górę.
*   Teraz zegnij palec środkowy w dół o 90 stopni.

Jeśli zrobiłeś to dobrze, Twój kciuk powinien wskazywać dodatnią oś x, palec wskazujący powinien być skierowany w kierunku dodatniej osi y i środkowy palec w kierunku dodatniej osi z. Gdybyś to zrobił lewą ręką, to zobaczysz, że oś z będzie odwrócona. Jest to znane jako lewoskrętny układ współrzędnych i jest powszechnie używany przez DirectX. Zauważ, że w znormalizowanych współrzędnych urządzenia (NDC) OpenGL wykorzystuje właściwie lewoskrętny układ współrzędnych (macierz projekcji przełącza "skrętność").

</div>

W następnym tutorialu omówimy bardziej szczegółowo, jak poruszać się po scenie. Teraz macierz widoku wygląda tak:

```cpp
glm::mat4 view;  
// Zauważ, że przesuwamy scenę w odwrotnym kierunku do miejsca, w którym chcemy się znaleźć  
view = glm::translate(view, glm::vec3(0.0f, 0.0f, -3.0f)); 
```

Ostatnią rzeczą jaką musimy zdefiniować jest macierz projekcji. Chcemy użyć projekcji perspektywicznej dla naszej sceny, więc zadeklarujmy macierz projekcji w następujący sposób:

```cpp
glm::mat4 projection;  
projection = glm::perspective(glm::radians(45.0f), screenWidth / screenHeight, 0.1f, 100.0f);
```

Teraz, gdy stworzyliśmy macierze transformacyjne, powinniśmy przekazać je do naszych shader'ów. Najpierw zadeklaruj macierze transformacji jako zmienne uniform w Vertex Shader i pomnóż je przez współrzędne wierzchołków:

```cpp
#version 330 core  
layout (location = 0) in vec3 position;  
...  
uniform mat4 model;  
uniform mat4 view;  
uniform mat4 projection;

void main()  
{  
    // Zauważ, że czytamy mnożenie macierzy od prawej do lewej  
    gl_Position = projection * view * model * vec4(position, 1.0f);  
    ...  
}
```

Powinniśmy też wysyłać macierze do programu cieniującego (zazwyczaj jest to wykonywane w każdej iteracji renderowania, ponieważ macierze transformacji mają tendencję do ulegania ciągłym zmianom):

```cpp
GLint modelLoc = glGetUniformLocation(ourShader.Program, "model"));  
glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));  
... // To samo dla View Matrix i Projection Matrix
```

Teraz, gdy nasze wierzchołki zostały przekształcane za pomocą macierzy modelu, widoku i projekcji, ostateczny obiekt powinien być:

*   Przechylony do tyłu w kierunku podłogi.
*   Nieco dalej od nas.
*   Wyświetlany z perspektywą (powinien być mniejszy im jego wierzchołki są dalej od nas).

Sprawdźmy, czy wynik rzeczywiście spełnia te wymagania:

![]({{ site.baseurl }}/img/learnopengl/coordinate_systems_result.png){: .center-image }

Rzeczywiście wygląda na to, że nasz płaszczyzna jest jest płaszczyzną 3D, która spoczywa na jakiejś wyimaginowanej podłodze. Jeśli nie otrzymasz tego samego wyniku, sprawdź kompletny [kod źródłowy](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/6.1.coordinate_systems/coordinate_systems.cpp).

## Więcej 3D

Do tej pory pracowaliśmy na płaszczyznach 2D, nawet w przestrzeni 3D. Wejdźmy więc na drogę poszukiwania przygód i rozszerzmy naszą płaszczyznę 2D do kostki 3D. Aby utworzyć sześcian potrzebujemy w sumie 36 wierzchołków (6 ścianki * 2 trójkąty * 3 wierzchołki każdy). 36 wierzchołków jest dużą ilością do utworzenia ale można je pobrać [stąd](https://learnopengl.com/code_viewer.php?code=getting-started/cube_vertices).

Dla zabawy, będziemy kostkę obracać się w czasie:

```cpp
model = glm::rotate(model, (float)glfwGetTime() * glm::radians(50.0f), glm::vec3(0.5f, 1.0f, 0.0f));
```

Później narysujmy sześcian przy użyciu <span class="fun">glDrawArrays</span>, ale tym razem z liczbą <span class="var">36</span>var> wierzchołków.

```cpp
glDrawArrays(GL_TRIANGLES, 0, 36);
```

Powinieneś otrzymać coś podobnego do obrazka poniżej:

<div align="center"><video width="600" height="450" loop="" controls="">  
<source src="https://learnopengl.com/video/getting-started/coordinate_system_no_depth.mp4" type="video/mp4">  
</video></div>

Wygląda raczej jak kostka, ale coś jest nie tak. Niektóre ściany kostki są rysowane na innych ściankach kostki. Dzieje się tak dlatego, że OpenGL rysując sześcian trójkąt po trójkącie, nadpisuje jego piksele, mimo tego że coś zostało już wcześniej narysowane. Z tego powodu niektóre trójkąty są rysowane jeden na drugim, podczas gdy nie powinny się one pokrywać.

Na szczęście, OpenGL przechowuje informacje o głębokości w buforze o nazwie <span class="def">z-bufor</span>, który pozwala OpenGL zdecydować, kiedy narysować piksel, a kiedy nie. Korzystając z z-buforu możemy powiedzieć OpenGL, aby przeprowadzał testy głębokości.

### Z-bufor

OpenGL przechowuje wszystkie informacje o głębokości w z-buforze, znanym również jako <span class="def">bufor głębokości</span>. GLFW automatycznie tworzy taki bufor za Ciebie (podobnie jak bufor kolorów, który przechowuje kolory obrazu wyjściowego). Głębokość jest zapisywana w każdym fragmencie (jako wartość <span class="var">z</span> fragmentu) i kiedy fragment chce wypisać swój kolor, OpenGL porównuje wartość głębokości tego fragmentu z wartością z z-bufora. Jeśli obecny fragment znajduje się za drugim fragmentem, jest on odrzucany, a w przeciwnym razie jest zastępowany. Ten proces nazywa się <span class="def">testem głębokości</span> (ang. _depth test_) i jest wykonywany automatycznie przez OpenGL.

Jeśli chcemy mieć pewność, że OpenGL faktycznie przeprowadza test głębokości, najpierw musimy poinformować OpenGL, że chcemy włączyć testy głębokości; ta opcja jest domyślnie wyłączona. Możemy włączyć testy głębokości za pomocą <span class="fun">glEnable</span>. Funkcje <span class="fun">glEnable</span> i <span class="fun">glDisable</span> umożliwiają włączenie/wyłączenie niektórych funkcji w OpenGL. Ta funkcjonalność jest wtedy włączana/wyłączana, dopóki nie zostanie wykonane inne wywołanie, aby włączyć/wyłączyć tą funkcjonalność. Teraz chcemy włączyć testy głębokości, włączając <span class="var">GL_DEPTH_TEST</span>:

```cpp
glEnable(GL_DEPTH_TEST);
```

Ponieważ używamy buforu głębokości, chcemy wyczyścić bufor głębokości przed każdą iteracją renderowania (w przeciwnym razie informacje o głębokości poprzedniej ramki pozostają w buforze). Podobnie jak wyczyszczenie bufora kolorów, możemy wyczyścić bufor głębokości, określając bit <span class="var">DEPTH_BUFFER_BIT</span> w funkcji <span class="fun">glClear</span>:

```cpp
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

Ponownie uruchommy nasz program i zobaczmy, czy teraz OpenGL przeprowadza testy głębokości:

<div align="center"><video width="600" height="450" loop="" controls="">  
<source src="https://learnopengl.com/video/getting-started/coordinate_system_depth.mp4" type="video/mp4">  
</video></div>

Mamy to! W pełni oteksturowana kostka z odpowiednim testem głębokości, która obraca się w czasie. Kod źródłowy możesz sprawdzić [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/6.2.coordinate_systems_depth/coordinate_systems_depth.cpp "undefined").

### Więcej kostek!

Powiedzmy, że chcielibyśmy wyświetlić 10 naszych kostek na ekranie. Każda kostka będzie wyglądać tak samo, ale będzie się różnić tylko pozycją świecie i rotacją. Układ graficzny sześcianu jest już zdefiniowany, więc nie musimy zmieniać buforów ani tablic atrybutów podczas renderowania większej liczby obiektów. Jedyną rzeczą jaką musimy zmienić dla każdego obiektu jest jego macierz modelu, za pomocą której przekształcamy kostki do przestrzeni świata.

Najpierw określmy wektor translacji dla każdej kostki, która określa jego pozycję w przestrzeni świata. Zdefiniujemy 10 pozycji kostki w tablicy <span class="var">glm::vec3</span>:

```cpp
glm::vec3 cubePositions[] = {  
    glm::vec3( 0.0f,  0.0f,  0.0f),  
    glm::vec3( 2.0f,  5.0f, -15.0f),  
    glm::vec3(-1.5f, -2.2f, -2.5f),  
    glm::vec3(-3.8f, -2.0f, -12.3f),  
    glm::vec3( 2.4f, -0.4f, -3.5f),  
    glm::vec3(-1.7f,  3.0f, -7.5f),  
    glm::vec3( 1.3f, -2.0f, -2.5f),  
    glm::vec3( 1.5f,  2.0f, -2.5f),  
    glm::vec3( 1.5f,  0.2f, -1.5f),  
    glm::vec3(-1.3f,  1.0f, -1.5f)  
};
```

Teraz, w obrębie pętli gry chcemy wywołać funkcję <span class="fun">glDrawArrays</span> 10 razy, ale za każdym razem przesyłając inną macierz modelu do Vertex Shader'a zanim dana kostka zostanie wyrenderowana. Tworzymy małą pętlę w obrębie pętli gry, która renderuje nasz obiekt 10 razy przy użyciu różnych macierzy modelu. Należy pamiętać, że dodajemy również niewielką rotację do każdego pojemnika.

```cpp
glBindVertexArray(VAO);  
for(unsigned int i = 0; i < 10; i++)  
{  
    glm::mat4 model;  
    model = glm::translate(model, cubePositions[i]);  
    float angle = 20.0f * i;  
    model = glm::rotate(model, glm::radians(angle), glm::vec3(1.0f, 0.3f, 0.5f));  
    ourShader.setMat4("model", model);

    glDrawArrays(GL_TRIANGLES, 0, 36);  
}
```

Ten fragment kodu zaktualizuje macierz modelu za każdym razem, gdy zostanie narysowany nowy sześcian i wykona to łącznie 10 razy. Teraz powinniśmy móc patrzeć na świat wypełniony 10 obróconymi kostkami:

![]({{ site.baseurl }}/img/learnopengl/coordinate_systems_multiple_objects.png){: .center-image }

Świetnie! Wygląda na to, że nasz pojemnik spotkał się z przyjaciółmi. Jeśli masz problemy, możesz porównać swój kod z [kodem źródłowym](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/6.3.coordinate_systems_multiple/coordinate_systems_multiple.cpp).

## Ćwiczenia

*   Spróbuj poeksperymentować z parametrami <span class="var">FoV</span> i <span class="var">aspect-ratio</span> funkcji projekcji perspektywicznej GLM. Zobacz, czy możesz dowiedzieć się, jak wpływają one na frustum perspektywiczne.
*   Pobaw się macierzą widoku, przesuwając ją w kilku różnych kierunkach i zobacz, jak zmienia się scena. Pomyśl o macierzy widoku jak o kamerze.
*   Postaraj się, aby co trzeci pojemnik (w tym pierwszy) obracał się w czasie, pozostawiając inne pojemniki bez zmian przy użyciu, tylko i wyłącznie, macierzy modelu: [rozwiązanie](https://learnopengl.com/code_viewer.php?code=getting-started/coordinate_systems-exercise3 "undefined").