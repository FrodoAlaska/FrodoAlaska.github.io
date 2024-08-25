---
layout: post
title: My Dream Game Engine
subtitle: And How I Failed To Create It
comments: true
tags: [gamedev, devlog, game-engine]
---

# Is That Even A Dream?
We always hear about the dream game. The game every game dev hopes to make. Whether it be an Open-world RPG with Dark Souls mechanics and cute characters like Animal Crossing or a Legend Of Zelda clone with horror elements, but we never hear about dream game engines, do we? 

There's a big reason for that: you use what's available. If you're an indie dev, you probably just use Unity, Godot, or Unreal. Nothing wrong with that obviously. But, if you are like me--mentally deranged, then you probably make your own game engine or even make your games from scratch. 

So, in this post, I'll be talking about that. The game engine of my dreams. The one I wish to use for all eternity. The one I wish to marry and have little game kids with. The one game engine I wish to die with... 

Wait, what we were talking about?? Oh... 

Before I outline to you my perfect big tiddy Goth game engine that I want to choke me, I'll first get into my ideas for the "perfect" game engine. Everyone's idea of a perfect game engine is different. So, it is only fair that I give you *my* own philosophies regarding libraries and their uses and, specifically, game engines. I also want to preface this by saying that this is *my* opinion. You might have a different opinion, and that is _completely_ okay. You might think I am wrong. No problem. Would love to hear why. But always remember, I am only stating my _opinion_ and not just hard _facts_. Those opinions are subject to change so don't take this as the bible of making game engines.

Good? Okay good. Now let's start.

# UI Or No UI. That Is The Question
Using the UI of the engine to make your game is great! One single button to add an entity to the scene. One button to add a child node. A few buttons to animate. Some other buttons to add a camera. However, I am a programmer. And that's a sentiment that is going to be echoed throughout this post. I like doing things the dirty way. Get down on my knees and... you get the point. Pressing on a button to add a scene is not intuitive to me. Making a `class` or a `struct` to hold the information of the scene _is_, however, intuitive to my programmer's brain. 

It might seem illogical to completely get rid of the UI and... it is. Having no UI for your engine at all is not the greatest idea. It is especially bad if you want your engine to be used by a wide variety of workers. Artists, musicians, designers, and what have you. Making the artist (who never coded in their life before, by the way) go for a deep-dive into your shitty codebase just to add an entity is, save to say, an absolutely _terrible_ idea. However, I don't want to make the prestigious programmer to get too comfortable and use the buttons to do the work either. So, what's the solution here? 

In my view, there are two separate parts to an engine: 
    - _Core_: This is where all of the fundamental logic of the engine exists. The rendering, the window creation, the event pooling, the input, the entity and scene management, and so on. 
    - _Editor_: This is where the front end of the engine lies. This layer will call functions from the _Core_ layer of the engine when a button is clicked, for example.

Therefore, in my opinion, you should give access to that core part of the engine to whoever desires to go that deep. You might even split the engine up into _three_ layers instead of two. The first is just the core (core rendering, window management, allocators, logging, input), the second layer which we will call the engine (lighting, shadows, entities, scenes, particles), and, finally, the third layer which is the editor as we discussed earlier. This might be more complex but it will make the user be able to choose the core framework of the engine if they don't like the way we handle entities for example (what assholes). 

There are already examples of this. Godot lets you access its own C++ layer so you can make your game without the need for an editor. Unreal, while not the same as Godot, does have its source code freely available on GitHub which you can just take and add or edit whichever you please. And Unity has... well, let's not talk about Unity. 

This way, you can appeal to Joe the programmer, and Abby the artist. Do you want to use the UI? Go ahead. You were dropped on your head as a child and now you're retarded and use this engine without UI? Sure, you can do that. And you shouldn't make the non-UI part of the engine hard to get up and running. Even though Godot has that GDNative plugin, it is a hassle to set up. The core engine is already a library. You can just give that library to the user and they can use it as they please.

# You, Me, And The User
Now, what is the most important thing about a library/framework? That's right. Say it with me now. Your mom. Yes. The user. Whoever is using this library or framework is the most important to you. 

Long ago (maybe like a week ago), I finished a 3D game of mine. Before starting the game, I went out of my way to make an engine for it. It was more of a template than an engine. It didn't have an editor or anything like that. However, it did make it easier to make games with OpenGL. I didn't have to touch one line of OpenGL or GLFW. Do you want to create a window? Just use this function: `window_create` done. Do you want to load a texture? Done. Get input? No problem. I thought everything was perfect. I thought I was the smartest person alive! But, as with everything in my life, I was wrong. It was the most horrendous library I ever used. Even worse than your mom yes.

It was called Gravel (the engine, not your mom). I only thought about the implementation of the engine. How could I load a 3D model? How can I make a texture rotate? How can I render text (cue Vietnam flashbacks)? I thought about it and then implemented it. No thought of the actual API. I didn't care how my code would be _used_. I only cared about how it will be _implemented_. In fact, with a lot of my libraries and frameworks, it was the same problem. The only thought I put into them was just the implementation.

That's why, in my opinion, it is _much_ better to outline the way you want the library to be used rather than focus on the implementation. You need to think about who will use this engine and what they are going to be using it for. If you have a solid foundation, you can always change the implementation later. Premature optimization is the devil. I don't know if that's true but Jared always tells me about it. 

