#opengl #graphics #gfx #engine #rendering 
# Resources
Study: 
- [Introduction | 3D Game Development with LWJGL 3](https://ahbejarano.gitbook.io/lwjglgamedev)
- [Learn OpenGL, extensive tutorial resource for learning Modern OpenGL](https://learnopengl.com/)
- [OpenGL with Java for Beginners : r/opengl](https://www.reddit.com/r/opengl/comments/1i1wrkx/opengl_with_java_for_beginners/) 
- [OpenGL 3D Game Tutorial 1: The Display](https://www.youtube.com/watch?v=VS8wlS9hF8E&list=PLRIWtICgwaX0u7Rf9zkZhLoLuZVfUksDP)

Downloads:
- [Customize - LWJGL](https://www.lwjgl.org/customize)

# Terms
## GLFW
[GLFW](https://www.glfw.org/)  -  a library to handle GUI components (Windows, etc.) and events (key presses, mouse movements, etc.) with an OpenGL context attached in a straightforward way.

Create a window:
```java
long window = glfwCreateWindow(300, 300, "Hello World!", NULL, NULL);
if (window == NULL)
    throw new RuntimeException("Failed to create the GLFW window");
```

Destroy window:
```java
// Free the window callbacks and destroy the window
glfwFreeCallbacks(window);
glfwDestroyWindow(window);

// Terminate GLFW and free the error callback
glfwTerminate();
glfwSetErrorCallback(null).free();

```

Window configurations:
```java 
// Configure GLFW
glfwDefaultWindowHints(); // optional, the current window hints are already the default
glfwWindowHint(GLFW_VISIBLE, GLFW_FALSE); // the window will stay hidden after creation
glfwWindowHint(GLFW_RESIZABLE, GLFW_TRUE); // the window will be resizable
```
## Memory Allocation
`MemoryStack` - class which allows allocation of native-accessible memory which is automatically cleaned when out of scope where `stackPush()` is called.
```java
MemoryStack stack = stackPush();
IntBuffer pointer = stack.mallocInt(1); // an int pointer
```


# Game Loop
Game loop structure:
```java
while (keepOnRunning) {
    input();
    update();
    render();
}
```

`input()` - Event handling
`update()` - updating game state (enemy positions, AI, etc..)
**Note** - if our `rendering` is not done in time it makes no sense to render old frames while processing our game loop. We have the flexibility to skip some frames.

## Support classes
An interface to encapsulate the game's logic

