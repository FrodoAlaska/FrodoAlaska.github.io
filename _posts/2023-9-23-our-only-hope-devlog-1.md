---
layout: post
title: Making The Engine
subtitle: Our Only Hope - Devlog 1
comments: true
tags: [gamedev, devlog, our-only-hope]
---

# Where The Hell Have You Been?
If you read my last devlog, you'd probably remember that I was going to do a devlog about collisions. If you read my last devlog, you'd also remember that it was a month ago, which was not my plan. What happened? Have I abandoned this game? Have I forsaken the great idea that is this game? Am I dead? All good questions that have the same answer: *no*. Even though the devlog about collisions is coming soon, the time between this devlog and the last was filled with a series of unfortunate events. Events that lead me to rewrite the whole project. Events that made me halt any progress on the game. What are those events you may ask? Well, they were a lot, some personal some not. However, the one thing that I learned throughout the last month is to always keep a backup of your project. A piece of advice I always heard but never bothered to actually listen to. 

However, even though my laptop decided to fuck me over and delete my progress (like wtf?!), I still have a lot of updates to give you. The progress has been slow but steady. Throughout this month I have written the engine for this game twice. I have implemented collisions, an asset system, an event system, a scene system, and many others that I'll talk about shortly. The game is chugging along slowly. Don't worry, everything is going well... he said, shortly before getting killed by an angry mob of trantulas. 

# Why Raylib?
In my last devlog, I said I was going to use SDL2 and C++ to make this game come to reality. While I still stuck with C++, the same is not true with SDL2. Don't get me wrong, I *love* SDL. It gives me a level of freedom I adore and the code is also very proffessional and very well documented. Yet, as I was writting out the game's engine, I discovered a lot of midly infuratting features that made me pull my hair out. One thing to keep is mind is that I know Raylib way more than SDL. I have used it for a year up until that point. And while I want to stick with SDL2 in the future, I decided to abandon it in favor of a faster tool to finish this game without any hiccups. But why leave Raylib and go to SDL2? Why not stick with Raylib? While Raylib is by far the best C++ game framework I used out there, it very much feels like I'm using a game engine without an editor. I want to dedicate my learning effort towards graphics programming and the inner workings of that field. If I'm going to spend all that time using Raylib--which is very much made for beginners and students--then I'm never going to learn what I want. 

Nonetheless, Raylib is pretty much the fastest tool to get this game done in time and without any hardships along the way (for me atleast).

# The Engine 
Now that you understand the context, let's get into the actual meat of this devlog: the engine. But, before I tell you the anatomy of this engine, I need to tell you a secret: I actually worked on this engine for a year before using it here. Let me explain. 

In the beginning, in the early dark ages, I did not understand anything that I was doing. I would pretty much just watch videos or read articles and copy whatever they were doing. However, through slowly hitting my head against the wall I started to learn and understand what's good and what's bad. I now know which system to use and I know when and how to use it. The engine that I'm about to spill all the secrets of just in a few minutes is one that I've used for the past year. I completed many games while using it. But I also reiterated over the engine **a lot**. This engine looks vastly different from the engine I used to make my first game with it. Some systems stayed the way they are (I'll talk about why later, don't worry), but there are also many things that did not stay around. 

This engine, as you will see, also uses a style and paradigm that I do not neccassirly like. I decided to use it, however, just because I knew I could make it in a short amount of time and with little effort. As I said I made this engine many times before, in fact a lot of the code is just copy pasted completely line by line from other games that I made. I think you're starting to see a pattern here, right? You'd be right to think that I'm not putting any effort into making this engine big, strong, and girthy (is that a dick joke? So immature), that's because I *am* not putting any effort into it. I want to get this game done before the end of the next month and for that to happen, I need to sacrifice the quality of either the engine or the gameplay and, for the longest time, that sacrifice was always the gameplay. I am planning to think about engine architicture and design after I finish this game but currently all I'm concerned about is the gameplay and only the gameplay. 

