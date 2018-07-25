-
layout: post
title: Tutorial 02 - Setup VC++ 2010 environment to cooperate with OpenGL
subtitle: Beginner OpenGL course
tags: [beginner-opengl-en, tutorial]
-
## Setting up the environment
Before we start creating our awesome graphical applications in OpenGL technology, first, we have to adjust ourprogramming environment to cooperate with this library. To begin, we have to have Microsoft Visual C++ 2010 Expressinstalled (here is the [link](http://www.microsoft.com/visualstudio/plk#downloads+d-2010-express)). 

Moreover, we have to download the following libraries:

**GLFW** - [http://www.glfw.org/](http://www.glfw.org/download.html "http://www.glfw.org/") - it's free, cross-platform library that will allow us to create window with OpenGL context and to handle I/O events (keyboard, mouse, etc.). We download 32-bit binaries for Windows (or 64-bit if want to develop 64 bit app).  
**GLM** - [http://glm.g-truc.net/](http://glm.g-truc.net/ "http://glm.g-truc.net/") - in my humble opinion, it is a very good mathematics library, which will help us with operations on matrices, etc. Let's download the newest version.  
**GLEW** - [http://glew.sourceforge.net/]( http://glew.sourceforge.net/ " http://glew.sourceforge.net/") - library that wraps the newest OpenGL library. Download 32-bit binaries for Windows. 

Next, we have to extract these libraries to separate folders. Then, from the main GLFW folder we copy the following files/folders and copy them to the proper locations:

*   Folder **include/GLFW** to location **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\include**
*   Files _*.lib_ from folder **lib-msvc100** to location **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\lib**
*   File  _*.dll_ from folder **lib-msvc100** to location **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\bin**

Now, let's head for GLEW:

*   Folder **include/GL** to location **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\include**
*   Files from folder **lib** to location **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\lib**
*   Files from folder **bin** to location **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\bin**

With GLM is easier:

*   Copy folder **glm** to location **C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\include**

Now, we have to tell the environment to use above libraries. Therefore, open Visual Studio 2010 C++ and choose: **File -> New -> Project**. In the next window choose **Win32 Console Application**, give a name to a project and choose a location where you want to store this project and click **Ok**. 

{% include lightbox src="img/beginner_opengl/11.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

In the next window click **Next**. Now, you should see a window with settings for the application. Check **Console Appliaction** and **Empty Project**, and click **Finish**.

{% include lightbox src="img/beginner_opengl/21.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

At the moment, we have project created. Before we start coding, we have to link the libraries that we previously downloaded. For this purpose, right click on the name of the project in Solution Explorer and choose **Properties**.

{% include lightbox src="img/beginner_opengl/31.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

In the new window, set Configuration field to **All Configurations**.

{% include lightbox src="img/beginner_opengl/41.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

Now let's go to **Configuration Properties -> Linker -> Input** and click on the field near to **Additional Dependencies** and choose **Edit…** In the new window type (everything should be separated with a new line): **opengl32.lib, glu32.lib, glfw3.lib, glfw3dll.lib, glew32.lib** and click **Ok**.

{% include lightbox src="img/beginner_opengl/51.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

## Code of the application
Finally, we can get our hands dirty and run the first OpenGL's window! To do this, right click on Source File and choose **Add->New Item…**

{% include lightbox src="img/beginner_opengl/61.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

In a new window choose **C++ File (.cpp)** and type a name for the file e.g.: _main_ and click **Add**.

{% include lightbox src="img/beginner_opengl/71.jpg" data="data" img-style="max-width:70%;" class="center-image" %}

Just now, we've crated new file to which re-type or copy and paste the code below (explanation will be in a moment):

{% highlight cpp linenos %}
/**  
** Listing taken from: http://www.glfw.org/documentation.html  
**/ 
#include <GLglew.h>  
#include <GLFW/glfw3.h>

int main(void)  
{  
    GLFWwindow* window; 

    /* Initialize the library */
    if (!glfwInit())  
        return -1; 
        
    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL); 

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
        
    /* Loop until the user closes the window */  
    while (!glfwWindowShouldClose(window))  
    {  
        /* Render here */ 

        /* Swap front and back buffers */  
        glfwSwapBuffers(window); 
        
        /* Poll for and process events */  
        glfwPollEvents();  
    } 
    
    glfwTerminate();  
    return 0;  
}  
{% endhighlight %}

Let's run and compile above code by pressing F5 on your keyboard or by clicking on a green arrow. The application should compile without errors and you should see a window with black background. Congratulations! You have crated your own OpenGL's window!

## Code explanation
In the first line, using the preprocessor command we include the header file, which will allow us using the newest OpenGL features and in the next line we attach file that will allow us creating window and OpenGL context. GLFW/glfw3.h file, defines all the constants, types and functions that are used by GLFW and attaches all the files that are needed for creating OpenGL application. That's why we don't need to worry about including files like: windows.h, GL/gl.h, etc. The only one file that isn't being attached is glu.h. If we want to include this file, we just have to add this line of code: 

```cpp
#define GLFW_INCLUDE_GLU  
#include <GLFW/glfw3.h>  
```

```cpp
GLFWwindow* window; 

/* Initialize the library */  
if (!glfwInit())  
    return -1;  
```
Here, we "auto-magically" initialize GLFW library. The method _glfwInit()_ returns zero if initialization step failed or non-zero value if it succeeded.

```cpp 
/* Create a windowed mode window and its OpenGL context */
window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
if (!window)
{
    glfwTerminate();
    return -1;
}
```

Next, we crate window with width 640px and height 480px and we give a name to the window "Hello World". If this operation fails, the NULL value will be returned. That's why we check if it succeeded. If it fails, we use __*glfwTerminate()*__ function to destroy all created windows and release all the resources that were occupied by GLFW. This method should be used when we want to turn off the application.

```cpp
/* Make the window's context current */  
glfwMakeContextCurrent(window);  
```

Before we can use OpenGL's functions, we have to crate its context for the window by calling the above function.

```cpp
/* Initialize GLEW */  
if(glewInit() != GLEW_OK)  
    return -1;  
```

Now, we initialize GLEW library and check if this step succeeded or not. From now, we can use OpenGL's features.

```cpp
/* Loop until the user closes the window */
while (!glfwWindowShouldClose(window))
{
    /* Render here */
    
    /* Swap front and back buffers */
    glfwSwapBuffers(window);

    /* Poll for and process events */
    glfwPollEvents();
}
```

The above loop is the heart of the any graphical application. It's called main application loop, where in every frame will be drawn our 3D models and all the algorithms will be evaluated.

In the while loop we check the condition if application's window is about to close down. The __*glfwWindowShouldClose()*__ returns 1 if a user presses the cross button (which is used to close the application's window) or when user presses the keys combination Alt+F4\. Further in the loop we call all the functions that are connected with rendering/drawing (which are not crated yet - that's why we see black background).

The GLFW window always uses double buffers, which prevents flickering when we render consecutive frames. The __*glfwSwapBuffers()*__ swaps the front and back buffers.

Our window, has to have the ability to handle events such as closing the window. In this case we use __*glfwPollEvents()*__ function that processes the events that are in an events queue and responses to them instantly.

More on this topic can be found in GLFW documentation (link is on the bottom of this post).

That's it for now. There's also an exercise which you can do to do some work and experiment with the above code. I highly recommend doing this exercise since the result of this exercise will be the base code of the next tutorial (the answer for this exercise will be available in the next tutorial). In the next part, we will draw the first triangle!

## Source Code
*   [VC++ 2010 solution](https://drive.google.com/file/d/0B0j4jdWAANaoQnhDUEV0dXJlM1U/view?usp=sharing)

## Exercise 
Divide the above program into the following functions:

*   main()
*   init(int width, int height)
*   update(float tpf)
*   render(float tpf)

To the update function we can pass the value from __*double glfwGetTime()*__ which returns the amount of time that elapsed from the time when we initialized GLFW. To make things simple, we can make _GLFWwindow* window;_ field a global field (I know, I know - that's not the way how it should be done :P).

## References
1. GLFW documentation, [http://www.glfw.org/docs/3.0/](http://www.glfw.org/docs/3.0/)
