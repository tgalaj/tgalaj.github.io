---
layout: post
title: Wiele świateł
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: oswietlenie
---

{% include learnopengl.md link="Lighting/Multiple-lights" %}

W poprzednich tutorialach sporo się nauczyliśmy o światłach w OpenGL. Dowiedzieliśmy się o cieniowaniu Phonga, materiałach, mapach oświetlenia i różnych rodzajach świateł. W tym samouczku połączymy całą uzyskaną wcześniej wiedzę, tworząc w pełni oświetloną scenę z 6 aktywnymi źródłami światła. Zamierzamy zasymulować światło słoneczne jako kierunkowe źródło światła, 4 punktowe światła rozproszone po scenie i latarkę.

Aby użyć więcej niż jednego źródła światła w scenie, chcemy zamknąć obliczenia oświetlenia w <span class="def">funkcjach</span> GLSL. Powodem tego jest to, że kod szybko staje się nieprzejrzysty, gdy chcemy wykonać obliczenia światła z wieloma światłami różnych typów światła, które wymagają różnych obliczeń. Gdybyśmy wykonywali wszystkie te obliczenia tylko w funkcji <span class="fun">main</span>, kod szybko stałby się trudny do zrozumienia.

Funkcje w GLSL są podobne do funkcji C. Mamy nazwę funkcji, typ zwracanej wartości i musielibyśmy zadeklarować prototyp na górze pliku, jeśli funkcja nie została zadeklarowana przed funkcją <span class="fun">main</span>. Stworzymy inną funkcję dla każdego z typów światła: kierunkowego, punktowego i reflektorowego.

Podczas korzystania z wielu świateł w scenie podejście jest zwykle następujące: mamy jeden wektor koloru, który reprezentuje kolor wyjściowy fragmentu. Dla każdego światła, wynikowy kolor światła dla danego fragmentu jest dodawany do wyjściowego wektora koloru fragmentu. Zatem każde światło na scenie oblicza swój indywidualny wpływ na wyżej wymieniony fragment i przyczyni się do końcowego koloru wyjściowego. Ogólna struktura wyglądałaby tak:

```glsl
    out vec4 FragColor;

    void main()
    {
      // zdefiniuj wyjściową wartość koloru
      vec3 output = vec3(0.0);
      // dodaj wpływ światła kierunkowego na kolor wyjściowy
      output += someFunctionToCalculateDirectionalLight();
      // zrób to samo dla wszystkich świateł punktowych
      for(int i = 0; i < nr_of_point_lights; i++)
      	output += someFunctionToCalculatePointLight();
      // i dodaj także inne światła (takie jak reflektory)
      output += someFunctionToCalculateSpotLight();

      FragColor = vec4(output, 1.0);
    }  
```

Rzeczywisty kod będzie się prawdopodobnie różnił w zależności od implementacji, ale ogólna struktura pozostaje taka sama. Definiujemy kilka funkcji, które obliczają wpływ każdego źródła światła i dodają jego wynikowy kolor do wyjściowego wektora koloru. Jeśli na przykład dwa źródła światła są blisko fragmentu, ich łączny wkład dałby bardziej oświetlony fragment niż jakby ten fragment był oświetlany przez pojedyncze źródło światła.

## Światło kierunkowe

Chcemy zdefiniować funkcję w Fragment Shader, która oblicza udział światła kierunkowego: funkcję, która pobiera kilka parametrów i zwraca obliczony kolor oświetlenia kierunkowego.

Najpierw musimy ustawić wymagane zmienne, których potrzebujemy do kierunkowego źródła światła. Możemy przechowywać zmienne w strukturze o nazwie <span class="fun">DirLight</span> i zdefiniować ją jako uniform. Wymagane zmienne powinny być znane z poprzedniego samouczka:

```glsl
    struct DirLight {
        vec3 direction;

        vec3 ambient;
        vec3 diffuse;
        vec3 specular;
    };  
    uniform DirLight dirLight;
```

