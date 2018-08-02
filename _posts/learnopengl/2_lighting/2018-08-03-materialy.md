---
layout: post
title: Materiały
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: oswietlenie
---

{% include learnopengl.md link="Lighting/Materials" %}

W świecie rzeczywistym każdy obiekt reaguje inaczej na światło. Obiekty stalowe są często jaśniejsze niż gliniany wazon, a drewniany pojemnik nie reaguje na światło tak samo jak stalowym pojemnik. Każdy obiekt reaguje inaczej na refleksy. Niektóre obiekty odbijają światło bez zbytniego rozpraszania, co powoduje małe rozbłyski, a inne rozpraszają się, dając większy rozbłysk. Jeśli chcemy symulować kilka typów obiektów w OpenGL, musimy zdefiniować właściwości <span class="def">materiałów</span>, które są specyficzne dla każdego obiektu.

W poprzednim tutorialu określiliśmy obiekt i kolor światła, aby zdefiniować wizualny efekt końcowy tego obiektu, w połączeniu z komponentem natężenia światła otoczenia i lustrzanego. Przy opisywaniu obiektów możemy zdefiniować kolor materiału dla każdego z 3 elementów oświetlenia: oświetlenia otoczenia, rozproszonego i lustrzanego. Określając kolor dla każdego komponentu, mamy drobnoziarnistą kontrolę nad wizualnym efektem końcowym obiektu. Teraz dodajmy zmienną odpowiedzialną za rozbłysk do tych 3 komponentów i dostaniemy wszystkie właściwości materiałów, których potrzebujemy:

```glsl
    #version 330 core
    struct Material {
        vec3 ambient;
        vec3 diffuse;
        vec3 specular;
        float shininess;
    }; 

    uniform Material material;
```

W Fragment Shader tworzymy strukturę `struct` do przechowywania właściwości materiału obiektu. Możemy również przechowywać je jako indywidualne wartości jednorodne, ale przechowywanie ich jako struktur sprawia, że ​​są bardziej uporządkowane. Najpierw definiujemy układ struktury, a następnie po prostu deklarujemy zmienną typu uniform z nowo utworzoną strukturą jako jej typem.

Jak widać, definiujemy wektor koloru dla każdego z komponentów oświetlenia Phong'a. Wektor materiału <span class="var">ambient</span> określa kolor, jaki ten przedmiot odbija w oświetleniu otoczenia; jest to zwykle tym samym, co kolor obiektu. Wektor materiału <span class="var">diffuse</span> określa kolor obiektu dla oświetlenia rozproszonego. Kolor rozproszenia jest (podobnie jak oświetlenie otoczenia) ustawiony na pożądany kolor obiektu. Wektor materiału <span class="var">specular</span> ustawia wpływ koloru światła lustrzanego na obiekt (a może nawet odzwierciedla specyficzny dla obiektu kolor rozbłysku). Na koniec, <span class="var">shininess</span> wpływa na promień rozbłysku światła.

