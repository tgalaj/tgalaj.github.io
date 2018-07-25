---
layout: post
title: strutil - simple std::string utility library
tags: [blog, projects]
gh-repo: shot511
gh-badge: [follow]
---

I was tired of re-implementing the same missing functionality over and over again, so I decided to put it in an easy to use, header only C++ 11 std::string utility library called **strutil**.

Link to the project's repository can be found in the [**Projects**]({{ site.baseurl }}/pages/projects) page. So feel free to grab the header file and include it in your project. strutil has the following features:

* Generic parsing methods - from std::string and to std::string.
* Splitting std::string to tokens with user defined delimiter (useful for CSV parsing).
* Replace a substring with another substring.
* Text manipulation functions: *repeat* (char or std::string), *to_lower*, *to_upper*, *trim* (also in-place).
* Checks: *contains*, *starts_with*, *ends_with*, *matches*.
* Compare two std::string with their case ignored. 
* Header only library - no building required.
* Does not require any dependencies.

I would also point out that it is very welcome to report bugs or new improvements/features - I'm open to any suggestions.
