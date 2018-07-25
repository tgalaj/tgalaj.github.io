---
layout: post
title: Wprowadzenie
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: intro-learnopengl
---

{% include learnopengl.md link="Introduction" %}

Skoro tutaj trafiłeś, prawdopodobnie chcesz nauczyć się tego jak dokładnie działa grafika komputerowa i robić wszystko to, co fajne dzieciaki robią samemu. Robienie rzeczy samemu to świetna zabawa, która pozwala szybciej i lepiej zrozumieć aspekty programowania grafiki. Jednak przed rozpoczęciem podróży w świat programowania grafiki komputerowej, należy wziąć pod uwagę kilka istotnych elementów.  

## Wymagania wstępne

Ponieważ OpenGL jest graficznym API, a nie samodzielną platformą, to wymagana jest znajomości języka programowania. Wybranym językiem jest C++, dlatego też wymagana jest jego znajomość, by móc efektywnie korzystać z tego kursu. Spróbuję jednak wyjaśnić większość zastosowanych pojęć, włącznie z zaawansowanymi aspektami C++ więc nie musisz być ekspertem w C++, ale powinieneś umieć napisać coś więcej niż tylko program 'Hello World'. Jeśli nie masz wystarczająco dużo doświadczenia z C++, to mogę zaproponować następujący, bezpłatny kurs [cpp0x.pl](http://cpp0x.pl/kursy/Kurs-C++/1 "undefined").

Również będziemy po drodze używać trochę matematyki (algebra liniowa, geometria i trygonometria) i będę próbował wyjaśnić wszystkie wymagane pojęcia potrzebne do zrozumienia danej lekcji. Z uwagi na to, że nie jestem matematykiem, to mimo tego, że moje wyjaśnienia mogą być łatwe do zrozumienia, to najprawdopodobniej będą niekompletne. W razie potrzeby przedstawię odnośniki do dobrych zasobów, które wyjaśniają matematykę w bardziej kompletny sposób. Nie przejmuj się potrzebną wiedzą matematyczną przed rozpoczęciem podróży w świat OpenGL. Prawie wszystkie pojęcia można zrozumieć z podstawową wiedzą matematyczną i postaram się zminimalizować matematykę w miarę możliwości. Większość funkcjonalności nie wymaga nawet zrozumienia całej matematyki, o ile wiesz, jak ją używać.

## Struktura

LearnOpenGL jest podzielony na kilka ogólnych tematów. Każdy temat zawiera kilka sekcji, które w bardzo szczegółowy sposób tłumaczą różne koncepcje. Każdy z przedmiotów można znaleźć w menu po lewej stronie (na stronie rtrclass.type.pl w górnym menu w sekcji LearnOpenGL.com). Tematy są przygotowane w sposób liniowy (zaleca się, aby rozpocząć od początku do końca, o ile nie wskazano inaczej), gdzie każda lekcja wyjaśnia tło teoretyczne i aspekty praktyczne.

Aby ułatwić śledzenie samouczków i nadanie im dodatkowej struktury, witryna zawiera _pola blokowe, bloki kodu, wskazówki kolorystyczne_ i _odwołania do funkcji_.

### Pola blokowe

{: .box-note }
**Niebieskie** pola blokowe zawierają kilka uwag lub przydatnych funkcji/wskazówek dotyczących OpenGL lub aktualnie omawianego tematu.

{: .box-error }
**Czerwone** pola blokowe zawierają ostrzeżenia lub inne funkcje, z którymi musisz być ostrożny.

### Kod

Znajdziesz mnóstwo małych fragmentów kodu w samouczkach, które znajdują w polach blokowych, a kod jest składniowo pokolorowany, co można zobaczyć poniżej:

```cpp 
// To pole zawiera kod  
```

Ponieważ dostarczają one tylko fragmenty kodu, gdziekolwiek będzie to konieczne, dostarczę odnośnik do całego kodu źródłowego wymaganego dla danego zagadnienia.

### Wskazówki kolorystyczne

Niektóre słowa są wyświetlane w innym kolorze, aby jasno pokazać, że te słowa posiadają specjalne znaczenie:

* <span class="def">Definicja:</span> zielone słowa określają definicję, t.j. ważny aspekt/nazwę, którą prawdopodobnie usłyszysz częściej.  
* <span class="fun">Logika programu:</span>czerwone słowa określają nazwy funkcji lub nazwy klas.  
* <span class="var">Zmienne:</span>niebieskie słowa określają zmienne jak i wszystkie stałe OpenGL.

### Odwołania do funkcji OpenGL

{: .box-warning}
W tym tłumaczeniu, niżej opisana funkcjonalność nie jest wspierana.

Szczególnie dobrze docenianą funkcją LearnOpenGL jest możliwość sprawdzania znaczenia większości funkcji OpenGL wszędzie tam, gdzie pojawja się ona w treści. Zawsze, gdy funkcja znajduje się w treści, która jest udokumentowana w witrynie, funkcja pojawi się z lekko zauważalnym podkreśleniem. Możesz przesunąć kursor myszy nad funkcję, a po krótkim odstępie czasowym, pojawi się okno, które pokaże istotne informacje o tej funkcji, w tym informacje na temat tego, co funkcja rzeczywiście robi. Najedź myszą na <span class="fun">glEnable</span>, aby zobaczyć to w akcji.

Teraz, kiedy znasz strukturę witryny, przejdź do sekcji Pierwsze Kroki, by rozpocząć podróż w świat OpenGL!
