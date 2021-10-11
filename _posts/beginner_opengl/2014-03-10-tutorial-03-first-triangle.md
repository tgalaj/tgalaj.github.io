---
layout: post
title: Tutorial 03 - First triangle
subtitle: Beginner OpenGL course
tags: [beginner-opengl-en, tutorial]
---

## Introduction

In this tutorial, we'll draw the first triangle using OpenGL. If you were one of few people who tried to do the "homework" from the previous part, you can verify your code below:

<details class="panel panel-success">
  <summary markdown="span" class="panel-heading">
    Homework
  </summary>

```cpp 
#include <GL/glew.h>
#include <GLFW/glfw3.h>

GLFWwindow* window;

int init(int width, int height)
{
    /* Initialize the library */
    if (!glfwInit())
        return -1;

    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(width, height, "Hello Triangle", NULL, NULL);

    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    /* Make the window's context current */
    glfwMakeContextCurrent(window);

    /* Initialize GLEW */
    if(glewInit() != GLEW_OK)
        return -1;

    return true;
}

void render(float tpf)
{
    //Render here
}

void update()
{
    float oldTime = 0.0f;
    float newTime = 0.0f;
    float gameTime = 0.0f;

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Update game time value */
        oldTime = newTime;
        newTime = (float)glfwGetTime();
        gameTime =  newTime - oldTime;

        /* Render here */
        render(gameTime );

        /* Swap front and back buffers */
        glfwSwapBuffers(window);

        /* Poll for and process events */
        glfwPollEvents();
    }
}

int main(void)
{
    if(!init(640, 480))
        return -1;

    update();
    glfwTerminate();

    return 0;
}
```

</details>

The code from above "answer" will be used as a basis for this tutorial. So, if you haven't done the "homework" from the previous part you can safely copy this code to your project. Let's get started then! :)

## Code of the application

Traditionally, I will first give you the code to go through and then I'll analyze and "translate" it to the human language.

```cpp 
#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <glm/glm.hpp>

GLFWwindow* window;

/* Initialize vertices of our triangle */
glm::vec3 vertices[] = { glm::vec3( 0.0f,  1.0f, 0.0f),
                         glm::vec3( 1.0f, -1.0f, 0.0f),
                         glm::vec3(-1.0f, -1.0f, 0.0f)
                       };

/* Initialize Vertex Buffer Object */
GLuint VBO = NULL;

int init(int width, int height)
{
    /* Initialize the library */
    if (!glfwInit())
        return -1;

    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(width, height, "Hello Triangle", NULL, NULL);

    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    /* Make the window's context current */
    glfwMakeContextCurrent(window);

    /* Initialize GLEW */
    if(glewInit() != GLEW_OK)
        return -1;

    /* Set the viewport */
    glViewport(0, 0, width, height);

    return true;
}

int loadContent()
{
    /* Create new buffer to store our triangle's vertices */
    glGenBuffers(1, &VBO);

    /* Tell OpenGL to use this buffer and inform that this buffer will contain an array of vertices*/
    glBindBuffer(GL_ARRAY_BUFFER, VBO);

    /* Fill buffer with data */
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    /* Enable a generic vertex attribute array */
    glEnableVertexAttribArray(0);

    /* Tell OpenGL how to interpret the data in the buffer */
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);

    return true;
}

void render(float tpf)
{
    /* Draw our triangle */
    glDrawArrays(GL_TRIANGLES, 0, 3);
}

void update()
{
    float oldTime = 0.0f;
    float newTime = 0.0f;
    float gameTime = 0.0f;

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Update game time value */
	oldTime = newTime;
	newTime = (float)glfwGetTime();
	gameTime =  newTime - oldTime;

        /* Render here */
        render(gameTime );

        /* Swap front and back buffers */
        glfwSwapBuffers(window);

        /* Poll for and process events */
        glfwPollEvents();
    }
}

int main(void)
{
    if(!init(640, 480))
        return -1;

    if(!loadContent())
        return -1;

    update();

    glfwTerminate();
    return 0;
}
```

## Code explanation

First of all, we include new header file from GLM library and initialize array of vertices of the triangle that we want to draw.

```cpp
#include <glm/glm.hpp>

/* Initialize vertices of our triangle */
glm::vec3 vertices[] = { glm::vec3( 0.0f, 1.0f, 0.0f),
                         glm::vec3( 1.0f, -1.0f, 0.0f),
                         glm::vec3(-1.0f, -1.0f, 0.0f)
                       };
```

