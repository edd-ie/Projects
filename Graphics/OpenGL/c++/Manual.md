#opengl #rendering #research #Cpp 

# Resources
- [Code produced](https://github.com/edd-ie/Graphics-Programming) 
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
Runs only for a special OpenGL primitive, `the patch`. 
Process **Subdivides the patch into smaller primitives** such as triangles. 
- This stage can be controlled by shader programs too.

### 5. Geometry Shader
For triangles, the Geometry Shader comes next. 
This shader can generate new primitives in addition to the currently processed ones, and you can use it to easily add debug information to your scene.

### 6. Primitive Assembly
All primitives are converted into triangles, transformed into viewport space (the screen dimensions), and clipped to the visible part in the viewport.

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


**MainRenderer::init()** :
- `gladLoadGLLoader()` initializes OpenGL via Glad, if fails return false to signal a failure to the creating Window class

`GLAD_GL_VERSION_4_6` integer is only satisfied if the graphics card and driver support OpenGL 4.6

Initialize other class variables
The vertex array initialization needs no separate check as this operation can only fail in a fatal way (such as out-of-memory errors)
If none of the steps fails, we return true to signal that the OpenGL initialization succeeded.

**MainRenderer::setSize()** :
- Resizes the framebuffer object and also the OpenGL viewport – the viewport information is important for the driver to know how to map the framebuffer to the output window.

**MainRenderer::uploadData()** :
- For correct usage of the uploaded vertex and triangle data, the method needs to store the size of `std::vector` with the triangle data. 
It hands over the input data to the vertex array object.


**MainRenderer::draw()** :
Responsible for displaying the triangles from the vertex array object to the framebuffers, and then to the screen
1. Start by binding the framebuffer object, which will let the framebuffer receive our vertex data.
2. Clear the screen with a very low grey colour. 
3. Clear the depth buffer
4. Back-face culling. 
	- Every triangle in the virtual world has two sides, front and back. 
	- The front side of the triangle usually faces the outside of the virtual objects we draw
	- The back side faces the inside. 
5. As the back of the objects will never be seen are occluded by the front faces
	- The triangles facing “away” from us don’t need to be drawn.  
	- Gives speedups, as the triangles are discarded early in the graphics pipeline before much work has been done with them
6. Draw the triangles stored in the vertex buffer :
	1. Load the shader program, which enables the processing of the vertex data
	2. Bind the texture to be able to draw textured triangles
	3. Bind the vertex array, so we have our triangle data available
7. Call `draw()` in vertex buffer:
	1. Sends vertex data to the GPU to be processed by the shaders.
8. Draw the content of the framebuffer to the screen

**MainRenderer::cleanup()** :
- Calls the cleanup() method of all other objects

### 2. Render Data
For ease of management of the vertex data, two structs will be used:
1. Holds the data for a single vertex, consists:
	1. 3-element `GLM vector` for its `position` 
	2. 2-element `vector` for the `texture coordinates`.
2. C++-style vector with elements of the first struct
	1. Creates a collection of all the vertices of a model.

`GLM` - allows data to be organized in the same way in the system memory as it would be on the GPU memory, allowing a simple copy to transfer the vertex data to the graphics card.


### 3. Buffer Types
#buffers
Memory of the graphics cards is managed by the driver; usually, all memory is seen as a single, large block.
This block will be divided into smaller parts `Buffer` for:
- triangle data
- texture 
- frame buffers, and more.
Buffer is accessible from by code via the driver.

#### i. Frame buffer
#buffers/frame_buffer
Creates the final picture displayed on the screen and stores the intermediate results of rendering.

Frame buffers need to be bound before modification:
- `glBindFramebuffer(GL_FRAMEBUFFER, buffer);` 

**NOTE :** include Glad and GLFW in the correct order 
- Glad first and GLFW second. 
- This is required for the OpenGL calls in the class to work

Framebuffers::init() 
- Takes size and width as parameters and initializes the framebuffers.
- `glGenFramebuffers()` - creates an OpenGL framebuffer object
- create a texture with the same size as the window, but without data
- Bind the created texture as a 2D texture type to alter it
- The texture created will have four 8-bit wide components: red, green, blue, and alpha (transparency)
- `GL_TEXTURE_MIN_FILTER` - handles #downscaling (minification) the texture if it is drawn far away
- `GL_TEXTURE_MAG_FILTER` - handles #upscaling (magnification)) the texture if it is drawn close to view
- `GL_NEAREST` - fastest way to up/downscale as it does no filtering at all.
- `GL_TEXTURE_WRAP_S / GL_TEXTURE_WRAP_T` - decides what happens on the positive and negative edges of the texture when drawn outside the defined area of the texture
	- edge-clamping sets the value of the texture data of the x or y position to 0.0 if the requested position is <0,  and to 1.0 if we are requesting a position of >1.
- unbind the texture by binding the (invalid) texture ID of 0
- binding the texture as texture attachment zero

Framebuffers::resize()
- Same parameters as `init()` and recreates the framebuffers to the given, new size. 
	- Deletes old colour texture and depth buffer
	- Calls the init() with the new parameters
- This is called from the renderer on window size changes to have matching framebuffer sizes.

Framebuffers::bind() 
- Enables the drawing to the framebuffers
	- `glBindFramebuffer(GL_DRAW_FRAMEBUFFER, buffer);` - called to enable modification

Framebuffers::unbind() 
- Disables the drawing to the framebuffers
	- `glBindFramebuffer(GL_DRAW_FRAMEBUFFER, 0);` - called to disable modification

`bind()`  & `unbind()` makes it possible to use multiple Framebuffer objects in a single function.
- #deferred_rendering - uses this technique and combines the buffers in a final method to the output picture.

Framebuffers::drawToScreen()
- Copies the data to the `GLFW window`. 
- Can be drawn internally to a separate buffer and not directly to the screen, just to show the flexibility of the rendering.
1. `glBindFramebuffer(GL_READ_FRAMEBUFFER, mBuffer)` - bind the frame buffer for reading
2. `glBindFramebuffer(GL_DRAW_FRAMEBUFFER, 0)` - disable drawing to the frame buffer
	- window becomes the output (draw) framebuffer
3. `glBlitFramebuffer(...)` - #blit the contents of the internal framebuffer to the window
	- a memory copy, a fast method to copy the contents of one framebuffer to another.
4. `glBindFramebuffer(GL_READ_FRAMEBUFFER, 0)` - unbind the framebuffer to stop reading it

Framebuffers::cleanup()
- `unbind()` - Disables drawing to the framebuffers
- `glDeleteTextures(1, &mColorTex)` - delete colour texture
- `glDeleteRenderbuffers(1, &depthBuffer)` - delete the depth buffer
- `glDeleteFramebuffers(1, &buffer)`  - delete the frame buffer

```c++::FrameBuffer.h
class Framebuffer {  
    unsigned int bufferWidth = 640;  
    unsigned int bufferHeight = 480;  
    GLuint buffer = 0;  
    GLuint colorTexture = 0;  
    GLuint depthBuffer = 0;  
    bool checkComplete();  
public:  
    bool init(unsigned int width, unsigned int height);  
    bool resize(unsigned int newWidth, unsigned int newHeight);  
    void bind(); // Enable drawing to the frame buffers  
    void unbind(); // Disable drawing to the frame buffers  
    void drawToScreen();  
    void cleanup();  
};
```

`bufferWidth` and `bufferHeight` - stores the current dimensions of the buffer
- They are required in the method for the final drawing to the output window. *(screen size)*

`GLuint` typed values are integers for internal buffers:
5. **buffer** - the overall framebuffer we draw to
6. **colourTexture** - the colour texture we use as data storage for the framebuffer
7. **depthBuffer** - stores the distance from the viewer for every pixel and ensures that only the colour value nearest to the viewer will be drawn.

`checkComplete()` - checks whether the framebuffer contains all components required to draw.
- Do this check when creating a framebuffer. 
- If the created framebuffer is missing parts of the configuration, accessing them would result in errors.

#### ii. Render buffer 
#buffers/render_buffer
In case don’t need to show or reuse the result of a drawing operation, we may use a render buffer instead of a texture.
- It can be written to like the texture in the framebuffer, but it cannot be read out easily.
- Most useful for intermediate buffers that are valid for a single frame, where the content is not needed for more than this single draw processing
```cpp
/* Use of render buffer to create a depth buffer */  
glGenRenderbuffers(1, &depthBuffer);  
glBindRenderbuffer(GL_RENDERBUFFER, depthBuffer);  
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH_COMPONENT24, width, height);
```
When a pixel in the colour attachment is about to be written: 
- depth buffer will be checked to see whether the pixel is closer to the viewer compared to a pixel already in that position (if any).
- If the new pixel is from a triangle closer to the viewer position, the depth buffer will be updated with the new, nearer value and the color attachment will be drawn. 
- If it is further away, both writes are discarded.

#### iii Vertex buffer and arrays
#buffers/vertex_buffer
**Vertex Buffer** - OpenGL data structures that hold all the required information for the rendering:
1. ordering of the data
2. How many elements are used per vertex, such as:
	- 3 for vertex coordinates, 4 for colour, and 2 for the texture coordinates
Stores all the data about the vertices we want to draw.
The buffer objects will contain the vertex and texture data

**Vertex array** - multiple vertex buffers bound together to be enabled or disabled by a single call as the source to draw from.

Vertex buffers inside a vertex array are bound tight to the shaders used.
- buffers will be used to “feed” the vertex data to the GPU
- **Format and positions** in the vertex array **have to match** the shader input definition
- Any differences, the *data will be misinterpreted*,** resulting in garbage** on the screen.

```cpp::VertexBuffer.h
#pragma once  
#include <vector>  
#include <glm/glm.hpp>  
#include <glad/glad.h>  
#include <GLFW/glfw3.h>  
#include "MainRenderData.h"  
  
class VertexBuffer {  
    GLuint vertexArray = 0;  
	GLuint vertexBuffer = 0;  
public:  
    void init();  
    void uploadData(OGLMesh vertexData);  
    void bind();  
    void unbind();  
    void draw(GLuint mode, unsigned int start,  
    unsigned int num);  
    void cleanup();  
};
```


`VertexBuffer::init() `
Set up the buffers
- `glGenVertexArray(1, &vertexArray) ` - creates a new vertex array object
- `glGenBuffers(1, &vertexBuffer)` - creates a vertex buffer object.
- `glBindVertexArray(vertexArray)` - bind the vertex array object
- `glBindBuffer(GL_ARRAY_BUFFER, vertexBuffer)` - bind the first buffer for the vertex data
- `glVertexAttribPointer()` - configures the buffer object with parameters:
	1. Input location in shaders, e.g. 0
	2. Number of elements, e.g. 3
	3. Element type, e.g. `GL_FLOAT`. 
	4. Are the elements normalized, for floating-point values `GL_FALSE` 
	5. Size of the vertex struct used, `sizeof(OGLVertex)` 
		- consisting of the position and the texture coordinate. 
	6. offset inside the `OGLVertex` struct, e.g. `offsetof(OGLVertex, uv)`
		- Used the C++ `offsetof()` macro to get the offsets of the position and the texture coordinates elements.
	
- ` glEnableVertexAttribArray(0)` - enable the vertex buffers which was just configured.
- `glBindBuffer(GL_ARRAY_BUFFER, 0)` - unbind the array buffer
- `glBindVertexArray(0)` - unbind the vertex array

`VertexBuffer::uploadData()`
Copies the data to the vertex buffers. We are using our own data type here to have all the per-vertex data as a single element. 
- `glBindVertexArray(vertexArray)`  - bind the vertex array
- `glBindBuffer(GL_ARRAY_BUFFER, vertexBuffer)`  - bind the vertex buffer
- `glBufferData()` - uploads the vertex data to the OpenGL buffer
	- calculates size by (number of elements in the vector) * (size of custom vertex data type)
	- Takes starting address for the memory copy, the address of the 1st vertex element
	- `GL_DYNAMIC_DRAW` - hints to the driver the data will be written and used multiple times
```c++
glBufferData(GL_ARRAY_BUFFER, 
	vertexData.vertices.size()* sizeof(OGLVertex), 
	&vertexData.vertices.at(0),  GL_DYNAMIC_DRAW);
```
- `glBindVertexArray(0)` - unbind the buffers


`VertexBuffer::bind() `and `VertexBuffer::unbind() `
similar to the Framebuffer class
- `glBindVertexArray(vertexArray)` - bind vertex array
- `glBindVertexArray(0)` - unbind vertex array


`VertexBuffer::draw() `
`glDrawArrays(mode, start, num)` - instructs OpenGL to draw the vertex array from the currently bound vertex array object
- `mode` - rendering mode. To draw triangles, use `GL_TRIANGLES`
- `start` - starting index in vertex array
- `num` - number of elements to be rendered

`VertexBuffer::cleanup()`
Frees the OpenGL resources
- `glDeleteBuffers(1, &vertexBuffer)`
- `glDeleteVertexArrays(1, &vertexArray)` 


#### iv. Textures
Used to make objects in the virtual world appear more realistic.
They can be :
- Procedurally generated
- generated from pictures taken from the real world
- Altered by graphics tools.

They can also be used to *transport vertex data to the GPU* in a very efficient way

To load images the project uses [STB image header](https://github.com/nothings/stb/blob/master/stb_image.h)
- A free header to load images from the system, such as PNG or JPEG, and make them available as a byte buffer for further usage.

```cpp
#define STB_IMAGE_IMPLEMENTATION
#include <stb_image.h>
#include "Texture.h"
```
`STB_IMAGE_IMPLEMENTATION` before the header is required only in a C++ file, to advise the header to activate the functions

```c++
#pragma once  
#include <string>  
#include <glad/glad.h>  
#include <GLFW/glfw3.h>  
  
class Texture {  
    GLuint mTexture = 0;  
public:  
    bool loadTexture(std::string textureFilename);  
    void bind();  
    void unbind();  
    void cleanup();  
};
```

`Texture::loadTexture()` 
Loads the file using the STB functions and creates the OpenGL texture
- 3 integer values are required for the STB loading function
	1. Texture width
	2. Texture height
	3. Number of channels (3 for RGB picture and 4 for RGBA, transparency channel).
- `stbi_set_flip_vertically_on_load()` - flips the image on the vertical axis, as the coordinate systems of the texture and the picture differ on the axis
- `stbi_load()` : 
	1. creates a memory area
	2. Reads the file from the system
	3. Flips the image as instructed before
	4. Fills the width, height, and channels with the values found in the image.
- If it can’t be loaded for some reason, such as the file was not found, free the memory
- `glGenTextures(1, &mTexture)` - generate a Texture object 
- `glBindTexture(GL_TEXTURE_2D, mTexture)` - bind the new texture as the current 2D texture
- `GL_TEXTURE_MIN_FILTER` - handles #downscaling (minification) the texture if it is drawn far away
	- `GL_LINEAR_MIPMAP_LINEAR` - trilinear sampling for minification in the project
- `GL_TEXTURE_MAG_FILTER` - handles #upscaling (magnification)) the texture if it is drawn close to view
	- `GL_LINEAR` - there is only linear filtering available.
- `GL_TEXTURE_WRAP_S & GL_TEXTURE_WRAP_T` - decides what happens on the positive and negative edges of the texture when drawn outside the defined area of the texture
	- `GL_REPEAT` - repeats the texture outside the range of 0 to 1. 
	- Similar to using only the fractional part of the texture coordinate, ignoring the integer part.
-  `glTexImage2D()` - loads the data to the graphics
	- loaded byte data from `stbi_load()`
	- `GL_RGBA` - for 4 component images
- `glGenerateMipmap(GL_TEXTURE_2D)` - generate #mipmaps. 
	- These are scaled-down versions of the original image, halving the width and height in every step.
	- Increases rendering speed - less data is read if the texture is far away
	- Reduces artifacts
- `glBindTexture(GL_TEXTURE_2D, 0)` - Unbind to prevent changes
- `stbi_image_free(textureData)` - free the memory allocated by the STB load call and return true

`Texture::bind() `and `Texture::unbind() `
similar to the Framebuffer class
- `glBindTexture(GL_TEXTURE_2D, mTexture)` - bind texture
- `glBindTexture(GL_TEXTURE_2D, 0)` - unbind texture