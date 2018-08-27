---
layout: post
title: Przegląd
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: pierwsze-kroki
---

{% include learnopengl.md link="Getting-started/Review" %}

Gratuluję dotarcia do końca sekcji _Pierwsze kroki_. Teraz umiesz już tworzyć okna z kontekstem OpenGL, tworzyć i kompilować shadery, wysyłać dane wierzchołkowe do shaderów poprzez obiekty buforów lub uniformy, rysować obiekty, używać tekstur, rozumiesz czym są wektory i macierze oraz umiesz połączyć całą tą wiedzę, aby stworzyć pełną scenę 3D z wirtualną kamerą.  

Uff, nauczyliśmy się bardzo dużo z ostatnich kilku rozdziałów. Staraj się bawić samouczkami, trochę eksperymentuj lub spróbuj zaimplementować własne pomysły. Jak tylko poczujesz, że wszystkie wcześniej omówione materiały nie kryją przed Tobą żadnych tajemnic, to nadszedł czas, aby przejść do następnych tutoriali.

## Słowniczek

*   <span class="var">OpenGL</span>: formalna specyfikacja graficznego API, która definiuje wygląd i rezultat każdej z funkcji.
*   <span class="var">GLAD</span>: biblioteka ładująca i ustawiająca wszystkie wskaźniki funkcji OpenGL za nas, dzięki czemu możemy używać wszystkich (nowoczesnych) funkcji OpenGL.
*   <span class="var">Viewport</span>: obszar, w którym renderujemy.
*   <span class="var">Potok renderowania</span>: cały proces jaki musi przejść wierzchołek zanim zostanie wyświetlony jako piksel na ekranie.
*   <span class="var">Shader</span>: mały program działający na karcie graficznej. Kilka etapów z potoku graficznego może używać stworzonych przez użytkownika programów cieniujących, w celu transformacji wierzchołków i wyświetleniu obiektów na ekranie.
*   <span class="var">Wierzchołek (ang. _Vertex_)</span>: zbiór danych reprezentujących jeden punkt.
*   <span class="var">Znormalizowane Współrzędne Urządzenia (NDC)</span>: układ współrzędnych, w którym znajdują się wierzchołki, po etapie obcinania i dzieleniu perspektywicznym na współrzędnych obcinania. Wszystkie pozycje wierzchołków w NDC są między -1.0 a 1.0 nie zostaną odrzucone i będą widoczne.
*   <span class="var">Vertex Buffer Object</span>: obiekt bufora, który przydziela pamięć i przechowuje wszystkie dane wierzchołkowe do użytku przez kartę graficzną.
*   <span class="var">Vertex Array Object</span>: przechowuje informacje o stanie bufora i atrybutów wierzchołkowych.
*   <span class="var">Element Buffer Object</span>: obiekt bufora, który przechowuje indeksy dla renderowania indeksowego.
*   <span class="var">Uniform</span>: specjalny typ zmiennej globalnej GLSL (każdy shader w programie może uzyskać dostęp do tej zmiennej) i musi być ustawiony tylko raz.
*   <span class="var">Tekstura</span>: specjalny typ obrazu, nakładany na obiekty 3D, dający iluzję, że obiekt posiada bardzo dużo szczegółów.
*   <span class="var">Zawijanie Tekstury (ang. _Texture Wrapping_)</span>: definiuje tryb określający sposób, w jaki OpenGL powinien próbkować tekstury, gdy współrzędne tekstur są poza zakresem: (0, 1).
*   <span class="var">Filtrowanie Tekstury (ang. _Texture Filtering_)</span>: definiuje tryb określający sposób, w jaki OpenGL powinien próbkować tekstury, gdy do wyboru jest kilka tekseli (pikseli tekstur). Zwykle występuje, gdy tekstura zostaje powiększana.
*   <span class="var">Mipmapy</span>: przechowywane mniejsze wersje tekstury, z których wybierana jest mipmapa o odpowiednim rozmiarze w oparciu o odległość do obserwatora.
*   <span class="var">stb_image</span>: biblioteka ładująca obrazy.
*   <span class="var">Jednostki Teksturujące (ang. _Texture Units_)</span>: pozwalają na używanie wielu tekstur na pojedynczym obiekcie, łącząc różne obiekty tekstur z innymi jednostkami teksturującymi.
*   <span class="var">Wektor</span>: byt matematyczny, która definiuje kierunki i/lub pozycje w dowolnym wymiarze.
*   <span class="var">Macierz</span>: prostokątna tablica wyrażeń matematycznych.
*   <span class="var">GLM</span>: biblioteka matematyczna stworzona z myślą o OpenGL.
*   <span class="var">Przestrzeń lokalna (ang. _Local Space_)</span>: przestrzeń obiektu 3D. Wszystkie współrzędne są określane względem punktu początkowego obiektu.
*   <span class="var">Przestrzeń świata (ang. _World Space_)</span>: wszystkie współrzędne są określane względem globalnego punktu początkowego.
*   <span class="var">Przestrzeń widoku (ang. _View Space_)</span>: wszystkie współrzędne widziane z perspektywy kamery.
*   <span class="var">Przestrzeń obcinania (ang. _Clip Space_)</span>: wszystkie współrzędne widziane z perspektywy kamery, ale z zastosowaniem projekcji. Jest to przestrzeń, w której współrzędne wierzchołków powinny znajdować się po operacjach w Vertex Shader. OpenGL zajmuje się resztą (przycinanie/dzielenie perspektywiczne).
*   <span class="var">Przestrzeń Ekranu (ang. _Screen Space_)</span>: wszystkie współrzędne wyświetlane na ekranie. Współrzędne wahają się w przedziale od 0 do szerokości/wysokości ekranu.
*   <span class="var">LookAt</span>: specjalny typ macierzy widoku, który tworzy układ współrzędnych, w którym wszystkie współrzędne są obracane i przesuwane w taki sposób, że użytkownik patrzy na określony cel z danej pozycji.
*   <span class="var">Kąty Eulera</span>: zdefiniowane jako <span class="var">yaw</span>, <span class="var">pitch</span> i <span class="var">roll</span>, które pozwalają nam stworzyć dowolny wektor kierunku 3D z tych trzech wartości.