The vertices array is of type _vec3_ and as you may already assumed it's a structure that defines three dimensional vector. A triangle has exactly three vertices and that's why there are three vectors in the vertices array that represents the points (vertices) of the triangle. Screen coordinates for axes X, Y and Z (of the OpenGL's window) are in the interval [-1.0, 1.0]. It's a crucial information for us because in this part we don't use programmable rendering pipeline yet - we don't have control over transforming vertices to screen coordinates. For now, we use fixed rendering pipeline, which will "display" the vertices of the triangle (that were saved to vertices array) and will 'color' the interior of the triangle on white.

In the _int init()_ function, I've added a new instruction _glViewport(x, y, width, height)_. It creates a rectangular viewport, where _x_ and _y_ are the lower left corner of the rectangle with a width _width_ and height _height_. In short, it is used to determine the size of a rectangular "window" through which we see the three-dimensional scene. This "window" may have the same dimensions as the window of our application, but it need not (then the scene will be shown for example in the left half of the window with the parameters _width = window_width/2_ and _height = window_height_).

Then we add a new function _int loadContent()_, which is responsible for loading and preparing the data for our application. It returns _true_ if all went well and _false_ otherwise. We immediately put a test in the _main_ function, which checks whether everything loaded properly.

```cpp
int loadContent()
{
}

int main(void)
{
    if(!init(640, 480))
        return -1;

    if(!loadContent())
        return -1;

    update((float)glfwGetTime());

    glfwTerminate();
    return 0;
}
```

Now we can move on to prepare our data for drawing. First, initialize global variable, which is a handler to a buffer that stores the vertices of the triangle.

```cpp 
/* Initialize Vertex Buffer Object */  
GLuint VBO = NULL;  
```

GLuint type is the type of OpenGL and can be compared to the unsigned int type. As we will see later, most of the OpenGL's objects are precisely of this type.

Now we can move to complete _loadContent()_ function.

```cpp 
/* Create new buffer to store our triangle's vertices */  
glGenBuffers(1, &VBO);  
```

Using this instruction we generate a new buffer. The function _glGenBuffers()_ takes two parameters: the first tells you how many objects/buffers you want to create, and the second is the address of the GLuint array, which holds handles that this function generates for us (make sure that it is large enough to accommodate the number of objects you want to generate). Subsequent calls of this function won't generate the same objects until you call _glDeleteBuffers()_ function.

```cpp
/* Tell OpenGL to use this buffer and inform that this buffer will contain an array of vertices*/  
glBindBuffer(GL_ARRAY_BUFFER, VBO);  
```

In this step, we tell OpenGL that we want to operate on the buffer (that we created earlier) and for it we will set some options. OpenGL works as a state machine - when we turn on something once, it is set to the point where you won't disable it. How does this apply to operations on buffers? If you call the function, e.g. _glBindBuffer()_, then the every operation on the buffers will be made to the buffer, which we passed as an argument to this function. The first parameter is a "type", to which we assign the buffer that is specified in the second parameter (there are more "types" of buffers, which we read about in the [documentation](http://www.opengl.org/registry/doc/glspec44.core.pdf)).

```cpp 
/* Fill buffer with data */  
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  
```

Then we "put" an array of vertices of our triangle into the buffer. The first parameter corresponds to the type of the buffer for which we put the data into. The second parameter is the size, in bytes, that we want to reserve in our buffer for data - we use of the _sizeof_ operator, which returns the size of our vertex array in bytes. The third parameter is a pointer to the data that we want to put in the buffer. The fourth parameter specifies whether we will change the data in the buffer and how often we will be using them, and how (more about these types can be found in the [documentation](http://www.opengl.org/registry/doc/glspec44.core.pdf)).

```cpp  
/* Enable a generic vertex attribute array */  
glEnableVertexAttribArray(0);  
```

After calling this function, OpenGL will have access to an array of vertices with index 0. It is especially important to remember to enable access to the array when we use functions for drawing such as: _glDrawArrays, glDrawElements, glDrawRangeElements, glMultiDrawElements, glMultiDrawArrays_.

```cpp
/* Tell OpenGL how to interpret the data in the buffer */  
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);  
```

This function determines how to interpret the data stored in the buffer. The first parameter specifies where the data was sent (to which array, for which index). The second parameter tells you how many components consists of a vertex attribute (in this case, his position). The third parameter is the data type of our vertices. The fourth parameter can normalize our position vectors for us if we pass GL_TRUE, otherwise this function will do nothing with them. The fifth parameter is the distance between the components of the vertices (in our case, consecutive components are tightly packed next to each other - hence the value 0). The sixth parameter is the distance in the buffer from which we store our data. Our buffer is made up of the same positions values so we set 0\. The last parameter is useful when in the buffer we store not only the information about the position, but also we have information about a vertex color, texture coordinates, etc.

```cpp
/* Draw our triangle */  
glDrawArrays(GL_TRIANGLES, 0, 3);  
```

Now, in the _render()_ function we call a method that will draw a triangle :) The first parameter is the type of primitive that will be rendered and constructed by the graphics card - this is due to the manner in which the vertices were saved in the _vertices_ array (more about these types is in the [documentation](http://www.opengl.org/registry/doc/glspec44.core.pdf)). The second parameter is the location of the first vertex position component in the buffer. The third parameter specifies how many position components has one vertex.

That's all! We can now compile our code and window should be shown with a large, white triangle:

![First triangle]({{ site.baseurl }}/img/beginner_opengl/tutorial-03-beginner-gl.png){: .center-image }

Below are exercises that will help you understand how some OpenGL functions work. I encourage you to experiment with the code and to comment below. Of course, exercise is only for willing people. In the [_Source Code_](#source_code) section is the code and VC++ solution of this tutorial ;) In the next part of the course we will look at the shaders, and we will color our triangle! :)

## Source Code {#source_code}
*   [VC++ 2010 solution](https://drive.google.com/file/d/0B0j4jdWAANaoUFZfTFd2MkJOSjg/view?usp=sharing&resourcekey=0-xvVNDHD8RxO5L9Da_VyRTQ)

## Exercises

1. What will happen if we will get out of the range [-1; 1] with the vertices of the triangle?
2. By using the _glViewport()_ function make the triangle to show up in the middle of the screen, but zoomed out twice.
3. Draw a square (Hint: use two triangles).
