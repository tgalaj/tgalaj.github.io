---
layout: post
title: Face culling
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: zaawansowany-opengl
---

{% include learnopengl.md link="Advanced-OpenGL/Face-culling" %}

Spróbuj mentalnie zwizualizować kostkę 3D i policz maksymalną liczbę ścianek (ang. *face*), które będziesz widzieć z dowolnego kierunku. Jeśli twoja wyobraźnia nie jest zbyt rozwinięta, prawdopodobnie uzyskałeś liczbę 3. Możesz zobaczyć sześcian z dowolnej pozycji i/lub kierunku, ale nigdy nie zobaczysz więcej niż 3 ścianek. Dlaczego mielibyśmy marnować wysiłek polegający na rysowaniu tych 3 pozostałych ścianek, których nawet nie widzimy? Gdybyśmy mogli odrzucić je w jakiś sposób, zaoszczędzilibyśmy ponad 50% inwokacji Fragment Shadera!

{: .box-note }
Mówimy _ponad 50%_ zamiast 50%, ponieważ z pewnych kierunków mogą być widoczne tylko 2 lub nawet 1 ścianka. W takim przypadku możemy zaoszczędzić **więcej** niż 50%.

To naprawdę świetny pomysł, ale musimy rozwiązać jeden problem: skąd wiemy, czy ścianka obiektu nie jest widoczna z punktu widzenia kamery?
Jeśli wyobrażamy sobie jakiś zamknięty kształt, każda z jego ścianek ma dwie strony. Każda strona będzie _skierowana_ w stronę użytkownika, albo będzie od niego odwrócona. Co by było, gdybyśmy mogli renderować tylko ścianki, które są skierowane w stronę kamery?

Właśnie to robi <def>face culling</def>. OpenGL sprawdza wszystkie ścianki, które są <def>skierowane przodem</def> do kamery i renderuje je, odrzucając wszystkie ścianki, które są <def>odwrócone</def>, oszczędzając nam wiele wywołań Fragment Shadera (są one drogie!). Musimy powiedzieć OpenGL, które ze ścianek, których używamy, są w rzeczywistości skierowane w naszą stronę, a które nie są. OpenGL używa do tego sprytnej sztuczki, analizując <def>kolejność definiowania</def> (ang. *winding order*) danych wierzchołków.

## Kolejność definiowania wierzchołków

Kiedy definiujemy zbiór wierzchołków trójkąta, definiujemy je w pewnym porządku, który jest albo <def>zgodnie z ruchem wskazówek zegara</def> (ang. *clockwise order*), albo <def>przeciwnie do ruchu wskazówek zegara</def> (ang. *counter-clockwise order*). Każdy trójkąt składa się z 3 wierzchołków i określamy te 3 wierzchołki w danej kolejności, patrząc od środka trójkąta.

![Kolejność definiowania wierzchołków trójkąta w OpenGL](/img/learnopengl/faceculling_windingorder.png){: .center-image }

Jak widać na rysunku, najpierw definiujemy wierzchołek `1`, a następnie możemy zdefiniować wierzchołek `2` lub `3`, a ten wybór definiuje kolejność wierzchołków tego trójkąta. Poniższy kod to ilustruje:

```cpp
    float vertices[] = {
        // zgodnie ze wskazówkami zegara
        vertices[0], // vertex 1
        vertices[1], // vertex 2
        vertices[2], // vertex 3
        // przeciwnie do ruchu wskazówek zegara
        vertices[0], // vertex 1
        vertices[2], // vertex 3
        vertices[1]  // vertex 2  
    };
```

Każdy zestaw 3 wierzchołków, które tworzą prymityw trójkąta, zawiera zatem kolejność wierzchołków. OpenGL używa tych informacji podczas renderowania prymitywów, aby określić, czy trójkąt jest <def>skierowany przodem</def> czy jest <def>odwrócony</def>. Domyślnie trójkąty zdefiniowane przeciwnie do ruchu wskazówek zegara są postrzegane jako te, które są skierowane przodem.

Definiując kolejność wierzchołków, wizualizujesz odpowiedni trójkąt tak, jakby był skierowany do Ciebie, więc każdy trójkąt, który definiujesz, powinien mieć kolejność definiowania wierzchołków przeciwną do ruchu wskazówek zegara tak, jakbyś był ustawiony bezpośrednio przodem do tego trójkąta. Fajną rzeczą w określaniu wszystkich twoich wierzchołków w ten sposób jest fakt, że aktualny porządek definiowania wierzchołków jest obliczany na etapie rasteryzacji. Wierzchołki są wtedy widziane z punktu widzenia **użytkownika**.

Wszystkie wierzchołki trójkąta, które obserwator widzi, są rzeczywiście w poprawnej kolejności definiowania. Natomiast, wierzchołki trójkątów po drugiej stronie sześcianu są teraz renderowane w taki sposób, że ich kolejność definiowania zostaje odwrócona. Powoduje to, że trójkąty, które widzimy, są postrzegane jako te przednie trójkąty, a trójkąty z tyłu są postrzegane jako skierowane tyłem do nas. Poniższy obraz pokazuje ten efekt:

![Kamera widzi trójkąty skierowane przodem lub tyłem](/img/learnopengl/faceculling_frontback.png){: .center-image }

