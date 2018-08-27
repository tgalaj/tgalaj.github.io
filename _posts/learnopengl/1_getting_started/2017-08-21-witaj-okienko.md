---
layout: post
title: Witaj Okienko
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: pierwsze-kroki
---

{% include learnopengl.md link="Getting-started/Hello-Window" %}

Sprawdźmy czy jesteśmy w stanie uruchomić GLFW. Na początek, utwórz nowy plik <span class="var">.cpp</span> i dołącz następujące pliki nagłówkowe na samej górze, wcześniej stworzonego pliku.  

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
```

{: .box-error }
Upewnij się, że najpierw dołączasz plik nagłówkowy GLAD, a później GLFW. W pliku nagłówkowym GLAD są zawarte instrukcje, które dołączają prawidłowe pliki nagłówkowe OpenGL (jak <span class="var">GL/gl.h</span>). Dlatego dołączanie GLAD przed dołączaniem innych plików nagłówkowych, które wymagają OpenGL załatwia sprawę.

Następnie utwórzmy funkcję <span class="fun">main</span>, gdzie zainicjalizujemy okno GLFW:

```cpp
int main()  
{  
    glfwInit();  
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);  
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);  
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);  
    //glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);

    return 0;  
}
```

W funkcji <span class="fun">main</span>, inicjalizujemy na początku GLFW za pomocą <span class="fun">glfwInit</span>, gdzie następnie konfigurujemy GLFW używając funkcji <span class="fun">glfwWindowHint</span>. Pierwszy argument <span class="fun">glfwWindowHint</span> mówi nam o tym, jaką opcję chcemy skonfigurować, gdzie mamy szeroki wybór tych opcji, każda z prefiksem <span class="var">GLFW_</span>. Drugi argument, to liczba całkowita, któa ustawia wartość dla danej opcji. Listę wszystkich dostępnych opcji, i ich możliwych wartości można znaleźć pod tym adresem [Dokumentacja GLFW](http://www.glfw.org/docs/latest/window.html#window_hints). Jeżeli teraz spróbujesz uruchomić aplikację i dostajesz dużo błędów typu _undefined reference_ to oznacza, że nie udało Ci się poprawnie zlinkować biblioteki GLFW.

Skoro w tym kursie skupiamy się na wersji OpenGL 3.3, to chcielibyśmy powiedzieć GLFW, że chcemy tej wersji właśnie używać. Dzięki temu, GLFW może poprawnie stworzyć kontekst OpenGL pod wybraną przez nas wersję. To gwarantuje, że jeżeli użytkownik nie posiada wsparcia dla konkretnej wersji OpenGL to GLFW wyrzuci błąd. Ustawiamy liczbę major (większą, przed kropką) i minor (mniejszą, po kropce) na wartość 3\. Oprócz tego mówimy GLFW, że chcemy jawnie używać profilu _core_. Jawne powiedzenie GLFW, że chcemy używać profilu core oznacza, że dostaniemy dostęp do jedynie małego podzbioru funkcjonalności OpenGL (bez wsparcia wstecznego dla starszych funkcjonalności, których nie chcemy dłużej używać). Zauważ, że jeżeli pracujesz na Mac OS X, to musisz również dodać <span class="fun">glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);</span> do inicjalizacji, by Twój kod zadziałał.

{: .box-note}
Upewnij się, że Twoja karta graficzna wspiera wersję OpenGL 3.3 lub wyższą. W przeciwnym razie Twoja aplikacja będzie się samoistnie wyłączać (ang. _crash_) lub będzie miała niezdefiniowane zachowanie. Żeby dowiedzieć się jaką wersję OpenGL wspiera Twój sprzęt, możesz wywołać z konsoli **glxinfo** (Linux) lub użyć dodatkowego programu jak [OpenGL Extension Viewer](http://download.cnet.com/OpenGL-Extensions-Viewer/3000-18487_4-34442.html) (Windows). Jeżeli wspierana wersja jest mniejsza, to sprawdź na stronie [GPU Info](http://opengl.gpuinfo.org/), czy Twoja karta w ogóle może wspierać tą wersję (no chyba, że karta jest bardzo stara) lub spróbuj uaktualnić sterowniki do karty graficznej.

Następnie musimy stworzyć obiekt okna. Obiekt okna przechowuje, wszystkie dane związane z oknem i jest często używany przez inne funkcje GLFW.

```cpp
GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", nullptr, nullptr);  

