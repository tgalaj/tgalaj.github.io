---
layout: post
title: Tutorial 03 - Pierwszy trójkąt
subtitle: Kurs OpenGL dla początkujących
tags: [beginner-opengl-pl, tutorial]
---
## Wstęp

W tej części kursu narysujemy swój pierwszy trójkąt. Jeżeli byłeś/aś jedną z nielicznych osób, które spróbowały rozwiązać pracę domową z poprzedniej części i chcesz ją zweryfikować, to tutaj znajduje się odpowiedź:

<details class="panel panel-success">
  <summary markdown="span" class="panel-heading">
    Odpowiedzi do ćwiczeń
  </summary>

```cpp  
#include <GL/glew.h>
#include <GLFW/glfw3.h>

GLFWwindow* window;

int init(int width, int height)
{
    /* Initialize the library */
    if (!glfwInit())
        return -1;

    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(width, height, "Hello Triangle", NULL, NULL);

    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    /* Make the window's context current */
    glfwMakeContextCurrent(window);

    /* Initialize GLEW */
    if(glewInit() != GLEW_OK)
        return -1;

    return true;
}

void render(float tpf)
{
    //Render here
}

void update()
{
    float oldTime = 0.0f;
    float newTime = 0.0f;
    float gameTime = 0.0f;

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Update game time value */
        oldTime = newTime;
        newTime = (float)glfwGetTime();
        gameTime =  newTime - oldTime;

        /* Render here */
        render(gameTime );

        /* Swap front and back buffers */
        glfwSwapBuffers(window);

        /* Poll for and process events */
        glfwPollEvents();
    }
}

int main(void)
{
    if(!init(640, 480))
        return -1;

    update();
    glfwTerminate();

    return 0;
}
```

</details>

Kod z powyższej odpowiedzi będzie mi służył jako podstawa do tej części kursu. Tak więc jeżeli nie odrobiliście pracy domowej możecie śmiało ten kod skopiować do swojego projektu. No to zaczynamy! :)

## Kod aplikacji

Tradycyjnie na początku podam kod całej aplikacji, a następnie będę go analizował.

```cpp  
#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <glm/glm.hpp>

GLFWwindow* window;

/* Initialize vertices of our triangle */
glm::vec3 vertices[] = { glm::vec3( 0.0f,  1.0f, 0.0f),
                         glm::vec3( 1.0f, -1.0f, 0.0f),
                         glm::vec3(-1.0f, -1.0f, 0.0f)
                       };

/* Initialize Vertex Buffer Object */
GLuint VBO = NULL;

int init(int width, int height)
{
    /* Initialize the library */
    if (!glfwInit())
        return -1;

    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(width, height, "Hello Triangle", NULL, NULL);

    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    /* Make the window's context current */
    glfwMakeContextCurrent(window);

    /* Initialize GLEW */
    if(glewInit() != GLEW_OK)
        return -1;

    /* Set the viewport */
    glViewport(0, 0, width, height);

    return true;
}

int loadContent()
{
    /* Create new buffer to store our triangle's vertices */
    glGenBuffers(1, &VBO);

    /* Tell OpenGL to use this buffer and inform that this buffer will contain an array of vertices*/
    glBindBuffer(GL_ARRAY_BUFFER, VBO);

    /* Fill buffer with data */
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    /* Enable a generic vertex attribute array */
    glEnableVertexAttribArray(0);

    /* Tell OpenGL how to interpret the data in the buffer */
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);

    return true;
}

void render(float tpf)
{
    /* Draw our triangle */
    glDrawArrays(GL_TRIANGLES, 0, 3);
}

void update()
{
    float oldTime = 0.0f;
    float newTime = 0.0f;
    float gameTime = 0.0f;

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Update game time value */
	oldTime = newTime;
	newTime = (float)glfwGetTime();
	gameTime =  newTime - oldTime;

        /* Render here */
        render(gameTime );

        /* Swap front and back buffers */
        glfwSwapBuffers(window);

        /* Poll for and process events */
        glfwPollEvents();
    }
}

int main(void)
{
    if(!init(640, 480))
        return -1;

    if(!loadContent())
        return -1;

    update();

    glfwTerminate();
    return 0;
}
```

## Wyjaśnienie kodu

Na początek dołączamy nowy plik nagłówkowy z biblioteki GLM i inicjalizujemy tablicę wierzchołków trójkąta, który chcemy narysować.

```cpp  
#include <glm/glm.hpp>

/* Initialize vertices of our triangle */  
glm::vec3 vertices[] = { glm::vec3( 0.0f, 1.0f, 0.0f),  
                         glm::vec3( 1.0f, -1.0f, 0.0f),  
                         glm::vec3(-1.0f, -1.0f, 0.0f)  
};  
```