W danych wierzchołków zdefiniowalibyśmy oba trójkąty w kierunku przeciwnym do ruchu wskazówek zegara (trójkąt przedni jako 1, 2, 3 i trójkąt tylny również jako 1, 2 i 3 (gdybyśmy widzieli trójkąt z przodu)). Jednak z punktu widza trójkąt tylny jest renderowany zgodnie z ruchem wskazówek zegara, jeśli narysujemy go w kolejności 1, 2 i 3 z bieżącego punktu widzenia widza. Mimo że podaliśmy trójkąt tylny w kolejności przeciwnej do ruchu wskazówek zegara, jest on teraz renderowany w kolejności zgodnej z ruchem wskazówek zegara. Właśnie to jest to, czego chcemy czyli <def>odrzucić</def> (ang. *cull*) niewidoczne ścianki (ang. *faces*)!

## Face culling

Na początku tutorialu powiedzieliśmy, że OpenGL jest w stanie odrzucić trójkąty, jeśli są skierowane tyłem (ang. *back-face triangle*). Teraz, gdy wiemy, jak ustawić kolejność wierzchołków, możemy zacząć używać opcji <def>face culling</def> w OpenGL, która jest domyślnie wyłączona.

Dane wierzchołków kostki, których używaliśmy podczas ostatnich ćwiczeń, nie zostały zdefiniowane w kolejności zgodnej z ruchem wskazówek zegara, więc zaktualizowałem dane wierzchołków, które można skopiować [stąd](https://learnopengl.com/code_viewer.php?code=advanced/faceculling_vertexdata). Dobrą praktyką jest to, aby spróbować wyobrazić sobie, że te wierzchołki są rzeczywiście zdefiniowane w kolejności przeciwnej do ruchu wskazówek zegara dla każdego trójkąta.

Aby włączyć face culling, musimy włączyć opcję <var>GL_CULL_FACE</var> w OpenGL:

```cpp
    glEnable(GL_CULL_FACE);  
```

Od tego momentu wszystkie ścianki, które nie są skierowane w stronę kamery, są odrzucane (spróbuj polatać wewnątrz sześcianu, aby zobaczyć, że wszystkie wewnętrzne ścianki są rzeczywiście odrzucane). Obecnie oszczędzamy ponad 50% wydajności renderowania fragmentów, ale zauważmy, że działa to tylko z zamkniętymi kształtami, takimi jak sześcian. Będziemy musieli ponownie wyłączyć funkcję usuwania ścianek, gdy narysujemy liście trawy z poprzedniego samouczka, ponieważ ich przednie **i** tylne ścianki powinny być widoczne.

OpenGL pozwala nam zmienić typ ścianki, którą chcemy odrzucać. Co jeśli chcemy odrzucać ścianki przednie, a nie tylne? Możemy zdefiniować to zachowanie, wywołując <fun>glCullFace</fun>:

```cpp
    glCullFace(GL_FRONT);  
```

Funkcja <fun>glCullFace</fun> przyjmuje trzy możliwe opcje:

*   `GL_BACK`: odrzuca tylko tylne ścianki.
*   `GL_FRONT`: odrzuca tylko przednie ścianki.
*   `GL_FRONT_AND_BACK`: odrzuca zarówno przednie, jak i tylne ścianki.

Domyślna wartość <fun>glCullFace</fun> to <var>GL_BACK</var>. Poza ściankami do odrzucenia możemy także powiedzieć OpenGL, że wolimy, aby ścianki, które są zdefiniowane zgodnie z ruchem wskazówek zegara były traktowane jako ścianki skierowane w naszą stronę. Możemy to zrobić za pomocą <fun>glFrontFace</fun>:

```cpp
    glFrontFace(GL_CCW);  
```

Wartością domyślną jest <var>GL_CCW</var> oznaczająca kolejność w przeciwną do ruchu wskazówek zegara. Drugą opcją jest <var>GL_CW</var>, która (oczywiście) oznacza kolejność zgodną z ruchem wskazówek zegara.

Jako prosty test mogliśmy odwrócić kolejność wierzchołków, mówiąc OpenGL, że ścianki przednie są teraz definiowane przez porządek zgodny z ruchem wskazówek zegara, a nie w kierunku przeciwnym do ruchu wskazówek zegara:

```cpp
    glEnable(GL_CULL_FACE);
    glCullFace(GL_BACK);
    glFrontFace(GL_CW);  
```

W rezultacie renderowane są tylko tylne ścianki:

![Renderowane są tylko tylne ścianki](/img/learnopengl/faceculling_reverse.png){: .center-image }

Zauważ, że możesz stworzyć ten sam efekt przez odrzucenie ścianek przednich z domyślną kolejnością przeciwną do ruchu wskazówek zegara:

```cpp
    glEnable(GL_CULL_FACE);
    glCullFace(GL_FRONT);  
```

Jak widać, funkcja face culling jest doskonałym narzędziem do zwiększania wydajności aplikacji OpenGL przy minimalnym wysiłku. Musisz sprawdzić, które obiekty będą faktycznie czerpać korzyści z odrzucania ścianek i które obiekty nie powinny być poddawane tej operacji.

## Ćwiczenia

*   Czy możesz ponownie zdefiniować dane wierzchołków, określając każdy trójkąt w kolejności zgodnej z ruchem wskazówek zegara, a następnie wyrenderować scenę z ustawieniem kolejności przednich ścianek zgodną z ruchem wskazówek zegara? [rozwiązanie](https://learnopengl.com/code_viewer.php?code=advanced/faceculling-exercise1)