Następnie możemy przekazać uniform <span class="var">dirLight</span> do funkcji z następującym prototypem:

```glsl
    vec3 CalcDirLight(DirLight light, vec3 normal, vec3 viewDir);  
```

{: .box-note }
Podobnie jak C i C++, jeśli chcemy wywołać funkcję (w tym przypadku wewnątrz funkcji <span class="fun">main</span>) funkcja powinna zostać zdefiniowana przed funkcją wywołującą. W tym przypadku wolimy zdefiniować funkcje poniżej funkcji <span class="fun">main</span>, więc to wymaganie nie jest spełnione. Dlatego musimy zadeklarować prototypy funkcji powyżej funkcji <span class="fun">main</span>, podobnie jak w C/C++.

Widać, że funkcja wymaga struktury <span class="fun">DirLight</span> i dwóch innych wektorów wymaganych obliczeń funkcji. Jeśli pomyślnie ukończyłeś poprzedni tutorial, zawartość tej funkcji nie powinna dziwić:

```glsl
    vec3 CalcDirLight(DirLight light, vec3 normal, vec3 viewDir)
    {
        vec3 lightDir = normalize(-light.direction);
        // światło rozproszone
        float diff = max(dot(normal, lightDir), 0.0);
        // światło lustrzane
        vec3 reflectDir = reflect(-lightDir, normal);
        float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
        // połącz wyniki
        vec3 ambient  = light.ambient  * vec3(texture(material.diffuse, TexCoords));
        vec3 diffuse  = light.diffuse  * diff * vec3(texture(material.diffuse, TexCoords));
        vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
        return (ambient + diffuse + specular);
    }  
```

Zasadniczo skopiowaliśmy kod z poprzedniego samouczka i użyliśmy wektorów podanych jako argumenty funkcji do obliczenia wektora koloru światła kierunkowego. Wynikowe komponenty światła otoczenia, rozproszonego i lustrzanego są następnie zwracane jako pojedynczy wektor koloru.

## Światło punktowe

Podobnie jak w przypadku świateł kierunkowych, chcemy również zdefiniować funkcję, która obliczy wkład światła punktowego na dany fragment, w tym jego tłumienie. Podobnie jak dla kierunkowych świateł chcemy zdefiniować strukturę, która określa wszystkie zmienne wymagane dla światła punktowego:

```glsl
    struct PointLight {    
        vec3 position;

        float constant;
        float linear;
        float quadratic;  

        vec3 ambient;
        vec3 diffuse;
        vec3 specular;
    };  
    #define NR_POINT_LIGHTS 4  
    uniform PointLight pointLights[NR_POINT_LIGHTS];
```

Jak widać, w GLSL zastosowaliśmy dyrektywę pre-procesorowa, aby zdefiniować maksymalną liczbę świateł punktowych, które chcemy mieć w naszej scenie. Następnie używamy tej stałej <span class="var">NR_POINT_LIGHTS</span>, aby utworzyć tablicę struktur <span class="fun">PointLight</span>. Tablice w GLSL są podobne do tablic C i mogą być tworzone za pomocą dwóch nawiasów kwadratowych. W tej chwili mamy 4 <span class="fun">PointLight</span> struktury do wypełnienia danymi.

{: .box-note }
Moglibyśmy również po prostu zdefiniować **jedną** dużą strukturę (zamiast różnych struktur na każdy typ światła), która zawiera wszystkie niezbędne zmienne dla **wszystkich** różnych typów światła i użyć tej struktury dla każdej funkcji i po prostu zignorować zmienne, których nie potrzebujemy. Jednak osobiście uważam, że obecne podejście jest bardziej intuicyjne i poza kilkoma dodatkowymi liniami kodu może zaoszczędzić trochę pamięci, ponieważ nie wszystkie typy światła wymagają wszystkich zmiennych.

Prototyp funkcji światła punktowego wygląda następująco:

