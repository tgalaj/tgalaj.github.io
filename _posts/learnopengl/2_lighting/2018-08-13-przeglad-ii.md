---
layout: post
title: Przegląd
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: oswietlenie
---

{% include learnopengl.md link="Lighting/Review" %}

Gratulacje, że dotarliście tak daleko! Nie jestem pewien, czy zauważyliście, ale we wszystkich tutorialach o oświetleniu nie dowiedzieliśmy się niczego nowego o OpenGL, z wyjątkiem kilku drobnych elementów, takich jak dostęp do tablic uniformów. Wszystkie tutoriale do tej pory polegały na manipulowaniu shaderami za pomocą technik i równań, aby uzyskać realistyczne efekty oświetleniowe. To ponownie pokazuje moc shaderów. Shadery są niezwykle elastyczne i widziałeś na własne oczy, że za pomocą zaledwie kilku wektorów 3D i niektórych konfigurowalnych zmiennych udało nam się stworzyć niesamowitą grafikę!

Kilka ostatnich tutoriali, poszerzyło naszą wiedzę o kolorach, modelu oświetlenia Phong'a (obejmujący oświetlenie otoczenia, rozproszone i lustrzane), materiałach obiektów, konfigurowalnych właściwościach światła, mapach diffuse i specular, różnych rodzajach świateł i jak połączyć całą tę wiedzę w jednej scenie. Koniecznie poeksperymentuj z różnymi światłami, kolorami materiałów, właściwościami światła i spróbuj stworzyć własną scenę za pomocą odrobiny kreatywności.

W następnych tutorialach będziemy dodawać bardziej zaawansowane kształty do naszej sceny, które wyglądają naprawdę dobrze pod wpływem omówionych modelach oświetlenia.

## Słowniczek

*   `Wektor koloru`: wektor przedstawiający większość rzeczywistych kolorów świata poprzez kombinację czerwonego, zielonego i niebieskiego komponentu (w skrócie `RGB`). Kolor obiektu jest w rzeczywistości odzwierciedlonymi składnikami koloru światła, których obiekt nie zaabsorbował.
*   `Model oświetlenia Phonga`: model przybliżający rzeczywiste oświetlenie poprzez obliczanie komponentu ambient, diffuse i specular.
*   `Światło otoczenia` (ang. *ambient lighting*): przybliżenie globalnego oświetlenia poprzez nadanie każdemu obiektowi małej jasności, aby obiekty nie były całkowicie ciemne, jeśli nie są oświetlane.
*   `Światło rozproszone` (ang. *diffuse lighting*): oświetlenie, które staje się mocniejsze, im bardziej wierzchołek/fragment jest zbliżony do źródła światła. Używa normalnych wektorów do obliczania kątów.
*   `Wektor normalny`: wektor (jednostkowy), który jest prostopadły do ​​powierzchni.
*   `Macierz normalnych`: macierz 3x3, która jest macierzą modelu (lub modelu-widoku) bez translacji. Jest również modyfikowana w taki sposób (odwrotność transpozycji), aby utrzymać wektory normalne skierowane we właściwą stronę po zastosowaniu nierównomiernego skalowania. W przeciwnym razie normalne wektory ulegają zniekształceniu, gdy stosuje się nierównomierne skalowanie.
*   `Światło lustrzane`: powoduje rozbłysk na powierzchni obiektu, im bliżej jest widza, który patrzy na odbicie źródła światła na powierzchni. W oparciu o kierunek widza, kierunek światła i wartość *shininess*, które określa wielkość rozbłysku.
*   `Cieniowanie Phonga`: model oświetlenia Phong zastosowany w Fragment Shaderze.
*   `Cieniowanie Gouraud`: model oświetlenia Phong zastosowany w Vertex Shader. Tworzy zauważalne artefakty przy użyciu niewielkiej liczby wierzchołków. Zyskuje efektywność kosztem utraty jakości wizualnej.
*   `Struktura GLSL`: struktura podobna do C, która działa jako kontener dla zmiennych shadera. Głównie używane do organizowania wejścia/wyjścia/uniformów.
*   `Materiał`: kolor światła otaczającego, rozproszenia i lustrzanego, który odbija obiekt. Ustawiają kolory obiektu.
*   `Światło (właściwości)`: natężenie światła otoczenia, rozproszenia i lustrzanego. Mogą one przyjmować dowolną wartość koloru i określać, w jakim kolorze/intensywności świeci źródło światła dla każdego określonego składnika Phonga.
*   `Mapa diffuse`: obraz tekstury, który ustawia kolor światła rozproszenia na każdy fragment obiektu.
*   `Mapa specular`: obraz tekstury, która ustawia intensywność/kolor światła lustrzanego na każdy fragment obiektu. Umożliwia zastosowanie rozbłysku tylko w określonych obszarach obiektu.
*   `Światło kierunkowe`: źródło światła opisane za pomocą kierunku. Jest on modelem światła, które znajduje się nieskończenie daleko, co powoduje, że wszystkie jego promienie świetlne mogą być uznane za równoległe, a zatem jego wektor kierunkowy pozostaje taki sam na całej scenie.
*   `Światło punktowe`: źródło światła z pozycją na scenie ze światłem, którego intensywność zanika wraz z odległością.
*   `Tłumienie`: proces zmniejszania natężenia światła wraz z odległością, stosowany w światłach punktowych i reflektorach.
*   `Reflektor`: źródło światła określone przez stożek, który skierowany jest w jednym, określonym kierunku.
*   `Latarka`: reflektor umieszczony z perspektywy widza.
*   `Tablica uniformów GLSL`: tablica uniformów. Działają tak, jak tablice C, z tym wyjątkiem, że nie można ich dynamicznie alokować.