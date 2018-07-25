---
layout: post
title: New version of OpenGL 4.6 with support for SPIR-V!
tags: [blog]
---

The latest version of OpenGL, numbered 4.6, was released on July 2017\. This latest release of the library includes a number of extensions that were developed by members of AMD, NVidia, Intel, and also includes support for SPIR-V for shaders.  

<img src="{{ site.baseurl }}/img/learnopengl/opengl.jpg" alt="OpenGL" style="float: left;">

SPIR-V is an intermediate language, defined and developed by the Khronos consortium for parallel computing on graphics cards. This allows developers to simplify their shaders by having only one shader version in SPIR-V, which should work on every hardware. Due to the fact that this functionality is included in the core of OpenGL API, it will be supported on every graphics card that supports OpenGL 4.6. In addition, you do not need to learn SPIR-V. Just knowing GLSL and using the *glslang* tool (link in additional materials) we can translate the GLSL code into SPIR-V.

OpenGL 4.6 has included extensions such as:

*   *GL_ARB_gl_spirv* and *GL_ARB_spirv_extensions* - includes a standardization of SPIR-V in OpenGL.
*   *GL_ARB_indirect_parameters* and *GL_ARB_shader_draw_parameters* - allow to reduce the CPU overhead associated with the batch rendering of geometry.
*   *GL_ARB_pipeline_statistics_query* and *GL_ARB_transform_feedback_overflow_query* - adds to the OpenGL functionality that was also available in DirectX.
*   *GL_ARB_texture_filter_anisotropic* (based on *GL_EXT_texture_filter_anisotropic*) - allows to improve the visual texture quality by standardizing the anisotropic filtering.
*   *GL_ARB_polygon_offset_clamp* (based on *GL_EXT_polygon_offset_clamp*) - resolves the problem of "light leak", which is related to shadow rendering.
*   *GL_ARB_shader_atomic_counter_ops* and *GL_ARB_shader_group_vote* - add the most basic functions built into shaders, which are standardized between driver vendors, which will allow for increased efficiency.
*   *GL_KHR_no_error* - (my favorite) reduces the CPU overhead by saying to OpenGL that the application does not expect any errors, so these errors will not even be generated (ideal solution for Release version of the application.)

In addition, there are new extensions:

*   *GL_KHR_parallel_shader_compile* - allows you to compile shaders from multiple threads
*   *WGL_ARB_create_context_no_error* and *GXL_ARB_create_context_no_error* - allow you to create OpenGL contexts that do not generate errors using WGL or GLX.

For more sophisticated applications, there is a way to merge OpenGL (ES) with Vulkan. This is possible thanks to the following extensions (more can be read in additional materials):

*   *GL_EXT_memory_object*
*   *GL_EXT_memory_object_fd*
*   *GL_EXT_memory_object_win32*
*   *GL_EXT_semaphore*
*   *GL_EXT_semaphore_fd*
*   *GL_EXT_semaphore_win32*
*   *GL_EXT_win32_keyed_mutex*

For those who would like to use the latest OpenGL today, NVidia has released the beta 382.88 (Windows) and 381.26.11 (Linux) drivers - the links are below.

In addition, if someone would be interested in viewing Khronos members' lectures from the **2017 SIGGRAPH** conference, then under this [link](https://www.youtube.com/playlist?list=PLYO7XTAX41FMmwf1i4JdDkjc0nbgxz5Fh) is their appearance - have a nice viewing!

### Additional materials and sources

*   [OpenGL 4.6 Specification and its extensions](https://khronos.org/registry/OpenGL/index_gl.php)
*   [glslang](https://github.com/KhronosGroup/glslang): the tool that translates the GLSL code to SPIR-V
*   [NVidia beta drivers](https://developer.nvidia.com/opengl-driver) for OpenGL 4.6
*   [GPU Caps Viewer 1.36.x](http://www.geeks3d.com/20170801/gpu-caps-viewer-1-36-x-released-with-opengl-4-6-support/): a tool to check what OpenGL version is available on our graphics card and which extensions are supported (except for information like GPU temperature, etc.)