Nonetheless, I'll stop talking about some nonesense you probably don't give a shit about and I'll start discussing how I made this engine and why I took the design decisions that are present in the engine. This engine is broken up into multiple pieces, each piece has it's own sets of functionalies that its resbonsible for. Besides very few exceptions, every system of the enigne is decoupled from everything else. No system in the engine knows about the existence of any other systems (again, besides few exceptions). If one system needs to communicate with another system, I use events to message that part of the engine. Without further a do, let's start with the first and most important system of the engine: the event system.

## The Event System 
While perhaps not the first thing that was implemented in the engine, the event system is probably one of the most important part of this engine. *Everything* in the engine communicates with the event system in one way or another (One Direction is that you?). You want to switch the current scene? Events. You want to play audio? Events. You want to solve a collision between two entities? Events. You grandma fell down the stairs and wants to get some help? Events! The event system is what makes the engine so decoupled which, therfore, makes it easier to work with. Before, I would have to inject dependecies into every system in the engine, making it very hard to work with and a chore to debug. However, with events things have gotten way easier. This is why this article is sponsored by events. Events, keep your head clean from blood.

But how did I implement an event sytem? Since this engine used OOP as its main programming paradigm, I did not shy away from any OOP ideas that might help me in the moment. That's why I used singletons to create an event system. In fact, the event system also uses templates and, besides one function, is completely implemented in the header file. Later, you'll see that I adopted the same design to implement asset loading. I am aware that this implementation is far from perfect and I am planning to make it better in the future, but for the time being, this is what worked for me.

If you go deep and think about what an event system really is, you'll find it to be a very simple concept. All an event system does is hold function pointers and call them when the time is right. In a header file somewhere in the project, there are a list of function pointer definitions to be used by the caller and calle of any event. When some part of the engine "listens" to an event, a function pointer is specified and the event system will store this type of function pointer in its appropriate container for later use. Once one of the systems of the engine "dispatchs" an event, the event system will once again take the function pointer that was specified (this is where templates are useful) and call every single instance of it in the appropriate container.

For example this is the list of all of the "events" that can be listened to and dispatched currently in the engine: 

