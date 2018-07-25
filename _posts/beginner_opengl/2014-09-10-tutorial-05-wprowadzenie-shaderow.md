---
layout: post
title: Tutorial 05 - Wprowadzenie do shader'ów
subtitle: Kurs OpenGL dla początkujących
tags: [beginner-opengl-pl, tutorial]
---
## Wstęp

W tej części kursu OpenGL nauczymy się jak napisać prosty shader, który narysuje nam trójkąt w takim kolorze, w jakim tylko będziemy chcieli! Od tej części kursu nie będę umieszczał całych listingów kodu (z powodu na zbyt dużą objętość) tylko będę od razu przechodził do części, w której będę analizował kod. Tam też zamieszczę najważniejsze linijki lub fragmenty kodu, które zmieniły się od poprzedniej części. Kod do tej części można pobrać [tutaj](https://drive.google.com/file/d/0B0j4jdWAANaoVzkyUnpTZWk1eGc/view?usp=sharing). Dodatkowo również od tej części kursu wszystko co bedziemy robić w OpenGL będzie korzystało z shader'ów jako z nowoczesnej metody programowania grafiki 3D. No to zaczynamy!

## Wyjaśnienie kodu aplikacji

Od poprzedniej części kursu (Tutorial 03) w kodzie zmieniło się kilka rzeczy:

*   dodana została funkcja _loadShader(std::string)_,
*   dodana została funkcja _loadAndCompileShaderFromFile(GLint, std::string, GLuint&)_,
*   została zmieniona funkcja _init()_ oraz _render()_
*   został dodany do projektu shader (właściwe to dwa shader'y - vertex i fragment), które zostały umieszczone w folderze Shaders.

Zacznijmy od początku (wyjaśnieniem shader'ów zajmę się na końcu). Funkcji _loadShader(std::string)_ nie będę tłumaczyć, ponieważ jest ona związana ściśle z językiem C++ i zakładam, że wszyscy korzystający z tego kursu znają ten język w takim stopniu, by rozumieć wszystko to, co w tej funkcji jest zawarte. Służy ona po prostu do wczytania kodu shader'a do pamięci komputera i do przechowania go w zmiennej typu _std::string_.

Zanim przejdziemy do tłumaczenia funkcji _loadAndCompileShaderFromFile(GLint, std::string, GLuint&)_, przejdźmy na chwilę do funkcji _init()_, w której zaszły pewne zmiany dotyczące shader'ów. Pierwszą nowością jest poniższa linijka:

```cpp  
/* Set clear color */  
glClearColor(1.0f, 1.0f, 1.0f, 1.0f);  
```

Zapisuje ona w maszynie stanów OpenGL'a, kolor, którym ma być czyszczony bufor koloru. Bufor koloru musi być czyszczony przy każdym renderowaniu ramki dlatego w funkcji _render()_ wywoływana jest instrukcja:

```cpp  
/* Clear the color buffer */  
glClear(GL_COLOR_BUFFER_BIT);  
```

Funkcja _void glClearColor(GLfloat red, GLfloat green, GLfloat blue, GLfloat alpha)_ przyjmuje cztery parametry typu GLfloat i są to kolejne składowe RGBA - kolory czerowny, zielony, niebieski, alfa (odpowiada za przezroczystość). Te parametry przyjmują wartości z przedziału [0, 1] i kiedy podamy wartość z poza tego przedziału zostanie ona "przycięta", by wpasować się w ten przedział (jeżeli podamy wartość 10.0f, zostanie ona zamieniona na wartość 1.0f, a gdy podamy wartość -8.0f, to zostanie ona zamieniona na wartość 0.0f).

Dzięki funkcji _void glClearColor(GLfloat red, GLfloat green, GLfloat blue, GLfloat alpha)_ możemy kontrolować kolor "tła" naszej wirtualnej sceny. Dzieje się tak dlatego, że najpierw czyszczony jest bufor koloru na domyślny kolor (zdefiniowany za pomocą ww. funkcji), a na to jest nakładana nasza geometria, która może mieć zupełnie inny kolor/kolory.

Teraz możemy przejść do części związanej tylko i wyłącznie z shader'ami.

```cpp  
/* Shader init */  
GLuint programHandle = glCreateProgram();

if(programHandle == 0)  
{  
    fprintf(stderr, "Error creating program object.\n");  
}  
```

Na początku tworzymy uchwyt do "programu" za pomocą funkcji _glCreateProgram()_, do którego "podepniemy" shader'y. Jeżeli zwróci wartość zero, to znaczy, że coś się popsuło podczas tworzenia tego obiektu - nie zobaczymy nic na ekranie, ponieważ nasze shader'y nie będą działać. Kiedy jest to wartość różna od zera, to wszystko jest w porządku.

```cpp  
/* Shader load from file and compile */  
loadAndCompileShaderFromFile(GL_VERTEX_SHADER, "Shaders/basic.vert", programHandle);  
loadAndCompileShaderFromFile(GL_FRAGMENT_SHADER, "Shaders/basic.frag", programHandle);  
```

Dwa razy wywołana zostaje funkcja _loadAndCompileShaderFromFile(GLint, std::string, GLuint&)_. Raz dla vertex shader'a i drugi raz dla fragment shader'a. Z poprzedniej lekcji wiemy, że jest to niezbędne minimum, jeżeli chcemy korzystać z możliwości oferowanych przez programy cieniujące. Ta funkcja korzysta z funkcji _loadShader(std::string)_ do wczytania kodu shader'a z pliku (można kod trzymać również w tablicy _char*_, ale nie jest to wygodne jeżeli mamy do czynienia z shader'ami, które mają dużo linijek kodu oraz kiedy chcemy taki shader debugować).

```cpp  
GLuint shaderObject = glCreateShader(shaderType);

if(shaderObject == 0)  
{  
    fprintf(stderr, "Error creating %s.\n", fileName.c_str());  
    return;  
}  
```

Pierwszym zadaniem funkcji _loadAndCompileShaderFromFile(...)_ jest stworzenie obiektu shader'a za pomocą funkcji _glCreateShader(GLuint type)_. Argumentem tej funkcji jest typ shader'a jaki mamy zamiar kompilować. Może to być: **GL_VERTEX_SHADER**, **GL_FRAGMENT_SHADER**, **GL_GEOMETRY_SHADER**, **GL_TESS_EVALUATION_SHADER** lub **GL_TESS_CONTROL_SHADER**, **GL_COMPUTE_SHADER**. Obiekt shader'a przechowujemy w lokalnej zmiennej _shaderObject_, by sprawdzić czy udało się taki obiekt stworzyć - wszystko będzie w porządku jeżeli będzie to wartość różna od zera, w przeciwnym wypadku coś poszło nie tak i zostanie wyświetlony odpowiedni komuniakt w konsoli.

```cpp  
std::string shaderCodeString = loadShader(fileName);

if(shaderCodeString.empty())  
{  
    printf("Shader code is empty! Shader name %s\n", fileName.c_str());  
    return;  
}

const char * shaderCode = shaderCodeString.c_str();  
const GLint codeSize = shaderCodeString.size();

glShaderSource(shaderObject, 1, &shaderCode, &codeSize);  
```

Następnie wczytywany jest kod shader'a, ze ścieżki podanej w argumencie _fileName_ i zapisany do zmiennej _shaderCode_. Teraz trzeba ten kod załadować do obiektu shader'a. Do tego celu służy funkcja _glShaderSource(...)_. Pierwszym argumentem jest obiekt shader'a, który chcemy stworzyć. Drugim parametrem jest liczba kodów, które chcemy skompilować (kompilujemy jeden kod na raz - stąd liczba 1). Trzeci argument to tablica łańcuchów znaków, która zawiera kody shader'ów. Nasza zawiera tylko jeden kod. Czwarty parametr jest to tablica, która zawiera długości łańcuchów znaków z trzeciego argumentu. Teraz kod shader'a został skopiowany do pamięci wewnętrznej OpenGL'a.

```cpp  
glCompileShader(shaderObject);

GLint result;  
glGetShaderiv(shaderObject, GL_COMPILE_STATUS, &result);

if(result == GL_FALSE)  
{  
    fprintf(stderr, "%s compilation failed!\n", fileName.c_str());

    GLint logLen;  
    glGetShaderiv(shaderObject, GL_INFO_LOG_LENGTH, &logLen);

    if(logLen > 0)  
    {  
        char * log = (char *)malloc(logLen);

        GLsizei written;  
        glGetShaderInfoLog(shaderObject, logLen, &written, log);

        fprintf(stderr, "Shader log: \n%s", log);  
        free(log);  
    }

    return;  
}  
```

Teraz możemy skompilować kod źródłowy naszego shader'a. W tym celu wywołujemy po prostu funkcję _glCompileShader(GLuint)_ przekazując jako parametr obiekt shader'a, który ma zostać skompilowany. Proces kompilacji może się nie powieść dlatego kolejnym krokiem jest sprawdzenie poprawności kompilacji i w razie czego wyświetleniu log'a, który poinformuje nas, w którym miejscu w shader'ze popełniliśmy błąd. Do tego służy funkcja _glGetShaderiv(...)_, która służy do pobierania różnych informacji o shader'ze. Póki co interesuje nas status kompilacji dlatego podajemy jako drugi argument wartość _GL_COMPILE_STATUS_. Pierwszym obiektem jest oczywiście obiekt shader'a, dla którego chcemy uzyskać daną informację, a trzecim parametrem jest zmienna, do której ma być zapisany status kompilacji (będzie w niej wartość _GL_TRUE_ lub _GL_FALSE_ zaleznie od tego czy proces się powiódł czy nie). Jeżeli kompilacja się nie powiodła to wyświetlamy stosowną informację w konsoli i następnie wyświetlamy log informujący nas gdzie dokładnie popełniliśmy błąd.

```cpp  
glAttachShader(programHandle, shaderObject);  
glDeleteShader(shaderObject);  
```

Kolejnym krokiem jest podpięcie obiektu shader'a do "programu", w którym będą przechowywane shader'y, które mają ze sobą współpracować. Kiedy podpięliśmy obiekt shader'a do programu, możemy go spokojnie skasować, by zwolnić pamięć.

```cpp  
/* Link */  
glLinkProgram(programHandle);  
```

Wracamy teraz do funkcji _init()_. Po udanej kompilacji shader'ów musimy poddać je procesowi linkowania (dokładniej linkujemy program shader'a). Ta operacja jest ważna z tego względu, że tworzone są połączenia między shader'ami - wyjście jednego shader'a jest łączone z odpowiednim wejściem drugiego, by możliwa była komunikacja i przesyłanie danych między nimi. Dodatkowo tworzone są połączenia między odpowiednimi wejściamy/wyjściami shader'a z odpowiednimi lokacjami w środowisku OpenGL. Tak samo jak przy kompilacji, linkowanie może się nie powieść, dlatego sprawdzamy w sposób praktycznie identyczny status linkowania programu, z tym, że wykorzystywana jest funkcja _glGetProgramiv(...)_.

```cpp  
/* Apply shader */  
glUseProgram(programHandle);  
```

Kiedy w naszym programie shader'y zostały poprawnie zlinkowane, możemy powiedzieć OpenGL'owi, że chcemy korzystać z danego zestawu shader'ów (programu), by rysował i kolorował obiekty tak jak zostało to zdefiniowane w kodzie shader'ów.

## Wyjaśnienie kodu shader'ów

Na początek przyjrzymy się vertex shader'owi:

```glsl  
#version 400

layout (location = 0) in vec3 vertexPosition;

void main()  
{  
    gl_Position = vec4(vertexPosition.x, vertexPosition.y, vertexPosition.z, 1.0f);

    // Alternatively we can write:  
    // gl_Position = vec4(vertexPosition, 1.0f);  
    // and the effect will be exactly the same  
}  
```

Z poprzedniej części wiemy, że vertex shader przetwarza jeden wierzchołek na raz. Język GLSL (OpenGL Shading Language) jest zbliżony do języka C++ dlatego nie jest on taki straszny do przyswojenia i nauczenia się.

```glsl  
#version 440  
```

Jest to dyrektywa preprocesora, która mówi o tym, z jakiej wersji GLSL'a mamy zamiar korzystać (lub jest napisany dany shader). W tym wypadku jest to wersja 4.4 z lipca 2013.

```glsl  
layout (location = 0) in vec3 vertexPosition;  
```

Za pomocą kwalifikatora wejścia **layout** definiujemy pod jakim indeksem, shader ma "szukać" wektora wierzchołków, pod który wcześniej wysłaliśmy dane na temat pozycji wierzchołków. W Tutorialu 03 "włączaliśmy" tą lokalizację za pomocą funkcji _glEnableVertexAttribArray(0)_, a za pomocą funkcji _glVertexAttribPointer()_ mówiliśmy OpenGL'owi, pod jaki indeks ma wysłać dane wierzchołków. Takich atrybutów może być kilka i mogą to być: wartości koloru danego wierzchołka, koordynaty tekstury dla danego wierzchołka lub wektor normalny dla danego wierzchołka.

```glsl  
void main()  
{  
    gl_Position = vec4(vertexPosition.x, vertexPosition.y, vertexPosition.z, 1.0f);

    // Alternatively we can write:  
    // gl_Position = vec4(vertexPosition, 1.0f);  
    // and the effect will be exactly the same  
}  
```

W każdym shader'ze musi być zdefiniowana funkcja _main()_, która jest główną funkcją każdego programu cieniującego (oprócz niej mogą być zdefiniowane inne funkcje pomocnicze). Do wbudowanej zmiennej wyjściowej vertex shader'a _gl_Position_ przypisujemy odpowiednie wartości z atrybutu wejściowego _vertexPosition_, by trójkąt miał takie współrzędne pozycji jakie zdefiniowaliśmy w programie. Zmienna _gl_Position_ jest strukturą typu _vec4_, która reprezentuje wektor 4-wymiarowy. Zauważmy, żeby odwoływać się do kolejnych elementów z wektora 3-wymiarowego _vertexPosition_ możemy korzystać z operatora "." (kropka), tak jak w klasach lub strukturach C++.

By ułatwić sobie życie i skrócić męki pisania kodu, możemy skorzystać z dogodności GLSL'a i skorzystać z konstruktora _vec4(vec3, float)_.

Zauważmy, że pod wartość _w_ (ostatnia wartość w konstruktorze _vec4_) jest podawana wartość 1.0f.

Warto zapamiętać, że:

*   Jeżeli w == 1, to wektor v(x, y, z, 1) jest **pozycją** w przestrzeni.
*   Jeżeli w == 0, to wektor v(x, y, z, 0) jest **kierunkiem** w przestrzeni.

Jest to ważne w przypadku translacji. W przestrzeni możemy przesunąć punkt, ale czy możliwe jest przesunięcie kierunku? Raczej nie :)

