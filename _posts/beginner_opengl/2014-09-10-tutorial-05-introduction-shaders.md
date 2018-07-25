---
layout: post
title: Tutorial 05 - Introduction to shaders
subtitle: Beginner OpenGL course
tags: [beginner-opengl-en, tutorial]
---

## Introduction

In this part of the OpenGL course we will learn how to write a simple shader that draws the triangle in the color in what color you want! From this part of the course, the whole code listings will not be placed throughout (due to too large volume) and therefore I will be immediately going to the part where the code analysis begin. There also will be put the most important lines or pieces of code that has changed from the previous part. You can download source code for this section from [here](https://drive.google.com/file/d/0B0j4jdWAANaoVzkyUnpTZWk1eGc/view?usp=sharing). In addition, starting from this part everything we will do in OpenGL will use the shaders as the modern method of 3D graphics programming. Here we go!

## Explanation of the application's code

From the previous part of the course (Tutorial 03) a few things has changed in the code:

*   _loadShader(std::string)_ function has been added,
*   _loadAndCompileShaderFromFile(GLint, std::string, GLuint&)_ function has been added,
*   changes has been made in _init()_ and _render()_ functions,
*   shader was added to the project (practically there are two shaders - vertex and fragment) that have been placed in the folder Shaders.

Let's start from the beginning (shaders explanation will be at the end). I will not explain the _loadShader(std::string)_ because it is linked closely with C ++ and I assume that all readers of this course are familiar with this language to such an extent, to understand everything that is included in this function. It is used to load shader code into the computer memory and to store it in a variable of type _std::string_.

Before going to explain the _loadAndCompileShaderFromFile(GLint, std::string, GLuin&)_, let's go for a moment to the _init()_, which has some changes related to shaders. The first novelty is the following line:

```cpp 
/* Set clear color */  
glClearColor(1.0f, 1.0f, 1.0f, 1.0f);  
```

It saves in the OpenGL state machine, the color which will be used to to clean color buffer. Color buffer must be cleaned with every frame that is rendered and because of it in the _render()_ is called the following instruction:

```cpp 
/* Clear the color buffer */  
glClear(GL_COLOR_BUFFER_BIT);  
```

The function _void glClearColor(GLfloat red, GLfloat green, GLfloat blue, GLfloat alpha)_ accepts four parameters of type GLfloat, and these are the consecutive components of RGBA - red, green, blue, alpha (responsible for transparency). These parameters take values from the interval [0, 1], and when we give a value outside this range it will be clamped to fit in this interval (if we give the value of 10.0f, it will be converted to a value of 1.0f, and when we give value -8.0f, it will be converted to a value 0.0f).

With the _void glClearColor(GLfloat red, GLfloat green, GLfloat blue, GLfloat alpha)_, we can control the color "background" of our virtual scene. This is because the color buffer is first cleared to the default color (defined by the above function), and on top of it we draw our geometry, which may have completely different color/colors.

Now we can go to the section related only with shaders.

```cpp 
/* Shader init */  
GLuint programHandle = glCreateProgram();

if(programHandle == 0)  
{  
    fprintf(stderr, "Error creating program object.\n");  
}  
```

First, create a handle to the "program" using the _glCreateProgram()_, to which we will attach the shader. If it returns zero, it means that something went wrong during the creation of this object - we will not see anything on the screen because our shader will not work. When it is a non-zero value, then everything is fine.

```cpp 
/* Shader load from file and compile */  
loadAndCompileShaderFromFile(GL_VERTEX_SHADER, "Shaders/basic.vert", programHandle);  
loadAndCompileShaderFromFile(GL_FRAGMENT_SHADER, "Shaders/basic.frag", programHandle);  
```

The function _loadAndCompileShaderFromFile(GLint, std::string, GLuint&)_ is called two times. Once for the vertex shader and the second time for the fragment shader. From the previous lesson, we know that it is the minimum, if we want to benefit from the opportunities offered by shaders. This function uses the _loadShader(std::string)_ to load the shader code (you can also keep the code in the array _char*_, but it is not convenient if we are facing with shaders that have a lot of lines of code and you want to debug the shader).

```cpp 
GLuint shaderObject = glCreateShader(shaderType);

if(shaderObject == 0)  
{  
    fprintf(stderr, "Error creating %s.\n", fileName.c_str());  
    return;  
}  
```

The first task of the _loadAndCompileShaderFromFile(...)_ is to create a shader object using the _glCreateShader(GLuint type)_. The argument of this function is the type of shader we are going to compile. It can be: **GL_VERTEX_SHADER**, **GL_FRAGMENT_SHADER**, **GL_GEOMETRY_SHADER**, **GL_TESS_EVALUATION_SHADER**, **GL_TESS_CONTROL_SHADER** or **GL_COMPUTE_SHADER**. Shader object is stored in the local variable _shaderObject_ used later to check whether the object was able to create - everything will be fine if it is non-zero value, otherwise something went wrong and will display a message in the console.

```cpp 
std::string shaderCodeString = loadShader(fileName);

if(shaderCodeString.empty())  
{  
    printf("Shader code is empty! Shader name %s\n", fileName.c_str());  
    return;  
}

const char * shaderCode = shaderCodeString.c_str();  
const GLint codeSize = shaderCodeString.size();

glShaderSource(shaderObject, 1, &shaderCode, &codeSize);  
```

Then the shader code is loaded from the path specified in the argument _fileName_ and saved to the _shaderCode_. Now you have to load this code to the shader object. For this purpose the _glShaderSource(...)_ is used. The first argument is the shader object that we want to create. The second parameter is the number of shader codes that we want to build (we compile one shader code at a time - hence the number 1). The third argument is an array of strings that contains the shader codes. Our contains only one code. The fourth parameter is an array that contains the length of the string in the third argument. Now the shader code has been copied to the internal memory of OpenGL.

```cpp 
glCompileShader(shaderObject);

GLint result;
glGetShaderiv(shaderObject, GL_COMPILE_STATUS, &result);
  
if(result == GL_FALSE)
{
    fprintf(stderr, "%s compilation failed!\n", fileName.c_str());
        
    GLint logLen;
    glGetShaderiv(shaderObject, GL_INFO_LOG_LENGTH, &logLen);

    if(logLen > 0)
    {
        char * log = (char *)malloc(logLen);

        GLsizei written;
        glGetShaderInfoLog(shaderObject, logLen, &written, log);

        fprintf(stderr, "Shader log: \n%s", log);
        free(log);
    }

    return;
}
```

Now we can compile the source code of our shader. To do this, simply call the _glCompileShader(GLuint)_ with a parameter of shader object that we want to compile. The build process may fail, so in the next steps we check the correctness of the compilation. If it fails you will see a log that will tell us where in the shader is a mistake. For this we call _glGetShaderiv(...)_, which is used to retrieve various information about shader. For now we are interested in the compilation status and that's why we give as the second argument value _GL_COMPILE_STATUS_. The first object is obviously a shader object for which you want to obtain the information, and the third parameter is the variable to which you want to save the compilation status (it will be _GL_TRUE_ or _GL_FALSE_ depending on whether the process was successful or not). If the compilation failed program will display relevant information in the console and then display the log telling us what the cause of failure was.

```cpp 
glAttachShader(programHandle, shaderObject);  
glDeleteShader(shaderObject);  
```

The next step is to attach shader object to program, which will store shaders that may work together. When we attached shader object to a program, we can safely delete it to free up memory.

```cpp 
/* Link */  
glLinkProgram(programHandle);  
```

We return now to the _init()_. After a successful shaders compilation we need to link them (more specifically we link shader program). This operation is important for this reason that they it is creating connections between shaders - the output of one shader is connected to the corresponding input of the second, to be able to communicate and transfer data between them. Additionally, connections are created between the corresponding input/output of a shader with relevant locations in OpenGL environment. Just like compiling, linking may fail, so we check the status of linking the program using the function _glGetProgramiv(...)_.

```cpp 
/* Apply shader */  
glUseProgram(programHandle);  
```

When in our program, shaders have been properly compiled and linked, we can say to OpenGL, that we want to use a given set of shaders (program) to draw and color objects the way that we defined in these shaders.

## Explanation of the shaders' code

At the beginning, let's look at the vertex shader code:  
```glsl 
#version 400

layout (location = 0) in vec3 vertexPosition;

void main()  
{  
    gl_Position = vec4(vertexPosition.x, vertexPosition.y, vertexPosition.z, 1.0f);

    // Alternatively we can write:  
    // gl_Position = vec4(vertexPosition, 1.0f);  
    // and the effect will be exactly the same  
}  
```

From the previous section, we know that the vertex shader transforms one vertex at a time. GLSL (OpenGL Shading Language) is a language similar to C++ and it is not so scary to assimilate and learn.

```glsl 
#version 440  
```

It is a preprocessor directive, which tells you which GLSL version we are going to use. In this case it is version 4.4 from July 2013.

```glsl 
layout (location = 0) in vec3 vertexPosition;  
```

Using the input qualifier **layout** we define under what index shader has to "look for" a vector of vertices, under which we previously sent data about the position of vertices. In Tutorial 03 we have enabled the location with the _glEnableVertexAttribArray(0)_, and used the _glVertexAttribPointer()_ to tell OpenGL, under which index the vertices data should be sent. We can define more attributes such as: the color of the vertex, texture coordinates for each vertex or normal vector for each vertex.

```glsl 
void main()  
{  
    gl_Position = vec4(vertexPosition.x, vertexPosition.y, vertexPosition.z, 1.0f);

    // Alternatively we can write:  
    // gl_Position = vec4(vertexPosition, 1.0f);  
    // and the effect will be exactly the same  
}  
```

In any shader, function _main()_ must be defined, which is the main function of any shader (in addition, there may be defined other functions). To the built-in output variable of vertex shader _gl_Position_ we assign the value of the input attribute _vertexPosition_ to make the triangle to have the same position coordinates that we defined in the program. The _gl_Position_ is a structure of type _vec4_, which represents the 4-dimensional vector. Note that if we want to refer to the following elements of 3-dimensional vector _vertexPosition_, we can use the operator "." (dot), just like in classes or structures in C++.

To make life easier and reduce the torment of writing the code, you can use the convenience of GLSL and use the constructor _vec4(vec3, float)_.

Note that the value of _w_ (the last value in the constructor _vec4_) is given the value of 1.0f.

It is worth remembering that:

*   If == 1, the vector v (x, y, z, 1) is **position** in 3D space.
*   If == 0, the vector v (x, y, z, 0) is **direction** in 3D space.

It is important for translation. In the 3D space we can move the point, but is it possible to shift direction? Probably not :)