if (window == nullptr)  
{  
    std::cout << "Failed to create GLFW window" << std::endl;  
    glfwTerminate();  
    return -1;  
}  

glfwMakeContextCurrent(window);
```

Funkcja <span class="fun">glfwCreateWindow</span> wymaga podania w dwóch pierwszych argumentach żądanej szerokości i wysokości okna. Trzeci argument pozwala nam dodać tytuł dla okna; na razie nazwiemy je <span class="var">"LearnOpenGL"</span>, ale możesz wybrać nazwę jaka Tobie odpowiada. Możemy zignorować 2 pozostałe parametry. Funkcja zwraca obiekt typu <span class="var">GLFWwindow</span>, którego będziemy później potrzebować dla innych operacji GLFW. Po tym, mówimy GLFW żeby uczynił kontekst OpenGL dla naszego okna, głównym kontekstem dla aktualnego wątku.

## GLAD

W poprzednim tutorialu, nadmieniłem, że GLAD zajmuje się zdobyciem wszystkich wskaźników na funkcje OpenGL, dlatego chcemy go zainicjalizować zanim użyjemy jakiejkolwiek funkcji OpenGL:

```cpp
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))  
{  
    std::cout << "Failed to initialize GLAD" << std::endl;  
    return -1;  
}
```

Przekazujemy bibliotece GLAD funkcję, która pozwoli na załadowanie adresów funkcji OpenGL, co jest specyficznym zadaniem dla każdego systemu operacyjnego. GLFW daje nam funkcję <span class="fun">glfwGetProcAddress</span>, która definiuje poprawne zachowanie, zależnie od tego na jakim systemie operacyjnym kompilujemy nasz program.

## Viewport (obszar renderowania)

Zanim zaczniemy cokolwiek renderować, musimy zrobić ostatnią rzecz. Musimy powiedzieć OpenGL na jakim obszarze okna chcemy rysować, tak by OpenGL wiedział jak wyświetlać obraz w naszym oknie. Możemy to ustawić te _wymiary_ za pomocą funkcji <span class="fun">glViewport</span>:

```cpp
glViewport(0, 0, 800, 600);
```

Pierwsze dwa parametry funkcji <span class="fun">glViewport</span> ustawiają lokalizację lewego dolnego rogu obszaru renderowania. Trzeci i czwarty argument ustawiają szerokość i wysokość obszaru renderowania wyrażoną w pikselach.

Możemy oczywiście ustawić wymiary obszaru renderowania na mniejsze niż dla okna GLFW; wtedy OpenGL będzie rysował na mniejszym obszarze niż obszar całego okna GLFW i wtedy poza viewportem OpenGL możemy wyświetlać inne rzeczy (np. kontrolki GUI).

{: .box-note }
Za kurtyną, OpenGL używa danych przekazanych do funkcji <span class="fun">glViewport</span> żeby przetransformować współrzędne 2D do przestrzeni współrzędnych Twojego ekranu (okna). Na przykład, przetwarzany punkt <span class="var">(-0.5, 0.5)</span> może być (jako jego ostatnia transformacja) zmapowany do punktu <span class="var">(200, 450)</span> na Twoim ekranie (oknie). Zauważ, że przetwarzane współrzędne w OpenGL są pomiędzy -1 i 1, dlatego w efekcie przedział (-1 do 1) jest mapowany do (0, 800) i (0, 600).

Jednakże, moment, w którym użytkownik zmienia rozmiar okna, obszar renderowania powinien być również zmieniony. Żeby to zrobić możemy zarejestrować wywołanie zwrotne (ang. _callback_) dla okna i będzie ono wywoływane za każdym razem kiedy okno będzie zmieniało rozmiar. Funkcja wywołania zwrotnego dla zmieniania rozmiaru okna ma następujący prototyp:

```cpp
void framebuffer_size_callback(GLFWwindow* window, int width, int height);
```

Ta funkcja w pierwszym parametrze przyjmuje obiekt <span class="fun">GLFWwindow</span> i dwie liczby całkowite, oznaczające nowy rozmiar okna. Kiedy okno zmienia swój rozmiar, GLFW wywołuje tą funkcję i wypełnia ją odpowiednimi argumentami, byś mógł je dalej sam przetworzyć.

```cpp
void framebuffer_size_callback(GLFWwindow* window, int width, int height)  
{  
    glViewport(0, 0, width, height);  
} 
```

Musimy powiedzieć GLFW, że powyższą funkcję chcemy wywoływać za każdym razem kiedy okno zmienia rozmiar. Dlatego musimy ją zarejestrować:

```cpp
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
```

Kiedy okno jest wyświetlone po raz pierwszy, zostaje wywołana funkcja <span class="fun">framebuffer_size_callback</span> z wynikowym rozmiarem okna. Dla ekranów retina, <span class="var">width</span> i <span class="var">height</span> będą trochę wyższe niż oryginalne wartości wejściowe.

Istnieje wiele wywołań zwrotnych za pomocą, których możemy zarejestrować nasze własne funkcje. Na przykład, możemy stworzyć funkcję wywołania zwrotnego, by przetwarzać zmiany danych wejściowych joystick'a, przetwarzać wiadomości o błędach itp. Wywołania zwrotne rejestrujemy wtedy, gdy obiekt okna został poprawnie stworzony, ale przed uruchomieniem głównej pętli gry (ang. _game loop_).

## Rozgrzewamy silniki

Nie chcemy, aby nasza aplikacja rysowała jeden obraz na ekranie, a zaraz po tym kończyła swoje działanie. Chcemy, by nasza aplikacja rysowała serię obrazków oraz odbierała dane wejściowe (np. z klawiatury, myszy), dopóki użytkownik nie zdecyduje, że sam chce zamknąć program. Z tego powodu, stworzymy pętlę _while_, którą nazwiemy pętlą gry (ang. _game loop_), która będzie działać w kółko dopóki nie powiemy GLFW, by z niej wyjść. Poniższy kod przedstawia bardzo prostą pętlę gry:

```cpp
while(!glfwWindowShouldClose(window))  
{  
    glfwPollEvents();  
    glfwSwapBuffers(window);  
}
```

Funkcja <span class="fun">glfwWindowShouldClose</span> sprawdza na początku każdej iteracji, czy GLFW był poinstruowany, by zamknąć okno, jeżeli tak, funkcja zwraca wartość _true_ i pętla gry kończy swoje działanie, po czym możemy zamknąć aplikację.  
Funkcja <span class="fun">glfwPollEvents</span> sprawdza czy zostały wywołane jakieś zdarzenia (ang. _events_), jak naciśnięcie przycisku na klawiaturze lub poruszenie myszą, i wywołuje odpowiednią funkcję (którą możemy ustawić przez wywołanie zwrotne). Funkcje, które zajmują się przetwarzaniem zdarzeń, zwykle wywołujemy na początku iteracji.  
Funkcja <span class="fun">glfwSwapBuffers</span> zamienia bufor koloru (duży bufor, który przechowuje wartości koloru dla każdego piksela w oknie GLFW), który był używany do narysowania ramki w tej iteracji z buforem zawierającym ramkę z poprzedniej iteracji i wyświetla wynik na ekranie.

{: .box-note }
**Podwójne buforowanie (ang.** _double buffer_)  
Kiedy aplikacja rysuje do pojedynczego bufora, obraz wynikowy może być wyświetlany w taki sposób, że będzie występował efekt migania (ang. _flickering_). Dzieje się to z powodu tego, że obraz wynikowy nie jest rysowany natychmiastowo, ale rysowany piksel po pikselu i zazwyczaj od górnego lewego rogu do prawego dolnego. Ponieważ te obrazy nie są też wyświetlane natychmiastowo na ekranie użytkownika, ale raczej wyświetlane po kawałku, to obraz wynikowy może zawierać artefakty (błędy w obrazie). By tego uniknąć, aplikacje okienkowe stosują podwójne buforowanie dla renderowania. **Przedni** bufor (ang. _front buffer_) zawiera finalny obraz wynikowy, który jest wyświetlany na ekranie, podczas, gdy całe renderowanie odbywa się przy użyciu **tylnego** bufora (ang. _back buffer_). Jak tylko wszystkie polecenia renderowania zostaną zakończone, **zamieniamy** (ang. _swap_) bufor tylni z buforem przednim, dzięki czemu obraz jest natychmiast wyświetlany na ekranie, usuwając wszystkie wcześniej wspomniane artefakty.

## Ostatnia rzecz

Jak tylko wyjdziemy z pętli gry, chcielibyśmy zadbać o poprawne czyszczenie/usuwanie zasobów, które wcześniej stworzyliśmy. Możemy to zrobić za pomocą funkcji <span class="fun">glfwTerminate</span>, którą wywołujemy na końcu funkcji <span class="fun">main</span>.

```cpp
glfwTerminate();  
return 0;
```

Powyższy kod usunie nam wszystkie zasoby związane z GLFW i poprawnie zamknie aplikację. Spróbuj teraz skompilować aplikację i jeżeli wszystko poszło dobrze powinieneś zobaczyć rezultat podobny do poniższego obrazka:

![Obraz wyjściowy okna GLFW jako najbardziej podstawowy przykład]({{ site.baseurl }}/img/learnopengl/hellowindow.png){: .center-image}

Jeżeli jest to bardzo monotonny i nudny czarny obraz, to wszystko zrobiłeś dobrze! Jeżeli nie dostałeś takiego wyniku, albo jesteś zmieszany tym, jak to wszystko jest powiązane, sprawdź pełny kod źródłowy [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/1.1.hello_window/hello_window.cpp).

Jeżeli masz problemy związane ze skompilowaniem aplikacji, upewnij się najpierw czy masz dobrze ustawione opcje linkera i czy poprawnie dodałeś odpowiednie ścieżki/lokalizacje do swojego IDE (zostało to wytłumaczone w poprzedniej części kursu). Dodatkowo upewnij się, że Twój kod jest poprawny; możesz to łatwo zweryfikować poprzez porównanie go z wcześniej udostępnionym kodem źródłowym. Jeżeli nadal masz problemy, napisz komentarz niżej opisując swój problem. Wtedy ja albo ktoś ze społeczności postara się Tobie pomóc.

## Wejście

Chcemy mieć w naszej aplikacji również możliwość przechwytywania zdarzeń z klawiatury bądź innego, podobnego urządzenia. GLFW udostępnia do tego celu kilka funkcji. Będziemy używać funkcji <span class="fun">glfwGetKey</span>, która w parametrze przyjmuje obiekt okna GLFW oraz klawisz. Funkcja zwraca <span class="var">true</span> jeśli dany klawisz jest w danej chwili wciśnięty. Stwórzmy funkcję <span class="fun">processInput</span>, by mieć wszystkie operacje wejścia zorganizowane w jednym miejscu:

```cpp
void processInput(GLFWwindow *window)  
{  
    if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)  
        glfwSetWindowShouldClose(window, true);  
}
```

W powyższym kodzie sprawdzamy, czy użytkownik wcisnął przycisk Escape (jeżeli nie został wciśnięty, <span class="fun">glfwGetKey</span> zwraca wartość <span class="var">GLFW_RELEASE</span>). Jeżeli użytkownik wcisnął klawisz Escape, mówimy GLFW by zakończył działanie poprzez ustawienie jego właściwości <span class="var">WindowShouldClose</span> na wartość <span class="var">true</span> używając funkcji <span class="fun">glfwSetwindowShouldClose</span>. Następnym razem gdy główna pętla będzie sprawdzać warunek, to on nie przejdzie i aplikacja zostanie zamknięta.

Następnie wywołujemy funkcję <span class="fun">processInput</span> przy każdej iteracji głównej pętli.

```cpp
while (!glfwWindowShouldClose(window))  
{  
    processInput(window);

    glfwSwapBuffers(window);  
    glfwPollEvents();  
} 
```

Daje nam to możliwość sprawdzenia w łatwy sposób reagowania na wciśnięcie danego klawisza w każdej nowej ramce.

## Renderowanie

Skoro chcemy wywoływać wszystkie operacje związane z renderowaniem w każdej iteracji, to musimy umieścić je w głównej pętli gry. Będzie to wyglądało mniej więcej tak:

```cpp
// pętla aplikacji  
while(!glfwWindowShouldClose(window))  
{  
    // wejście  
    processInput(window);

    // komendy renderowania  
    ...

    // sprawdź i wywołaj zdarzenia oraz zamień bufory koloru  
    glfwPollEvents();  
    glfwSwapBuffers(window);  
}
```

Żeby sprawdzić czy wszystko działa jak należy, wyczyśćmy ekran za pomocą koloru, który sami wybierzemy. Na początku każdej iteracji pętli, chcemy zawsze czyścić ekran. W przeciwnym razie będziemy obserwować wyniki z poprzedniej iteracji (może to być celowy efekt, ale zazwyczaj nie jest). Możemy wyczyścić bufor koloru naszego ekranu używając funkcji <span class="fun">glClear</span>, gdzie przekazujemy bufory, w formie bitów, by wybrać, który bufor chcemy wyczyścić. Możliwe wartości bitowe jakie możemy tu przekazać to <span class="var">GL_COLOR_BUFFER_BIT</span>, <span class="var">GL_DEPTH_BUFFER_BIT</span> i <span class="var">GL_STENCIL_BUFFER_BIT</span>. Na chwilę obecną, interesują nas tylko wartości koloru dlatego wyczyścimy bufor koloru.

```cpp
glClearColor(0.2f, 0.3f, 0.3f, 1.0f);  
glClear(GL_COLOR_BUFFER_BIT);
```

Zauważ, że ustawiamy również kolor czyszczący za pomocą funkcji <span class="fun">glClearColor</span>, którym będziemy czyścić ekran. Za każdym razem kiedy wywołamy funkcję <span class="fun">glClear</span> z zamiarem czyszczenia bufora koloru, to cały ten bufor będzie wyczyszczony za pomocą wartości zdefiniowanych za pomocą funkcji <span class="fun">glClearColor</span>. Te operacje powinny dać nam kolor zbliżony do ciemno-zielono-niebieskiego koloru.

{: .box-note }
Jak możesz sobie przypominać z części kursu pt. _OpenGL_, funkcja <span class="fun">glClearColor</span> jest funkcją _ustawiającą stan_ (ang. _state-setting_), natomiast funkcja <span class="fun">glClear</span> jest funkcją _używającą stanu_ (ang. _state-using_) - używa obecnego stanu by pobrać wartość koloru czyszczącego.

![Image of GLFW's window creation with glClearColor defined]({{ site.baseurl }}/img/learnopengl/hellowindow2.png){: .center-image}

Pełny kod źródłowy aplikacji możesz znaleźć [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/1.2.hello_window_clear/hello_window_clear.cpp).

Mamy na razie wszystko czego nam potrzeba, by wypełnić pętlę gry mnogością różnych wywołań renderujących, ale to zostawmy na kolejną część kursu.