---
layout: post
title: Tutorial 04 - Czym jest programowalny potok renderingu?
subtitle: Kurs OpenGL dla początkujących
tags: [beginner-opengl-pl, tutorial]
---
## Wstęp

Na początku tej części chciałbym przeprosić, że już od dłuższego czasu nie było żadnych nowości - postaram się to nadrobić w nadchodzących tygodniach :-) . Poniżej znajduje się link do odpowiedzi do ćwiczeń z poprzedniej części. Zachęcam do zweryfikowania swoich odpowiedzi.

<details class="panel panel-success">
  <summary markdown="span" class="panel-heading">
    Odpowiedzi do ćwiczeń
  </summary>

**1.** Trójkąt zostanie narysowany poprawnie, ale będzie powiększony, tak, że jego wierzchołki "wyjdą" poza zakres okna OpenGL.

**2.** Obraz musi być dwa razy pomniejszony, czyli dwa ostatnie parametry naszej rzutni (viewport) mają odpowiednio szerokość: _width/2_ i wysokość: _height/2_, gdzie _width_ i _height_ to szerokość i wysokość naszego okna OpenGL.

Następnie musimy ustawić tą rzutnię na środku ekranu. Czyli lewy dolny róg rzutni musi być w 1/4 szerokości i 1/4 wysokości okna OpenGL (pamiętajmy, że lewy dolny róg okna OpenGL to punkt _(0, 0)_). Zatem, pierwsze dwa parametry mają odpowiednio wartości: _width/4_ i _height/4_.  

```cpp  
glViewport(width/4, height/4, width/2, height/2);  
```

**3.** Żeby narysować kwadrat, trzeba skorzystać z dwóch trójkątów. W tym celu aktualizujemy tablicę _vertices_ o nowe wartości:  

```cpp  
glm::vec3 vertices[] = { glm::vec3(-1.0f, -1.0f, 0.0f),  
                         glm::vec3(-1.0f,  1.0f, 0.0f),  
                         glm::vec3( 1.0f, -1.0f, 0.0f),  
                         glm::vec3(-1.0f,  1.0f, 0.0f),  
                         glm::vec3( 1.0f,  1.0f, 0.0f),  
                         glm::vec3( 1.0f, -1.0f, 0.0f) };  
```  
Oraz zmieniamy ostatni parametr w funkcji _glDrawArrays()_, który mówi o tym ile wierzchołków z tej tablicy chcemy narysować (w tym wypadku 6 - dwa trójkąty; dwa punkty się powtarzają - o uniknięciu tej nadmiarowości będzie w kolejnych częściach tego kursu).  

```cpp  
glDrawArrays(GL_TRIANGLES, 0, 6);  
```

</details>

W tej części kursu będzie sama teoria dotycząca tego jak właściwie przebiega cały proces renderowania w OpenGL'u - będzie to taki wstęp do programów cieniujących (shaderów). Uważam, że jest to dosyć istotny aspekt przy nauce programowania grafiki 3D, ponieważ pozwala on zrozumieć zachowania OpenGL'a oraz będziemy bardziej świadomi tego co dzieje się za "kulisami" narysowania wirtualnej sceny na ekranie. Dodatkowo, ta wiedza ułatwi nam zrozumienie działania shaderów, byśmy mogli z łatwością "pokolorować" nasz trójkąt oraz jakoś go później przekształcić.

## Programowalny potok renderingu

Dawno, dawno temu, gdy na topie był OpenGL 1.0, w kartach graficznych był zaimplementowany tzw. _stały potok renderingu_. Jego zaletą było to, że w niewielkim czasie mogliśmy narysować trójkąt na ekranie, pokolorować go i dowolnie obracać. Dokładniej mówiąc, programiści byli ograniczeni tylko do używania "cegiełek", które ktoś wcześniej zaimplementował, by stworzyć coś nowego.

Takie podejście było dobre do pewnego momentu, w którym możliwości tych "cegiełek" się wyczerpały i programiści chcieli stworzyć coś nowego, unikalnego, szybszego. Dlatego producenci kart graficznych wymyślili _programowalny potok renderingu_, w którym na pewnych etapach rysowania geometrii, programista mógł mieć wpływ (pisząc programy cieniujące - shadery) na to jak geometria będzie wyglądać (operacje na wierzchołach) i jak zostanie pokolorowana (operacje na pikselach). Wraz z kolejnymi wersjami OpenGL'a można było mieć wpływ na coraz więcej poszczególnych etapów renderowania oraz zaczęto odchodzić coraz bardziej od przestarzałego, stałego potoku renderingu. Dzisiejszy proces renderowania jest przedstawiony na poniższym obrazku:

{% include lightbox src="img/beginner_opengl/GL-Pipeline.jpg" data="data" title="Potok renderingu OpenGL" alt="Potok renderingu OpenGL" img-style="max-width:70%;" class="center-image" %}