```glsl 
#version 400

out vec4 fragColor;

void main()  
{  
    fragColor = vec4(0.0f, 0.0f, 0.0f, 1.0f);  
}  
```

In the fragment shader, everything looks like in the vertex shader with the difference that we define own output variable of type _vec4_, for the fragment's color (thus the qualifier _out_). To the _fragColor_ variable we assign color RGBA (red, green, blue, alpha) whose successive components take values in the range of [0.0f, 1.0f]. In this case, a triangle is colored in black. If we wanted to change the color to red just convert the first component to the value of 1.0f.

The overall effect should be as follows:

![Black triangle]({{ site.baseurl }}/img/beginner_opengl/tutorial-05-beginner-gl.png){: .center-image }

## Conclusions

That's all for today. If something was not clear from today's lesson, please write to me on email or in the comments below. In the next lesson we will look at the issue of interpolation that OpenGL does automatically :) .

## Source Code {#source_code}
*   [VC++ 2010 solution](https://drive.google.com/file/d/0B0j4jdWAANaoVzkyUnpTZWk1eGc/view?usp=sharing)

## Exercises

1.  Change the background color to blue.
2.  Change the color of the triangle/square to green.

## References
1. More information in OpenGL's [documentation](http://www.opengl.org/registry/doc/glspec45.core.pdf)
2. More information about GLSL and shaders in GLSL's [documentation](http://www.opengl.org/registry/doc/GLSLangSpec.4.50.pdf)