```glsl  
#version 400

out vec4 fragColor;

void main()  
{  
    fragColor = vec4(0.0f, 0.0f, 0.0f, 1.0f);  
}  
```

W przypadku fragment shader'a wszystko wygląda podobnie jak przy vertex shader'ze z tą różnicą, że definiujemy własną zmienną wyjściową typu _vec4_, dla koloru fragmentu (stąd kwalifikator _out_). Do zmiennej _fragColor_ przypisujemy kolor RGBA (czerwony, zielony, niebieski, alfa), którego kolejne komponenty przyjmują wartości z zakresu [0.0f, 1.0f]. W tym wypadku trójkąt będzie pokolorowany na czarno. Gdybyśmy chcieli zmienić kolor na czerwony wystarczy pierwszy komponent zamienić na wartość 1.0f.

Efekt całości powinien być następujący:

![Czarny trójkąt]({{ site.baseurl }}/img/beginner_opengl/tutorial-05-beginner-gl.png){: .center-image }

## Zakończenie

To już wszystko na dzisiaj. W razie gdyby było coś nie jasne z dzisiejszej lekcji, proszę pisać do mnie na maila, bądź w komentarzach poniżej. W kolejnej lekcji przyjrzymy się zagadnieniu interpolacji kolorów, którą OpenGL robi automatycznie :) .

## Kod źródłowy {#source_code}
*   [Solucja VC++ 2010](https://drive.google.com/file/d/0B0j4jdWAANaoVzkyUnpTZWk1eGc/view?usp=sharing)

## Ćwiczenia

1.  Zmień kolor tła na niebieski.
2.  Zmień kolor trójkąta/kwadratu na zielony.

## Dodatkowe źródła

1. Więcej informacji w [dokumentacji OpenGL](http://www.opengl.org/registry/doc/glspec45.core.pdf)
2. Więcej informacji w [dokumentacji GLSL](http://www.opengl.org/registry/doc/GLSLangSpec.4.50.pdf)