Tablica wierzchołków jest typu _vec3_, czyli jak można się domyślić jest to struktura opisująca wektor trójwymiarowy. Jak wiadomo trójkąt ma trzy wierzchołki, dlatego tutaj zapełniliśmy tą tablicę trzema wektorami, które reprezentują punkty trójkąta. Współrzędne ekranu dla osi X, Y i Z (okienka OpenGL) mieszczą się w zakresie [-1, 1]. Jest to dla nas ważna informacja, ponieważ w tej części kursu nie korzystamy jeszcze z programowalnego potoku renderingu – nie mamy kontroli nad przekształcaniem wierzchołków do układu współrzędnych ekranu. Korzystamy ze stałego potoku renderingu, który sam "wyświetli" wierzchołki trójkąta (dokładnie te same, które zostały podane w tablicy wierzchołków) i „pokoloruje” go na biało.

W funkcji _int init()_ doszła nowa instrukcja _glViewport(x, y, width, height)_. Tworzy ona prostokątną rzutnię, gdzie _x_ i _y_ to lewy dolny róg prostokąta o szerokości _width_ i wysokości _height_. Mówiąc krótko, służy ona do określenia rozmiarów prostokątnego "okna" przez które będziemy oglądać trójwymiarową scenę. To "okno" może mieć takie same wymiary jak okno naszej aplikacji, ale nie musi (wtedy scena będzie wyświetlana np. w lewej połowie okna przy parametrach _width = window_width/2_ i _height = window_height_).

Następnie dodajemy nową funkcję _int loadContent()_, która jest odpowiedzialna za załadowanie i przygotowanie danych do naszej aplikacji. Zwraca _true_ jeżeli wszystko poszło dobrze, albo _false_ w przeciwnym wypadku. Od razu umieszczamy też test w funkcji _main_, który sprawdza czy wszystko załadowało się poprawnie.

```cpp  
int loadContent()  
{  
}

int main(void)  
{  
    if(!init(640, 480))  
        return -1;

    if(!loadContent())  
        return -1;

    update((float)glfwGetTime());

    glfwTerminate();  
    return 0;  
}  
```

Teraz możemy przejść do przygotowania naszych danych do rysowania. Najpierw inicjalizujmy zmienną globalną, która jest „uchwytem” do bufora, w którym są przechowywane wierzchołki trójkąta.

```cpp  
/* Initialize Vertex Buffer Object */  
GLuint VBO = NULL;  
```

Typ GLuint jest typem OpenGL’a i można go porównać z typem unsigned int. Jak zobaczymy później, większość obiektów OpenGL’a jest właśnie tego typu.

Teraz możemy przejść do uzupełnienia funkcji _loadContent()_.

```cpp  
/* Create new buffer to store our triangle's vertices */  
glGenBuffers(1, &VBO);  
```

Poprzez tą instrukcję generujemy nowy bufor. Funkcja _glGenBuffers()_ przyjmuje dwa parametry: pierwszy mówi o tym ile obiektów/buforów chcemy utworzyć, a drugi jest adresem do tablicy GLuint, która przechowuje uchwyty, które ta funkcja generuje za nas (najlepiej się upewnić, że jest ona wystarczająco duża by pomieścić ilość obiektów, które chcemy wygenerować). Kolejne wywołania tej funkcji nie wygenerują tych samych obiektów dopóki wcześniej nie wywołamy funkcji _glDeleteBuffers()_.

```cpp  
/* Tell OpenGL to use this buffer and inform that this buffer will contain an array of vertices*/  
glBindBuffer(GL_ARRAY_BUFFER, VBO);  
```

