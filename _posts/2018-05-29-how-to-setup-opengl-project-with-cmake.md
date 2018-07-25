---
layout: post
title: How to setup OpenGL project with CMake
tags: [cmake, other, opengl, tutorial]
---

I decided to create this tutorial about CMake and OpenGL since I couldn't find many articles about this specific topic on the Internet. Most CMake tutorials focus on the very basic usage - one file *main.cpp* and then create project with one CMake function *add_executable* and that's it. That was not enough for me since setting up an OpenGL project is quite cumbersome and requires some additional libraries to configure.

Therefore, I've dived into this topic and with some luck I was able to create working CMake script, that will build a project for IDE of my choice, which in my case is *Visual Studio Community*.

I would like to state I'm not a CMake expert, so any improvements/comments/etc. to this tutorial and the CMake code are greatly welcome.

## Prerequisites
First of all, we have to have CMake installed and added to the system PATH variable. You can download CMake [here](https://cmake.org/download/).

Secondly, we need some project files that we can work with. For this purpose, I've prepared a small package with basic OpenGL project that opens a window and renders an object. You can download this sample project [here][sample-project-zip].

What is more, this project contains the following dependencies:
* GLAD
* GLFW3
* GLM
* stb_image

The one dependency that is missing is Assimp library (responsible for loading 3D models). It was left on purpose - we will learn how to use CMake by building Assimp ourselves.

## First step: using CMake
### Download sources
To be able to load 3D models with the pre-made project we need Assimp library. Authors of this library don't distribute binary files. Therefore, we have to build it from given source files. You can download Assimp's source files from the [original page](http://www.assimp.org/). Then just simply click the *Download* button, and you will be redirected to their download page. Then, go ahead and download *Source code (zip)*.

{: .box-note }
At the time, when this article was written, the newest version of Assimp was 4.1.0.

### Building Assimp
When we have downloaded the source files, we can extract them. Now, in the location where the extracted files are, create new folder and call it *build*. This folder will contain the CMake generated Visual Studio files.

To create Visual Studio solution files using CMake, we have to options:
1. use command line
2. use CMake GUI

Here, I will show how to use both approaches.

### Using command line
Simply open command line tool inside the newly created *build* directory and type:

```
cmake -G "Visual Studio 15" -DLIBRARY_SUFFIX="" ..
```

And that's it. Generated Visual Studio solution files should be inside the *build* folder.

### Using CMake GUI
Run **CMake-gui** application. You should see a window similar to this one:

{% include lightbox src="img/cmake/cmake-gui.png" data="data" img-style="max-width:60%;" class="center-image" %}

Then fill two boxes with proper paths:
* *Where is the source code*
* *Where to build the binaries*

I think that these boxes are self-explanatory. Then, you should have something like this:

{% include lightbox src="img/cmake/cmake-gui-paths.png" data="data" img-style="max-width:60%;" class="center-image" %}

Now click on the *Configure* button and select the generator for our project. In our case it is *Visual Studio 15 2017*. Then click *Finish*.

{% include lightbox src="img/cmake/cmake-gui-generator.png" data="data" img-style="max-width:60%;" class="center-image" %}

When configuration step has completed, make sure that **LIBRARY_SUFFIX** is empty:

{% include lightbox src="img/cmake/cmake-gui-variables.png" data="data" img-style="max-width:60%;" class="center-image" %}

Now you can click on the *Generate* button to generate Visual Studio solution files. CMake will save them inside the *build* folder.

### Building binaries
If you generated Visual Studio solution files using one of the above methods, go to the *build* folder and open **Assimp.sln**. Then, right click on the *assimp* project and select *Set as StartUp project*.

{% include lightbox src="img/cmake/vs-startup.png" data="data" img-style="max-width:60%;" class="center-image" %}

Now select the build configuration to **Release** and platform to **Win32** and build the project (press F7 or choose Build->Build solution).

{% include lightbox src="img/cmake/vs-config.png" data="data" img-style="max-width:60%;" class="center-image" %}

When everything is ok (I hope it is), you should see a message in Visual Studio console:

{% include lightbox src="img/cmake/vs-output.png" data="data" img-style="max-width:70%;" class="center-image" %}

### One more step
When you have successfully built Assimp binaries, you can do the following steps:

1. Copy **assimp.lib** from *build/code/Release* to *lib* folder of our Sample project.
2. Create *dlls* folder in the root directory of our Sample project.
3. Copy **assimp.dll** from *build/code/Release* to *dlls* folder of our Sample project.
4. Copy *build/include/assimp* folder to *include* folder of our Sample project.
5. Copy *include/assimp* folder to *include* folder of our Sample project.

That's it. We can now proceed to writing a CMake script that will build Visual Studio (or any other) solution for Simple project.

## Second step: CMake script for our Simple project
Finally, we can talk something about core of this tutorial - creating a CMake script.

First of all, create file *CMakeLists.txt* in the root directory of Sample project. In this file we will define the build rules for our project.

Every CMake script should define the minimum required version of CMake and name of the project (and in our case also a solution name). Let's set the minimum CMake version to 3.2 and project name to *OpenGLExample*.

```cmake
cmake_minimum_required(VERSION 3.2 FATAL_ERROR)
project(OpenGLExample)
``` 

### Source files and libraries

Now we will create two CMake variables to hold _*.c/*.cpp_ and _*.h/*.hpp_ source files. First variable will be called *SOURCE_FILES* and second *HEADER_FILES*. To do so, we will use *file* command with *GLOB_RECURSE* parameter.

```cmake
# Add source files
file(GLOB_RECURSE SOURCE_FILES 
	${CMAKE_SOURCE_DIR}/src/*.c
	${CMAKE_SOURCE_DIR}/src/*.cpp)
	
# Add header files
file(GLOB_RECURSE HEADER_FILES 
	${CMAKE_SOURCE_DIR}/src/*.h
	${CMAKE_SOURCE_DIR}/src/*.hpp)
```

The *file* command can take as a first parameter a lot of options, but we will focus only on two of them:
* **GLOB** - it will generate a list of files that match the globbing expression, which is very similar to regular expressions.
* **GLOB_RECURSE** - it works the same as the **GLOB** with exception that it will traverse all subdirectories of the matched directory and will match the files.

So, in our case using *GLOB_RECURSE* is much convenient as we store source files in different subdirectories. However, if we would like to use *GLOB*, we should add these subdirectories to the *file* command ourselves.

You can also notice the global *CMAKE_SOURCE_DIR* variable. This variable points to the folder where CMakeLists.txt file is located. CMake defines much more global variables which you can view [here](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html).

In the next step, we specify where linker should look for static libraries for our project. It's done using *link_directories* command and passing the directory as a parameter.

```cmake
# Add .lib files
link_directories(${CMAKE_SOURCE_DIR}/lib)
```

### Resources
Every OpenGL project needs some additional resources which hold different data - 3D models, textures, etc. According to my knowledge, CMake doesn't have a nice and clean way to specify the resources directory. However, the best way to overcome this problem is to use *configuration file*. This file will moved by CMake to different folder with changed content. 

Using this knowledge, we will create a configuration file, that after modified by CMake, will contain a C++ macro that will hold an absolute path to the root directory of our project (where CMakeLists.txt resides).

To do so, create *helpers* folder inside *src* folder, and then create file *RootDir.h.in* in the newly created folder. Then, type the following code to our configuration file.

```cpp
#pragma once
#define ROOT_DIR "@CMAKE_SOURCE_DIR@/"
```

As you can see, we referenced a CMake variable *@CMAKE_SOURCE_DIR@*. This variable will be modified by CMake with the absolute path, where the file CMakeLists.txt is located.

We also need to tell CMake where is our configuration file, and where CMake should put it. To do so, we call a command *configure_file*.

```cmake
# Configure assets header file
configure_file(src/helpers/RootDir.h.in src/helpers/RootDir.h)
include_directories(${CMAKE_BINARY_DIR}/src)
```

### Executable
When we have our source files configured, it's time to configure out main project that will create the executable. All we need to do is to call the following function:

```cmake
# Define the executable
add_executable(${PROJECT_NAME} ${HEADER_FILES} ${SOURCE_FILES})
```

It will create the project with a name that was specified using *project* command with source and header files saved in variables SOURCE_FILES and HEADER_FILES.

### External libraries
The last big thing that we need to do, in order to complete the CMake script is to tell CMake what additional libraries are required by our project, and where these libraries can be found. All of these can be accomplished with *find_package* command.

This command take as a parameter a module name. Modules in CMake aid with finding various libraries and packages. CMake, by default, supports a lot of well-known libraries such as OpenGL. You can list all modules that your version of CMake supports by typing:

```
cmake help-module-list
```

Unfortunately, CMake doesn't support by default every library (that would be impossible). However, we can create our modules (or download it from the web). Fortunately, our Simple project comes with 3 modules that will allow us to find Assimp, GLM and GLFW3 libraries.

First we have to tell CMake where our modules are.

```cmake
# We need a CMAKE_DIR with some code to find external dependencies
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
```

Then we can tell CMake to find desired libraries.

```cmake
# OpenGL
find_package(OpenGL REQUIRED)

# GLM
find_package(GLM REQUIRED)
message(STATUS "GLM included at ${GLM_INCLUDE_DIR}")

# GLFW
find_package(GLFW3 REQUIRED)
message(STATUS "Found GLFW3 in ${GLFW3_INCLUDE_DIR}")

# ASSIMP
find_package(ASSIMP REQUIRED)
message(STATUS "Found ASSIMP in ${ASSIMP_INCLUDE_DIR}")
```

What about GLAD and stb_image? These libraries are header-only libraries so for them we will create separate targets (projects) in our solution.

```cmake
# STB_IMAGE
add_library(STB_IMAGE "thirdparty/stb_image.cpp")

# GLAD
add_library(GLAD "thirdparty/glad.c")
```

Then, we will save all found directories into a single variable *LIBS*.

```cmake
# Put all libraries into a variable
set(LIBS glfw3 opengl32 assimp STB_IMAGE GLAD)
```

The last thing that we will do is to link selected libraries with our main project and define include directories, that will be used by compiler in order to search for include files.

```cmake
# Define the include DIRs
include_directories(
	"${CMAKE_SOURCE_DIR}/src"
	"${CMAKE_SOURCE_DIR}/include"
)

# Define the link libraries
target_link_libraries(${PROJECT_NAME} ${LIBS})
```

### Visual Studio filters (folders)
All of the above steps create the valid CMake script that will create a valid Visual Studio project. But to make this project look a little bit nicer in our IDE we will create a macro (and then we will run it) to put source and header files in Visual Studio filters (folders).

```cmake
# Create virtual folders to make it look nicer in VS
if(MSVC_IDE)
	# Macro to preserve source files hierarchy in the IDE
	macro(GroupSources curdir)
		file(GLOB children RELATIVE ${PROJECT_SOURCE_DIR}/${curdir} ${PROJECT_SOURCE_DIR}/${curdir}/*)

		foreach(child ${children})
			if(IS_DIRECTORY ${PROJECT_SOURCE_DIR}/${curdir}/${child})
				GroupSources(${curdir}/${child})
			else()
				string(REPLACE "/" "\\" groupname ${curdir})
				string(REPLACE "src" "Sources" groupname ${groupname})
				source_group(${groupname} FILES ${PROJECT_SOURCE_DIR}/${curdir}/${child})
			endif()
		endforeach()
	endmacro()

	# Run macro
	GroupSources(src)
endif()
```

### Auto-copy DLLs
After creating the Visual Studio solution using CMake script defined above, we will have to copy ourselves the required DLLs to correct directories. In order to omit this problem, we can define a custom command that will automatically copy the selected DLLs (or the whole folder) to the correct place. This command will be executed after the process of building the project.

```cmake
# Copy dlls
if(WIN32)
	add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_directory
		"${PROJECT_SOURCE_DIR}/dlls"
		$<TARGET_FILE_DIR:${PROJECT_NAME}>)
endif()
```

### Build the Sample project
In order to test our script, let's build our Sample project. To do so, you can use command line tool or cmake-gui application. Here, I'm gonna use the command line tool.

So let's call this set of commands in the root directory of our project.

```
mkdir build
cd build
cmake -G "Visual Studio 15" ..
```

This will create Visual Studio solution inside the build directory. Now, open *OpenGLExample.sln* solution and run the *OpenGLExample* project. You should see a window with rotating statue.

{% include lightbox src="img/cmake/final.png" data="data" img-style="max-width:60%;" class="center-image" %}

## The end
If you had any troubles creating your CMakeLists.txt script you can reference the script created especially for this tutorial [here](https://github.com/Shot511/OpenGLSampleCmake/blob/master/CMakeLists.txt).

You can also download the final Sample project with CMake script from the [GitHub repository](https://github.com/Shot511/OpenGLSampleCmake).

If you noticed any errors or typos, or something wasn't working just let me know in the comment section down below. Also, any improvements/comments/etc. to this tutorial and the CMake code are greatly welcome.

## Source code
* [Sample project][sample-project-zip]
* [Final CMake project on GitHub](https://github.com/Shot511/OpenGLSampleCmake)

## References
1. CMake official [documentation](https://cmake.org/cmake/help/latest/)
2. CMake Community [Wiki](https://gitlab.kitware.com/cmake/community/wikis/home)

[sample-project-zip]: https://drive.google.com/file/d/13FSX7GcrVXpGMM1zUwJnjwinS8Ym5G-H/view?usp=sharing