```glsl
    vec3 CalcPointLight(PointLight light, vec3 normal, vec3 fragPos, vec3 viewDir);  
```

Funkcja pobiera wszystkie potrzebne dane jako swoje argumenty i zwraca wartość `vec3`, która reprezentuje kolor, który jest oświetlany przez dane światło punktowe. Ponownie, inteligentne kopiowanie i wklejanie skutkuje następującą funkcją:

```glsl
    vec3 CalcPointLight(PointLight light, vec3 normal, vec3 fragPos, vec3 viewDir)
    {
        vec3 lightDir = normalize(light.position - fragPos);
        // światło rozproszone
        float diff = max(dot(normal, lightDir), 0.0);
        // światło lustrzane
        vec3 reflectDir = reflect(-lightDir, normal);
        float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
        // tłumienie
        float distance    = length(light.position - fragPos);
        float attenuation = 1.0 / (light.constant + light.linear * distance + 
      			            light.quadratic * (distance * distance));    
        // połącz wyniki
        vec3 ambient  = light.ambient  * vec3(texture(material.diffuse, TexCoords));
        vec3 diffuse  = light.diffuse  * diff * vec3(texture(material.diffuse, TexCoords));
        vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
        ambient  *= attenuation;
        diffuse  *= attenuation;
        specular *= attenuation;
        return (ambient + diffuse + specular);
    } 
```

Umieszczenie tej funkcjonalności w zewnętrznej funkcji ma tę zaletę, że możemy łatwo obliczyć oświetlenie dla wielu świateł punktowych bez potrzeby duplikowania kodu. W funkcji <span class="fun">main</span> po prostu tworzymy pętlę, która iteruje po tablicy świateł punktowych, która wywołuje <span class="fun">CalcPointLight</span> dla każdego światła punktowego.

## Łączymy wszystko razem

Teraz, gdy zdefiniowaliśmy zarówno funkcję dla świateł kierunkowych, jak i funkcję dla świateł punktowych, możemy umieścić je razem w funkcji <span class="fun">main</span>.

```glsl
    void main()
    {
        // przygotowanie wektorów
        vec3 norm = normalize(Normal);
        vec3 viewDir = normalize(viewPos - FragPos);

        // faza 1: światło kierunkowe
        vec3 result = CalcDirLight(dirLight, norm, viewDir);
        // faza 2: światła punktowe
        for(int i = 0; i < NR_POINT_LIGHTS; i++)
            result += CalcPointLight(pointLights[i], norm, FragPos, viewDir);    
        // faza 3: światło reflektorowe
        //result += CalcSpotLight(spotLight, norm, FragPos, viewDir);    

        FragColor = vec4(result, 1.0);
    }
```

Każdy rodzaj światła dodaje swój wkład do powstałego koloru wyjściowego, dopóki wszystkie źródła światła nie zostaną przetworzone. Wynikowy kolor zawiera wpływ koloru wszystkich źródeł światła w scenie. Jeśli chcesz, możesz również zaimplementować reflektor i dodać jego efekt również do koloru wyjściowego. Pozostawiam implementację funkcji <span class="fun">CalcSpotLight</span> jako ćwiczenie dla czytelnika.

Ustawienie uniformów dla struktury światła kierunkowego nie powinno być zbyt trudne, ale możesz się zastanawiać, w jaki sposób możemy ustawić uniformy dla wartości świateł punktowych, ponieważ uniform świateł punktowych jest teraz tablicą struktur <span class="fun">PointLight</span>. To nie jest coś, o czym mówiliśmy wcześniej.

Na szczęście, nie jest to zbyt skomplikowane. Ustawienie uniformu tablicy struktur działa tak samo, jak ustawienie uniformów struktur, ale tym razem musimy również zdefiniować odpowiedni indeks podczas pobierania lokalizacji uniformu:

```cpp
    lightingShader.setFloat("pointLights[0].constant", 1.0f);
```

