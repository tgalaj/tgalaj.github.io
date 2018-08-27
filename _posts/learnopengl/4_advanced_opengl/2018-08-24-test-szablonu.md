---
layout: post
title: Test szablonu
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: zaawansowany-opengl
---

{% include learnopengl.md link="Advanced-OpenGL/Stencil-testing" %}

Po przetworzeniu fragmentu przez Fragment Shader zostaje wykonany tzw. <def>test szablonu</def> (ang. *stencil test*), który podobnie jak test głębi, ma możliwość odrzucania fragmentów. Następnie pozostałe fragmenty zostają przekazane do testu głębokości, który może odrzucić jeszcze więcej fragmentów. Test szablonu oparty jest na zawartości kolejnego bufora o nazwie <def>bufora szablonu</def> (ang. *stencil buffer*), który możemy aktualizować podczas renderowania w celu uzyskania interesujących efektów.

Bufor szablonu zawiera (zwykle) `8` bitowe <def>wartości szablonu</def>, co daje łącznie `256` różnych wartości szablonu na piksel/fragment. Następnie możemy ustawić te wartości szablonu na wartości wybrane przez nas, a następnie możemy odrzucić lub zachować fragmenty, gdy dany fragment ma określoną wartość szablonu.

{: .box-note }
Każda biblioteka okienkowa musi dla Ciebie stworzyć bufor szablonu. GLFW robi to automatycznie, więc nie musimy mówić GLFW, aby go utworzyć, ale inne biblioteki okienkowe nie mogą domyślnie tworzyć bufora szablonów, więc koniecznie sprawdź dokumentację biblioteki.

Prosty przykład zastosowania bufora szablonu jest pokazany poniżej:

![Prosta demonstracja bufora szablonu](/img/learnopengl/stencil_buffer.png){: .center-image }

Bufor szablonu jest najpierw czyszczony zerami, a następnie prostokąt `1` zostaje ustawiony w buforze szablonu. Fragmenty sceny są renderowane wtedy (pozostałe są odrzucane), gdy wartość szablonu tego fragmentu zawiera wartość `1`.

Operacje bufora szablonu pozwalają nam ustawić ten bufor dla określonych wartości wszędzie tam, gdzie renderujemy fragmenty. Zmieniając zawartość bufora szablonu podczas renderowania, _zapisujemy_ dane do bufora szablonu. W tej samej (lub następnej) iteracji renderowania możemy odczytać te wartości, aby odrzucić lub narysować pewne fragmenty. Korzystając z bufora szablonu, możesz się trochę pogubić, ale ogólny schemat korzystania z tego bufora jest zwykle następujący:

*   Włącz zapisywanie do bufora szablonu.
*   Narysuj obiekty, aktualizując zawartość bufora szablonu.
*   Wyłącz zapisywanie do bufora szablonu.
*   Narysuj (inne) obiekty, tym razem odrzucając pewne fragmenty w oparciu o zawartość bufora szablonu.

Korzystając z bufora szablonu możemy w ten sposób odrzucić pewne fragmenty na podstawie fragmentów innych narysowanych obiektów w scenie.

Możesz włączyć test szablonu, włączając opcję <var>GL_STENCIL_TEST</var>. Od tego momentu wszystkie wywołania renderingu będą miały wpływ na bufor szablonu w taki czy inny sposób.

```cpp
    glEnable(GL_STENCIL_TEST);    
```

Zauważ, że musisz także wyczyścić bufor szablonu przed każdą nową iteracją, podobnie jak bufor koloru i głębi:

```cpp
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT); 
```

Podobnie jak dla testu głębokości istnieje funkcja <fun>glDepthMask</fun>, tak również istnieje odpowiednia funkcja dla bufora szablonu. Funkcja <fun>glStencilMask</fun> pozwala nam ustawić maskę bitową `AND` z wartością szablonu, która ma zostać zapisana w buforze. Domyślnie jest ona ustawiona na maskę bitową `1`, które nie modyfikują wyjścia, ale jeśli ustawimy ją na `0x00`, to wszystkie wartości szablonu zapisane w buforze skończą się jako `0`. Jest to równoważne z funkcją testu głębi <fun>glDepthMask(GL_FALSE)</fun>:

```cpp
    glStencilMask(0xFF); // każdy bit jest zapisywany w buforze szablonu, tak jak jest
    glStencilMask(0x00); // każdy bit kończy się jako 0 w buforze szablonu (wyłączanie zapisywania)
```

W większości przypadków po prostu wpisujesz `0x00` lub `0xFF` jako maskę szablonu, ale dobrze jest wiedzieć, że istnieje opcja ustawienia niestandardowych masek bitowych.

## Funkcje testu szablonu

