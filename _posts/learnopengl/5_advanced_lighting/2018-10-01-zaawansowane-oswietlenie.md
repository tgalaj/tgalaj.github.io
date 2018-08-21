---
layout: post
title: Zaawansowane oświetlenie
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: advanced-lighting
mathjax: true
---

{% include learnopengl.md link="Advanced-Lighting/Advanced-Lighting" %}

W tutorialach o [oświetleniu]({% post_url /learnopengl/2_lighting/2018-08-01-podstawy-oswietlenia %}) wprowadziliśmy model oświetlenia Phong'a, aby wprowadzić do naszych scen trochę realizmu. Model Phong'a wygląda całkiem nieźle, ale ma kilka niuansów, na których skupimy się w tym samouczku.

## Blinn-Phong

Oświetlenie Phong'a to świetne i bardzo wydajne przybliżenie oświetlenia, ale jego odbicia lustrzane nie wyglądają dobrze w pewnych warunkach, szczególnie gdy właściwość połysku jest niska, co daje duży (szorstki) obszar lustrzany. Poniższy obrazek pokazuje, co dzieje się, gdy używamy wykładnika połysku równego `1.0` na płaskiej, oteksturowanej płaszczyźnie:

![Wynik odbicia lustrzanego Phonga z niskim wykładnikiem](/img/learnopengl/advanced_lighting_phong_limit.png){: .center-image }

Na krawędziach widać, że obszar odbicia lustrzanego jest natychmiast odcięty. Dzieje się tak dlatego, że kąt między wektorem patrzenia a wektorem odbicia nie może przekroczyć 90 stopni. Jeśli kąt jest większy niż 90 stopni, wynikowy iloczyn skalarny staje się ujemny, co daje w wyniku wykładnik o wartości `0.0`. Prawdopodobnie myślisz, że to nie będzie problem, ponieważ nie powinniśmy uzyskać żadnego oświetlenia dla kątów wyższych niż 90 stopni, prawda?

Nieprawda, dotyczy to tylko elementu rozproszonego, w którym kąt większy niż 90 stopni między wektorem normalnym a wektorem źródła światła oznacza, że ​​źródło światła znajduje się poniżej oświetlonej powierzchni, a zatem światło rozproszone powinno wynosić `0.0`. Jednak przy oświetleniu lustrzanym nie mierzymy kąta między źródłem światła a wektorem normalnym, ale między wektorem patrzenia a kierunkiem odbicia. Spójrz na następujące dwa obrazy:

![Obraz wektorów odbicia Phonga jest niepoprawny, gdy jest większy niż 90 stopni](/img/learnopengl/advanced_lighting_over_90.png){: .center-image }

Tutaj problem powinien stać się oczywisty. Lewy obraz pokazuje odbicia Phong'a, gdzie $\theta$ ma mniej niż 90 stopni. Na prawym obrazie widzimy, że kąt $\theta$ pomiędzy wektorem patrzenia a kierunkiem odbicia jest większy niż 90 stopni i w rezultacie anuluje wkład oświetlenia lustrzanego. Zasadniczo nie stanowi to problemu, ponieważ kierunek patrzenia jest daleki od kierunku odbicia, ale jeśli używamy wykładnika o niskim połysku, promień zwierciadlany jest wystarczająco duży, aby mieć udział w tych warunkach. W takim przypadku anulujemy ten wkład pod kątem większym niż 90 stopni (jak widać na pierwszym obrazie).

W 1977 roku został wprowadzony model cieniowania <def>Blinn'a-Phong'a</def> przez Jamesa F. Blinn'a jako rozszerzenie cieniowania Phong'a, którego używaliśmy do tej pory. Model Blinna-Phonga jest w dużej mierze podobny do modelu Phonga, ale podchodzi nieco inaczej do oświetlenia lustrzanego, który w rezultacie rozwiązuje nasz problem. Zamiast polegać na wektorze odbicia, używamy tzw. <def>wektora w połowicznego</def>, który jest wektorem jednostkowym dokładnie w połowie między kierunkiem patrzenia a kierunkiem światła. Im bliżej ten wektor połoowiczny wyrównuje się z wektorem normalnym powierzchni, tym większy jest udział oświetlenia lustrzanego.

![Ilustracja wektora połowicznego Blinna-Phonga](/img/learnopengl/advanced_lighting_halfway_vector.png){: .center-image }

Gdy kierunek patrzenia jest idealnie wyrównany z (teraz wyobrażonym) wektorem odbicia, wektor połowiczny idealnie dopasowuje się do wektora normalnego. Im bardziej odbiorca patrzy w kierunku oryginalnego odbicia, tym silniejszy staje się rozbłysk.

