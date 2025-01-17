#sdl #inheritance #oop #polymorphism #Cpp 
# Inheritance
Share common functionality between similar classes and also create subtypes from existing types.

All games have objects of various types. In most cases, these objects will have a lot of the same data and require a lot of the same basic functions.
-  Almost all of our objects will be drawn to the screen, thus requiring a `draw` *function*
- A location to draw to, that is, `x` and `y` *position variables*
- We don't want static objects all the time, so we will need an `update` *function* 
- Finally a `clean` *function*, for destroying.

## Parent

```c++
#ifndef GAMEOBJECT_H  
#define GAMEOBJECT_H  
  
#include <string>  
#include <SDL.h>  
#include "TextureManager.h"  
  
class GameObject {  
protected:  
    std::string textureID;  
    int currentFrame;  
    int currentRow;  
    int xPos;  
    int yPos;  
    int objWidth;  
    int objHeight;  
public:  
    void load(int x, int y, int width, int height, std::string texture_id); 
    void draw(SDL_Renderer* renderer);  
    void update();  
    void clean();  
};  
  
#endif //GAMEOBJECT_H
```

**Note** - `Protected` status restricts access to class and sub classes.
`GameObject`.c :

```c++
#include "GameObject.h"
#include <utility>


void GameObject::load(int x, int y, int width, int height, std::string texture_id){
    xPos = x;
    yPos = y;
    objWidth = width;
    objHeight = height;
    textureID = std::move(texture_id); //Sends string to the function and the caller of the function will never use the string again
    currentRow = 1;
    currentFrame = 1;
}

void GameObject::draw(SDL_Renderer* renderer){
    TextureManager::Instance()->drawFrame(textureID,
        xPos, yPos,
        objWidth, objHeight,
        currentRow, currentFrame,
        renderer);
}

void GameObject::update() {
    xPos += 1;
}
```

`std::move` read : #string 
- [std::move - cppreference.com](https://en.cppreference.com/w/cpp/utility/move)
- [c++ - Passing std::string by Value or Reference - Stack Overflow](https://stackoverflow.com/questions/10789740/passing-stdstring-by-value-or-reference)

## Child
Inherit from it and create a class called `Player`.h:

```c++
#include "GameObject.h"
class Player : public GameObject // inherit from GameObject
{
public:
	void load(int x, int y, int width, int height, std::string textureID);
	void draw(SDL_Renderer* renderer);
	void update();
	void clean();
};   
```

The `::` operator is called the **scope** resolution operator, used to:
- Identify the specific place that some data or function resides.

Player.c :
```c++
void Player::load(int x, int y, int width, int height, std::string textureID){
	GameObject::load(x, y, width, height, std::move(textureID));
} 

void Player::draw(SDL_Renderer* renderer){
	GameObject::draw(renderer);
}

void Player::update() { 
	xPos += 1; 
}
```

## Implementation

Create these objects in the `Game` header file: 
```c++
GameObject gameObj; 
Player player;
```

Load them in the `init` *function* :
```c++
gameObj.load(100, 100, 128, 82, "animate"); 
player.load(300, 300, 128, 82, "animate");
```

Add to the render and update functions:
```c++
void Game::render(){
	SDL_RenderClear(renderer); // clear to the draw colour
	gameObj.draw(renderer);
	player.draw(renderer);
	SDL_RenderPresent(renderer); // draw to the screen
}

void Game::update(){
	gameObj.update();
	player.update();
}
```

Cap our frame rate slightly else the objects will move far too fast:
```c++
while(g_game->running()){
	g_game->handleEvents();
	g_game->update();
	g_game->render();
	SDL_Delay(10); // add the delay
}
```

Now build and run to see the separate objects.

# Polymorphism
Reference an object through a pointer to its parent or base class.

This allows storage of a list of `GameObject` in `Game` class with our explicitly referring to the class.

Instead of  creating individual objects, you can place them in a vector of their parent classes
```c++
std::vector gameObjects;

// Add the objects
GameObject* m_player = new player();
GameObject* m_enemy1 = new enemy();
GameObject* m_enemy2 = new enemy();
GameObject* m_enemy3 = new enemy();

gameObjects.push_back(m_player); 
gameObjects.push_back(m_enemy1); 
gameObjects.push_back(m_enemy2); 
gameObjects.push_back(m_enemy3);

```

The `Game::draw` function can now look something like this:
```c++
void Game::draw()
 {
	for(std::vector<GameObject*>::size_type i = 0;
	i != m_gameObjects.size(); i++) {
		  m_gameObjects[i]->draw(m_pRenderer);
	}

	// Or an enhanced for loop
	for(auto & gameObject : gameObjects){  
	    gameObject->update();  
	}
 }
```


Alter `GameObject` function to allow polymorphism:
```c++
virtual void load(int x, int y, int width, int height, std::string textureID);
virtual void draw(SDL_Renderer* pRenderer);
virtual void update();
virtual void clean();
```

`Virtual` keyword means that when calling this function through a pointer, it *uses  
the definition from the type of the object itself*, not the type of its pointer:
- This function would always call the `draw` function contained in `GameObject`, neither `Player` nor `Enemy`.