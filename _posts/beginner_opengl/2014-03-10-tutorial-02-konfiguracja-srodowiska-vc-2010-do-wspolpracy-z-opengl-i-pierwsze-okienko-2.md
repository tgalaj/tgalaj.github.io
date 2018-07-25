---
layout: post
title: Tutorial 02 - Konfiguracja środowiska VC++ 2010 do współpracy z OpenGL i pierwsze okienko
subtitle: Kurs OpenGL dla początkujących
tags: [beginner-opengl-pl, tutorial]
---

## Konfiguracja środowiska

Zanim zaczniemy tworzyć nasze niesamowite aplikacje graficzne w technologii OpenGL musimy najpierw przystosować nasze środowisko do współpracy z ww. technologią. Na początek instalujemy środowisko pracy - Microsoft Visual C++ 2010 Express (tutaj przypominam [link](http://www.microsoft.com/visualstudio/plk#downloads+d-2010-express)).

Ponadto ściągamy z Internetu trzy biblioteki:  
**GLFW** - [http://www.glfw.org/](http://www.glfw.org/download.html "http://www.glfw.org/") - jest to darmowa, przenośna biblioteka dzięki, której z łatwością utworzymy okienko OpenGL i będziemy mogli obsługiwać zdarzenia I/O (klawiatura, mysz, etc.). Ściągamy 32-bitowe binarki dla Windows'a (32-bit Windows binaries).  
**GLM** - [http://glm.g-truc.net/](http://glm.g-truc.net/ "http://glm.g-truc.net/") - bardzo dobra biblioteka matematyczna, która załatwi za nas mnożenie macierzy, tworzenie macierzy projekcji/widoku. Ściągamy najnowszą wersję.  
**GLEW** - [http://glew.sourceforge.net/]( http://glew.sourceforge.net/ " http://glew.sourceforge.net/") -biblioteka opakowująca najnowszą bibliotekę OpenGL. Ściągamy 32-bitowe binarki dla Windows'a (Windows 32-bit and 64-bit).

Rozpakowujemy je do osobnych folderów. Następnie z folderu głównego biblioteki GLFW kopiujemy pliki/foldery do odpowiednich lokalizacji:

*   Folder **include/GLFW** do lokalizacji** **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\include****
*   Pliki _*.lib_ z folderu **lib-msvc100** do lokalizacji **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\lib**
*   Plik _*.dll_ z folderu **lib-msvc100** do lokalizacji **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\bin**

Teraz biblioteka GLEW:

*   Folder **include/GL** kopiujemy do lokalizacji **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\include**
*   Pliki z folderu **lib** kopiujemy do lokalizacji **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\lib**
*   Pliki z folderu **bin** kopiujemy do lokalizacji **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\bin**

Z biblioteką GLM jest już prościej:

*   Kopiujemy folder **glm** do lokalizacji **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\include**

Teraz wystarczy powiedzieć naszemu środowisku, by korzystał z tych bibliotek. W tym celu otwieramy Visual Studio 2010 C++ i wybieramy: **File -> New -> Project**. W nowym okienku wybieramy **Win32 Console Application**, nadajemy nazwę nowemu projektowi i wybieramy lokalizację, w której ma być on utworzony i klikamy **Ok**.

{% include lightbox src="img/beginner_opengl/11.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

W kolejnym oknie klikamy **Next**. Pojawia się teraz okienko z ustawieniami naszej aplikacji. Zaznaczamy **Console Appliaction** i **Empty Project**, i klikamy **Finish**.

{% include lightbox src="img/beginner_opengl/21.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

Teraz mamy stworzony swój projekt. Zanim zaczniemy pracę musimy do naszego projektu podpiąć biblioteki, które wcześniej instalowaliśmy. W tym celu klikamy prawym przyciskiem myszy na nazwie naszego projektu w Solution Explorerze i wybieramy **Properties**.

{% include lightbox src="img/beginner_opengl/31.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

W nowym oknie ustawiamy pole Configuration na **All Configurations**.

{% include lightbox src="img/beginner_opengl/41.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

Teraz przechodzimy do **Configuration Properties -> Linker -> Input** i klikamy w pole obok **Additional Dependencies** i wbieramy **Edit…** W nowym okienku wpisujemy (wszystko oddzielamy Enterami): **opengl32.lib, glu32.lib, glfw3.lib, glfw3dll.lib, glew32.lib** i klikamy **Ok**.

{% include lightbox src="img/beginner_opengl/51.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

## Kod aplikacji

Teraz możemy zabrać się do właściwej pracy i uruchomić nasze pierwsze okienko OpenGL’a! :D W tym celu prawym przyciskiem myszy klikamy na Source File i wybieramy **Add->New Item…**

{% include lightbox src="img/beginner_opengl/61.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

W nowym oknie wybieramy **C++ File (.cpp)** i wpisujemy nazwę pliku np.: main i klikamy **Add**.

{% include lightbox src="img/beginner_opengl/71.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

Właśnie utworzyliśmy nowy plik, do którego kopiujemy poniższy kod (wytłumaczenie kodu za chwilę):

{% highlight cpp linenos %}
/**  
** Listing taken from: http://www.glfw.org/documentation.html  
**/ 
#include <GL/glew.h>  
#include <GLFW/glfw3.h>

int main(void)  
{  
    GLFWwindow* window; 

    /* Initialize the library */
    if (!glfwInit())  
        return -1; 
        
    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL); 

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
        
    /* Loop until the user closes the window */  
    while (!glfwWindowShouldClose(window))  
    {  
        /* Render here */ 

        /* Swap front and back buffers */  
        glfwSwapBuffers(window); 
        
        /* Poll for and process events */  
        glfwPollEvents();  
    } 
    
    glfwTerminate();  
    return 0;  
}  
{% endhighlight %}

Teraz możemy skompilować i uruchomić naszą aplikację naciskając klawisz F5 lub klikając zieloną strzałkę. Aplikacja powinna bez problemu się skompilować i powinno pokazać się okienko z czarnym tłem. Jeżeli tak się stało to gratuluję! Właśnie stworzyłeś okienko OpenGL’a! :D

## Wyjaśnienie kodu

W pierwszej linijce za pomocą dyrektywy preprocesora dołączamy plik nagłówkowy, dzięki któremu będzie można używać najnowszej biblioteki OpenGL, a w drugiej dołączamy plik, który pozwoli nam na stworzenie nowego okna. Ten plik (GLFW/glfw3.h) definiuje wszystkie stałe, typy i funkcje używane przez GLFW oraz dołącza wszystkie pliki, które są nam potrzebne do tworzenia aplikacji OpenGL na Windowsie i nie tylko, czyli nie musimy martwić się o dołączanie plików windows.h, GL/gl.h itp. Jedyne co nie jest dołączane to plik glu.h. Jeżeli chcemy by był dołączony to przed tą dyrektywą umieszczamy linijkę:

```cpp  
#define GLFW_INCLUDE_GLU  
#include <GLFW/glfw3.h>  
```

```cpp  
GLFWwindow* window; //Tworzymy „uchwyt” do naszego okienka.

/* Initialize the library */  
if (!glfwInit())  
    return -1;  
```

Tutaj, „auto-magicznie”, odbywa się inicjalizacja biblioteki GLFW. Zwraca zero jeżeli się nie udało lub inną wartość niż zero jeżeli się powiodło.

```cpp  
/* Create a windowed mode window and its OpenGL context */  
window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);

if (!window)  
{  
    glfwTerminate();  
    return -1;  
}  
```

Tworzymy nowe okno o szerokości 640px i wysokości 480px z tytułem okna „Hello World”. Jeżeli ta operacja się nie powiedzie, to zostanie zwrócona wartość NULL, dlatego sprawdzamy czy się powiodło. Jeżeli się nie powiodło to za pomocą funkcji _**glfwTerminate()**_, która niszczy wszystkie stworzone okienka i zwalnia zasoby zajęte przez GLFW. Powinna być używana jeżeli chcemy wyłączyć naszą aplikację.

```cpp  
/* Make the window's context current */  
glfwMakeContextCurrent(window);  
```

Zanim będziemy mogli używać funkcji OpenGL, musimy stworzyć kontekst dla okna poprzez wywołanie ww. funkcji.

```cpp  
/* Initialize GLEW */  
if(glewInit() != GLEW_OK)  
    return -1;  
```  
Teraz inicjalizujemy bibliotekę GLEW i sprawdzamy, czy przebiegła ona pomyślnie. Od teraz możemy używać funkcji OpenGL :)

```cpp  
/* Loop until the user closes the window */  
while (!glfwWindowShouldClose(window))  
{  
    /* Render here */

    /* Swap front and back buffers */  
    glfwSwapBuffers(window);

    /* Poll for and process events */  
    glfwPollEvents();  
}  
```

W tej pętli zaczyna się dziać to co nas będzie interesować najbardziej. Jest to tzw. główna pętla aplikacji, w której co klatkę będą rysowane w oknie nasze modele 3D i będą aktualizowane wszystkie zmienne, które tego wymagają.

W pętli sprawdzamy warunek czy okno naszej aplikacji ma być zamknięte. Funkcja **_glfwWindowShouldClose()_** zwraca 1 jeżeli użytkownik naciśnie krzyżyk do zamykania okna, lub naciśnie kombinację klawiszy Alt+F4\. Dalej w tej pętli wywołujemy wszystkie operacje dotyczące renderowania/rysowania (których teraz nie ma, dlatego mamy czarne tło).

Okienko GLFW zawsze używa podwójnych buforów, co zapobiega miganiu ekranu przy renderowaniu kolejnych klatek. Funkcja **_glfwSwapBuffers()_** zamienia ze sobą przedni i tylni bufor.

Nasze okno musi móc odbierać zdarzenia chociażby takie jak zamknięcie okna. W tym celu używamy metody **_glfwPollEvents()_**, która przetwarza zdarzenia, które zostały zgłoszone do kolejki zdarzeń i odpowiada na nie w trybie natychmiastowym.

Więcej na temat GLFW można dowiedzieć się w dokumentacji (link na samym dole strony).

To na tyle. Dla chętnych jeszcze zostawiam ćwiczenie do samodzielnego rozwiązania, które przyda się w następnym wydaniu tego kursu (tam też znajdzie się odpowiedź do tego ćwiczenia). W kolejnej części kursu narysujemy pierwszy trójkąt!

## Kod źródłowy
*   [Solucja VC++ 2010](https://drive.google.com/file/d/0B0j4jdWAANaoQnhDUEV0dXJlM1U/view?usp=sharing)

## Ćwiczenie

Podziel program na funkcje:

*   main()
*   init(int width, int height)
*   update(float tpf)
*   render(float tpf)

Do funkcji update można przekazać wartość funkcji _**double glfwGetTime()**_, która zwraca ilość sekund jaka upłynęła od czasu zainicjalizowania GLFW. Dla ułatwienia pole GLFWwindow* window; można uczynić globalnym (wiem, wiem – nie powinno się tak robić :-P).

## Dodatkowe źródła
1. Dokumentacja GLFW w wersji angielskiej, [http://www.glfw.org/docs/3.0/](http://www.glfw.org/docs/3.0/)