Podobnie jak przy teście głębokości, mamy pewną kontrolę nad tym, kiedy test szablonu powinien przejść lub nie i jak powinien wpływać na bufor szablonu. Istnieją dwie funkcje, za pomocą których możemy skonfigurować test szablonu: <fun>glStencilFunc</fun> i <fun>glStencilOp</fun>.

Funkcja <fun>glStencilFunc(GLenum func, GLint ref, maska ​​GLuint)</fun> przyjmuje trzy parametry:

*   `func`: ustawia funkcję testu szablonu. Ta funkcja testu jest aplikowana do zapisanej wartości bufora szablonu i wartości `ref` funkcji <fun>glStencilFunc</fun>. Możliwe opcje to: <var>GL_NEVER</var>, <var>GL_LESS</var>, <var>GL_LEQUAL</var>, <var>GL_GREATER</var>, <var>GL_GEQUAL</var>, <var>GL_EQUAL</var>, <var>GL_NOTEQUAL</var> i <var>GL_ALWAYS</var>. Ich znaczenie semantyczne jest podobne do funkcji bufora głębi.
*   `ref`: określa wartość referencyjną (odniesienia) dla testu szablonu. Zawartość bufora szablonu jest porównywana z tą wartością.
*   `mask`: określa maskę, która jest poddawana operacji `AND` z wartością referencyjną i jest zapisywana do bufora szablonu, zanim test je porówna. Początkowo ustawione na `1`.

Tak więc w przypadku prostego przykładu zastosowania bufora szablonu, który pokazaliśmy na początku, funkcja będzie ustawiona na:

```cpp
    glStencilFunc(GL_EQUAL, 1, 0xFF)
```

Mówi to OpenGL'owi, że ilekroć wartość szablonu fragmentu jest równa (`GL_EQUAL`) wartości odniesienia `1`, ten fragment przechodzi test i zostaje narysowany, w przeciwnym wypadku zostaje on odrzucony.

Ale funkcja <fun>glStencilFunc</fun> opisuje tylko to, co OpenGL powinien zrobić z zawartością bufora szablonu, a nie jak możemy zaktualizować ten bufor. Tutaj pojawia się funkcja <fun>glStencilOp</fun>.

Funkcja <fun>glStencilOp(GLenum sfail, GLenum dpfail, GLenum dppass)</fun> zawiera trzy opcje, z które określają jakie działania podjąć:

*   `sfail`: czynność do wykonania, jeśli test szablonu nie powiedzie się.
*   `dpfail`: czynność do wykonania, jeśli test szablonu przejdzie, ale test głębokości nie powiedzie się.
*   `dppass`: czynność, którą należy podjąć, jeśli przechodzi zarówno test szablonu, jak i test głębi.

Następnie dla każdej z opcji możesz wykonać następujące czynności:

<table align="center">
  <tbody><tr>
  	<th style="text-align:center;">Akcja</th>
  	<th style="text-align:center;">Opis</th>
  </tr>  
  <tr>
    <td style="text-align:center;">GL_KEEP</td>
 	<td style="text-align:center;">Aktualnie przechowywana wartość szablonu jest zachowywana.</td>
  </tr>
  <tr>
    <td style="text-align:center;">GL_ZERO</td>
 	<td style="text-align:center;">Wartość szablonu jest ustawiona na `0`.</td>
  </tr>
  <tr>
    <td style="text-align:center;">GL_REPLACE</td>
 	<td style="text-align:center;">Wartość szablonu zastępowana jest wartością odniesienia ustawioną za pomocą <fun>glStencilFunc</fun>.</td>
  </tr>
  <tr>
    <td style="text-align:center;">GL_INCR</td>
 	<td style="text-align:center;">Wartość szablonu zostaje zwiększona o 1, jeśli jest niższa od wartości maksymalnej.</td>
  </tr><tr>
    <td style="text-align:center;">GL_INCR_WRAP</td>
 	<td style="text-align:center;">Robi to samo co <var>GL_INCR</var>, z tym, że "zawija" wartość z powrotem do `0`, gdy tylko zostanie przekroczona maksymalna wartość.</td>
  </tr> 
  <tr>
    <td style="text-align:center;">GL_DECR</td>
 	<td style="text-align:center;">Wartość szablonu zostaje zmniejszona o 1, jeśli jest wyższa niż wartość minimalna.</td>
  </tr>
  <tr>
    <td style="text-align:center;">GL_DECR_WRAP</td>
 	<td style="text-align:center;">Robi to samo co <var>GL_DECR</var>, z tym, że "zawija" wartość do wartości maksymalnej, jeśli jest ona mniejsza niż `0`.</td>
  </tr>
  <tr>
    <td style="text-align:center;">GL_INVERT</td>
 	<td style="text-align:center;">Bitowo odwraca bieżącą wartość bufora szablonu.</td>
  </tr>
</tbody></table>