For example, let us assume you have a function called something like this: `render_model`. This function, as the name implies, will render a 3D model at the specified transform position. Now, how is that function implemented? Doesn't fucking matter. Rather, the question you _should_ be asking is how will this function be used by the user. Is this function suitable the way it is set up currently to render a model without hassle? Try it. Make a game or two with it. Perhaps it doesn't make sense. Perhaps this function needs an extra parameter. Perhaps it doesn't. You _need_ to use your library in order to get a feeling of how the user will be using it. Stop thinking about the implementation for once and just think about the user. Take all of the hate and anger you endured from the C++ STD library and try to be the good guy here. 

# Everything Or Nothing
As programmers, we _love_ to overscope everything. We shall add lighting to this engine. Also, we will add some Navmesh pathfinding. And, after breakfast, we shall conquer that juicy ass. Calm down. You are not Unity. You are not Godot. And you are _not_ Unreal for damn sure. You are just one guy who is trying to allocate some free time to make a game engine. A game engine no one asked for. 

Let's assume you are trying to make a racing game. You have two choices: use Unity (shoot me in the balls) or use this other engine. Now, since I know you more than I know myself, I know you would have just picked Unity right? I'm right. I'm always right. However, that other engine is actually an engine specifically made for racing games. Many games were made and tested with this game engine. It has all the specifics and tools to help you make a racing game. Now, can you make a first-person shooter with this engine? Well, yes. You still have a camera that might be set to first person anyway. There's a way to render models and meshes, check collisions, fire events, and so on. Nothing is stopping you from making a FPS game with this engine. However, you'll be using its absolute _full_ potential if you make a racing game with it. 

And that's the problem with Unity. Unity is trying to be too many things at once. It's trying to appeal to the indie market (arguably, its biggest user base to date) _and_, at the same time, it is trying to emulate Unreal. Unity is trying to pull too many strings at once which is hurting its potential. At least with Unreal, it knows what it is: a pretty capable and strong game engine, ready to make any 3D game you have in mind. And, with Godot, it's trying to appeal to that indie audience. Making small simple games that don't require the complexity of many tools Unreal provides you with and at the same time being quite capable. 

Am I saying you should _not_ make a general-purpose game engine able to do every game in existence? No. What I _am_ saying, however, is to focus your attention on one area of the game engine. There are millions (too many but fuck you) of game engines out there. So many to even count. Each and every single one of them has a quirk. Something that makes it stand out from the others. Anyone with a sizable amount of experience can create a general-purpose game engine. What makes yours so special? 

Maybe your engine is really good at making grand strategy games. Maybe it is an engine that is specifically designed for ASCII games on the terminal. Perhaps your engine has a weird name like Game Maker. Ew. 

# Pick Your Battles, Son
*Write once and run everywhere.*

That's a sentence you probably heard before. Java, the all-great programming language, was marketed as a language that could run on any and all systems. And, to their credit, it was correct. You see, Java was made on top of a Virtual Machine (VM). This meant, to grossly oversimplify, it didn't have to write its own implementation on every CPU out there. You could just write the Java application and it _will_ run. 

If you ever tried to make a game with Unity, you would know that Unity could build and package up your game to multiple platforms. Windows, Linux, Mac, Switch, Android, IOS, and the Web. Now that's amazing! Any game you make you can play it anywhere! Expect that's not really the case. 

As you and I know, cross-compilation sucks major ass. I mean you can build for these systems, but then you will have to get a Linux VM up and running, get a Mac computer somehow, have both an Android _and_ an IOS device, know how to use emscripten or something similar to build for the web, and, finally, you will have to get a SDK for the Switch from Nintendo. Yeah. That's not possible. Perhaps (maybe), you can get Windows and Linux up and running. Maybe even Mac if you have the hardware (which I doubt). But building for mobile? Well, you _have_ to have a Mac computer in order to build for an IOS device. Android has its quirks that you will have to learn and worry about. And the web... well no cares about that. We can just give it a cookie. It'll be happy.

You have to pick and choose your battles. For example, currently, I can build for both Windows and Linux. Web and Android if I can be bothered. And, even though it is a pain in my Henry Tudor, I can still do it with no problem. However, the other platforms? No thanks. I will not risk my time or sanity for 2.5% of the desktop market. 

And this is not just about being platform-agnostic either. This is about everything you will come across on your engine-making journey. In a perfect world for me, I don't need any libraries. I don't have to import or compile any libraries I need for my project. I can just build _my_ project and that's it. However, I can't be bothered to load in and read PNG or JPEG files. So just use `stb_image` for that. Same thing with audio. Same thing with font loading. However, there are a few things _I am_ willing to spend time on. Dealing with the different OS libraries to create and handle window creation. Physics, math, rendering, and entity management. All of these things I am fond of and I wish to learn more about. I actually _do_ want to spend time delving deeper into t these topics. 

Let `stb_image` handle my textures for me. It is a pretty trusted library that I have used for _many_ of my projects. I don't care much about image loading either so why not?  

You have to pick your battles. You can't just do everything by yourself. Make sure to _not_ waste your time on something that you do not care about. Making an engine is a hobby after all. Treat it that way. Forever.
 
# The Dream Game Engine
 So, after all of that, what's the point? What's the point of all that rambling I meant not the... yeah. Well, these are thoughts I have had for the better part of a year. I wanted to make my _own_ game engine but I never found a good fit. I wanted to make something that I could be proud to show to people. An engine I can actually use for my own games without vomiting from disgust and anger. 

 Perhaps this is the best opportunity to announce my new series in this blog. The Panda Engine devlog series. This is an engine I have been preparing to make for a long time. An engine I thoroughly thought about. I can't tell you it will be the best engine in the damn world, but it is an engine that I am very passionate and excited about. 

 The upcoming devlog series will be somewhat educational. As I will try to explain the concepts I have learned and know along the way. Hopefully, you will get something out of it. And if you didn't... then I will seriously cry, dude.
