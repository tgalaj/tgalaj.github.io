---
layout: post
title: Mapy oświetlenia
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: oswietlenie
---

{% include learnopengl.md link="Lighting/Lighting-maps" %}

W [poprzednim]({% post_url /learnopengl/2_lighting/2018-08-03-materialy %}) tutorialu omawialiśmy możliwość posiadania przez każdy obiekt własnego, unikalnego materiału, który odpowiednio reaguje na światło. Wspaniale nadaje to każdemu obiektowi unikalny wygląd w porównaniu z innymi obiektami w oświetlonej scenie, ale nadal nie zapewnia zbyt dużej elastyczności w kontrolowaniu wizualnego efektu.

W poprzednim tutorialu zdefiniowaliśmy materiał dla całego obiektu, ale obiekty w świecie rzeczywistym zwykle nie składają się z jednego materiału, ale składają się z kilku materiałów. Pomyśl o samochodzie: jego zewnętrzna część składa się z błyszczącego materiału, ma okna, które częściowo odbijają otaczające środowisko, jego opony są lśniące, więc nie mają refleksów i mają bardzo błyszczące obręcze (jeśli dobrze myjesz swój samochód). Samochód ma również składowe kolorów otoczenia i rozproszenia, które nie są takie same dla całego obiektu; samochód posiada wiele różnych składowych kolorów otoczenia/rozproszenia. W sumie taki obiekt ma różne właściwości materiałowe dla każdej z jego części.

Zatem system materiałów w poprzednim samouczku nie jest wystarczający dla wszystkich modeli, dlatego musimy rozszerzyć poprzedni system, wprowadzając mapy _diffuse_ i _specular_. Dzięki temu możemy wpływać na składową rozproszenia (i pośrednio na składową otoczenia, ponieważ prawie zawsze są takie same) oraz na komponent lustrzany obiektu z znacznie większą precyzją.

# Mapy diffuse (ang. *diffuse maps*)

To co chcemy zrobić to, móc w jakiś sposób ustawić rozproszony komponent koloru obiektu dla każdego pojedynczego fragmentu. Jakiś system, w którym możemy uzyskać wartość koloru na podstawie położenia fragmentu na obiekcie.

To prawdopodobnie powinno brzmieć bardzo znajomo i szczerze mówiąc, używamy takiego systemu już od jakiegoś czasu. Brzmi to bardzo podobnie do _tekstury_, o których obszernie dyskutowaliśmy w jednym z [wcześniejszych]({% post_url /learnopengl/1_getting_started/2017-09-11-tekstury %}) tutoriali  i zasadniczo jest to po prostu: tekstura. Używamy tylko innej nazwy dla tej samej podstawowej zasady: za pomocą obrazu owiniętego wokół obiektu, który możemy indeksować dla unikalnych wartości kolorów dla każdego z fragmentów. W oświetlonych scenach nazywa się to zwykle <span class="def">mapą diffuse</span> (ang. *diffuse map*) (tak ogólnie nazywają to artyści 3D), ponieważ obraz tekstury reprezentuje wszystkie składowe rozproszenia koloru obiektu.

Aby zademonstrować mapę diffuse, użyjemy [następującego obrazu](/img/learnopengl/container2.png) drewnianego pojemnika ze stalową obwódką:

![](/img/learnopengl/container2.png){: .center-image }

Używanie mapy diffuse w Fragment Shader jest dokładnie takie samo, jak w tutorialu dotyczącym tekstur. Tym razem jednak przechowujemy teksturę jako `sampler2D` wewnątrz struktury <span class="fun">Material</span>. Zastępujemy wcześniej zdefiniowany wektor koloru `vec3` z mapą diffuse.

{: .box-error }
Należy pamiętać, że `sampler2D` jest tak zwanym <span class="def">typem nieprzezroczystym</span> (ang. *opaque type*), co oznacza, że ​​nie możemy instancjonować tych typów, ale możemy je tylko zdefiniować jako uniformy. Gdybyśmy stworzyli tę strukturę inną niż uniform (jak np. parametr funkcji), GLSL mógłby rzucić dziwne błędy; to samo odnosi się do każdej struktury posiadającej takie nieprzezroczyste typy.