Tutaj widać, że niezależnie od kierunku, w którym patrzy widz, kąt między wektorem połowicznym a wektorem normalnym powierzchni nigdy nie przekracza 90 stopni (chyba że światło jest znacznie poniżej powierzchni). Daje to nieco inne wyniki w porównaniu do odbić Phonga, ale w większości przypadków wygląda lepiej wizualnie, szczególnie w przypadku niskich wykładników. Model cieniowania Blinna-Phonga jest również dokładnym modelem cieniowania stosowanym we wcześniejszym, stałym potoku OpenGL.

Uzyskanie wektora połowicznego jest łatwe, dodajemy wektor kierunku światła i wektor patrzenia i normalizujemy wynik:

$$\bar{H} = \frac{\bar{L} + \bar{V}}{||\bar{L} + \bar{V}||}$$

Przekłada się to na kod GLSL w następujący sposób:

```glsl
    vec3 lightDir   = normalize(lightPos - FragPos);
    vec3 viewDir    = normalize(viewPos - FragPos);
    vec3 halfwayDir = normalize(lightDir + viewDir);
```

Wtedy obliczenie parametru odbicia w zasadzie staje się obciętym iloczynem skalarnym między wektorem normalnym i wektorem połowicznym, aby uzyskać cosinus kąta pomiędzy nimi, który ponownie podnosimy do potęgi połyskliwości (ang. *shininness*):

```glsl
    float spec = pow(max(dot(normal, halfwayDir), 0.0), shininess);
    vec3 specular = lightColor * spec;
```

To wszystko na temat modelu Blinna-Phonga. Jedyną różnicą między odbiciem lustrzanym Blinna-Phonga i Phonga jest to, że mierzymy teraz kąt pomiędzy wektorem normalnym i wektorem połowicznym w porównaniu do kąta między kierunkiem patrzenia a wektorem odbicia.

Wraz z wprowadzeniem wektora połowicznego do obliczania najciemniejszych punktów, nie powinniśmy już mieć odcięcia lustrzanego jak to było w modelu cieniowania Phonga. Poniższy obrazek przedstawia obszar lustrzany obu metod z wykładnikiem lustrzanym równym `0.5`:

![Porównanie cieniowania Phonga i Blinna-Phonga z niskim wykładnikiem](/img/learnopengl/advanced_lighting_comparrison.png){: .center-image }

Inną subtelną różnicą cieniowania Phonga i Blinna-Phonga jest to, że kąt między wektorem połowicznym a wektorem normalnym jest często krótszy niż kąt pomiędzy wektorem patrzenia a  wektorem odbicia. W rezultacie, aby uzyskać podobne wyniki do cieniowania Phonga, współczynnik połyskliwości musi być nieco wyższy. Ogólna zasada polega na ustawieniu od 2 do 4 razy większego współczynnika połyskliwości Phonga.

Poniżej znajduje się porównanie obu modeli odbicia lustrzanego z wykładnikiem Phonga ustawionym na `8.0` i Blinna-Phonga na `32.0`:

![Porównanie cieniowania Phonga i Blinna-Phonga z normalnymi wykładnikami](/img/learnopengl/advanced_lighting_comparrison2.png){: .center-image }

Widać, że wykładnik połyskliwości Blinna-Phonga jest nieco ostrzejszy w porównaniu do Phonga. Zazwyczaj wymaga to trochę ulepszeń, aby uzyskać podobne wyniki, co wcześniej, ale cieniowanie Blinna-Phonga daje ogólnie bardziej wiarygodne wyniki w porównaniu do cieniowania Phonga.

Tutaj użyliśmy prostego Fragment Shadera, który przełącza się pomiędzy modelem Phonga, a modelem Blinna-Phonga:

```glsl
    void main()
    {
        [...]
        float spec = 0.0;
        if(blinn)
        {
            vec3 halfwayDir = normalize(lightDir + viewDir);  
            spec = pow(max(dot(normal, halfwayDir), 0.0), 16.0);
        }
        else
        {
            vec3 reflectDir = reflect(-lightDir, normal);
            spec = pow(max(dot(viewDir, reflectDir), 0.0), 8.0);
        }
```

Możesz znaleźć kod źródłowy dla prostej wersji demonstracyjnej [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/5.advanced_lighting/1.advanced_lighting/advanced_lighting.cpp). Po naciśnięciu przycisku `b` demo przełącza się z modelu Phonga na model Blinna-Phonga i vica versa.