Tutaj indeksujemy pierwszą strukturę <span class="fun">PointLight</span> w tablicy <span class="var">pointLights</span> i pobieramy lokalizację jej zmiennej <span class="var">constant</span>. To niestety oznacza, że ​​musimy ręcznie ustawić wszystkie uniformy dla każdego z 4 świateł punktowych, co prowadzi do 28 wywołań do ustawiania uniformów dla samych świateł punktowych, co jest nieco żmudnym zadaniem. Można spróbować to nieco uprościć, definiując klasę światła punktowego, która ustawi uniformy za ciebie, ale i tak musisz ustawić wszystkie wartości uniformów światła w ten sposób.

Nie zapominajmy, że musimy również zdefiniować wektor pozycji dla każdego ze świateł punktowych, więc rozrzućmy je nieco po scenie. Zdefiniujmy kolejną tablicę `glm :: vec3`, która zawiera pozycje świateł punktowych:

```glsl
    glm::vec3 pointLightPositions[] = {
    	glm::vec3( 0.7f,  0.2f,  2.0f),
    	glm::vec3( 2.3f, -3.3f, -4.0f),
    	glm::vec3(-4.0f,  2.0f, -12.0f),
    	glm::vec3( 0.0f,  0.0f, -3.0f)
    };  
```

Następnie, zaindeksuj odpowiednią strukturę <span class="fun">PointLight</span> w tablicy <span class="var">pointLights</span> i ustaw jej zmienną <span class="var">position</span> jako jedną z pozycji, które właśnie zdefiniowaliśmy. Pamiętaj też, aby teraz narysować 4 kostki reprezentujące światła zamiast 1. Po prostu utwórz kolejną macierz dla każdej kostki reprezentującej światło, tak jak zrobiliśmy to z pojemnikami.

Jeśli dodatkowo użyjesz latarki, wynik wszystkich połączonych świateł wygląda mniej więcej tak:

![](/img/learnopengl/multiple_lights_combined.png){: .center-image }

Jak widać, wydaje się, że gdzieś na niebie jest jakaś forma globalnego światła (jak słońce), mamy 4 światła rozproszone po scenie i latarkę, która jest widoczna z perspektywy gracza. Wygląda całkiem nieźle, prawda?

Możesz znaleźć pełny kod źródłowy skończonej aplikacji [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/2.lighting/6.multiple_lights/multiple_lights.cpp).

Obrazek pokazuje wszystkie źródła światła ustawione z domyślnymi właściwościami światła, które stosowaliśmy we wszystkich poprzednich samouczkach, ale jeśli będziesz bawić się tymi wartościami, możesz uzyskać całkiem interesujące wyniki. Artyści i edytorzy poziomów zazwyczaj zmieniają wszystkie te zmienne świetlne w dużym edytorze, aby upewnić się, że oświetlenie pasuje do otoczenia. Korzystając z prostego, oświetlonego środowiska, które właśnie stworzyliśmy, możesz stworzyć ciekawe efekty wizualne, po prostu zmieniając ich atrybuty oświetlenia:

![](/img/learnopengl/multiple_lights_atmospheres.png){: .center-image }

Zmieniliśmy również kolor czyszczenia tła, aby lepiej oddać oświetlenie. Widać, że po prostu dostosowując niektóre parametry oświetlenia, możesz stworzyć zupełnie inny klimat.

Teraz powinieneś dobrze rozumieć oświetlenie w OpenGL. Dzięki dotychczasowej wiedzy możemy już tworzyć ciekawe i bogate wizualnie środowiska. Spróbuj pobawić się różnymi wartościami, aby stworzyć własną, unikalną scenę.

## Ćwiczenia

*   Czy możesz (w pewnym sensie) odtworzyć różne sceny z ostatniego obrazu, modyfikując wartości atrybutów światła? [rozwiązanie](https://learnopengl.com/code_viewer.php?code=lighting/multiple_lights-exercise2).