Usuwamy również wektor koloru otoczenia materiału, ponieważ kolor otoczenia jest prawie we wszystkich przypadkach równy kolorowi rozproszenia, więc nie ma potrzeby przechowywania go oddzielnie:

```glsl
    struct Material {
        sampler2D diffuse;
        vec3      specular;
        float     shininess;
    }; 
    ...
    in vec2 TexCoords;
```

{: .box-note }
Jeśli jesteś trochę uparty i nadal chcesz ustawić kolory otoczenia na inną wartość (inną niż wartość rozproszenia), możesz zachować ambient `vec3`, ale wtedy kolory otoczenia pozostaną takie same dla całego obiektu. Aby uzyskać różne wartości otoczenia dla każdego fragmentu, musisz użyć innej tekstury tylko dla wartości otoczenia.

Zauważ, że będziemy potrzebować ponownie współrzędnych tekstury w Fragment Shader, więc zadeklarowaliśmy dodatkową zmienną wejściową. Następnie po prostu próbkujemy teksturę, aby uzyskać wartość rozproszonego koloru fragmentu:

```glsl
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));  
```

Nie zapomnij również ustawić koloru otoczenia materiału równego kolorowi rozproszenia materiału:

```glsl
    vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
```

To wystarczy, aby używać mapy diffuse. Jak widać, nie jest to nic nowego, ale zapewnia dramatyczny wzrost jakości obrazu. Aby to działało, musimy zaktualizować dane wierzchołków o dane współrzędnych tekstury, przenieść je jako atrybuty wierzchołków do Fragment Shader, załadować teksturę i powiązać teksturę z odpowiednią jednostką tekstury.

