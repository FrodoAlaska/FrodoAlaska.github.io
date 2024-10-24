---
layout: post
title: Making The Engine
subtitle: Our Only Hope - Devlog 1
comments: true
tags: [gamedev, devlog, our-only-hope]
---

# Where The Hell Have You Been?
If you read my last devlog, you'd probably remember that I was going to do a devlog about collisions. If you read my last devlog, you'd also remember that it was a month ago, which was not my plan. What happened? Have I abandoned this game? Have I forsaken the great idea that is this game? Am I dead? All good questions that have the same answer: *no*. Even though the devlog about collisions is coming soon, the time between this devlog and the last was filled with a series of unfortunate events. Events that led me to rewrite the whole project. Events that made me halt any progress on the game. What are those events you may ask? Well, there were a lot, some personal some not. However, the one thing that I learned throughout the last month is to always keep a backup of your project. A piece of advice I always heard but never bothered to actually listen to. 

However, even though my laptop decided to fuck me over and delete my progress (like wtf?!), I still have a lot of updates to give you. The progress has been slow but steady. Throughout this month I have written the engine for this game twice. I have implemented collisions, an asset system, an event system, a scene system, and many others that I'll talk about shortly. The game is chugging along slowly. Don't worry, everything is going well... he said, shortly before getting killed by an angry mob of tarantulas. 

# Why Raylib?
In my last devlog, I said I was going to use SDL2 and C++ to make this game come to reality. While I am still stuck with C++, the same is not true with SDL2. Don't get me wrong, I *love* SDL. It gives me a level of freedom I adore and the code is also very professional and very well documented. Yet, as I was writing out the game's engine, I discovered a lot of mildly infuriating features that made me want to pull my hair out. One thing to keep in mind is that I know Raylib way more than SDL. I have used it for a year up until that point. And, while I want to stick with SDL2 in the future, I decided to abandon it in favor of a faster tool to finish this game without any hiccups. But why leave Raylib and go to SDL2? Why not stick with Raylib? While Raylib is by far the best C++ game framework I used out there, it very much feels like I'm using a game engine without an editor. I want to dedicate my learning effort to graphics programming and the inner workings of that field. If I'm going to spend all that time using Raylib--which is very much made for beginners and students--then I'm never going to learn what I want. 

Nonetheless, Raylib is pretty much the fastest tool to get this game done in time and without any hardships along the way (so far at least).

# The Engine 
Now that you understand the context, let's get into the actual meat of this devlog: the engine. But, before I tell you the anatomy of this engine, I need to tell you a secret: I actually worked on this engine for a year before using it here. Let me explain. 

In the beginning, in the early dark ages, I did not understand anything that I was doing. I would pretty much just watch videos or read articles and copy whatever they were doing. However, through slowly hitting my head against the wall I started to learn and understand what's good and what's bad. I now know which system to use and I know when and how to use it. The engine that I'm about to spill all the secrets of just in a few minutes is one that I've used for the past year. I completed many games while using it. But I also reiterated over the engine **a lot**. This engine looks vastly different from the engine I used to make my first game with it. Some systems stayed the way they are (I'll talk about why later, don't worry), but there are also many things that did not stay around. 

This engine, as you will see, also uses a style and paradigm that I do not necessarily like. I decided to use it, however, just because I knew I could make it in a short amount of time and with little effort. As I said I made this engine many times before, in fact, a lot of the code is just copy-pasted completely line by line from other games that I made. I think you're starting to see a pattern here, right? You'd be right to think that I'm not putting any effort into making this engine big, strong, and girthy (is that a dick joke? So immature), that's because I *am* not putting any effort into it. I want to get this game done before the end of the next month and for that to happen, I need to sacrifice the quality of either the engine or the gameplay and, for the longest time, that sacrifice was always the gameplay. I am planning to think about engine architecture and design after I finish this game but, currently, all I'm concerned about is the gameplay and only the gameplay. 

Nonetheless, I'll stop talking about some nonsense you probably don't give a shit about and I'll start discussing how I made this engine and why I took the design decisions that are present in the engine. This engine is broken up into multiple pieces, each piece has its own sets of functionalities and responsibilities. Besides very few exceptions, every system of the engine is decoupled from everything else. No system in the engine knows about the existence of any other systems (again, besides a few exceptions). If one system needs to communicate with another system, I use events to message that part of the engine. Without further ado, let's start with the first and most important system of the engine: the event system.

