---
layout: post
title: Tworzenie okna
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: pierwsze-kroki
---

{% include learnopengl.md link="Getting-started/Creating-a-window" %}

Pierwszą rzeczą jaką musimy zrobić, zanim zaczniemy tworzyć zapierające dech w piersi efekty graficzne, to musimy stworzyć kontekst OpenGL oraz okienko aplikacji, do którego będziemy rysować. Jednakże, te operacje są inne dla każdego systemu operacyjnego, a OpenGL specjalnie nie daje żadnego API do zrobienia tych rzeczy. Dlatego sami musimy stworzyć okienko, zdefiniować kontekst i przechtywyać sygnały z urządzeń wejściowych takich jak np. kalwiatura i mysz.  

Na szczęście, istnieje kilka bibliotek, które dostarczają już odpowiednich funkcjonalności, których nam tutaj potrzeba. Niektóre z tych bibliotek są specjalnie pisane pod zastosowanie OpenGL. Dzięki nim możemy zaoszczędzić sobie sporo pracy związanej z specyficznymi operacjami dla danego systemu i dostarczają nam gotowe okienko z kontekstem OpenGL, do którego możemy rysować. Niektóre spośród najbardziej popularnych bibliotek to GLUT, SDL, SFML i GLFW. Na potrzeby tego kursu będziemy używać **GLFW**.

## GLFW

GLFW jest biblioteką napisaną w języku C, ukierunkowaną na OpenGL i dostarczającą tylko niezbędne funkcje jakie są potrzebne podczas renderowania rzeczy na ekranie. Pozwala nam ona na stworzenie kontekstu OpenGL, zdefiniowanie parametrów okna oraz przechytywanie sygnałów z urządzeń wejściowych.

<img src="{{ site.baseurl }}/img/learnopengl/glfw.png" alt="GLFW logo" class="right">

Celem tego i następnego szkolenia jest zainstalowanie i uruchomienie GLFW w naszym projekcie, upewniając się, że na pewno dobrze tworzy ona nam kontekst OpenGL i poprawnie wyświetla okno, do którego będziemy mogli renderować. Ten kurs przeprowadzi Cię krok po kroku przez etapy pobierania, budowania i linkowania biblioteki GLFW. W tym celu będziemy używać środowiska zintegrowanego (IDE) Microsoft Studio 2015 (cały proces jest ten sam nawet w najnowszych wersjach Visual Studio). Jeżeli nie używasz Visual Studio (lub używasz starszej wersji) nie martw się, proces instalacji będzie przebiegał podobnie na większości innych IDE.

## Budowanie GLFW