Domyślnie funkcja <fun>glStencilOp</fun> jest ustawiona na `(GL_KEEP, GL_KEEP, GL_KEEP)`, więc niezależnie od wyniku któregokolwiek z testów, bufor szablonu zachowuje jego wartości. Domyślne zachowanie nie powoduje aktualizacji bufora szablonu, więc jeśli chcesz pisać do bufora szablonu, musisz określić co najmniej jedną akcję dla dowolnej opcji.

Używając funkcji <fun>glStencilFunc</fun> i <fun>glStencilOp</fun> możemy dokładnie określić, kiedy i jak chcemy zaktualizować bufor szablonu i możemy również określić, kiedy test szablonu ma przejść a kiedy nie np. kiedy fragmenty należy odrzucić.

# Obramowanie obiektów

Byłoby mało prawdopodobne, gdybyś w pełni zrozumiał, w jaki sposób test szablonu działa tylko na bazie poprzednich sekcji, więc przedstawimy konkretną przydatną funkcjonalność, którą można zaimplementować za pomocą samego testu szablonu zwanego <def>obramowaniem obiektu</def> (ang. `object outlining`).

![Obiekt obramowany za pomocą bufora szablonu](/img/learnopengl/stencil_object_outlining.png){: .center-image }

Obramowanie obiektów robi dokładnie to, co mówi nazwa. Dla każdego obiektu (lub tylko jednego) tworzymy małe kolorowe obramowanie wokół (połączonych) obiektów. Jest to szczególnie użyteczny efekt, gdy chcesz na przykład wybrać jednostki w grze strategicznej i musisz pokazać użytkownikowi, która z jednostek została wybrana. Procedura obramowania obiektów jest następująca:

1.  Ustaw funkcję testu szablonu na <var>GL_ALWAYS</var> przed rysowaniem obiektów (które mają być obramowane), aktualizując bufor szablonu za pomocą wartości `1` wszędzie tam, gdzie renderowane są fragmenty obiektów.
2.  Narysuj obiekty.
3.  Wyłącz pisanie do bufora szablonu i test głębokości.
4.  Przeskaluj każdy z obiektów o niewielką wartość.
5.  Użyj innego Fragment Shader'a, który na wyjściu daje pojedynczy kolor (obramowanie).
6.  Narysuj obiekty ponownie, ale tylko wtedy, gdy wartości szablonu nie są równe `1`.
7.  Włącz ponownie pisanie do bufora szablonu i test głębokości.

Ten proces ustawia zawartość bufora szablonu na `1` dla każdego fragmentu obiektu, a gdy chcemy narysować obramowanie, w zasadzie rysujemy powiększone wersje obiektów i wszędzie tam, gdzie test szablonu przechodzi, przeskalowana wersja obiektu jest rysowana wokół obiektu. Zasadniczo odrzucamy wszystkie fragmenty przeskalowanych wersji obiektów, które są częścią oryginalnych fragmentów obiektów przy użyciu bufora szablonu.

Dlatego najpierw stworzymy bardzo prosty Fragment Shader, który będzie rysował kolor obramowania. Po prostu ustawiamy zakodowaną wartość koloru i uruchamiamy shader <var>shaderSingleColor</var>:

```glsl
    void main()
    {
        FragColor = vec4(0.04, 0.28, 0.26, 1.0);
    }
```

Zamierzamy tylko dodać obrys obiektu do dwóch pojemników - podłogę zostawimy tak jak jest. Chcemy najpierw narysować podłogę, następnie dwa pojemniki (podczas zapisywania do bufora szablonu), a następnie narysować przeskalowane pojemniki (jednocześnie odrzucając fragmenty, które pokrywają się z wcześniej narysowanymi fragmentami kontenera).

Najpierw chcemy włączyć test szablonu i ustawić akcje, które mają zostać wykonane, gdy którykolwiek z testów zakończy się sukcesem lub niepowodzeniem:

```cpp
    glEnable(GL_STENCIL_TEST);
    glStencilOp(GL_KEEP, GL_KEEP, GL_REPLACE);  
```

Jeśli którykolwiek z testów zakończy się niepowodzeniem, nie robimy nic, po prostu przechowujemy aktualnie zapisaną wartość, która znajduje się w buforze szablonu. Jeśli jednak zarówno test szablonu, jak i test głębi się powiodą, chcemy zastąpić zapisaną wartość szablonu wartością odniesienia ustawioną za pomocą <fun>glStencilFunc</fun>, którą później ustawimy na wartość `1`.

Czyścimy bufor szablonu do `0`, a dla kontenerów aktualizujemy bufor szablonu do `1` dla każdego narysowanego fragmentu:

```cpp
    glStencilFunc(GL_ALWAYS, 1, 0xFF); // wszystkie fragmenty powinny aktualizować bufor szablonu
    glStencilMask(0xFF); // włącz zapisywanie do bufora szablonu
    normalShader.use();
    DrawTwoContainers();
```