Jak widać z powyższego diagramu, na początku zaczynamy od przesłania danych wierzchołków prymitywów, które chcemy narysować. Prymityw jest to podstawowa figura geometryczną, którą możemy narysować. OpenGL oferuje nam takie prymitywy jak: punkty (GL_POINTS), linie (GL_LINES), łamane (GL_LINE_STRIP), łamane zamknięte (GL_LINE_LOOP), trójkąty (GL_TRIANGLES), paski trójkątów (GL_TRIANGLE_STRIP), wachlarze trójkątów (GL_TRIANGLE_FAN). Poniżej znajduje się obrazek przedstawiający wcześniej wymienione prymitywy geometryczne.

{% include lightbox src="img/beginner_opengl/GL-Primitives.jpg" data="data" title="Prymitywy OpenGL" alt="Prymitywy OpenGL" img-style="max-width:70%;" class="center-image" %}

Następnie obróbką tych danych zajmuje się Vertex Shader, który przekształca nam wierzchołki z lokalnego układu współrzędnych obiektu, do współrzędnych ekranu (więcej o transformacjach będzie w części poświęconej transformacjom); Tessellation Shader, który tak naprawdę składa się z dwóch osobnych programów cieniujących oraz Geometry Shader (więcej o Tessellation i Geometry shader'ach będzie w następnych częściach tego kursu). Następnie są tworzone prymitywy, które potem są "obcinane" jeżeli wyjdą poza obszar widoczności wirtualnego "oka". Na koniec uruchamiany jest Fragment Shader, który koloruje piksele na odpowiednie kolory. Po tym są uruchamiane jeszcze różne testy (nożyczek (scissor test) alfa, szablonu (stencil test), głębi, mieszania (blending)) i na koniec uzyskujemy wyrenderowaną scenę 3D.

W kolejnych sekcjach przyjrzymy się z bliska każdemu z tych etapów, by dowiedzieć się co każdy z nich dokładnie robi.

## Vertex Data

Na początku przygotowujemy nasze dane, które przedstawiają jakiś kształt np. trójkąt i umieszczamy je w odpowiednej tablicy (tablica _vertices_ z poprzedniej części). Kiedy mamy przygotowane wierzchołki musimy je przesłać do obiektu OpenGL, który te dane może przechować - bufor. Przesyłanie danych do bufora OpenGL odbywa się poprzez wywołanie funkcji _glBuferData()_. Kiedy dane znajdują się w buforze możemy je narysować poprzez wywołanie funkcji _glDrawArrays()_. Rysowanie oznacza przesłanie tych danych dalej w potoku renderingu.

## Vertex Shader

Następnym procesem, do którego trafiają dane po wyrażeniu chęci rysowania jest Vertex Shader. Jest to proces, nad którym mamy pełną kontrolę i sami go definiujemy. Jest wymagane by był zaimplementowany i użyty przynajmniej jeden Vertex Shader podczas uruchomienia aplikacji.

Vertex Shader jest zwykłym programem, który piszemy i jest wywoływany dla każdego wierzchołka, który chcemy narysować. Jego głównym celem jest przekształcenie wierzchołków do współrzędnych ekranu, ale też może zostać użyty np. do przekształcania pozycji tych wierzchołków.

## Tessellation Control & Evaluation Shader

Teraz dane znajdują się na etapie Tessellation Control Shader'a i Tessellation Evaluation Shader'a. Są to podobne programy jak Vertex Shader, nad którymi mamy wpływ co się w nich stanie. Nie są one obowiązkowe, ponieważ służą do specjalnych celów. W odróżnieniu od Vertex Shader'a, Tessellation Shader'y działają na _łatach (patch)_. Generalnie służą one do teselacji geometrii, czyli na zwiększeniu liczy prymitywów geometrycznych w danym kształcie geometrycznym po to by uzyskać np. bardziej wygładzoną siatkę modelu.

## Geometry Shader

Następnym procesem, do którego trafiają dane jest Geometry Shader. Jest on, podobnie jak Tessellation Shader'y, nieobowiązkowym etapem (można ale nie trzeba go pisać) i służy on do dodatkowego przetworzenia geometrii (przesłanych danych), by np. stworzyć nowe prymitywy geometryczne jeszcze przed ich rasteryzacją.

## Primitive Setup

Wcześniejsze etapy operowały jedynie na wierzchołkach, które miały stworzyć odpowiednie prymitywy geometryczne. W tym etapie, tworzone są te kształty geometryczne na podstawie tych wierzchołków, które wcześniej przetworzyliśmy (albo i nie).

## Clipping

Czasami może zarzyć się tak, że wierzchołki mogą znaleźć się poza rzutnią (viewport'em) - obszarem na którym możemy rysować - figura geometryczna znajduje się częściowo w obszarze rzutni, a częściowo poza nią. W tym celu wierzchołki, które leżą poza rzutnią są modyfikowane w ten sposób, by żaden z nich nie był już poza rzutnią.

Jeżeli figura znajduje się całkowicie w obszarze rzutni, to jej wierzchołki nie są modyfikowane, a jeżeli leży całkowicie poza rzutnią to te wierzchołki nie zostaną uwzględnione w kolejnych krokach - zostaną odrzucone.

Jest to automatyczny proces, którym zajmuje się sam OpenGL.

## Rasterization

Następnie, prymitywy są przesyłane do rasteryzera. Jego zadaniem jest określenie, które piksele rzutni są pokryte przez dany prymityw geometryczny. W tym etapie generowane są _fragmenty_ czyli informacja o pozycji danego piksela oraz interpolowany kolor wierzchołka i interpolowana koordynata tekstury.

Przetwarzaniem tych fragmentów zajmują się dwa kolejne etapy.

## Fragment Shader

Tak jak Vertex Shader, jest to etap, nad którym mamy pełną kontrolę i jest on obowiązkowy do zdefiniowania (bo skąd OpenGL ma wiedzieć jak pokolorować geometrię?). W tym etapie jest pisany Fragment Shader, który wykonuje się raz dla każdego fragmentu z procesu rasteryzacji. W tym procesie możemy zdefiniować finalny kolor (czy to zdefiniowany przez programistę czy wyliczony z kalkulacji światła) fragmentu oraz czasami wartość głębi (_depth value_). Możemy w tym kroku zamiast zwykłego koloru, nałożyć na fragment teksturę, bu urzeczywistnić naszą trójwymiarową scenę.

Fragment Shader może również zaprzestać przetwarzania danego fragmentu jeżeli uzna, że dany fragment nie powinien być rysowany.

Różnica między Vertex (wliczając w to Tessellation i Geometry Shader'y), a Fragment Shader'em jest taka, że Vertex Shader zajmuje się umiejscowieniem prymitywu na ekranie, a Fragment Shader zajmuje się nadaniem koloru temu fragmentowi.

## Operacje po Fragment Shader

Dodatkowo po operacjach, które możemy sami zdefiniować w Fragment Shader, wykonywane są finalne działania na poszczególnych fragmentach. Te dodatkowe czynności są opisane w kolejnych podrozdziałach.

### Scissor test

Test nożyczek. W aplikacji możemy zdefiniować prostokąt, co do którego ograniczy się zakres renderowania. Każdy fragment, który znajdzie się poza tym zdefiniowanym obszarem nie zostanie wyrenderowany.

### Alpha test

Po teście nożyczek następuje test kanału alfa. Służy on do tego by określić przezroczystość danego fragmentu. Ten test porównuje wartość alfa danego fragmentu i wartość, która została zdefiniowana w programie. Następnie sprawdzana jest relacja między tymi wartościami (czy większa, czy mniejsza, czy równa, itd.) jaka powoduje przejście tego testu. Jeżeli relacja nie powoduje przejścia testu, fragment jest odrzucany.

### Stencil test

Kolejnym testem jest test szablonu, który odczytuje wartości z bufora szablonu (_stencil buffer_) w pozycji danego fragmentu i porównuje je z wartościami zdefiniowanymi przez aplikację. Test szablonu przechodzi tylko wtedy gdy odpowiednia relacja jest spełniona (wartość jest równa, większa, mniejsza, itd.). W przeciwnym wypadku, test nie powodzi się i dany fragment jest odrzucany.

W tym wypadku, możemy zdefiniować co się stanie w buforze szablonu jeżeli test szablonu się powiedzie (używane jest to w jednej technice renderowania cieni).

### Depth test

W teście głębi, porównywana jest głębia danego fragmentu z głębią w buforze głębi (_depth buffer_). Jeżeli głębia fragmentu nie spełnia relacji (która jest określona w aplikacji) z wartością w buforze głębi, dany fragment jest odrzucany. Domyślnie ta relacja jest ustawiona jako "mniejszy lub równy" w OpenGL, ale możemy ją zmienić. Czyli jeżeli fragment ma wartość głębi mniejszą lub równą wartości z bufora głębi, wartość w buforze jest zastępowana przez wartość głębi tego fragmentu.

Jest to ważny test, który pozwala nam na przesłanianie obiektów innymi obiektami (wiemy co jest za czymś lub przed czymś).

### Blending

Kiedy wszystkie testy się zakończą, kolor danego fragmentu jest mieszany z kolorem w buforze obrazu (_image buffer_). Wartość koloru danego fragmentu jest łączona z kolorem w buforze obrazu (lub kolor fragmentu może zastępować wartość w buforze obrazu). Ten etap może zostać tak skonfigurowany, by otrzymać efekt przezroczystości.

To już na tyle. Gratulacje dla wytrwałych, którzy doszli do końca tego artykułu i pogłębili swoją wiedzę. Jeżeli czegoś nie rozumiecie na tym etapie - nie martwcie się! Wszystko się wyjaśni w następnych częściach tego kursu, kiedy to mam nadzieję, będą już praktyczne lekcje. W następnej lekcji zajmiemy się pisaniem pierwszego programu cieniującego (shader'a).

## Dodatkowe źródła
1. [Dokumentacja](http://www.opengl.org/registry/doc/glspec44.core.pdf) OpenGL 4.4 w wersji angielskiej
2. Mathematics for 3D Game Programming and Computer Graphics, Lengyel Eric, 2012
3. Real-Time Rendering Third Edition, Akenine-Moller T., Haines E., Hoffman N., 2008