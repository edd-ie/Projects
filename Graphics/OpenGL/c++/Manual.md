#opengl #rendering #research #Cpp 

# Resource
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
Use of `GLFW` - an open source toolkit that is used to handle the tasks around the application window
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
function
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
All events are stored in an `event queue` and must be handled by the application code.
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