## The Event System 
While perhaps not the first thing that was implemented in the engine, the event system is probably one of the most important parts of this engine. *Everything* in the engine communicates with the event system in one way or another (One Direction is that you?). Do you want to switch the current scene to another scene? Events. Do you want to play audio? Events. Do you want to solve a collision between two entities? Events. Did your grandma fall down the stairs and you want to get some help? Events! The event system is what makes the engine so decoupled which, therefore, makes it easier to work with. Before, I would have to inject dependencies into every system in the engine, making it very hard to work with and a chore to debug. However, with events things have gotten way easier. This is why this article is sponsored by events. Events, keep your head clean from blood.

But how did I implement an event system? Since this engine uses OOP as its main programming paradigm, I did not shy away from any OOP ideas that might help me at the moment. That's why I used singletons to create an event system. In fact, the event system also uses templates and, besides one function, is completely implemented in the header file. Later, you'll see that I adopted the same design to implement asset loading. I am aware that this implementation is far from perfect and I am planning to make it better in the future, but for the time being, this is what worked for me.

If you go deep and think about what an event system really is, you'll find it to be a very simple concept. All an event system does is hold function pointers and call them when the time is right. In a header file somewhere in the project, there is a list of function pointer definitions to be used by the caller and the callee of any event. When some part of the engine "listens" to an event, a function pointer is specified and the event system will store this type of function pointer in its appropriate container for later use. Once one of the systems of the engine "dispatches" an event, the event system will once again take the function pointer that was specified (this is where templates are useful) and call every single instance of it in the appropriate container.

For example, this is the list of all of the "events" that can be listened to and dispatched currently in the engine: 

```cpp
using OnEntityCollision = std::function<void(std::string&, std::string&)>;
using OnSceneChange = std::function<void(SceneType)>;
using OnSoundPlay = std::function<void(std::string&&)>;
using OnMusicPlay = std::function<void(std::string&&)>;
using OnMusicStop = std::function<void(std::string&)>;
using OnQuit = std::function<void(void)>;
```

