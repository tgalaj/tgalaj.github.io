---
layout: post
title: Repozytorium kodu
subtitle: LearnOpenGL.com
tags: [learnopengl, tutorial]
subtag: intro-learnopengl-2
---

{% include learnopengl.md link="Code-repository" %}

Możesz znaleźć wszystkie odpowiednie przykłady kodu online w każdym tutorialu, ale jeśli chcesz szybko uruchomić dema samouczków lub porównać swój kod z działającymi przykładami, możesz znaleźć repozytorium kodu [tutaj](https://github.com/JoeyDeVries/LearnOpenGL) na Github.

W tej chwili plik `CMakeLists.txt` może poprawnie generować pliki projektów Visual Studio, Makefile i działa zarówno w systemie Windows, jak i Linux. Nie był on szeroko testowany na systemie Apple X OS X, ani na wszystkich IDE, więc możesz zostawić komentarz lub zaktualizować `CMakeLists.txt` za pomocą pull request, jeśli uruchomisz ten kod na innych systemach/IDE.

Chciałbym podziękować Zwookiemu za ogromną pomoc przy tworzeniu skryptu CMake na systemie Linux. Dzięki aktualizacjom Zwookiego plików CMakeLists, CMake z powodzeniem generuje pliki projektów zarówno w systemie Windows, jak i Linux.

Sprawdź także projekt [Glitter](https://github.com/Polytonic/Glitter) autorstwa Polytonic, który jest bardzo prostym projektem dla tych samouczków, który jest wstępnie skonfigurowany z odpowiednimi bibliotekami.