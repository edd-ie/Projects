#opengl #rendering #research #Cpp 
# Window

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