And, if you want to listen to an event, you would have to call the event manager class, get its instance (since it's a singleton), and call the *ListenToEvent* function, specifying the event type and pass in a lambda or a function with the same signature as the event type that was passed into the template argument.

```cpp
EventManager::Get().ListenToEvent<OnEntityCollision>([&](std::string& entityID1, std::string& entityID2){
    if(entityID1 == "Player" && entityID2 == "Enemy")
        playerHealth -= 10;
});
```

This function call above would have to be done in the constructor of the class that will listen to the event. However, with dispatching events, you can do it anywhere in the engine. And, by the way, nullptr functions are accounted for, so no segmentation faults could happen.

```cpp
EventManager::Get().DispatchEvent<OnEntityCollision>("Player", "Enemy");
```

The same thing goes with dispatching any type of event, just pass in the event type into the template and the appropriate arguments. But, you're probably wondering how can I pass in arguments to dispatch an event function call without knowing how many arguments (if any) the event takes? Well, with variadic types, of course. I think they're called something different (an argument pack?) in C++, but the same idea is there. Just let the function take in ellipses into the second template argument and forward it into the function pointer so that it can unpack the arguments there and use it. In fact, I'll just show you. What you're about to see below is the modern C++ way of making you puke. It sure is ugly but it works like a charm.

```cpp
template<typename T, typename... Args>
void DispatchEvent(Args&&... args)
```

 The first template argument is the event type and the second is just an unknown number of arguments that will be forwarded to the function pointer later. While this implementation does make you pass in any function signature that takes in a different number of arguments, it does not account for an event that has a return value. I actually never implemented an event system that has events of different return values. However, this system has worked for me so far and, till now, I have not needed an event type with a return value anyway. This event system allows for any place in the engine to dispatch any type of event it wants. While convenient, it can cause problems with debugging later, and that's precisely why a lot of the event systems out there don't allow for multiple places to dispatch any type of event. Only the "creator" of the event type *can* dispatch *that* event and no one else. There's always one publisher but multiple subscribers.

But how are the functions implemented anyway? That is something that I'm not going to show you. Not because I'm afraid of code being stolen or anything (why would you steal a piece of crap?) but, rather, because I don't want this article to cause you an excruciating eye sore that you've never felt before. As I said before, the implementation of the *DispatchEvent* function and the *ListenToEvent* function are completely done in the header file (since I am working with templates) but that's not the problem, the problem is the hundreds of if statements that are present in those implementations. Why you ask? To deduce the type of the template given, of course. In the *EventManager* class there are vectors of every type of event in the engine. A vector for the *OnEntityCollision*, another for the *OnQuit*, and another for the *OnSoundPlay* and every time one of the above functions is called, I try to guess every type of event the template could be and, when I guess the type correctly, I add the event type given into its appropriate vector. That way, whenever I need to call that event type, all I need to do is try to deduce the type of the template given again and call every event type in the appropriate vector, checking for nullptrs along the way. If you're really curious, you can go to the GitHub page here: [OurOnlyHope/Managers/EventManager.hpp]("https://github.com/FrodoAlaska/OurOnlyHope/blob/master/src/Managers/EventManager.hpp") and check for the code yourself.  

## Handaling The Assets 
While I can be given all credit for all of the systems that are in this engine, the *AssetManager* is one that I completely *cannot* take any credit for. This implementation of an asset handler is shamelessly taken from the amazing javidx9. Of course, this system has changed a lot throughout the times I've implemented it but the idea and spirit still go back to that one implementation I saw on Youtube: [Code-It-Yourself! Role Playing Game]("https://www.youtube.com/watch?v=xXXt3htgDok&t=2248s&ab_channel=javidx9"). Many of the systems in this engine do take "inspiration" from other implementations of the same idea, but this one is just more than inspiration. However, while the implementation of this system does work for my game and javidx9's game, it will not work for everyone or every type of game. As you will see in a second, this system loads all of the assets at once, even ones the player might not need in their current playthrough. For a small game with a few amount of assets, that's fine, but with a big game with a lot of assets, this system will completely not work. Obviously, you'd need to find a better, more sophisticated system.

But how does it work, anyway? As I said before, this system will load all of the assets at once at the beginning of the game. Music, sound, textures, and fonts all will be taken from the disk and completely loaded into the game. Even if you might not need a specific asset this playthrough, the asset will get loaded either way. So, for that reason, an appropriate function needs to be called at the beginning of the initialization stage of the engine. That function looks looks like this:

```cpp
AssetManager::Get().LoadAssets();
```

Oh yeah, did I forget to say that the *AssetManager* is also a singleton? Well, now you know. This function actually calls several other functions that will then load the assets. I made specific Load functions for every type of asset for better visualization, for both myself and other people who might read the code. 

```cpp
void AssetManager::LoadAssets()
{
  LoadFonts();
  LoadSprites();
  LoadSounds();
  LoadMusic();
}
```

As you can see, the *LoadAssets* function calls 4 different functions that will each load different assets. And, when the game is closed, the *UnloadAssets* function of the *AssetManager* needs to be called to, as the name says, unload all of the assets from memory.

```cpp
void AssetManager::UnloadAssets()
{
  UnloadFonts();
  UnloadSprites();
  UnloadSounds();
  UnloadMusic();
}
```

And, like the *LoadAssets* function the above function will also call to other functions that will unload the appropriate asset type. Ok great, but how are assets *actually* loaded and unloaded? The *AssetManager* singleton class has a bunch of *unordered_map*s with a key and value pair to denote the name (or ID) of an asset and the value which is the asset type (Music, Sound, Texture2D, etc.). This will make it easier to retrieve the appropriate assets when needed. For example, if, somewhere in the code, a system needs the "Zombie_Grunt" sound, that system only needs to call the *GetSound* function, passing in the appropriate string and the function will return the appropriate Sound (if it exists, of course).

```cpp
AssetManager::Get().GetSound("Zombie_Grunt");
```

And, as you can imagine, all of the asset types are handled this way. Besides *GetSound* there is also *GetMusic*, *GetSprite*, and *GetFont*. Every function will return the appropriate asset type depending on the given string. Now, the horrible thing about this system is that there is no error handling if the wrong string is passed into one of the functions. If the string that was passed in did not exist in the database or a misspelling of an already existing asset occurred, there would be no errors at all. But why? My only defense to this is that I work on my own on this engine and there has never been a need to implement such a feature since I never make a mistake in my life. In all seriousness though, this sort of error can be really terrible and it should be handled. However, I will add this feature to the list of features I need to add to this engine once I finish this game.  

As for the actual loading of the assets, all I do is add the appropriate key with the appropriate value (using LoadSound() from Raylib, for example) to the map and that's it. The function I'm about to show you might look complicated but all it does is make a utility lambda that takes the name of the asset ID and its path, it then creates a new entry and adds it to the map with the given ID and passes the path to the appropriate Raylib asset loading function. 

```cpp
void AssetManager::LoadSounds()
{
  auto load = [&](const std::string& sound, const std::string& path) {
     m_sounds[sound] = LoadSound(path.c_str()); 
  };

  // Adding sounds
  load("Button_Click", "assets/audio/button_click.wav");
  load("Sword_Swing", "assets/audio/sword_swing.wav");
  load("Zombie_Death-1", "assets/audio/sword_kill_zombie-1.wav");
  load("Zombie_Death-2", "assets/audio/sword_kill_zombie-2.wav");
  load("Zombie_Grunt", "assets/audio/zombie_grunt.wav");
}
```

Again, this system was taken from javidx9. It has saved me a lot throughout the times that I've used it. And, until I find a more sophisticated one that doesn't require an OOP mentality, I will gladly use it for the time being.

## Scenes And The Scene System
The scene system is possibly one of the worst systems in this whole engine. It is unintuitive, unscalable, and a heap of garbage that has infested my workflow ever since I implemented it. When I finish this game, the scene system is *the* first thing I'll completely remove from this engine. But how bad is it? 

Every scene in the engine inherits from a *Scene* interface. The *Scene* interface has two functions: *Update* and *Render*. Seems good so far. In the engine, there is a system that will handle the different scenes and the loading and unloading process. The scene system has an *unordered_map* of scenes where the key is an enum called *SceneType*, where all of the possible scenes are stored. And the value of the *unordered_map* is a *unique_ptr* of a *Scene* interface. Also, the scene system has another *Scene unique_ptr* which denotes the current scene. The scene system will use the current scene variable to call the *Update* and *Render* functions. This means the current scene variable will *point* to the desired scene in the *unordered_map*. The scene system also listens to the *OnSceneChange* event where it proceeds to load the desired scene that was given by the event parameter. But what seems to be the problem? Loading the scenes isn't the problem in this implementation. Rather, the problem comes whenever I want to create a new scene. 

The process of creating a new scene goes as follows: create a new class, make the class inherit from the *Scene* interface, override the virtual *Update* and *Render* functions, and proceed to add whatever functionality you want in the scene. Doesn't seem too bad, right? It doesn't... so far. I have used this system a lot throughout the last few months. I made a lot of games using this system. It has served me well... with small games. You see, the previous games I made with this system have all been small. At maximum, there were only three scenes that the game had. But, and this is where the problem arrives, when you start to have more scenes, way more scenes--more than 3 at least--you begin to have a problem. For example, in this game, I would like to have more than 3 scenes. In fact, I *do* have more than 3 scenes. All in all, I have 7 scenes. And, in the future, I would like to have more than just 7 scenes. So if I have to make a class (a .h and a .cpp) for *each* scene, my hair will start to go gray. A better implementation and a new system would have to be made... just not now. Add it to the list of things that need to be done after finishing this game. 

Currently, the main scene I work with is the *GameScene* which has everything to do with... well, the game. The player, the zombies, the score, the waves, the pausing, and everything else that has to do with gameplay. So, more often than not, the *GameScene* is my central hub of development in this engine. Later, once I get to it, I'll start working on UI which is where I'll give the other scenes more love and attention. 

## Audio
Currently, I don't have any control over any audio system in the game. Pretty much everything is interfaced with Raylib and its audio system. The only thing I do is use the *LoadSound/LoadMusic* function to load the sound/music, the *PlaySound/PlayMusicStream* function to play the sound/music, and *SetSoundVolume/SetMusicVolume* to control the volume of the sound/music. There is only the *AudioListener* class which sets some arbitrary volume values for the sounds and music. It also listens to all of the appropriate sound/music events. Essentially, the *AudioListener* will listen to the sound/music events, set the appropriate volume (which can be set by the player later on), and play the appropriate sounds/music. 

## Entities
Ever since I decided to take the plunge and make games from scratch, organizing entities has been one of the many things I struggled with. Of course, it is not the only thing I struggled with but it was--and still is--one of my major struggles. Previously, long ago, I had my entities broken up into two types: a dynamic entity and a static entity. However, I soon discovered that the two entities approach can lead to a lot of headaches, especially when you learn that a dynamic entity is actually a child class of the static entity class and the static entity class is a child of *another* base entity class. So you can see why I stopped using that complicated system. Right now, I have an entity system that, while good, is still lacking in some areas. But how does it work? 

Like the scenes, every entity inherits from an *Entity* class. The *Entity* class is a pure virtual class, meaning the functions that it has *must* be implemented by child classes. The *Entity* class has the *Update* and *Render* functions, just like the *Scene* interface. But, unlike the *Scene* interface, the *Entity* class has three variables. In the previous entity system, I had a lot of variables in the base entity class. Variables that not every entity needs. Learning from that mistake, I made the base entity class have as few variables as possible. Only the bare minimum every entity will need. Or, at least, what I think is the bare minimum an entity needs relative to this engine. Essentially, the entities in the engine only inherit a *Transform* component (oh, we'll get to that later), an ID which is a string, and a boolean flag which indicates if the entity is active or not. 

I have another section for components so I'll not get into it now. What I'll talk about here, though, is the other two variables: the ID and the active flag. I'll start with the ID as it is probably the least useful and it's pretty much used in one area of the engine. At the initialization stage of every entity, they can assign a unique ID that will help later with knowing which entity collided with whom. However, this ID is not necessarily unique for *every* entity in the game. Rather, the ID is used to denote the *type* of the entity. If the entity that is currently being initialized is the player, then the ID of the entity will be "Player". If that entity was a zombie, then the ID would be "Zombie". And so on. But could you not have a better way to handle that? Perhaps an enum? Passing a reference of the type somehow? I think you know what I'm going to say next. That's right, put it in the things-to-fix-after-finishing-this-game pile. But what about the active flag? Well, to explain that, I have to tell you about the way entities are managed in the engine.

In the *GameScene*, there is a class called the *EntityManager* which, as the name implies, manages every entity in the game. The *EntityManager* class has an *Update* and a *Render* function just like every entity. As you can see, the *EntityManager* is quite similar to the *SceneManager* so far. However, that is where the similarities end. Unlike the *SceneManager*, the *EntityManager* does not have an *unordered_map* of all of the entities. Instead, it has a *unique_ptr* of individual entities. A pointer to the Player, a pointer to a *ZombieManager* class which just manages all of the zombies in the game (I'll talk about it in depth in a later devlog), and so on and so forth. In the initialization stage, the *EntityManager* will initialize all of the entities that need to be initialized. In both the *Update* and the *Render* functions, a check is made before any of the entities get updated or rendered. Every entity that currently has its active flag set to true can be updated and rendered. Otherwise, the entity will be ignored completely. Doing things this way will ensure I don't have to delete and reallocate new entities during the runtime. All of the entities in the game--like the assets--get allocated and initialized once at the beginning of the game. If the player or a zombie dies, they will not be updated or rendered. They will simply be ignored. The only problem with this system is that I have to manually decide how to reset each entity. But, at the cost of fewer allocations, it is a very much welcome side effect. 

But what happens inside those *Update* and *Render* functions? In fact, how are the entities getting rendered? For that, I use components. Or, rather, composition.

## Composition Over Inheritance
If I ever decide to stick with OOP (for some reason) I will definitely value composition over inheritance every time. As I said before, organizing entities and giving them only the bare minimum that they need at a base level was always a struggle for me. I always had the dilemma of "what if". What if there is an entity that needs a texture? Don't all of the entities need textures of some sort? What if there is an entity that needs a collider? Don't all entities need colliders of some sort? At the end of these existential thoughts, I just end up with a base entity class that has a bunch of variables that some--but not all--entities need. But what happens when an entity doesn't require a collider or a texture? Should I just set them to null? What if these entities have different parameters and the collider has a fixed size across all entities? I think you understand where I'm coming from. I value composition over inheritance because I hit my head against the wall too many times from excessive inheritance. But what are the components in my engine anyway?

Previously, I talked about there being a *Transform* component in the base *Entity* class. The *Transform* component is just a class that holds three variables: a position, a rotation, and a scale. There are some functions associated with the *Transform* component but they aren't used as much as the variables. Besides the *Transform* component, there is also the *Sprite* component which gives an entity the ability to add a texture and draw that texture. The *Sprite* component has only one function: the *Render* function which takes in an lvalue reference to a transform component. Since, most of the time, you need the position, rotation, and scale of an entity to render a sprite in the exact position, with the specified rotation, and an applicable scale. The *Sprite* component itself only has a texture, the size of the sprite, and an origin. The constructor of the *Sprite* component takes a sprite ID and the specified size of the sprite. The *Sprite* component takes the sprite ID and retrieves the texture from the *AssetManager* using the sprite ID given. It then assigns the size of the sprite to be used later and sets the origin to the middle of the texture.

There are also two other components in the engine and those are the *PhysicsBody* and the *Collider*. These I will talk about in depth in the next devlog where everything physics and collisions will be present.

## Box2D, The Contact Listener, And Debug Draw
Since I will be making a devlog specifically about physics, Box2D, and collisions, this section will be quite short I promise. The integration of Box2D is not the easiest of things to do. Box2D uses real-world measurements (cm, m, kg, etc.) instead of pixels to assign its values. I had to extensively use conversion utility functions I made specifically to convert all of the real-world metrics into pixels. Basically, I had to cut a lot of corners in order to make Box2D a part of the engine. But, since I used Box2D before, I made these utility functions in hindsight before writing a single line of the *PhysicsBody* or *Collider* components.

After finishing these rather small but very useful functions, I went on to implement the Box2D world, Contact Listener, and Box2D's Debug Draw. Box2D makes it easy for you to create a physics world. All you have to do is declare a *b2World* class somewhere, pass a gravity value into it, call the *Step* function every frame and then that's it. I made *b2World* a global variable to make it easier to add bodies to the world and to get the head of the quadtree for the *ContactListener*. It's horrible I know but I'm ready to sacrifice true software engineering values in order to make a game.  Now that I had an easier way to add bodies into the world, I went on to create the *ContactListener* which will help with knowing exactly *which* body collided with which. 

Box2D stores its bodies in a quadtree. So, as is often the case, when you want to iterate over that quadtree to check which of the bodies are colliding with each other, you need to get the head or the beginning of the quadtree in order to go down the tree body by body. And, in order to get the beginning of the tree, you need to get a reference of the world and call a function called *GetContactList* which will return a *b2Body* pointer which you can use to go down the tree. In order to get the next node in the quadtree, you need to call the *GetNext* function that exists in the body. I'm not going to go over the implementation of the ContactListener for now since, again, I'll be making a specific devlog for all things collisions. 

And, finally, the last system of the engine is the *DebugDraw*. All it does, as the name suggests, is draw lines around the active bodies in the world. This is great for debugging purposes only and nothing else. It has become one of the best tools for me whenever I use Box2D. Again, I'm not going to go over it in this devlog. However, if you're curious, you can go look at it on the GitHub page of this project. 

# What's Next?
So, now that the engine is finished, what's next? Currently, the game is in a very stable state. Even though I'm currently writing about the engine, I actually moved on to other things in the game. My concern with the game right now is the player movement and the combat, which I will make devlogs about in the future. Keep in mind that I do these devlogs way after I finish whatever I'm talking about. So, don't worry, development is (currently) going smoothly. Unlike last time, I promise that the next devlog is going to be about collisions and my fight with Box2D and physics as a whole (man I wish that apple was an anvil). And, as usual, I actually already implemented collisions and the various Box2D systems as you can see above. But, nonetheless, I hope you're having a wonderful day and always remember...

Huzzah!