Dzięki tym 4 komponentom definiującym materiał obiektu możemy symulować wiele rzeczywistych materiałów. Tabela znaleziona na stronie [devernay.free.fr](http://devernay.free.fr/cours/opengl/materials.html) pokazuje kilka właściwości materiałów, które symulują prawdziwe materiały znalezione w świecie zewnętrznym. Poniższy obrazek pokazuje wpływ kilku z tych materiałów rzeczywistego świata na naszą kostkę:

![](/img/learnopengl/materials_real_world.png){: .center-image }

Jak możesz zauważyć, poprzez poprawnie określone właściwości materiału obiektu, zmienia się percepcja obiektu. Efekty są wyraźnie zauważalne, ale dla najbardziej realistycznych rezultatów będziemy potrzebować bardziej skomplikowanych kształtów niż sześcianu. W następujących sekcjach samouczka omówimy bardziej skomplikowane kształty.

Uzyskanie poprawnych materiałów dla różnych obiektów jest trudnym zadaniem, które wymaga przede wszystkim eksperymentowania i dużego doświadczenia, więc nie jest czymś niezwykłym, aby całkowicie zniszczyć wizualną jakość obiektu przez niewłaściwie parametry materiału.

Spróbujmy wprowadzić taki system materiałów do shader'ów.

# Ustawianie materiałów

Stworzyliśmy zmienną uniform o typie struktury materiału w Fragment Shader, dlatego też zmienimy obliczenia dotyczące oświetlenia, aby zachować zgodność z nowymi właściwościami materiałów. Ponieważ wszystkie zmienne materiałowe są przechowywane w strukturze, możemy uzyskać do nich dostęp z poziomu uniformu <span class="var">material</span>:

```glsl
    void main()
    {    
        // ambient
        vec3 ambient = lightColor * material.ambient;

        // diffuse 
        vec3 norm = normalize(Normal);
        vec3 lightDir = normalize(lightPos - FragPos);
        float diff = max(dot(norm, lightDir), 0.0);
        vec3 diffuse = lightColor * (diff * material.diffuse);

        // specular
        vec3 viewDir = normalize(viewPos - FragPos);
        vec3 reflectDir = reflect(-lightDir, norm);  
        float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
        vec3 specular = lightColor * (spec * material.specular);  

        vec3 result = ambient + diffuse + specular;
        FragColor = vec4(result, 1.0);
    }
```

Jak widać, teraz mamy dostęp do wszystkich właściwości struktury `material`, gdziekolwiek ich potrzebujemy i tym razem możemy obliczyć finalny kolor za pomocą kolorów materiału. Każdy z atrybutów materiału obiektu mnożony jest przez odpowiednie komponenty oświetlenia.

Możemy ustawić materiał obiektu w aplikacji, ustawiając odpowiednie uniformy. Struktura w GLSL nie jest jednak szczególna pod żadnym względem przy ustawianiu uniformów. Struktura działa tylko jako hermetyzacja uniformów, więc jeśli chcemy wypełnić strukturę, musimy jeszcze ustawić poszczególne uniformy, ale tym razem poprzedzone nazwą struktury:

```cpp
    lightingShader.setVec3("material.ambient",  1.0f, 0.5f, 0.31f);
    lightingShader.setVec3("material.diffuse",  1.0f, 0.5f, 0.31f);
    lightingShader.setVec3("material.specular", 0.5f, 0.5f, 0.5f);
    lightingShader.setFloat("material.shininess", 32.0f);
```

Ustawiamy komponent ambient i diffuse na kolor, który chcemy nadać obiektowi, i ustawiamy `specular` obiektu na średnio jasny kolor; nie chcemy, aby komponent lustrzany był zbyt mocny na tym konkretnym obiekcie. Zachowujemy również `shininess` o wartości `32`. Teraz możemy łatwo wpływać na materiał obiektu z poziomu aplikacji.

Uruchomienie programu powinno dać coś takiego:

![](/img/learnopengl/materials_with_material.png){: .center-image }

To tak naprawdę nie wygląda zbyt dobrze, prawda?

## Właściwości światła

Obiekt jest zbyt jasny. Powód, dla którego obiekt jest zbyt jasny, jest taki, że kolory światła otoczenia, rozproszenia i odbicia są odbijane z pełną siłą dla dowolnego źródła światła. Źródła światła mają również różne natężenia odpowiednio dla ich elementów otoczenia, rozproszenia i odbicia. W poprzednim samouczku rozwiązaliśmy to poprzez zmianę intensywności światła otoczenia i odbicia. Chcemy teraz zrobić coś podobnego, ale tym razem poprzez określenie wektorów intensywności dla każdego z komponentów oświetlenia. Gdybyśmy zwizualizowali <span class="var">lightColor</span> jako `vec3 (1.0)` kod wyglądałby tak:

```glsl
    vec3 ambient  = vec3(1.0) * material.ambient;
    vec3 diffuse  = vec3(1.0) * (diff * material.diffuse);
    vec3 specular = vec3(1.0) * (spec * material.specular); 
```

Tak więc każda właściwość materiału obiektu jest zwracana z maksymalną intensywnością dla każdego ze składników światła. Te wartości `vec3 (1.0)` mogą mieć indywidualny wpływ dla każdego źródła światła i jest to zwykle to, czego chcemy. W tej chwili komponent światła otoczenia obiektu w pełni wpływa na kolor kostki, ale komponent światła otoczenia nie powinien tak bardzo wpływać na ostateczny kolor, więc możemy ograniczyć kolor światła otoczenia poprzez ustawienie natężenia oświetlenia światła na niższą wartość:

```glsl
    vec3 ambient = vec3(0.1) * material.ambient;  
```

Możemy w ten sam sposób wpływać na natężenie źródła światła rozproszonego i lustrzanego. Jest to podobne do tego, co zrobiliśmy w poprzednim samouczku; można powiedzieć, że stworzyliśmy już pewne właściwości światła, aby indywidualnie wpływać na każdy komponent oświetlenia. Będziemy chcieli stworzyć coś podobnego do struktury materiału dla właściwości światła:

```glsl
    struct Light {
        vec3 position;

        vec3 ambient;
        vec3 diffuse;
        vec3 specular;
    };

    uniform Light light;  
```

Źródło światła ma inną intensywność dla komponentów światła <span class="var">ambient</span>, <span class="var">diffuse</span> i <span class="var">specular</span>. Światło otoczenia jest zazwyczaj ustawione na małą intensywność, ponieważ nie chcemy, aby kolor otoczenia był zbyt dominujący. Rozproszona składowa źródła światła jest zwykle ustawiona na dokładnie taką wartość, jaką chcemy aby miało światło; często jest to jasny, biały kolor. Składnik lustrzany jest zwykle przechowywany w `vec3(1.0)` świeci z pełną intensywnością. Zauważ, że dodaliśmy wektor pozycji światła do struktury.

Podobnie jak w przypadku uniforma materiału, musimy zaktualizować Fragment Shader:

```glsl
    vec3 ambient  = light.ambient * material.ambient;
    vec3 diffuse  = light.diffuse * (diff * material.diffuse);
    vec3 specular = light.specular * (spec * material.specular);  
```

Następnie chcemy ustawić intensywność światła w aplikacji:

```cpp
    lightingShader.setVec3("light.ambient",  0.2f, 0.2f, 0.2f);
    lightingShader.setVec3("light.diffuse",  0.5f, 0.5f, 0.5f); // przyciemnij nieco światło, aby pasowało do sceny
    lightingShader.setVec3("light.specular", 1.0f, 1.0f, 1.0f); 
```

Teraz, gdy zmodyfikowaliśmy, w jaki sposób światło ma wpływać na wszystkie materiały obiektów, otrzymujemy wizualny efekt końcowy, które wygląda podobnie do efektu końcowego z poprzedniego samouczka. Tym razem jednak mamy pełną kontrolę nad oświetleniem i materiałem obiektu:

![](/img/learnopengl/materials_light.png){: .center-image }

Zmiana wizualnych aspektów obiektów jest teraz stosunkowo łatwa. Ulepszmy trochę nasze rozwiązanie!

## Różne kolory światła

Do tej pory używaliśmy kolorów światła, aby tylko zmieniać intensywność poszczególnych składników oświetlenia, wybierając kolory w zakresie od białego do szarego do czarnego, nie wpływając na faktyczne kolory obiektu (tylko jego intensywność). Ponieważ mamy teraz łatwy dostęp do właściwości światła, możemy zmieniać ich kolory w czasie, aby uzyskać naprawdę interesujące efekty. Ponieważ wszystko jest już ustawione Fragment Shader, zmiana kolorów światła jest łatwa i natychmiast tworzy pewne ciekawe efekty:

<div align="center"><video width="600" height="450" loop="" controls="">  
<source src="https://learnopengl.com/video/lighting/materials.mp4" type="video/mp4">  
![](/img/learnopengl/materials_light_colors.png){: .center-image}
</video></div>

Jak widać, inny kolor światła ma duży wpływ na kolorystykę obiektu. Ponieważ kolor światła ma bezpośredni wpływ na kolory, które obiekt może odzwierciedlić (jak można to sobie przypomnieć w samouczku Kolory), ma to znaczący wpływ na wizualny efekt końcowy.

Możemy z łatwością zmieniać kolory światła w czasie, zmieniając barwę komponentu otoczenia i rozproszenia za pomocą <span class="fun">sin</span> i <span class="fun">glfwGetTime</span>:

```cpp
    glm::vec3 lightColor;
    lightColor.x = sin(glfwGetTime() * 2.0f);
    lightColor.y = sin(glfwGetTime() * 0.7f);
    lightColor.z = sin(glfwGetTime() * 1.3f);

    glm::vec3 diffuseColor = lightColor   * glm::vec3(0.5f); // decrease the influence
    glm::vec3 ambientColor = diffuseColor * glm::vec3(0.2f); // low influence

    lightingShader.setVec3("light.ambient", ambientColor);
    lightingShader.setVec3("light.diffuse", diffuseColor);
```

Spróbuj poeksperymentować z różnymi wartościami parametrów światła i materiałów i zobacz, jak wpływają one na jakość obrazu. Możesz znaleźć kod źródłowy aplikacji [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/2.lighting/3.1.materials/materials.cpp).

## Ćwiczenia

*   Czy możesz zasymulować niektóre z obiektów w świecie rzeczywistym, definiując ich odpowiednie materiały, jak widzieliśmy na początku tego samouczka? Zwróć uwagę, że wartości komponentu otoczenia [tabelka](http://devernay.free.fr/cours/opengl/materials.html) nie są takie same jak wartości komponentu rozproszenia; nie uwzględniają intensywności światła. Aby poprawnie ustawić ich wartości, musisz ustawić wszystkie intensywności światła na `vec3 (1.0)`, aby uzyskać takie samo wyjście: [rozwiązanie](https://learnopengl.com/code_viewer_gh.php?code=src/2.lighting/3.2.materials_exercise1/materials_exercise1.cpp) cyjanowego plastikowego pojemnika.