Korzystając z funkcji testu szablonu <var>GL_ALWAYS</var>, upewniamy się, że każdy z fragmentów kontenerów aktualizuje bufor szablonu za pomocą szablonu o wartości `1`. Ponieważ fragmenty zawsze przechodzą test szablonu, bufor szablonu jest aktualizowany z wartością odniesienia wszędzie tam, gdzie je narysowaliśmy.

Teraz, gdy bufor szablonu jest aktualizowany za pomocą `1`, gdzie pojemniki zostały narysowane, rysujemy przeskalowane pojemniki, ale tym razem wyłączając zapisy do bufora szablonu:

```cpp
    glStencilFunc(GL_NOTEQUAL, 1, 0xFF);
    glStencilMask(0x00); // wyłącz zapisywanie do bufora szablonu
    glDisable(GL_DEPTH_TEST);
    shaderSingleColor.use(); 
    DrawTwoScaledUpContainers();
```

Ustawiamy funkcję szablonu na <var>GL_NOTEQUAL</var>, która zapewnia, że narysujemy tylko części kontenerów, których wartości bufora szablonu nie są równe `1`, a zatem tylko narysują części kontenerów, które znajdują się poza wcześniej narysowanymi kontenerami. Pamiętaj, że wyłączamy również testowanie głębokości, więc przeskalowane kontenery - obramowanie, nie zostanie nadpisane przez podłogę.

Upewnij się, że ponownie włączasz bufor głębi, gdy skończysz.

Cały kod przedstawiający algorytm dla naszej sceny będzie wyglądał mniej więcej tak:

```cpp
    glEnable(GL_DEPTH_TEST);
    glStencilOp(GL_KEEP, GL_KEEP, GL_REPLACE);  

    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT); 

    glStencilMask(0x00); // upewnij się, że nie aktualizujemy bufora szablonu podczas rysowania podłogi
    normalShader.use();
    DrawFloor()  

    glStencilFunc(GL_ALWAYS, 1, 0xFF); 
    glStencilMask(0xFF); 
    DrawTwoContainers();

    glStencilFunc(GL_NOTEQUAL, 1, 0xFF);
    glStencilMask(0x00); 
    glDisable(GL_DEPTH_TEST);
    shaderSingleColor.use(); 
    DrawTwoScaledUpContainers();
    glStencilMask(0xFF);
    glEnable(GL_DEPTH_TEST);  
```

Tak długo, jak rozumiesz ogólną ideę testu szablonu, to ten fragment kodu nie powinien być zbyt trudny do zrozumienia. W przeciwnym razie postaraj się uważnie przeczytać poprzednie sekcje i spróbuj dokładnie zrozumieć, co każda z funkcji robi po tym jak zobaczyłeś przykład jej użycia.

Wynik tego algorytmu obramowania obiektów, w scenie z tutorialu Testowanie głębokości, wygląda następująco:

![Scena 3D z obrysowaniem obiektu za pomocą bufora szablonu](/img/learnopengl/stencil_scene_outlined.png){: .center-image }

Sprawdź kod źródłowy [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/4.advanced_opengl/2.stencil_testing/stencil_testing.cpp), aby zobaczyć pełny kod algorytmu obrysowania obiektów.

{: .box-note }
Widać, że granice obramowania obu pojemników pokrywają się, co zwykle jest efektem, którego pożądamy (myślę o grach strategicznych,w których chcemy wybrać 10 jednostek, zwykle potrzebujemy scalania obramowania). Jeśli chcesz uzyskać pełną ramkę na każdy obiekt, musisz wyczyścić bufor szablonu dla każdego obiektu i odrobinę pokombinować z buforem głębi.

Algorytm obrysowania obiektów jest dość często używany w grach do wizualizacji zaznaczonych obiektów (pomyśl o grach strategicznych) i taki algorytm może być łatwo zaimplementowany w klasie Model. Następnie można po prostu ustawić flagę typu bool w klasie modelu, czy mamy narysować ten obiekt z obramowaniem czy bez niego. Jeśli chcesz być puścić wodze kreatywności, możesz nawet nadać obramowaniu bardziej naturalny wygląd za pomocą filtrów post-processingu, takich jak rozmycie Gaussa (ang. *Gaussian Blur*).

Testowanie szablonów ma wiele innych zastosowań, poza obrysowywaniem obiektów, takich jak rysowanie tekstur wewnątrz lusterka wstecznego samochodu, dzięki czemu idealnie pasuje ona do kształtu lusterka lub renderowania cieni w czasie rzeczywistym za pomocą bufora szablonu o nazwie <def>shadow volumes</def>. Bufor szablonu jest kolejnym fajnym narzędziem w naszym rozbudowanym już zestawie narzędzi OpenGL.