Zaktualizowane dane wierzchołków można znaleźć [tutaj](https://learnopengl.com/code_viewer.php?code=lighting/vertex_data_textures). Dane wierzchołków zawierają teraz pozycje wierzchołków, wektory normalne i współrzędne tekstury dla każdego z wierzchołków sześcianu. Zaktualizujmy Vertex Shader, aby przyjmował współrzędne tekstury jako atrybut wierzchołka i by przekazał je do Fragment Shader:

```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aNormal;
    layout (location = 2) in vec2 aTexCoords;
    ...
    out vec2 TexCoords;

    void main()
    {
        ...
        TexCoords = aTexCoords;
    }  
```

Upewnij się, że zaktualizowałeś wskaźniki atrybutów wierzchołków obu VAO, aby dopasować je do nowych danych wierzchołków i załadować obraz kontenera jako teksturę. Przed narysowaniem kontenera chcemy przypisać preferowaną jednostkę tekstury do uniformu samplera <span class="var">material.diffuse</span> i powiązać teksturę kontenera z tą jednostką tekstury:

```cpp
    lightingShader.setInt("material.diffuse", 0);
    ...
    glActiveTexture(GL_TEXTURE0);
    glBindTexture(GL_TEXTURE_2D, diffuseMap);
```

Teraz, korzystając z mapy diffuse, ponownie uzyskujemy ogromny wzrost szczegółowości i tym razem z dodatkowym oświetleniem pojemnik naprawdę zaczyna lśnić (całkiem dosłownie). Twój kontener prawdopodobnie wygląda teraz mniej więcej tak:

![](/img/learnopengl/materials_diffuse_map.png){: .center-image }

Możesz znaleźć pełny kod źródłowy aplikacji [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/2.lighting/4.1.lighting_maps_diffuse_map/lighting_maps_diffuse.cpp).

# Mapy specular (ang. *specular maps*)

Prawdopodobnie zauważyłeś, że rozbłyski wyglądają nieco przygaszone, ponieważ nasz obiekt to pojemnik, który składa się głównie z drewna i wiemy, że drewno nie zbyt dużych rozbłysków. Możemy to naprawić, ustawiając materiał lustrzany obiektu na `vec3(0.0)`, ale to oznaczałoby, że stalowe krawędzie pojemnika przestałyby również pokazywać rozbłyski światła, a także wiemy, że stal **powinna** pokazywać rozbłyski. Ponownie, chcielibyśmy kontrolować, które części obiektu powinny pokazywać rozbłyski o różnym natężeniu. Jest to problem, który wygląda bardzo podobnie do map diffuse. Przypadek? Nie sądzę.

Możemy również użyć tekstur tylko dla rozbłysków. Oznacza to, że musimy wygenerować czarno-białą (lub kolorową, jeśli chcemy) teksturę, która zdefiniuje intensywność rozbłysku każdej części obiektu. Przykładem [mapy specular](/img/learnopengl/container2_specular.png) jest następujący obraz:

![](/img/learnopengl/container2_specular.png){: .center-image }

Intensywność oświetlenia zwierciadlanego jest uzyskiwana dzięki jasności każdego piksela na obrazie. Każdy piksel mapy specular może być wyświetlany jako wektor koloru, gdzie  na przykład, czarny reprezentuje wektor koloru `vec3(0.0)` i szary wektor koloru `vec3(0.5)`. W Fragment Shader próbkujemy odpowiednią wartość koloru i mnożymy tę wartość z intensywnością oświetlenia lustrzanego. Im "bielszy" jest piksel, tym wyższy jest wynik mnożenia, a tym samym jaśniejszy staje się komponent lustrzany obiektu.

Ponieważ pojemnik składa się głównie z drewna, a drewno jako materiał nie powinno mieć żadnych błyszczących punktów, cała drewniana część tekstury diffuse została zamieniona na czerń: czarne sekcje nie mają żadnych błyszczących punktów. Stalowa krawędź pojemnika ma zmienną intensywność lustrzaną, przy czym sama stal jest stosunkowo podatna na odblaski światła, podczas gdy pęknięcia nie są.

{: .box-note }
Z technicznego punktu widzenia drewno ma również refleksy, chociaż o znacznie mniejszej wartości połysku (bardziej rozprasza światło) i mniejszym wpływie. Jednak, dla celów edukacyjnych możemy po prostu udawać, że drewno reaguje na światło zwierciadlane.

Za pomocą takich narzędzi jak _Photoshop_ lub _Gimp_ stosunkowo łatwo można przekształcić teksturę diffuse na teksturę specular, w ten sposób, że wycinając niektóre części, przekształcając je na czarno-białe i zwiększając jasność/kontrast.

## Próbkowanie map specular

Mapa specular jest podobna do każdej innej tekstury, więc kod jest podobny do kodu mapy diffuse. Upewnij się, że prawidłowo załadowałeś obraz i wygenerowałeś obiekt tekstury. Ponieważ używamy innego samplera tekstury w tym samym Fragment Shader, musimy użyć innej jednostki tekstury (patrz [Tekstury]({% post_url /learnopengl/1_getting_started/2017-09-11-tekstury %})) dla mapy specular, zatem powiążmy go z odpowiednią jednostką tekstury przed renderowaniem:

```cpp
    lightingShader.setInt("material.specular", 1);
    ...
    glActiveTexture(GL_TEXTURE1);
    glBindTexture(GL_TEXTURE_2D, specularMap);  
```

Następnie zaktualizuj właściwości materiału w Fragment Shader, aby akceptował `sampler2D` jako jego składnik lustrzany zamiast `vec3`:

```glsl
    struct Material {
        sampler2D diffuse;
        sampler2D specular;
        float     shininess;
    };  
```

Na koniec chcemy spróbkować mapę specular, aby pobrać odpowiednią intensywność lustrzaną dla fragmentu:

```glsl
    vec3 ambient  = light.ambient  * vec3(texture(material.diffuse, TexCoords));
    vec3 diffuse  = light.diffuse  * diff * vec3(texture(material.diffuse, TexCoords));  
    vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
    FragColor = vec4(ambient + diffuse + specular, 1.0);   
```

Korzystając z map specular możemy określić z ogromną dokładnością, jakie części obiektu faktycznie mają właściwości _rozbłyskowe_. Możemy nawet ustawić ich odpowiednią intensywność. Mapy specular dają nam dodatkową warstwę kontroli.

{: .box-note }
Jeśli nie chcesz podążać z nurtem, możesz również użyć rzeczywistych kolorów na mapie specular, aby nie tylko ustawić intensywność rozbłysku każdego fragmentu, ale także kolor oświetlenia zwierciadlanego. Realistycznie, jednak kolor oświetlenia jest przeważnie określony przez samo źródło światła, więc nie generuje realistycznych wizualizacji (dlatego obrazy są zazwyczaj czarno-białe: zależy nam tylko na intensywności).

Jeśli teraz uruchomisz aplikację, wyraźnie zobaczysz, że materiał pojemnika jest teraz podobny do rzeczywistego drewnianego pojemnika z stalowymi ramami:

![](/img/learnopengl/materials_specular_map.png){: .center-image }

Możesz znaleźć pełny kod źródłowy aplikacji [tutaj](https://learnopengl.com/code_viewer_gh.php?code=src/2.lighting/4.2.lighting_maps_specular_map/lighting_maps_specular.cpp).

Dzięki mapom diffuse i specular możemy naprawdę dodać ogromną ilość detali do stosunkowo prostych obiektów. Możemy nawet dodać więcej szczegółów do obiektów przy użyciu innych map, takich jak <span class="def">map normalnych</span> (ang. *normal/bump maps*) i/lub <span class="def">map odbić</span> (ang. *reflection maps*), ale to jest coś, co zarezerwujemy na późniejsze samouczki. Pokaż swój pojemnik wszystkim znajomym i rodzinie i zadowalaj się faktem, że nasz pojemnik może pewnego dnia stać się jeszcze ładniejszym niż już jest!

## Ćwiczenia

*   Powygłupiaj się z wektorami komponentów światła otoczenia, rozproszenia i lustrzanego i zobacz, jak wpływają one wizualnie na pojemnik.
*   Spróbuj odwrócić wartości kolorów mapy specular w Fragment Shader tak, aby drewno pokazywało rozbłyski, a stalowe obramowania nie (zauważ, że ze względu na pęknięcia w stalowej ramce granice wciąż wykazują pewne odbicie, choć z mniejszą intensywnością) : [rozwiązanie](https://learnopengl.com/code_viewer.php?code=lighting/lighting_maps-exercise2).
*   Spróbuj utworzyć mapę specular z tekstury diffuse, która używa rzeczywistych kolorów zamiast czerni i bieli i zobacz, że wynik nie wygląda zbyt realistycznie. Możesz użyć tej [kolorowej mapy specular](https://learnopengl.com/img/lighting/lighting_maps_specular_color.png), jeśli nie możesz sam jej wygenerować: [wynik](https://learnopengl.com/img/lighting/lighting_maps_exercise3.png).
*   Dodaj też coś, co jest nazywane <span class="def">mapą emisji</span>, która jest teksturą przechowującą wartości emisji na każdy fragment. Wartości emisji są kolorami, które obiekt może _emitować_, jak gdyby sam w sobie zawierał źródło światła; w ten sposób obiekt może świecić niezależnie od warunków oświetleniowych. Mapy emisji są często tym, co widzisz, gdy obiekty w grze świecą (jak [oczy robota](http://www.witchbeam.com.au/unityboard/shaders_enemy.jpg) lub [paski świetlne na pojemniku](http://www.tomdalling.com/images/posts/modern-opengl-08/emissive.png)). Dodaj [tę](https://learnopengl.com/img/textures/matrix.jpg) teksturę (autorstwa creativesam) jako mapę emisji do kontenera, tak jakby litery emitowały światło: [rozwiązanie](https://learnopengl.com/code_viewer_gh.php?code=src/2.lighting/4.3.lighting_maps_exercise4/lighting_maps_exercise4.cpp); [wynik](https://learnopengl.com/img/lighting/lighting_maps_exercise4.png).