W tym kroku mówimy OpenGL’owi, że chcemy działać na danym buforze i dla niego będziemy ustawiać pewne opcje. OpenGL działa jak maszyna stanów - jak raz coś włączymy, to jest to ustawione do momentu, w którym tego nie wyłączymy. Jak to się ma do działania na buforach? Jeżeli wywołujemy funkcję np. _glBindBuffer()_, to wszystko co tyczy się operacji na buforach będzie dokonywane na tym buforze, który wskazaliśmy w ww. funkcji. Pierwszy parametr to "typ", do którego przypisujemy bufor, który jest podany w drugim parametrze (jest więcej "typów" buforów, o których możemy poczytać w [dokumentacji](http://www.opengl.org/registry/doc/glspec44.core.pdf)).

```cpp  
/* Fill buffer with data */  
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  
```

Następnie "wkładamy" do bufora tablicę wierzchołków naszego trójkąta. Pierwszy parametr odpowiada "typowi" bufora, do którego wkładamy dane. Drugi parametr to wielkość w bajtach, jaką chcemy zarezerwować w naszym buforze na dane - korzystam tutaj z operatora _sizeof_, który zwraca wielkość naszej tablicy wierzchołków w bajtach. Trzeci parametr to wskaźnik do danych, które chcemy umieścić w buforze. Czwarty parametr określa to czy będziemy zmieniać dane w buforze i jak często będziemy z nich korzystać, i w jaki sposób (więcej o tych typach można znaleźć w [dokumentacji](http://www.opengl.org/registry/doc/glspec44.core.pdf)).

```cpp  
/* Enable a generic vertex attribute array */  
glEnableVertexAttribArray(0);  
```

Po wywołaniu tej funkcji, OpenGL będzie miał dostęp do tablicy wierzchołków o indeksie 0\. Jest to szczególnie ważne, by pamiętać o włączeniu dostępu do tej tablicy jeżeli przy rysowaniu korzystamy z funkcji takich jak: _glDrawArrays, glDrawElements, glDrawRangeElements, glMultiDrawElements, glMultiDrawArrays_.

```cpp  
/* Tell OpenGL how to interpret the data in the buffer */  
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);  
```

Ta funkcja określa jak dane zapisane w buforze mają być interpretowane. Pierwszy parametr określa gdzie zostały wysłane dane (do jakiej tablicy, o jakim indeksie). Drugi parametr mówi o tym z ilu komponentów składa się dany atrybut wierzchołka (w tym wypadku jego pozycja). Trzeci parametr to typ danych jakiego są poszczególne komponenty naszych wierzchołków. Czwarty parametr znormalizuje nam nasze wektory pozycji jeżeli podamy GL_TRUE, w przeciwnym wypadku nic z nimi nie zrobi. Piąty parametr to odległość między komponentami pozycji wierzchołków (w naszym przypadku kolejne komponenty są ściśle upakowane obok siebie - stąd wartość 0). Szósty parametr to odległość w buforze, od której zaczynają się nasze dane pozycji. Nasz bufor składa się z samych wartości pozycji dlatego ustawiamy 0. Ostatni parametr jest przydany wtedy, kiedy w buforze oprócz informacji o pozycji mamy też informacje o kolorze wierzchołka, współrzędnych teksturowania, itp.

```cpp  
/* Draw our triangle */  
glDrawArrays(GL_TRIANGLES, 0, 3);  
```

Teraz w funkcji _render()_ wywołujemy metodę, która narysuje nam trójkąt :) Pierwszy parametr to rodzaj prymitywów, jaki będzie renderowany i konstruowany przez kartę graficzną - wynika to z tego w jaki sposób zapisaliśmy wierzchołki w tablicy _vertices_ (więcej o tych typach w [dokumentacji](http://www.opengl.org/registry/doc/glspec44.core.pdf)). Drugi parametr to lokalizacja pierwszego komponentu pozycji wierzchołka w buforze. Trzeci parametr określa ile komponentów pozycji ma jeden wierzchołek.

To wszystko! Możemy teraz skompilować nasz kod i powinno okazać się okienko z dużym, białym trójkątem:

![Pierwszy trójkąt]({{ site.baseurl }}/img/beginner_opengl/tutorial-03-beginner-gl.png){: .center-image }

Poniżej umieszczone są ćwiczenia, które pomogą zrozumieć jak działają niektóre funkcje OpenGL'a. Zachęcam do własnego poeksperymentowania i komentowania. Oczywiście ćwiczenia są tylko i wyłącznie dla chętnych, w sekcji [_Kod źródłowy_](#source_code) znajduje się kod i solucja z tego tutoriala ;) W następnej części kursu zapoznamy się z programami cieniującymi (shaderami) i sprawimy, że nasz trójkąt zmieni kolor! :)

## Kod źródłowy {#source_code}
*   [Solucja VC++ 2010](https://drive.google.com/file/d/0B0j4jdWAANaoUFZfTFd2MkJOSjg/view?usp=sharing&resourcekey=0-xvVNDHD8RxO5L9Da_VyRTQ)

## Ćwiczenia

1. Co się stanie jeżeli wyjdziemy z wierzchołkami trójkąta poza zakres [-1; 1]?
2. Za pomocą funkcji _glViewport()_ sprawy, by trójkąt pokazywał się na środku ekranu, ale pomniejszony dwukrotnie.
3. Narysuj kwadrat (Podpowiedź: wykorzystaj dwa trójkąty).