`cpp
using OnEntityCollision = std::function<void(std::string&, std::string&)>;
using OnSceneChange = std::function<void(SceneType)>;
using OnSoundPlay = std::function<void(std::string&&)>;
using OnMusicPlay = std::function<void(std::string&&)>;
using OnMusicStop = std::function<void(std::string&)>;
using OnQuit = std::function<void(void)>;
`
And, if you want to listen to an event, you would have to call the event manager class, get its instance (since it's a singleton), and call the *ListenToEvent* function, specifing the event type and pass in a lambda or a function with the same signature as the event type that was passed into the template argument.

`cpp
EventManager::Get().ListenToEvent<OnEntityCollision>([&](std::string& entityID1, std::string& entityID2){
    if(entityID1 == "Player" && entityID2 == "Enemy")
        playerHealth -= 10;
});
`
This function call above would have to be done in the constructor of the class that will listen to the event. However, with dispatching events, you can do it anywhere in the engine. And, by the way, nullptr functions are accounted for, so no segmentation faults could happen.

`cpp
EventManager::Get().DispatchEvent<OnEntityCollision>("Player", "Enemy");
`

The same thing goes with dispatching any type of event, just pass in the event type into the template and the appropriate arguments. But, you're probably wondering how can I pass in arguments to dispatch an event function call without knowing how many arguments (if any) the event takes? Well, with veriadic types, of course. I think they're called something different (an argument pack?) in C++, but the same idea is there. Just let the function take in elipses into the second template argument and forward it into the function pointer so that it can unpack the arguments there and use it. In fact, I'll just show you. What you're about to see below is the modern C++ way of making you puke. It sure is ugly but it works like a charm.

`cpp
template<typename T, typename... Args>
void DispatchEvent(Args&&... args)
`

 The first template argument is the event type and the second is just an unknown number of arguments that will be forwarded to the function pointer later. While this implementation does make you pass in any function signature that takes in different number of arguments, it does not account for an event that has a return type. I actually never implemented an event system that has events of different return type. However, this system has worked for me so far and, till now, I have not needed an event type with a return type anyway. This event system allows for any place in the engine to dispatch any type of event it wants. While convienent, it can cause problems with debugging later, and that's precicesly why a lot of the event systems out there don't allow for multiple places to dispatch any type of event. Only the "creator" of the event type *can* dispatch *that* event and no one else. There's always one publisher but multiple subscribers.

But how are the functions implemented anyways? That is something that I'm not going to show you. Not because I'm afraid of code being stolen or anything (why would you steal a piece of crap?) but, rather, because I don't want this article to cause you pain that you've never felt before. As I said before, the implementation of the *DispatchEvent* function and the *ListenToEvent* function are completely done in the header file (since I am working with templates) but that's not the problem, the problem is the hundreds of if statements that are present in those implementations. Why you ask? To deduce the type of the template given, of course. In the *EventManager* class there are vectors of every type of event in the engine. A vector for the *OnEntityCollision*, another for the *OnQuit*, and another for the *OnSoundPlay* and everytime one of the above functions are called, I try to guess every type of event the template could be and, when I guess the type correctly, I add the event type given into its appropriate vector. That way, whenever I need to call that event type, all I need to do is try to deduce the type of the template given again and call every event type in the appropriate vector, checking for nullptrs along the way. If you're really curious, you can go to the github page here: [OurOnlyHope/Managers/EventManager.hpp]("https://github.com/MohamedAG2002/OurOnlyHope/blob/master/src/Managers/EventManager.hpp") and check for the code yourself.  

## Handaling The Assets 
While I can be given all credit for all of the systems that are in this engine, the *AssetManager* is one that I completely *cannot* take any credit for. This implementation of an asset handler is shamelessly taken from the amazing javidx9. Of course, this system has changed a lot throughout the times I've implemented it but the idea and spirit still goes back to that one implementation I saw from Youtube: [Code-It-Yourself! Role Playing Game]("https://www.youtube.com/watch?v=xXXt3htgDok&t=2248s&ab_channel=javidx9"). Many of the systems in this engine does take "inspiration" from other implementations of the same idea, but this one is just more than inspiration. However, while the implementation of this system does work for my game and javidx9's game, it will not work for everyone or every type of game. As you will see in a second, this system loads all of the assets at once, even ones the player might not need in their current playthrough. For a small game with a few amount of assets, that's fine, but with a big game with a lot of assets, this system will completely not work. Obviously, you'd need to find a better, more sophisticated system.

But how does it work, anyways? As I said before, this system will load all of the assets at once in the beginning of the game. Music, sound, textures, and fonts all will get taken from the disk and completely loaded into the game. Even if you might not need a specific asset this playthrough, the asset will get loaded eitherway. So, for that reason, an appropriate function needs to be called at the beginning of the initialization stage of the engine. That function looks looks like this:

`cpp
AssetManager::Get().LoadAssets();
`

Oh yeah, did I forget to say that the *AssetManager* is also a singleton? Well, now you know. This function actually calls several other functions that will then load the assets. I made specific Load functions for every type of asset for better visualization, for both myself and other people who might read the code. 

`cpp
void AssetManager::LoadAssets()
{
  LoadFonts();
  LoadSprites();
  LoadSounds();
  LoadMusic();
}
`

As you can see, the *LoadAssets()* function calls 4 different functions that will each do different things. And, when the game is closed, the *UnloadAssets* function of the *AssetManager* needs to be called to, as the name says, unload all of the assets from memory.

`cpp
void AssetManager::UnloadAssets()
{
  UnloadFonts();
  UnloadSprites();
  UnloadSounds();
  UnloadMusic();
}
`

And, like the *LoadAssets* function the above function will also call to other functions that will unload the appropriate asset type. Ok great, but how are assets *actually* loaded and unloaded? The *AssetManager* singleton class has a bunch of *unordered_maps* with a key and value pair to denote the name (or ID) of an asset and the value which is the asset type (Music, Sound, Texture2D, etc...). This will make it easier for retrieving the appropriate assets when needed. For example, if, somewhere in the code, a system needs the "Zombie_Grunt" sound, that system only needs to call the *GetSound* function, passing in the appropriate string and the function will return the appropriate Sound (if it exists, of course).

`cpp
AssetManager::Get().GetSound("Zombie_Grunt");
`

And, as you can imagine, all of the asset types are handled this way. Besides *GetSound* there is also *GetMusic*, *GetSprite*, and *GetFont*. Every function will return the appropriate asset type depending on the given string. Now, the horrible thing about this system is that there is no error handaling if the wrong string was passed into one of the functions. If the string that was passed in did not exist in the database or a misspleaning of an already existting asset occured, there would be no errors at all. But why? My only defense to this is that I work on my own on this engine and there has never been a need to implement such a feature since I never make a mistake in my life. In all seriousness though, this sort of error can be really terrible and it should be handled. However, I will add this feature to the list of features I need to add to this engine once I finish this game.  

As for the actual loading of the assets, all I do is add the appropriate key with the appropriate value (using LoadSound() from Raylib, for example) to the map and that's it. The function I'm about to show you might look complicated but all it does is make a utility lambda that takes the name of the asset ID and its path, it then indexes (or, rather, creates a new entry with the new key given) the map with the given ID and passes the path to the appropriate Raylib asset loading function. 

`cpp
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
`

Again, this system was taken from javidx9. It has saved me a lot throughout the times that I've used it. And, until I find a more sophisticated one that doesn't require OOP mentality, I will gladly use it for the time being.

## Scenes And The Scene System

## Audio

## Entities

## Composition Over Inheritence

## Box2D, The Contact Listener, And Debug Draw
Since I will be making a devlog specifically about physics, Box2D, and collisons, this section will be quite short I promise. The integration of Box2D is not the easiest of things to do. As you've seen above with the *PhysicsBody* and *Collider* components I had to use the conversion utility functions a lot. Again, Box2D works with real life metrics and that doesn't translate very well to pixels, or at least it did not in my case. So, since I used Box2D before, I made these utility functions before I wrote a single line of the *PhysicsBody* or *Collider* components.

`cpp
Vector2 B2Vec2ToVector2(b2Vec2 vec);
b2Vec2 Vector2ToB2Vec2(Vector2 vec);
b2BodyType BodyTypeToB2BodyType(BodyType type);
BodyType B2BodyTypeToBodyType(b2BodyType type);
`

After finishing these rather small but very useful functions, I went on to implement the Box2D world, Contact Listener, and the Box2D's Debug Draw. Box2D makes it easy for you to create a physics world. All you have to do is declare a *b2World* class somewhere, pass a gravity value into it, call the *Step* function every frame and then that's it. And, as disscussed above, I made *b2World* a global variable to make it easier to add bodies to the world. But, that was not the only reason I made the world variable global. The Contact Listener needs the world to do other things. Box2D stores its bodies in a quad tree. So, as often the case, when you want to iterate over that quad tree to check which of the bodies are colliding with each other, you need to get the head or the beginning of the quad tree in order to go down the tree body by body. And, in order to get the beginning of the tree, you need to get a reference of the world and call a function called *GetContactList* which will return a *b2Body* pointer which then you can use to go down the tree by calling the *GetNext* function that exists in the body. 

I'm not going to go over the implementation of the ContactListener for now since, again, I'll be making a specific devlog for all things collisions. And, finally, the last system of the engine is the DebugDraw. All it does, as the name suggest, is draw lines aroung the active bodies in the world. This is great for debugging purposses only and nothing else. It has become one of the best tools for me whenever I use Box2D. Again, I'm not going to go over it in this devlog. However, if you're curious, you can go look at it in the github page of this project. 

# What's Next?
So, now that the engine is finished, what's next? Currently, the game is in a very stable state. Even though I'm currently writting about the engine, I actually moved on to other things in the game. My concern with the game right now is the player movement and the combat, which I will make devlogs about in the future. Keep in mind that I do these devlogs way after I finish whatever I'm talking about. So, don't worry, development is (currently) going smoothly. Unlike last time, I promise that the next devlog is going to be about collisions and my fight with Box2D and physics as a whole (man I wish that apple was an anvil). And, as usual, I actually already implemented collisions and the various Box2D systems as you can see above. But, nonetheless, I hope you're having a wonderful day and always remember...

Huzzah!