GLFW może zostać pobrany z ich własnej [strony internetowej](http://www.glfw.org/download.html). GLFW posiada pre-kompilowane pliki binarne i pliki nagłówkowe dla Visual Studio 2013/2015, ale dla kompletności tego kursu, zbudujemy GLFW sami z kodów źródłowych. Pobierzmy zatem _Source package_.

{: .box-error }
Jeżeli masz zamiar użyć pre-kompilowanych plików binarnych, upewnij się, że pobierasz wersję 32 bitową, a nie 64 bitową (no chyba, że wiesz dokładnie co robisz). Wersja 64 bitowa (podobno) sprawiała dużo kłopotów dla większości czytelników.

Kiedy pobrałeś już pliki źródłowe, wypakuj je i otwórz zawartość folderu. Jesteśmy zainteresowani tylko kilkoma rzeczami:

*   Wynikowa biblioteka po kompilacji.
*   Folder **include**.

"Własnoręczne" zbudowanie biblioteki z kodów źródłowych gwarantuje, że wynikowa biblioteka będzie idealnie dopasowna do naszego CPU/OS, czego wygodne, pre-kompilowane wersje nie mogą zagwarantować (czasami pre-kompilowane biblioteki nie są dostępne dla danego systemu). Jednym problemem związanym z udostępnianiem światu kodu źródłowego jest to, że nie każdy używa tego samego IDE do tworzenia własnych aplikacji, co oznacza, że dostarczone pliki projektu/solucji mogą nie być kompatybilne z IDE używanymi przez inne osoby. Dlatego inni ludzie muszą budować sobie sami własne projekty/solucje z dostępnych plików .c/.cpp i .h/.hpp, co jest żmudnym zajęciem. Dlatego, z tego powodu powstało takie narzędzie jak <span class="var">CMake</span>.

### CMake

CMake jest narzędziem, które pozwala na wygenerowanie gotowego projektu/solucji dla wybranego przez użytkownika IDE (np. Visual Studio, Code::Blocks, Eclipse) z zestawu plików źródłowych używając pre-definiowanych skryptów CMake. Pozwala to nam na wygenerowanie projektu Visual Studio 2012 z paczki plików źródłowych GLFW, których możemy użyć do skompilowania tejże biblioteki. Na wstępie musimy sciągnąć i zainstalować CMake, który możesz znaleźć na ich [stronie](http://www.cmake.org/cmake/resources/software.html). Osobiście (Joey de Vries) użyłem instalatora Win32\.

Jak tylko CMake zostanie zainstalowany, możesz wybrać czy chcesz uruchomić CMake z lini poleceń czy za pomocą ich GUI. Jako, że nie chcemy komplikować sobie rzeczy, użyjemy GUI. CMake wymaga folderów z kodem źródłowym i folderu dla plików wynikowych - binarek (ang. _destination folder_). Jako folder z kodem źródłowym wybierzemy główny folder, sciągniętych wcześniej plików źródłowych GLFW. Natomiast folder dla plików wynikowych musimy stworzyć sami - najlepiej pod nazwą _build_ i następnie w GUI wybierzmy ten folder.

<img src="{{ site.baseurl }}/img/learnopengl/cmake.png" alt="Logo CMake" class="center-image">

Jak już ustwaliśmy foldery z plikami źródłowymi i dla plików wynikowych, kliknij przycisk <span class="var">Configure</span>, co sprawi, że CMake odczyta ustawione parametry oraz pliki źródłowe. Następnie musimy wybrać generator dla projektu. Skoro używamy Visual Studio 2015 wybierzmy opcję <span class="var">Visual Studio 14</span> (Visual Studio 2015 jest również nazywany jako Visual Studio 2014). Następnie CMake wyświetli możliwe opcje budowania, do skonfigurowania wynikowej biblioteki. Możemy je zostawić tak jak są i kliknąć ponownie przycisk <span class="var">Configure</span> do zapisania ustawień. Jak wszystkie opcje zostały już ustawione, klikamy na przycisk Generate i wynikowe pliki projektu zostaną wygenerowane i zapisane we wcześniej stworzonym folderze <span class="var">build</span>.

### Kompilacja

W folderze <span class="var">build</span>, znajdź plik o nazwie <span class="var">GLFW.sln</span> i otwórz go za pomocą Visual Studio 2015\. Skoro CMake sam wygenerował nam projekt, który zawiera prawidłową konfigurację, możemy od razu klkiknąć przycisk <span class="var">Build Solution</span>. Wynikowa, skompilowana biblioteka może zostać znaleziona w folderze <span class="var">src/Debug</span> o nazwie <span class="var">glfw3.lib</span> (zauważ, że używamy wersji 3).

Jak tylko biblioteka została wygenerowana, musimy się upewnić, że nasze IDE wie, gdzie znajduje się biblioteka oraz pliki nagłówkowe. Są na to dwa sposoby:

1.  Znajdujemy foldery <span class="var">/lib</span> i <span class="var">/include</span> naszego IDE/Kompilatora. Do folderu <span class="var">/include</span> IDE wrzucamy zawartość folderu <span class="var">/include</span> GLFW i to samo robimy dla pliku <span class="var">glfw3.lib</span> z tym, że wrzucamy go do folderu <span class="var">/lib</span> naszego IDE. To rozwiązanie działa, ale nie jest rekomendowanym podejściem. Ciężko jest śledzić zmiany w folderach <span class="var">lib</span> oraz <span class="var">include</span>, a nowe instalacje naszego IDE/Kompilatora mogą skutkować usunięciem tych plików.
2.  Zalecanym sposobem jest stworzenie nowego zbioru folderów, w wybranej przez nas lokalizacji, które będą zawierały pliki nagłówkowe oraz pliki bibliotek, do których możemy się odnieść z poziomu IDE/Kompilatora. Osobiście (Joey de Vries) używam folderów <span class="var">Libs</span> oraz <span class="var">Include</span>, gdzie przechowuję wszystkie biblioteki i ich pliki nagłówkowe na potrzeby projektów OpenGL. Dzięki temu, wszystkie moje dodatkowe biblioteki są zorganizowane w jednej lokacji (która może być współdzielona przez wiele komputerów). Jedynym wymaganiem jest to, że za każdym razem kiedy tworzymy nowy projekt, musimy powiedzieć naszemu IDE gdzie ma szukać tych folderów.

Jak tylko przeniesiesz odpowiednie pliki do wygodnej Tobie lokalizacji, możemy zabrać się za stworzenie pierwszego projektu OpenGL wykorzystującego GLFW!

## Nasz pierwszy projekt

Na początku, otwórzmy Visual Studio i stwórzmy nowy projekt. Wybierz Visual C++, jeżeli zostanie Ci podanych kilka opcji i zaznacz opcję <span class="var">Empty Project</span> (nie zapomnij nadać swojemu projektowi odpowiedniej nazwy). Od teraz mamy nasz obszar roboczy do stworzenia naszej pierwszej aplikacji OpenGL.

## Linkowanie

W celu użycia GLFW w naszym projekcie, musimy podłączyć tą bibliotekę do naszego projektu. Może to zostać zrobione poprzez powiedzenie Visual Studio, że chcemy używać <span class="var">glfw3.lib</span> w ustawieniach linkera, ale jest mały problem - nasze IDE nie wie gdzie może ten plik znaleźć, ponieważ przenieśliśmy ten plik do innej lokalizacji. Musimy zatem dodać te lokalizacje do naszego projektu.

Możemy dodać te foldery (gdzie VS powinien szukać bibliotek/plików nagłówkowych) poprzez otworzenie ustawień projektu (prawy przycisk myszy na nazwie projektu w Solution Explorer) i wybranie <span class="var">VC++ Directories</span>, jak zostało to przedstawione na poniższym obrazku:

[![Image of Visual Studio's VC++ Directories configuration]({{ site.baseurl }}/img/learnopengl/vc_directories.png)](http://www.rtrclass.type.pl/wp-content/uploads/2017/03/vc_directories.png){: .center-image}

Możesz tutaj dodać swoje własne lokalizacje, żeby projekt wiedział gdzie ma szukać odpowiednich plików. Może to zostać zrobione manualnie poprzez wpisanie odpowiedniej ścieżki lub poprzez kliknięcie na tekście odpowiedniej lokalizacji i wybranie opcji <span class="var">\<Edit..\></span>. Powinieneś zobaczyć takie oto okno dla np. <span class="var">Include Directories</span>:

[![Image of Visual Studio's Include Directories configuration]({{ site.baseurl }}/img/learnopengl/include_directories.png)](http://www.rtrclass.type.pl/wp-content/uploads/2017/03/include_directories.png){: .center-image}

Możesz tutaj dodać tyle dodatkowych ścieżek ile tylko chcesz, i od tego momentu IDE będzie przeszukiwało te lokalizacje w poszukiwaniu plików nagłówkowych. Jak tylko Twój folder <span class="var">Include</span>, który zawiera folder GLFW, zostanie dołączony do projektu, będziesz mógł odwoływać się z poziomu kodu do jego plików nagłówkowych poprzez dyrektywę <span class="var">#include <GLFW/..></span>. To samo tyczy się folderów z plikami bibliotek.

Jak już VS jest w stanie odnaleźć wszystkie potrzebne pliki, możemy w końcu przejść do podłączenia GLFW do naszego projektu. W tym celu w ustawieniach projektu wybieramy zakładkę <span class="var">Linker</span> i wybieramy <span class="var">input</span>:

[![Image of Visual Studio's link configuration]({{ site.baseurl }}/img/learnopengl/linker_input.png)](http://www.rtrclass.type.pl/wp-content/uploads/2017/03/linker_input.png){: .center-image}

Żeby podłączyć bibliotekę, musisz podać jej nazwę linkerowi. Skoro biblioteka ma nazwę <span class="var">glfw3.lib</span>, dodajemy tą nazwę do pola <span class="var">Additional Dependencies</span> (albo poprzez manualne wpisanie tekstu, albo poprzez opcję <span class="var">\<Edit..\></span>). Teraz GLFW będzie linkowane kiedy będziemy kompilować nasz projekt. Oprócz GLFW powinieneś również dodać do linkowania bibliotekę OpenGL - może to się różnić w zależności od systemu operacyjnego.

### Biblioteka OpenGL na Windows

Jeżeli używasz systemu Windows, biblioteka <span class="var">opengl32.lib</span> jest zawarta w Microsoft SDK, które jest domyślnie instalowane kiedy instalujesz Visual Studio. Skoro w tym kursie używamy kompilatora VS i systemu Windows, dodajemy <span class="var">opengl32.lib</span> do ustawień linkera.

### Biblioteka OpenGL na Linux

Na systemach Linux musisz dołączyć plik biblioteki <span class="var">libGL.so</span> poprzez dodanie <span class="var">-lGL</span> do ustawień linkera. Jeżeli nie możesz znaleźć pożądanej biblioteki, musisz zapewne zainstalować deweloperskie paczki Mesa, NVidia lub AMD. Nie będę zagłębiał się w detale ponieważ wszystko to zależy od konkretnej platformy (dodatkowo nie jestem ekspertem od systemów Linux).

Jak już dodałeś obie biblioteki do ustawień linkera (GLFW oraz OpenGL) możemy dołączyć odpowiednie pliki nagłówkowe poprzez wpisanie w kodzie następującej linijki:

```cpp
#include <glfw\glfw3.h>
```

{: .box-note }
Użytkownikom systemów Linux, którzy kompilują za pomocą GCC, następująca komenda może być pomocna w celu skompilowania projektu <span class="var">-lglfw3 -lGL -lX11 -lpthread -lXrandr -lXi</span>. Nie prawidłowe zlinkowanie tych bibliotek spowoduje wiele niezdefiniowanych błędów odwołań.

To wszystko kończy instalację i konfigurację GLFW.

## GLAD

Jeszcze do końca nie skończyliśmy konfiguracji projektu, ponieważ została nam do zrobienia jeszcze jedna rzecz. Skoro OpenGL jest standardem/specyfikacją, to do obowiązków dostawy sterowników należy implementacja tej specyfikacji w sterowniku karty graficznej, która daną wersję OpenGL obsługuje. Ponieważ istnieje wiele wersji sterowników OpenGL, to lokalizacja jego funkcji nie jest znana w czasie kompilacji i musi być uzyskana w czasie uruchomienia aplikacji. Jest to z kolei zadaniem programisty, by uzyskać lokalizacje tych funkcji, których potrzebuje i musi przechować do nich wskaźniki na późniejsze ich zastosowanie. Pobieranie tych lokalizacji jest [zależne od systemu operacyjnego](https://www.khronos.org/opengl/wiki/Load_OpenGL_Functions) i w systemie Windows wygląda to mniej więcej tak:

```cpp
// Zdefiniuj prototyp funkcji  
typedef void (*GL_GENBUFFERS) (GLsizei, GLuint*);  
// Znajdź funkcję i przypisz do niej wskaźnik do funkcji  
GL_GENBUFFERS glGenBuffers = (GL_GENBUFFERS)wglGetProcAddress("glGenBuffers");  
// Funkcja może być teraz wywołana normalnie  
GLuint buffer;  
glGenBuffers(1, &buffer);
```

Jak możesz zauważyć, powyższy kod wygląda dosyć skomplikowanie i jest to bardzo żmudny proces, by uzyskać te lokalizację dla każdej funkcji, której możesz potrzebować, a która nie jest jeszcze zadeklarowana. Na szczęście, istnieją biblioteki, które zajmują się tym zagadnieniem, a popularną i aktualną z nich jest biblioteka **GLAD**.

### Instalacja GLAD

GLAD jest biblioteką [Open Source](https://github.com/Dav1dde/glad), która zajmuje się tą żmudną pracą, o której było mówione powyżej. GLAD posiada trochę inny rodzaj konfiguracji niż większość znanych bibliotek Open Source. Glad używa [web service'u](http://glad.dav1d.de/), za pomocą, którego możemy wskazać, której wersji OpenGL chcemy używać, by GLAD mógł załadować wszystkie funkcje zdefiniowane dla tej wybranej wersji OpenGL.

Przejdź do [web service'u](http://glad.dav1d.de/), upewnij się, że jest wybrany język C/C++. W sekcji API wybierz przynajmniej wersję OpenGL 3.3 (która jest wersją, której będziemy tutaj używać; jak wybierzesz wyższą wersję to też nic się nie stanie). Dodatkowo, upewnij się, że profil jest ustawiony na _Core_ oraz, że pole _Generate a loader_ jest zaznaczone. Zignoruj rozszerzenia (na razie) i naciśnij przycisk _Generate_ do stworzenia plików wynikowych biblioteki.

GLAD powinien dostarczyć Ci plik zip, który zawiera dwa foldery <span class="var">include</span> i jeden plik <span class="var">glad.c</span>. Skopiuj te dwa foldery (<span class="var">glad</span> i <span class="var">KHR</span>) do swojego folderu <span class="var">include</span> (albo dodaj kolejny wiersz w ustawieniach projektu wskazujący na tą lokalizację) i dodaj plik <span class="var">glad.c</span> do źródeł swojego projektu (musi być widoczny w _Solution Explorer_).

Po ostatnim kroku, powinieneś być wstanie dodać następującą dyrektywę na samej górze kodu źródłowego:

```cpp
#include <glad/glad.h>
```

Kompilator nie powinien Ci na tym etapie zwrócić żadnych błędów. Na tym etapie jesteśmy przygotowani do następnego etapu kursu, gdzie będziemy dyskutować o tym jak właściwie możemy użyć GLFW i GLAD do skonfigurowania kontekstu OpenGL i do uruchomienia pierwszego okna. Upewnij się, że wszystkie Twoje lokalizacje folderów <span class="var">include</span> oraz <span class="var">lib</span> są wpisane poprawnie oraz czy nazwy bibliotek są podane poprawnie w ustawieniach linkera. Jeżeli dalej stoisz w miejscu, sprawdź komentarze, przejrzyj dodatkowe materiały lub zadaj pytanie poniżej.

## Dodatkowe materiały

*   [GLFW: Window Guide](http://www.glfw.org/docs/latest/window_guide.html): oficjalny poradnik GLFW na temat ustawiania i konfiguracji okna.
*   [Building applications](http://www.opengl-tutorial.org/miscellaneous/building-your-own-c-application/): duża garść informacji na temat kompilowania/linkowania aplikacji oraz podaje listę możliwych błędów (wraz z rozwiązaniami), które mogą się przydarzyć.
*   [GLFW with Code::Blocks](http://wiki.codeblocks.org/index.php?title=Using_GLFW_with_Code::Blocks): budowanie GLFW w Code::Blocks.
*   [Running CMake](http://www.cmake.org/runningcmake/): krótki przegląd tego jak uruchomić CMake zarówno na Windows jak i na Linux.
*   [Writing a build system under Linux](https://learnopengl.com/demo/autotools_tutorial.txt): kurs automatycznych narzędzi autorstwa Wouter Verholst jak napisać system budowania (ang. build system) pod Linux konkretnie na potrzeby tego kursu.
*   [Polytonic/Glitter](https://github.com/Polytonic/Glitter): bardzo prosty projekt, który jest pre-konfigurowany wszystkimi potrzebnymi bibliotekami; świetne rozwiązanie dla tych, którzy chcą mieć przykładowy projekt, bez użerania się z budowaniem wszystkich bibliotek.