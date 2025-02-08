#opengl #rendering #research #Cpp 

# Resources
- [GLFW: Introduction to the API](https://www.glfw.org/docs/latest/intro_guide.html)
- [How include Vulkan & GLFW in CLion using CMAKE - YouTube](https://www.youtube.com/watch?v=82taqgqkdeU)
- [glfw/glfw: A multi-platform library for OpenGL, OpenGL ES, Vulkan, window and input](https://github.com/glfw/glfw)
- [GLFW: Introduction](https://www.glfw.org/docs/latest/) 
# Window
[GLFW: Window guide](https://www.glfw.org/docs/latest/window_guide.html) <-- Resource
Creating classes for the `Window` and `Logger` for errors
## Main
Window creation
- Includes the `memory` header, because of use of `unique smart pointer` 
- Initialize the window using the width, height, and title text.
- If this initialization fails, we print out a log message and exit the program

The `log()` call has:
 - Output level as the first parameter,
 - C-style printf string. 
 - The __FUNCTION__ macro is recommended to print out the function

Loop: 
- If the init() call was successful, we enter the mainLoop() function of the Windows class.
- This handles all the window events, drawings, and more. 

Closing the window ends the main loop. After this, we clean up the window and return

```cpp
#include <memory>  
#include "Display/Window.h"  
#include "Display/Logger.h"  
  
  
int main(int argc, char *argv[]) {  
    std::unique_ptr<Window> w = std::make_unique<Window>();  
  
    if (!w->init(640, 480, "Test Window")) {  
        Logger::log(1, "%s error: Window init error\n", __FUNCTION__);  
        return -1;  
    }  
  
    w->mainLoop();  
    w->cleanup();  
    return 0;  
}
```

## Logger
Simplify the debugging process. Logging messages with different *logging levels*, enabling you to control the number of logs being shown.

## Window
Use of `GLFW` - an open source toolkit that is used to handle the tasks around the application window #glfw/window  
- Create and destroy the application window
- Handle the window events (such as minimize, resize, or close)
- Add an #OpenGL context or #Vulkan support to enable 3D rendering
- Get the input from the mouse, keyboard, and gamepads/joysticks

```cpp title:Window.h
#include <string>
#include <GLFW/glfw3.h>
class Window {
private:
    GLFWwindow *mainWindow = nullptr;
public:
    bool init(unsigned int width, unsigned int height, const std::string& title);
    void mainLoop();
    void cleanup();

};
```

The `init()` checks whether GLFW could be initialized at all. If something unexpected happens, it will return false in the `main()` function and stop the program.

The `mainLoop()` checks whether the user generated an event to close the window, that is, by selecting the close button. 
- If this is not the case, it instructs `GLFW` to poll any events. 
- This call is required to react to anything happening to the window itself 
	- keyboard presses
	- mouse events
	- window operations such as minimizing or even closing:

```cpp
while (!glfwWindowShouldClose(mainWindow)) {  
    /* poll events in a loop */  
    glfwPollEvents();  
}
```

The `cleanup()` destroys the window and terminates GLFW.

## OpenGL context

Link CMake:
```CMake
set(OpenGL_GL_PREFERENCE GLVND)
find_package(OpenGL REQUIRED)
target_link_libraries(${PROJECT_NAME} glfw  OpenGL::GL)
```

`glfwMakeContextCurrent()` – it gets the OpenGL context, which contains the
global state of the rendering, and makes it the context of the current thread.

```cpp
bool Window::init(unsigned int width, unsigned int height, std::string
title) {
	if (!glfwInit()) {...}
	if (!mainWindow) {...}
	glfwMakeContextCurrent(mainWindow);
}
```

we can use some simple OpenGL calls inside the `Window::mainloop()` 

```cpp
void Window::mainLoop(){
	// wait for the vertical sync 
	glfwSwapInterval(1);
	...
}
```

### Background
`glfwSwapInterval()` - activate the wait for the vertical sync with a call to this GLFW
function  #glfw/background 
- Without waiting for it: the window might *flicker, tearing might occur*, as the update and buffer switch will be done as fast as possible.

```cpp
void Window::mainLoop(){
	...
	//Background
	float color = 0.0f;
	
	while (!glfwWindowShouldClose(mWindow)) {
		color >= 1.0f ? color = 0.0f : color += 0.01f;
		glClearColor(color, color, color, 1.0f);
		glClear(GL_COLOR_BUFFER_BIT);
		
		/* swap buffers */
		glfwSwapBuffers(mWindow);
		
		...
	}
```

`glClearColor()` - Sets the new colour to be used when clearing the draw buffer

`glClear()` - with the value set to clear only the colour buffer
- gives the window a simple grey background:

`GLFW` activates `double buffering` for the OpenGL window. 
- Has two separate graphics buffers of the same size, a `front buffer` and a `back buffer`.

`Front buffer` - contains the image created by the previous rendering calls.
`Back buffer` - changes to the final picture occur in while showing the front buffer, 

`glfwSwapBuffers()` - After the drawing of the back buffer has finished, it swaps the two buffers and displays the content of the back buffer, making the previous front buffer the new back buffer for the hidden drawing:

# Event Handling
All events are stored in an `event queue` and must be handled by the application code. #glfw/eventHandling 
- If you never request the events of that queue, the application window won’t even close properly thus can only be killed using Task Manager.

`glfwPollEvents()` - required to empty the event queue and runs any configured call-backs. 
- Without this the window will do nothing, not even close down.
- Should be at end of main loop to process new calls

`glfwWaitEvents()` - also clears the event queue and fire the call-backs
- puts the thread to sleep and waits until at least one event has been generated for the window.
- Usually used in non-interactive applications that are waiting for any input from the user

For events that react to user actions by performing another action, eg. message display when a user tries closing the application need two parts:
1. Function called by GLFW 
2. call that sets the function as a `callback`

### Lambda functions

A small piece of code, running as an anonymous function.

GLFW is written in C and has no concept of C++ features like OOP thus a small helper is added to every window that might be created – a pointer that can be set and read by the user:
```cpp
void glfwSetWindowUserPointer(GLFWwindow *win, void *ptr);
```

Can store any arbitrary data in the user pointer like:
- C++ Window object, and inside the lambda, this pointer will be read and used just like in any other C++ call.

The callback for window close event:
- requires a pointer to a function and returns either NULL

```cpp
GLFWwindowclosefun glfwSetWindowCloseCallback (GLFWwindow *window, GLFWwindowclosefun callback);
```

**Callbacks** created using `typedef`. This is done to avoid writing the expression in the second braces every time we use the function.
```cpp
typedef void(* GLFWwindowclosefun) (GLFWwindow *window)
```

In the `init()`, lambda is introduced by the square brackets, [], followed by the parameters the function takes.
```cpp
glfwSetWindowUserPointer(mWindow, this);
glfwSetWindowCloseCallback(mWindow, [](GLFWwindow *win) {
	auto thisWindow = static_cast<Window*>(
	glfwGetWindowUserPointer(win));
	thisWindow->handleWindowCloseEvents();
});
```

Termination:
```cpp
void Window::handleWindowCloseEvents() {  
    Logger::log(1, "%s: Window got close event... bye!\n", __FUNCTION__);  
}
```


## Keyboard Presses

Similar to window events
1. create a member function to be called 
2. add the lambda-encapsulated call to GLFW

GLFW offers callback to get the events for keyboard key presses/releases.

Plain key input receives four values:
1. The ASCII key code of the key
	- `GLFW_KEY_A` - 7-bit ASCII code of the letter you pressed, >256.
2. The (platform-specific) scan code of that key
	- hardcoding it into your code is a bad idea. 
3. The action you carried out 
	- `GLFW_PRESS` - key press
	- `GLFW_RELEASE` - key release
	- `GLFW_REPEAT` - key pressed for longer, but note this is not issued for all keys.
4. The status of the modifier key, such as Shift, Ctrl, or Alt
	- bitmap to see whether the users pressed keys such as `Shift`, `Ctrl`, or `Alt`
	- Can enable the reporting of `Caps Lock` and `Num Lock` – *not enabled* in the normal input mode
```cpp
glfwSetKeyCallback(window, key_callback);
void key_callback(GLFWwindow* window, int key, int scancode, int
action, int mods)
```

Example GLFW callback for key presses:
```cpp
//place in class under public to give callbak access to window pointer
public:
void handleKeyEvents(int key, int scancode, int action,
int mods);

/*In init()*/
//Callback for key events  
glfwSetKeyCallback(mainWindow, [](GLFWwindow *win, int key,  
int scancode, int action, int mods) {  
auto thisWindow = static_cast<Window*>(glfwGetWindowUserPointer(win));  
thisWindow->handleKeyEvents(key, scancode, action, mods);  
});
```

Simple key Event handler:
```cpp
void Window::handleKeyEvents(int key, int scancode, int action, int mods) {
    std::string actionName;
    switch (action) {
        case GLFW_PRESS:
            actionName = "pressed";
        break;
        case GLFW_RELEASE:
            actionName = "released";
        break;
        case GLFW_REPEAT:
            actionName = "repeated";
        break;
        default:
            actionName = "invalid";
        break;
    }
    const char *keyName = glfwGetKeyName(key, 0);
    Logger::log(1, "%s: key %s (key %i, scancode %i) %s\n",
    __FUNCTION__, keyName, key, scancode,
    actionName.c_str());
}
```

## Mouse movement
### 1. Movement adjusted by the OS 

Returns the value with all the *optional settings* you might have defined, such as `mouse acceleration`, which speeds up the cursor if you need to move the cursor across the screen.

-  The following is a callback function, which gets informed if the mouse position changes:
```cpp
glfwSetCursorPosCallback(window, cursor_position_callback);
void cursor_position_callback(GLFWwindow* window, double xpos, double ypos)
```

Alternatively, you can poll the current mouse position in your code manually:
```cpp
double xpos, ypos;
glfwGetCursorPos(window, &xpos, &ypos);
```

### 2. Raw movement.
Excludes these settings and provides you with the `precise level of movement` of the mouse. 

To enable raw mode:
1. Disable the mouse cursor in the window (not only hide it)
2. Try to activate it:
```cpp
glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);
if (glfwRawMouseMotionSupported()) {
	glfwSetInputMode(window, GLFW_RAW_MOUSE_MOTION,GLFW_TRUE);
}
```

To `exit raw mode`, go back to the `normal mouse mode`:
```cpp
glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_NORMAL);
```

Example use cases:
1. Adjusting the settings using an onscreen menu, OS movement is perfect.
2. Rotate or move the model, or change the view in the virtual world use the raw mode instead.
## Mouse presses
To add a mouse button callback, add the function call to `Window.h`:
```cpp
private:
void handleMouseButtonEvents(int button, int action, int mods);
```

Callback handling and the function itself in `Window::init()`:
```cpp
glfwSetMouseButtonCallback(mainWindow, [](GLFWwindow *win, int button, int action, int mods) {
	auto thisWindow = static_cast<Window*>(
	glfwGetWindowUserPointer(win));
	thisWindow->handleMouseButtonEvents(button, action,
	mods);
});
```
Gets back the pressed button, the action (`GLFW_PRESS` or `GLFW_RELEASE`), and any pressed modifiers such as the `Shift` or `Alt` .

# Building an OpenGL Renderer
#opengl/renderer
## Requirements
1. Main.cpp and the OpenGL window
2. `Glad`, the OpenGL loader generator
3. `stb_image`, a single-header loader for image files
4. The [OpenGL Mathematics (GLM)](https://github.com/g-truc/glm) library

```CMake
include(FetchContent)

FetchContent_Declare(
	glm
	GIT_REPOSITORY	https://github.com/g-truc/glm.git
	GIT_TAG 	bf71a834948186f4097caa076cd2663c69a10e1e #refs/tags/1.0.1
)

FetchContent_MakeAvailable(glm)

target_link_libraries(main PRIVATE glm::glm)
```

## Graphics pipeline

#graphics/pipeline
![[Pasted image 20250207221216.png]]

### 1. Vertex data
The characters to be drawn are made of triangles, and the `Vertex Data of these triangles` is
sent from the application to the graphics card.

### 2. Primitive processing
This input data is processed per `Primitive` 
- For every triangle we send, OpenGL sends the primitive type with the draw call.
### 3. Vertex Shader 
The Vertex Shader transforms the per-vertex data into the so-called `clip space`, 
- a normalized space with a range between -1.0 and 1.0. 
This makes the processing of further transformation easier; any *coordinate outside the range will not be visible*.

### 4. Tessellation
The Tessellation stage runs only for a special OpenGL primitive, `the patch`. 
The tessellation operation will **subdivide the patch into smaller primitives** such as triangles. 
- This stage can be controlled by shader programs too.

### 5. Geometry Shader
For triangles, the Geometry Shader comes next. 
This shader can generate new primitives in addition to the currently processed ones, and you can use it to easily add debug information to your scene.

### 6. Primitive Assembly
All primitives are converted into triangles, transformed into viewport space (our screen dimensions), and clipped to the visible part in the viewport.

### 7. Rasterization
*Converts* the *incoming primitives into* so-called `fragments`, which will eventually become screen pixels. 
- It also interpolates the vertex values, such as colour or texture, across the face of the primitive.

### 8. Fragment Shader
Determines the final colour of the fragment. 
- It can be used to blend textures or add fog.

### 8. Per-Sample Operations 
Tests which decides whether the fragment will be used for the final screen or discarded:
- Include scissor or stencil tests (to “cut out” parts of the screen) 
- The depth test

### 9. Screen
At the end of the stage, a final picture will have been created. This picture will eventually be displayed on the computer screen.


## OpenGL loader generator Glad
To translate the function pointers back to more human-readable function names, several helper
libraries exist. Document uses [Glad](https://glad.dav1d.de/).
1. Select OpenGL for Specification,** Version 4.6** for API
2. **Core** for Profile.
3. For Extensions, choose **ADD ALL**.
4. Keep **Generate a loader** checked and the other two options unchecked
5. Press the **GENERATE** button
6. Download the ZIP file and unpack it to the project root, including the folders

```CMakeList
file(GLOB SOURCES  
        glad/src/glad.c  
)
...
target_include_directories(${PROJECT_NAME} PUBLIC include src window tools opengl model)
```

## Anatomy of the OpenGL renderer
The renderer will be split into five classes to collect all operations and data required for different
OpenGL objects:
1. **Main renderer** - which is called from `mainLoop()` in the *Window* class.
2. **Framebuffer** - which is responsible for the creation of the buffers.
3. **Vertex array** - which stores the vertex data that will be drawn to the screen
4. **Shader** - which loads and compiles the shader programs
5. **Texture** - which loads PNG image files from the system and creates an OpenGL texture out of them

### 1. Main Renderer class
Make sure glad.h is included before glfw3.h as GLFW changes its behavior and will not include
the basic system headers if OpenGL functionality is already found

`init()` - used for the first initialization; it creates the OpenGL objects we need to draw anything at all. 
`cleanup()` - removes the objects, called from the *Window class* after we close the application window.
`setSize()` - is used during window resizes; it will be called from the *Window class*.
`uploadData()` - stores triangle and texture data from the model in the renderer class, 
`draw()` - the triangles will be drawn to the framebuffer

we add local objects of our four classes in the list to create, and a counter of the triangles we upload to the renderer. 
The counter is needed for the draw() call to display the correct amount of triangles from the vertex array.

```cpp:mainRender.h
#ifndef MainRENDERER_H  
#define MainRENDERER_H  
  
#pragma once  
#include <vector>  
#include <string>  
#include <glm/glm.hpp>  
#include <glad/glad.h>  
#include <GLFW/glfw3.h>  
  
#include "Framebuffer.h"  
#include "VertexBuffer.h"  
#include "Texture.h"  
#include "Shader.h"  
#include "MainRenderData.h"  
  
class MainRenderer {  
    Shader basicShader{};  
    Framebuffer frameBuffer{};  
    VertexBuffer vertexBuffer{};  
    Texture texture{};  
    int triangleCount = 0;  
public:  
    bool init(unsigned int width, unsigned int height);  
    void setSize(unsigned int width, unsigned int height);  
    void cleanup();  
    void uploadData(OGLMesh vertexData);  
    void draw();  
};  
  
#endif //MainRENDERER_H
```