#ui #sdl 
# Installation

Tutorial [HOWTO: Setup CLion for SDL2 Gamedev](https://www.youtube.com/watch?v=N5CZLSVU0DA)
Alternative -[C++/SDL2 RPG Platformer Tutorial for Beginners Part 1 | Setting up SDL2 on Windows](https://www.youtube.com/watch?v=KsG6dJlLBDw&list=PL2RPjWnJduNmXHRYwdtublIPdlqocBoLS)

1. Ensure SDL2 is downloaded
2. Create `cmake_modules` directory in your project
3. Create a `FindSDL2.cmake` in the new dir
4. Paste [SDL finder code](https://gist.githubusercontent.com/416rehman/4b5a88f5e5f033dfe8f7abd7bb768bde/raw/e1ffd1d8aa8f5047ae7526c246581938b463eeba/gistfile1.txt) in the new cmake
5. Create a `FindSDL_image.cmake` in the new dir
6. Paste: 
```
SET(SDL_IMAGE_SEARCH_PATHS  
        ~/Library/Frameworks  
        /Library/Frameworks  
        /usr/local  
        /usr  
        /sw # Fink  
        /opt/local # DarwinPorts  
        /opt/csw # Blastwave  
        /opt  
        ${SDL_IMAGE_PATH}  
)  
  
  
FIND_PATH(SDL_IMAGE_INCLUDE_DIR  
        SDL_image.h  
        HINTS  
        $ENV{SDL_IMAGEDIR}  
        PATH_SUFFIXES include/SDL2 include  
        PATHS ${SDL_IMAGE_SEARCH_PATHS}  
)  
  
  
  
find_library(SDL_IMAGE_LIBRARY  
        NAMES SDL_image  
        PATHS ${SDL_IMAGE_PATH}/lib  
        NO_DEFAULT_PATH  
)  
  
include(FindPackageHandleStandardArgs)  
find_package_handle_standard_args(SDL_image # Keep this consistent with find_package(SDL_IMAGE)  
        REQUIRED_VARS SDL_IMAGE_INCLUDE_DIR SDL_IMAGE_LIBRARY  
)  
  
mark_as_advanced(SDL_IMAGE_INCLUDE_DIR SDL_IMAGE_LIBRARY)  
  
# Define upper case variable for consistency  
set(SDL_IMAGE_INCLUDE_DIRS ${SDL_IMAGE_INCLUDE_DIR})  
set(SDL_IMAGE_LIBRARIES ${SDL_IMAGE_LIBRARY})
```

7. Use this template to configure the main `CMAKEList.txt`
```cmake
cmake_minimum_required(VERSION 3.30)  
project(sdlTest)  
  
set(CMAKE_CXX_STANDARD 23)  
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake_modules)  
  
set(SDL2_PATH "X:/Code/C++/SDL2/x86_64-w64-mingw32")  
set(SDL_IMAGE_PATH "X:/Code/C++/SDL2_image/x86_64-w64-mingw32")  
  
find_package(SDL2 REQUIRED)  
include_directories(${SDL2_INCLUDE_DIR})  
  
find_package(SDL_image REQUIRED)  
include_directories(${SDL_IMAGE_INCLUDE_DIR})  
  
add_executable(sdlTest main.cpp)  
  
target_link_libraries(${PROJECT_NAME} ${SDL2_LIBRARY})  
target_link_libraries(${PROJECT_NAME} ${SDL_IMAGE_LIBRARY})
```

8. Add the `SDL.dll` file to the `cmake-build-debug` folder. The `SDL.dll` file is located at `..\SDL2\x86_64-w64-mingw32\bin`.
	- When you want to distribute your game to another computer, you will have to share this file as well as the executable.
9. Add the `SDL2_image.dll` file to the `cmake-build-debug` folder. The `SDL2_image.dll` file is located at `..\SDL2_image\x86_64-w64-mingw32\bin`.

# Getting Started
First, we included the `SDL.h` header file so that we have access to all of SDL's functions: 
```c++
#include <SDL.h>
```

## Create window & renderer. 

Make 2 global variable:
1. First is a pointer to an `SDL_Window` function, which will be set using the `SDL_CreateWindow` function. 
2. The second is a pointer to an `SDL_Renderer` object; set using the `SDL_CreateRenderer function`
``` c++
SDL_Window* g_pWindow = 0; 
SDL_Renderer* g_pRenderer = 0;
```

3. Initialize SDL. This example initializes all of SDL's subsystems using the `SDL_INIT_EVERYTHING` flag, but this does not always have to be the case *(see SDL initialization flags)*:
```c++ 
if(SDL_Init(SDL_INIT_EVERYTHING) >= 0){...}
else return 1 //Failed initialization
``` 

4. If the `SDL initialization` was successful, we can create the pointer to our window. `SDL_CreateWindow` **returns** a `pointer to a window` matching the passed parameters. 
- The parameters are:
	1. window title 
	2. x position of the window 
	3. y position of the window,
	4. width
	5. height
	6. any required SDL_ flags 
	- `SDL_WINDOWPOS_CENTERED` will center our window relative to the screen:
```c++
// if succeeded create our window
g_pWindow = SDL_CreateWindow("Setting up SDL", SDL_ WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, 640, 480, SDL_WINDOW_ SHOWN);
```

5. Check whether the window creation was successful, and if so, move on to set the pointer to our renderer.
- Passing the parameters:
	1. Window we want the renderer to use as a parameter, `g_pWindow` pointer. 
	2. The index of the rendering driver to initialize; in this case, 
		- we use **-1** to use the `first capable driver`. 
	3. The final parameter is `SDL_Renderer_Flag` (see SDL renderer flags):
```c++
// if the window creation succeeded create our renderer 
if(g_pWindow != 0) { 
	g_pRenderer = SDL_CreateRenderer(g_pWindow, -1, 0); 
} else { 
	return 1; // sdl could not create window 
}
```

## Show windows
If everything was successful, we can now create and show our window:
```c++
// everything succeeded lets draw the window  
// set to black // This function expects Red, Green, Blue and  
// Alpha as color values  
SDL_SetRenderDrawColor(g_pRenderer, 0, 0, 0, 255);
// clear the window to black  
SDL_RenderClear(g_pRenderer);
// show the window  
SDL_RenderPresent(g_pRenderer);  
// set a delay before quitting  
SDL_Delay(5000);  
// clean up SDL  
SDL_Quit();
```

# SDL Flags 
## Initialization <a id="init_flags"></a>
#sdl_init_flags
Event handling, file I/O, and threading subsystems are all initialized by default in SDL. Other subsystems can be initialized using the following flags:

| Flag                 | Initialized subsystem(s)   |
| -------------------- | -------------------------- |
| SDL_INIT_HAPTIC      | Force feedback subsystem   |
| SDL_INIT_AUDIO       | Audio subsystem            |
| SDL_INIT_VIDEO       | Video subsystem            |
| SDL_INIT_TIMER       | Timer subsystem            |
| SDL_INIT_JOYSTICK    | Joystick subsystem         |
| SDL_INIT_EVERYTHING  | All subsystems             |
| SDL_INIT_NOPARACHUTE |  Don't catch fatal signals |
We can also use bitwise `|` to initialize more than one subsystem. 
- To initialize only the audio and video subsystems, we can use a call to `SDL_Init`, for example: 
```c++
SDL_Init(SDL_INIT_AUDIO | SDL_INIT_VIDEO);
``` 

Checking whether a subsystem has been initialized or not can be done with a call to the `SDL_WasInit()` function: 
```c++
if(SDL_WasInit(SDL_INIT_VIDEO) != 0) { 
	cout << "video was initialized"; 
}
``` 

## Renderer 
#sdl_render_flags
When initializing an `SDL_Renderer` flag, we can pass in a flag to determine its behaviour. 

| Flags                      | Purpose                                                |
| -------------------------- | ------------------------------------------------------ |
| SDL_RENDERER_SOFTWARE      | Use software rendering                                 |
| SDL_RENDERER_ACCELERATED   | Use hardware acceleration                              |
| SDL_RENDERER_PRESENTVSYNC  | Synchronize renderer update with screen's refresh rate |
| SDL_RENDERER_TARGETTEXTURE | Supports render to texture                             |

## Window
#sdl_window_flags
`SDL_CreateWindow` takes an enumeration value of type `SDL_WindowFlags`. 
- These values set how the window will behave.


| Flag                     | Purpose                                      |
| ------------------------ | -------------------------------------------- |
| SDL_WINDOW_FULLSCREEN    | Make the window fullscreen                   |
| SDL_WINDOW_OPENGL        | Window can be used with as an OpenGL context |
| SDL_WINDOW_SHOWN         | The window is visible                        |
| SDL_WINDOW_HIDDEN        | Hide the window                              |
| SDL_WINDOW_BORDERLESS    | No border on the window                      |
| SDL_WINDOW_RESIZABLE     | Enable resizing of the window                |
| SDL_WINDOW_MINIMIZED     | Minimize the window                          |
| SDL_WINDOW_MAXIMIZED     | Maximize the window                          |
| SDL_WINDOW_INPUT_GRABBED | Window has grabbed input focus               |
| SDL_WINDOW_INPUT_FOCUS   | Window has input focus                       |
| SDL_WINDOW_MOUSE_FOCUS   | Window has mouse focus                       |
| SDL_WINDOW_FOREIGN       | The window was not created using SDL         |


# Game
## Loop
The loop involves these processes: 
- initialize
- get input
- do physics
- render
- exit. 
We will generalize these functions to:
	- init()
	- handleEvents()
	- update()
	- render()
	- clean()

## Class
Separate the functions into their own class, `Game.h`
- Classes in C++ are automatically private to make things public it has to be specified

**Game.h** :
```c++
#ifndef GAME_H  
#define GAME_H  
#include<SDL.h>  
  
  
class Game {  
	SDL_Window* g_pWindow = nullptr;  
	SDL_Renderer* g_pRenderer = nullptr;
    bool running;  
public:  
    Game() {  
        running = false;  
    }  
  
    void init(const char* title, const int xPos, const int yPos, const int  
        height, const int width, const int flags) {  
  
        running = true;  
    }  
  
    void render(){}  
    void update(){}  
    void handleEvents(){}  
    void clean(){}  
  
    bool isRunning() {  
        return running;  
    }  
};  
  
#endif //GAME_H
```

**main.cpp** :
```c++
#include "Game.h"  
  
// Game object  
Game* game = nullptr;  
  
int main(int argc, char* argv[])  
{  
    game = new Game();  
      
    game->init("Chapter 1", 100, 100, 640, 480, 0);  
    while(game->isRunning())  
    {  
        game->handleEvents();  
        game->update();  
        game->render();  
    }  
    game->clean();  
      
    return 0;  
}
```


# Drawing
SDL can use two structures to draw to the screen. 
1. `SDL_Surface` structure - which contains a collection of pixels and is rendered using software rendering processes (not the GPU). 
2. `SDL_Texture` - this can be used for hardware-accelerated rendering (more efficient).

create some rectangles to be used when drawing the texture.
```c++
SDL_Texture* m_pTexture; // the new SDL_Texture variable 
SDL_Rect m_sourceRectangle; // the first rectangle 
SDL_Rect m_destinationRectangle; // another rectangle
```
## Creating an SDL texture 
Create a pointer to an `SDL_Texture` object as a member variable in our *Game.h* header file.

We can load this texture in our game's `init` function for now. Open up `Game.cpp` and follow the steps to load and draw an `SDL_Texture`:

1. Create an assets/resources folder to hold our images, place this in the same folder as your source code (not the executable code). When you want to distribute the game you will copy this assets folder along with your executable.

### Load Image 
 2. In `init` function we can load our image. We will use the `SDL_LoadBMP` function which returns an `SDL_Surface*`. 
	 - From this `SDL_Surface*` we can create `SDL_Texture` structure using the `SDL_ CreateTextureFromSurface` function. 
	 - We then free the temporary surface, releasing any used memory.
	 -
``` c++
SDL_Surface* pTempSurface = SDL_LoadBMP("assets/rider.bmp"); 
m_pTexture = SDL_CreateTextureFromSurface(m_pRenderer, pTempSurface); SDL_FreeSurface(pTempSurface);
```

3. `SDL_Texture` ready to be drawn to the screen. Get the dimensions of the texture we have just loaded, and use them to set the width and height of `m_sourceRectangle` so that we can draw it correctly. 

```c++
SDL_QueryTexture(m_pTexture, nullptr, nullptr, 
	&m_sourceRectangle.w, &m_sourceRectangle.h);
```
- Querying the texture will allow us to set the width and height of our source rectangle to the exact dimensions needed

### Source and destination rectangles
Used for topics such as tile map loading and drawing. They are also important for sprite sheet animation.

**Source rectangle** - Defining the area we want to copy from a texture onto the window.(eg. LxW of a png)
**Destination triangle** - Specific area and position of the renderer.

```c++
sourceRectangle.w = 50; 
sourceRectangle.h = 50;
```
50 x 50 square of the image has been copied across to the renderer.

```c++
destinationRectangle.x = 100; 
destinationRectangle.y = 100;
```
The location that we want to display on the renderer

```c++
sourceRectangle.x = 50; 
sourceRectangle.y = 50;
```
The section of the image we want to display from (crop)

### Render image

In game's `render` function and we will add the code to draw our texture. 
Put this function between the calls to `SDL_ RenderClear` and `SDL_RenderPresent`.

```c++
//Option 1
SDL_RenderCopy(gRenderer, gTexture, &sourceRectangle,  &destinationRectangle);

//Option 1
SDL_RenderCopy(gRenderer, gTexture, &sourceRectangle,  nullptr);

//Option 2
SDL_RenderCopy(gRenderer, gTexture, nullptr,  &destinationRectangle);

//Option 3
SDL_RenderCopy(gRenderer, gTexture, nullptr, nullptr);
```

Option:
1. Renders image with constraints of both source and destination rectangle
2. Render section of image defined by source rectangle to full screen
3. Render full image to the potion of the renderer defined by destination triangle
4. Render full image to full screen.

## Animating a sprite sheet
#animating_sprites
**Sprite sheet** - a series of animation frames all put together into one image. 
- The separate frames need to have a very specific width and height so that they create a seamless motion.

Load the image using `SDL_image.h` :
```c++
SDL_Surface* pTempSurface =  IMG_Load("resources/pack/Block-Sheet.png");
```

Follow the steps to setup an image.
Configure the source image to display 1 section of the sheet.
```c++
destinationRectangle.x = sourceRectangle.x = 0;  
destinationRectangle.y = sourceRectangle.y = 0;  
sourceRectangle.w = 32;  
sourceRectangle.h = 32;  
destinationRectangle.w = 50;  
destinationRectangle.h = 50;
```

### Playing the animation
Once the image is setup, you'll need to move through the animation sheet to play the animation.
In the `update()` move through the sheet by multiplying the width/hieght by number of images*
- horizontal using `x-axis` 
- vertically using `y-axis`:
```c++
int sourceWidth = 32;
int numImages = 5;
int timeMs = 100;
sourceRectangle.x = sourceWidth * static_cast<int>((SDL_GetTicks() / timeMs) % numImages);
```

- `SDL_GetTicks()` used find out the amount of milliseconds since SDL was initialized, divide this by the amount of time (in ms) we want between frames % amount of frames we have in our animation. 
- This code will (every 100 milliseconds) shift the x value of our source rectangle by 32 pixels (the width of a frame), multiplied by the current frame we want

To play the animation is similar to displaying an image
```c++
SDL_RenderCopy(gRenderer, gTexture, &sourceRectangle,  &destinationRectangle);
```

## Rotating an image

`SDL_RenderCopyEx` takes the same parameters as `SDL_RenderCopy` but also takes specific parameters for rotation and flipping. 
- Fifth parameter -  the angle we want the image to be displayed 
- Sixth parameter - the center point we want for the rotation. 
- The final parameter - an enumerated type called `SDL_RendererFlip`.

| SDL_RendererFlip value | Purpose                       |
| ---------------------- | ----------------------------- |
| SDL_FLIP_NONE          | No flipping                   |
| SDL_FLIP_HORIZONTAL    | Flip the texture horizontally |
| SDL_FLIP_VERTICAL      | Flip the texture vertically   |
```c++
//Flip the texture horizontally
SDL_RenderCopyEx(m_pRenderer, m_pTexture, &m_sourceRectangle, &m_destinationRectangle, 0, 0, SDL_FLIP_HORIZONTAL);
```

## Texture Manager
#texture_manager
A class that creates functions that allow us to:
1. **Create/load** an `SDL_ Texture` structure from an image file
2. **Draw** the texture (either static or animated),
3. Hold a **list** of `SDL_Texture*`

### Create function 
The class will have 2 draw functions: `draw() & drawFrame`

Both take parameters:
1. ID of the texture we want to draw
2. x and y position we want to draw to
3. The height and width of the frame / image we are using
4. The renderer we will copy to
5. An `SDL_RendererFlip` value (default is *SDL_FLIP_NONE*). 

The `drawFrame` function has two additional parameters:
1. the current frame we want to draw 
2. Which row it is on in the sprite sheet.

```c++
TextureManager();  
~TextureManager();

// draw 
void draw(std::string id, int x, int y, int width, int height, SDL_Renderer* pRenderer, SDL_RendererFlip flip = SDL_FLIP_NONE); 

// drawframe 
void drawFrame(std::string id, int x, int y, int width, int height, int currentRow, int currentFrame, SDL_Renderer* pRenderer, SDL_RendererFlip flip = SDL_FLIP_NONE);
```

### Load function
As parameters, the function takes:
1. Filename of the image
2. The ID we want to use to refer to the texture
3. The renderer we want to use.
```c++
bool load(std::string fileName,std::string id, SDL_Renderer* pRenderer);
```

### List of texture objects
We will represent it a `std::map` where *texture_id* = **key** and *texture_pointer* = **value**

```c++
std::map<std::string, SDL_Texture*> textureMap;
```

### code
In `TextureManager.cpp` :
```c++
#include "TextureManager.h"  
#include <SDL.h>  
#include <SDL_image.h>  
#include <string>

TextureManager::TextureManager() {  
    textureMap = {};  
}

bool TextureManger::load(std::string fileName,std::string id,  
          SDL_Renderer* renderer) {  
    SDL_Surface* temp_surface = IMG_Load(fileName.c_str());  
  
    if(temp_surface == nullptr){  
        return false;  
    }  
    SDL_Texture* texture = SDL_CreateTextureFromSurface(renderer, temp_surface);  
    SDL_FreeSurface(temp_surface);  
  
    if(texture == nullptr){  
        // reaching here means something went wrong  
        return false;  
    }  
  
    // everything went ok, add the texture to our list  
    textureMap[id] = texture;  
    return true;  
}  
  
void TextureManger::draw(std::string id, int x, int y, int width, int height,  
    SDL_Renderer* pRenderer, SDL_RendererFlip flip) {  
    SDL_Rect srcRect;  
    SDL_Rect destRect;  
  
    srcRect.x = 0;  
    srcRect.y = 0;  
    srcRect.w = destRect.w = width;  
  
    srcRect.h = destRect.h = height;  
    destRect.x = x;  
    destRect.y = y;  
    SDL_RenderCopyEx(pRenderer, textureMap[id], &srcRect,  
    &destRect, 0, nullptr, flip);  
}  
  
  
void TextureManger::drawFrame(std::string id, int x, int y, int width, int height,  
    int currentRow, int currentFrame, SDL_Renderer* pRenderer,  
    SDL_RendererFlip flip) {  
  
    SDL_Rect srcRect;  
    SDL_Rect destRect;  
      
    srcRect.x = width * currentFrame;  
    srcRect.y = height * (currentRow - 1);  
  
    srcRect.w = destRect.w = width;  
    srcRect.h = destRect.h = height;  
    destRect.x = x;  
    destRect.y = y;  
  
    SDL_RenderCopyEx(pRenderer, textureMap[id], &srcRect,  
    &destRect, 0, 0, flip);  
}
```

In the `Game` class create 2 class variable:
1. Current frame - int.
2. Texture manager - pointer.
```c++
// Drawing  
int currentFrame;  
TextureManager* textureManager;
```

Load a texture in the game's `init` function. 
```c++
currentFrame = 0;
textureManager = new TextureManager();
textureManager->load("resource/animate-alpha.png", "animate", renderer);
```

Drawing a static image at 0,0 and an animated image at 100,100. 
In `render` function:
```c++
void Game::render() {  
    SDL_RenderClear(renderer); // clear the renderer to the draw color  
  
    textureManager->draw("animate", 0,0, 32, 32,  renderer);  
    textureManager->drawFrame("animate", 100,100, 32, 32, 1, currentFrame, renderer);  
  
    SDL_RenderPresent(renderer); // draw to the screen  
}
```

`drawFrame` function uses our `currentFrame` member variable. 
Increment it in the `update` function by doing the calculation of the source rectangle inside the draw function: *(refer to `playing the animation`)*
```c++
void Game::update() {  
    currentFrame = static_cast<int>(((SDL_GetTicks() / 100) % 5));  
}
```


### Singleton
A class that can only have one instance.
We want to have only 1 instance of the texture manager at any given time thus converting it to a singleton.

First making its constructor private. 
- This is to ensure that it cannot be created like other objects.
- Create a pointer to an instance of the class
- Create an instance function:
	- This function checks whether we already have an instance of our `TextureManager`.
	-  If not, then it constructs it, otherwise it simply returns the static instance.
- **Typedef**: A typedef is provided for ease of use, allowing you to refer to `TextureManager` as `TheTextureManager`.

```c++
class TextureManager {  
    std::map<std::string, SDL_Texture*> textureMap;  
  
    //conversion to Singleton = can only have 1 instance in a program  
    static TextureManager* instance;  
    TextureManager() {  
        textureMap = {};  
    }  
public:  
    static TextureManager* Instance(){  
        if(instance == nullptr)  
        {  
            instance = new TextureManager();  
            return instance;  
        }  
        return instance;  
    }
};

typedef TextureManager TheTextureManager;
```

Initialize the instance outside the class declaration, in the `TextureManager.cpp` file.
```c++
// TextureManager.cpp 
TextureManager* TextureManager::instance = nullptr;
```


Using it in the game class:
**Loading a texture** :
```c++
if(!TheTextureManager::Instance()->load("resources/pack/Effect Block-Sheet.png", "animate", renderer))  
{  
    std::cout << "failed to load animation texture\n";  
}
```

**Rendering an texture** :
```c++
TheTextureManager::Instance()->drawFrame("animate", 100,100, 32, 32, 1, currentFrame, renderer);
```
