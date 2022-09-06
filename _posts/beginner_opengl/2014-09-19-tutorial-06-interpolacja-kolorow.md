---
layout: post
title: Tutorial 06 - Interpolacja
subtitle: Kurs OpenGL dla początkujących
tags: [beginner-opengl-pl, tutorial]
---

## Wstęp

W dzisiejszym tutorialu przyjrzymy się ważnemu etapowi w potoku renderowania - interpolacji, którą wykonuje rasteryzer na wartościach, które wychodzą z vertex shader'a. Była o tym drobna wzmianka w [Tutorial 04]({{ site.baseurl }}{% post_url beginner_opengl/2014-06-08-tutorial-04-czym-jest-programowalny-potok-renderowania %} "Tutorial 04 – Czym jest programowalny potok renderingu?") w sekcji dotyczącej rasteryzacji, a dzisiaj zobaczymy jak to działa w praktyce. Odpowiedzi do ćwiczeń z poprzedniego kursu są zamieszczone poniżej:

<details class="panel panel-success">
  <summary markdown="span" class="panel-heading">
    Odpowiedzi do ćwiczeń
  </summary>

**1.** Musimy zmienić kolor czyszczący tła na niebieski. W modelu RGB, kolor niebieski jest przedstawiany jako trójka (0, 0, 1). Stąd mała zmiana w kodzie:

```cpp  
glClearColor(0.0f, 0.0f, 1.0f, 1.0f);  
```

**2.** Kolorowaniem zajmuje się fragment shader, dlatego tam musimy zrobić zmianę - musimy sprawić by kolorem wyjściowym nie był czarny, ale zielony. Kolor zielony, w modelu RGB, jest przedstawiany jako trójka (0, 1, 0), dlatego zmiana w fragment shader'ze wygląda tak:

```glsl  
fragColor = vec4(0.0f, 1.0f, 0.0f, 1.0f);  
```

</details>

Rasteryzer interpoluje (uśrednia) wartości między trzema wierzchołkami trójkąta, a następnie "odwiedza" każdy piksel poprzez wywołanie fragment shader'a, który zwraca kolor danego piksela, który jest zapisywany przez rasteryzer do bufora koloru. Czyniąc długą historię krótką, jeżeli mamy zdefiniowany kolor dla każdego wierzchołka, przyjmijmy lewy-dolny czarny, prawy-dolny czerwony, górny zielony, to w końcowym efekcie wartość koloru w tych wierzchołkach będzie wynosił odpowiednio: (0, 0, 0), (1, 0, 0), (0, 1, 0). Następnie wywoływany jest fragment shader, dla każdego piksela na ekranie (nas interesuje to co dzieje się podczas kolorowania prymitywu) i podczas tego procesu fragment shader koloruje każdy piksel uśrednionym kolorem.

Ten sam proces jest wykonywany dla innych wartości, które z reguły przypisane są do wierzchołków. Jedną z tych wartości są wektory normalne, które są wykorzystywane do obliczeń światła oraz współrzędne tekstury, które zastępują nam kolor.

Ta teoria może z początku wydawać się dziwna i niezrozumiała dlatego przejdźmy do części praktycznej.

## Wyjaśnienie kodu

Zmian w kodzie jest niewiele i dotyczą one tylko vertex i fragment shader'a. Zacznijmy od vertex shader'a:

```glsl  
#version 440

layout (location = 0) in vec3 vertexPosition;

out vec3 pos;

void main()  
{  
    pos = vertexPosition;

    gl_Position = vec4(vertexPosition.x, vertexPosition.y, vertexPosition.z, 1.0f);  
}  
```

Pojawiła się nowa zmienna _pos_ z kwalifikatorem _out_, co oznacza, że będzie ona dostępna w kolejnych etapach procesu renderowania - będziemy mogli wziąć jej wartości i wykorzystać np. w fragment shader. Następnie do tej zmiennej przypisujemy wartość pozycji z atrybutu wejściowego _vertexPosition_.

Do czego nam jest potrzebna wartość pozycji wierzchołka w fragment shaderze? Dlatego, że wykorzystamy tę informację do tego, by mieć różne kolory na każdym wierzchołku.

Przyjrzyjmy się fragment shader'owi:

```glsl  
#version 440

in vec3 pos;

out vec4 fragColor;

void main()  
{  
    fragColor = vec4(pos.x, pos.y, pos.z, 1.0f);  
}  
```

Tutaj też pojawiła się nowa zmienna, z tą samą nazwą co w vertex shaderze, tylko z innym kwalifikatorem. Tutaj jest to _in_, co oznacza, że jest to wartość która wchodzi do tego shader'a. Musi mieć ona taką samą nazwę jak w shaderze, z którego "wychodzi". Tutaj jako kolejne składowe koloru podajemy kolejno współrzędne pozycji, które nadadzą każdemu wierzchołkowi inny kolor, a środek trójkąta będzie zawierał uśrednione kolory. Jak widzimy proces uśredniania kolorów dla reszty pikseli jest automatyczny i zajmuje się nim rasteryzer.

Efekt działania jest przedstawiony poniżej:

![Interpolowane kolory pomiędzy wierzchołkami trójkąta]({{ site.baseurl }}/img/beginner_opengl/tutorial-06-beginner-gl.png){: .center-image }

Solucję można jak zwykle pobrać z sekcji [_Kod źródłowy_](#source_code).

## Zakończenie

Mam nadzieję, że tutorial rozjaśnił Wam czym jest interpolacja i co jest odpowiedzialne za nią. W razie wątpliwości proszę o pisanie do mnie na maila bądź po prostu o zostawienie komentarza poniżej. W następnej części kursu wejdziemy razem w trzeci wymiar, w którym nasz trójkąt wyewoluuje do postaci piramidy!

## Kod źródłowy {#source_code}
*   [Solucja VC++ 2010](https://drive.google.com/file/d/0B0j4jdWAANaoczF2dXhlSTBOTE0/view?usp=sharing&resourcekey=0-_oGIuBVTC92kypKjPHl-HQg)
