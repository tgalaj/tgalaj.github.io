---
layout: post
title: Zaawansowane zarządzanie danymi
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: zaawansowany-opengl
---

{% include learnopengl.md link="Advanced-OpenGL/Advanced-Data" %}

Od dłuższego czasu używamy buforów w OpenGL do przechowywania danych. Istnieją bardziej interesujące sposoby manipulowania buforami, a także inne interesujące metody przekazywania dużych ilości danych do shaderów za pośrednictwem tekstur. W tym samouczku omówimy bardziej interesujące funkcje operujące na buforach i sposób wykorzystania obiektów tekstur do przechowywania dużych ilości danych (część samouczka dot. tekstur nie została jeszcze napisana).

Bufor OpenGL jest tylko obiektem, który zarządza częścią pamięci i niczym więcej. Ustawiamy znaczenie bufora po powiązaniu go z określonym  <def>przeznaczeniem bufora</def> (ang. *buffer target*). Bufor to tylko bufor tablicy wierzchołków, gdy wiążemy go z <var>GL_ARRAY_BUFFER</var>, ale możemy go też łatwo połączyć z <var>GL_ELEMENT_ARRAY_BUFFER</var>. OpenGL wewnętrznie przechowuje bufor na każde jego przeznaczenie i bazując na tym przeznaczeniu przetwarza bufory w inny sposoby.

Do tej pory wypełniamy pamięć zarządzaną przez obiekty bufora, wywołując <fun>glBufferData</fun>, która przydziela kawałek pamięci i dodaje dane do tej pamięci. Gdybyśmy przekazali `NULL` jako argument danych, funkcja przydzieliłaby tylko pamięć i jej nie wypełniała. Jest to użyteczne, jeśli najpierw chcemy _zarezerwować_ określoną ilość pamięci, a następnie powrócić do tego bufora, aby wypełnić go kawałek po kawałku.

Zamiast wypełniać cały bufor jednym wywołaniem funkcji, możemy również wypełnić określone obszary bufora, wywołując <fun>glBufferSubData</fun>. Ta funkcja przyjmuje jako argument przeznaczenie bufora, przesunięcie, rozmiar danych i rzeczywiste dane. Nowością w tej funkcji jest to, że możemy teraz podać przesunięcie, które określa _od_ jakiego miejsca chcemy wypełnić bufor. To pozwala nam wstawiać/aktualizować tylko niektóre części pamięci bufora. Zwróć uwagę, że bufor powinien mieć wystarczającą ilość pamięci. Wynika z tego, że wywołanie <fun>glBufferData</fun> jest konieczne przed wywołaniem <fun>glBufferSubData</fun> na buforze.

```cpp
    glBufferSubData(GL_ARRAY_BUFFER, 24, sizeof(data), &data); // Zakres: [24, 24 + sizeof(data)]
```

Jeszcze inną metodą zapisywania danych do bufora jest pobranie wskaźnika do pamięci bufora i bezpośrednie skopiowanie danych do bufora. wywołanie <fun>glMapBuffer</fun> OpenGL zwraca wskaźnik do pamięci aktualnie powiązanego bufora:

```cpp
    float data[] = {
      0.5f, 1.0f, -0.35f
      ...
    };
    glBindBuffer(GL_ARRAY_BUFFER, buffer);
    // pobierz wskaźnik
    void *ptr = glMapBuffer(GL_ARRAY_BUFFER, GL_WRITE_ONLY);
    // teraz skopiuj dane do pamięci
    memcpy(ptr, data, sizeof(data));
    // pamiętaj, aby powiedzieć OpenGL, że skończyliśmy z operacjami na wskaźniku
    glUnmapBuffer(GL_ARRAY_BUFFER);
```

Mówiąc OpenGL, skończyliśmy operację na wskaźniku za pomocą <fun>glUnmapBuffer</fun> OpenGL wie, że ten wskaźnik będzie już niepotrzebny. Po odpięciu wskaźnika, staje się on niepoprawny.

Korzystanie z <fun>glMapBuffer</fun> jest przydatne do bezpośredniego mapowania danych bufora, bez wcześniejszego przechowywania ich w pamięci tymczasowej. Pomyśl o bezpośrednim odczytywaniu danych z pliku i kopiowaniu ich do pamięci bufora.

## Grupowanie atrybutów wierzchołków

Używając <fun>glVertexAttribPointer</fun> byliśmy w stanie określić układ atrybutów w buforze wierzchołków. W buforze wierzchołków <def>przeplataliśmy</def> atrybuty; to jest, umieściliśmy współrzędne pozycji, normalne i/lub tekstury obok siebie dla każdego wierzchołka. Teraz, gdy wiemy nieco więcej o buforach, możemy przyjąć inne podejście.

Moglibyśmy również zgrupować wszystkie dane wektora w duże porcje danych według typu atrybutu zamiast ich przeplatania. Zamiast przeplatanego układu `123123123123` stosujemy podejście grupowania `111122223333`.

Podczas ładowania danych wierzchołków z pliku zazwyczaj pobierasz tablicę pozycji, wektorów normalnych i/lub tablicę współrzędnych tekstury. Może to kosztować trochę wysiłku, aby połączyć te tablice w jedną dużą tablicę przeplatanych danych. Podejście grupowania jest wtedy prostszym rozwiązaniem, które możemy łatwo wdrożyć za pomocą <fun>glBufferSubData</fun>:

```cpp
    float positions[] = { ... };
    float normals[] = { ... };
    float tex[] = { ... };
    // wypełnij bufor
    glBufferSubData(GL_ARRAY_BUFFER, 0, sizeof(positions), &positions);
    glBufferSubData(GL_ARRAY_BUFFER, sizeof(positions), sizeof(normals), &normals);
    glBufferSubData(GL_ARRAY_BUFFER, sizeof(positions) + sizeof(normals), sizeof(tex), &tex);
```

W ten sposób możemy bezpośrednio przenieść tablice atrybutów wierzchołków jako całość do bufora bez konieczności ich wcześniejszego przetwarzania. Mogliśmy również połączyć je w jedną dużą tablicę i natychmiast wypełnić bufor za pomocą <fun>glBufferData</fun>, ale funkcja <fun>glBufferSubData</fun> idealnie nadaje się do takich zadań.

Będziemy również musieli zaktualizować wskaźniki atrybutów wierzchołków, aby odzwierciedlić te zmiany:

```cpp
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), 0);  
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)(sizeof(positions)));  
    glVertexAttribPointer(
      2, 2, GL_FLOAT, GL_FALSE, 2 * sizeof(float), (void*)(sizeof(positions) + sizeof(normals)));  
```

Zauważ, że parametr `stride` jest równy rozmiarowi atrybutu wierzchołka, ponieważ następny wektor atrybutów wierzchołka można znaleźć bezpośrednio po jego 3 (lub 2) komponencie.

To daje nam jeszcze inne podejście do ustawiania i określania atrybutów wierzchołków. Korzystanie z obu metod nie przynosi natychmiastowej korzyści OpenGL, jest to w większości bardziej zorganizowany sposób ustawiania atrybutów wierzchołków. Podejście, które wybierzesz, jest oparte wyłącznie na twoich preferencjach i rodzaju aplikacji.

## Kopiowanie buforów

Po wypełnieniu buforów danymi możesz chcieć udostępnić te dane innym buforom lub skopiować zawartość bufora do innego bufora. Funkcja <fun>glCopyBufferSubData</fun> umożliwia nam kopiowanie danych z jednego bufora do drugiego. Prototyp funkcji jest następujący:

```cpp
    void glCopyBufferSubData(GLenum readtarget, GLenum writetarget, GLintptr readoffset,
                             GLintptr writeoffset, GLsizeiptr size);
```

Parametry `readtarget` i `writetarget` przewidują podanie przeznaczenia buforów, obu buforów uczestniczących w wymianie danych. Możemy na przykład skopiować z bufora <var>VERTEX_ARRAY_BUFFER</var> do bufora <var>VERTEX_ELEMENT_ARRAY_BUFFER</var>, określając odpowiednio te przeznaczenia buforów jako bufory do odczytu (readtarget) i do zapisu (writetarget). Bufory obecnie powiązane z tymi typami przeznaczenia zostaną użyte do operacji kopiowania.

Ale co by było, gdybyśmy chcieli odczytywać i zapisywać dane do dwóch różnych buforów, które są jednocześnie buforami tablicy wierzchołków? Nie możemy powiązać dwóch buforów w tym samym czasie z tym samym przeznaczeniem bufora. Z tego powodu, OpenGL daje nam dwa dodatkowe przeznaczenia buforów o nazwach <var>GL_COPY_READ_BUFFER</var> i <var>GL_COPY_WRITE_BUFFER</var>. Następnie wiążemy wybrane przez nas bufory do nowych typów przeznaczenia i ustawiamy je jako argumenty `readtarget` i `writetarget`.

Funkcja <fun>glCopyBufferSubData</fun> następnie odczytuje dane o rozmiarze `size` z danego przesunięcia `readoffset` i zapisuje je w buforze `writetarget` rozpoczynając od pozycji `writeoffset`. Przykład kopiowania zawartości dwóch buforów tablic wierzchołków pokazano poniżej:

```cpp
    float vertexData[] = { ... };
    glBindBuffer(GL_COPY_READ_BUFFER, vbo1);
    glBindBuffer(GL_COPY_WRITE_BUFFER, vbo2);
    glCopyBufferSubData(GL_COPY_READ_BUFFER, GL_COPY_WRITE_BUFFER, 0, 0, sizeof(vertexData));
```

Mogliśmy to również zrobić, poprzez powiązanie tylko bufora `writetarget` z jednym z nowych typów przeznaczenia buforów:

```cpp
    float vertexData[] = { ... };
    glBindBuffer(GL_ARRAY_BUFFER, vbo1);
    glBindBuffer(GL_COPY_WRITE_BUFFER, vbo2);
    glCopyBufferSubData(GL_ARRAY_BUFFER, GL_COPY_WRITE_BUFFER, 0, 0, sizeof(vertexData));  
```

Dzięki dodatkowej wiedzy na temat sposobu manipulowania danymi buforów możemy już z nich korzystać w bardziej interesujący sposób. Im dalej wejdziesz w OpenGL, tym bardziej przydatne stają się te nowe metody zarządzania danymi. W następnym samouczku omówimy <def>obiekty buforów uniformów</def> (ang. *uniform buffer objects*) i zrobimy dobry użytek z <fun>glBufferSubData</fun>.