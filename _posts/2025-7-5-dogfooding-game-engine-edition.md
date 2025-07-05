---
layout: post
title: Dogfooding - Game Engine Edition
comments: true
tags: [gamedev, game-development, game-engine, nikola-engine, nikola, discussion]
---

## Why Would I Feed My Dogs?

Have you ever heard this term before? _Dogfooding_. It sounds so weird, yet it is a very prevalent saying in the software development space. Essentially, it means **_to actively use the products you are creating_**. 

And if you're an eagle-eyed reader (nice glasses, by the way), you'll know what I'll talk about today. 

In the past 8 months, I have been creating my own game engine from scratch. Using a few libraries here and there, I was able to create a very steady base that can be used to make games in the future. And, in the last month or so, I was able to _finally_ finish and release quite a simple [game](https://frodoalaska.itch.io/crossing-the-line) using it. 

Now, yes, indeed, the game does not look the best, nor am I claiming that it's the next Metal Gear Solid. However, through the process of creating this game, I discovered a lot of things about myself as a developer and about the game engine I've been creating. 

I want to go in-depth about what I discovered while creating this game about game engines, my personal experience, and, perhaps, something useful you can take away from all of this nonsense. 

## The Two Kinds

Before going into greater detail about anything, I first need to establish a few discoveries I have made while being in the game engine sphere for the past 4 years or so. Call them _observations_, if you will. They may be wrong, but they are what _I_ have seen. Feel free to denounce them or disagree with them. This is the internet, after all, where all civil discussions are known to take place.

Firstly, what I have seen is that there are two types of people who get into game engine development in the first place: _Those who wish to make a game from scratch_ and _those who wish to make game engines_. There is a third category that I noticed, which is _those who just wish to dip their toes a bit and try new things_. However, for the purpose of this discussion, that category is too broad and doesn't concern us either way. 

Even though categorizing people is quite fun (and sometimes dangerous), it might not be enough. We have to understand the motives of both sides and what makes them take this path. 

In the first category, the motives are quite clear: _To make a game_. But, especially in modern times, why would one make a game without a game engine? Isn't it harder and way more prone to human error to use a custom game engine as opposed to making one using a game engine that has existed for many years and has been developed by _thousands_ of talented engineers? While it certainly is more, well, let's just say, "harder" to make a game without an engine, it is also _way_ more enjoyable, at least in my opinion. I have gone [in-depth](https://frodoalaska.github.io/2023-07-13-why-make-a-game-from-scratch/) about why _I_, personally, make games without game engines, so I won't repeat the same points here. 

As for the second category, the motives are to make a _game engine_. In other words, they intend to make _software_ for others to create games with. It is purely a _technical_ endeavor. And while I am not claiming that there is no need for creativity in a purely technical setting, I am claiming that the _type_ of creativity a game engine programmer requires is quite different from one who wishes to create a game. Especially when game design is concerned, the type of creativity differs wildly. And while these types of engineers do like video games in general, they are more interested in the technical aspects of video games and how they are made rather than the _design_ of the game itself. Once again, there's absolutely nothing wrong with that, but do keep that in mind as we continue our discussion.

Even though the motives of these two categories differ, their _path_ to achieve their motives is, in my opinion, quite similar. And I would even go as far as to say that they share the same motives in principle. Perhaps even the same feeling. The feeling that **the current state of game engines is in a dire state**. 

Ominous, perhaps, but let me explain. And since I don't want to drag anyone in particular down, let me give a small example that I know a lot about: my games and game engines. 

## Me And Game Engines 

As I said before, I have been developing a [game engine](https://github.com/FrodoAlaska/Nikola.git) for the past 8 months now. However, that is not the first game engine I ever created, and it certainly won't be the last. In fact, I have been developing engines for the past 4 years now. That doesn't mean that I never used popular game engines like Unity or the very famous Construct 3 game engines, but it does mean that I have developed more games using my own engines than using popular ones.

At the very beginning, I used a wide variety of frameworks. And, by the way, people often mix up the terminology; frameworks are not game engines. It is exactly what it sounds like, a _framework_. A library. A set of functions and types. While game engines are _software_. Just like Adobe Premier, Blender, or Gimp are software. There's a _big_ difference there. Sorry about that, but it has always been a pet peeve.

Anyhow, along the way, I gained a lot of experience working with SDL, Raylib, and SFML. And while these frameworks are _fantastic_, they always lacked the level of control needed to make a game with all the types of necessities _I_ needed. For example, most of these frameworks are mainly _2D_. And yes, Raylib does have 3D capabilities, but it is very limited, and it is only great if you want to play around with minimal 3D settings. Often, as well, the performance with some of these frameworks is not the best. And I know that most of these frameworks are open source, and I can just fork their repositories and work from there, but I'd much rather work with my own code than dive deep into someone else's codebase. I do that for work already, and it's not fun. At the time, though, that did not concern me. I only wanted to do 2D games because that's the only thing I _thought_ I could do. 

The "architecture" -- if you can even call it that -- of these games was quite simple. A `struct` here and there or some `class`es if I ever felt too academic, and that was it. As I went along, I tried to adopt concepts from other game engines I used, like scenes and entities, for example. Of course, being the immature programmer that I was, I failed miserably, since the entities either were way too generic or way too specific. On top of that, since the games were quite simple and the codebase somewhat small (some games did not pass 3000 lines of code), I would throw away every piece of code I've written and start from scratch on my next project. 

Eventually, though, as I moved away from frameworks and more towards OpenGL and GLFW, I discovered that throwing away code was not suitable anymore. It was a waste of time to have to write the same thing again and again. It wasted precious time I would have rather spent making or designing the game. Yet, when I did try to make a solid base that I could use, I'd always fail. And by "fail," I mean "hate" it. 

My first attempt at this was a game engine I named "Gilgamesh". Aside from the very cool name, the engine lacked any kind of user-friendly behaviour. The functions were sporadic and did not make any sense. The types were all over the place, and sometimes big chunks of useful functionality were missing from the engine entirely because I did not know what I was doing. The second and, arguably, a more "successful" attempt was the "Gravel" game engine. Again, great time but a terrible piece of software. 

The problem with every game engine I created was that I _assumed_ the functionality I needed, and that _assumption_ led me into making terrible decisions. Besides the assumptions, I also never truly thought about _how_ the engine would be used, but instead, I just thought it would be cool if my game engine had a very cool Entity Component System. I never asked if adding a certain functionality was useful or not; I just wanted to add it because I thought it was cool and I saw people talking about it. 

With this newest game engine of mine, I decided to remove assumptions completely. If I ever _assumed_ anything, I would immediately throw it out. Any system that would benefit my future games directly will be added with care. If any system, on the other hand, had no place to be here, and I saw no use case for it, then I would not even consider it. How cool or interesting it might be. 

That's a cool underground story, but what does it have to do with the current state of game engines? Well...

## Feed Your Dogs, Please

Game engines are software that are created solely for making games. I mean, it's in the name. However, in the past decade or so, game engines have also been used in other fields like the animation and film industry. And while that is a very cool prospect with lots of promise, it diminishes the power and versatility of a _game_ engine. Let me explain. 

In the past, say the 1990s, "game engines" were not really a thing. Well, not exactly. 

There were game engines that did exist at the time. Namely, the Id Tech engine was the first of its kind. But, in general, every game studio made its own proprietary game engine to be used for _their_ specific genre. The folks at Interplay (the people who created the first Fallout) used their own custom game engine since they were making games for a very specific genre. Naughty Dog, as well, used their own engine. So did Blizzard and Valve. Every game studio had a custom game engine for their games. 

Now, let's fast forward a bit to modern times, practically every game studio uses Unreal. Apart from a few outliers here and there, Unreal _dominates_ the game development scene. And why wouldn't they? They have the best rendering technology the world has ever seen. Their engine supports multiple platforms, including all kinds of game consoles. They have a pretty chill licensing model. And the pros go on and on. So what's the problem here? 

The difference between now and then is that game engines back then were made _purely_ for game creation purposes. After the release of every new game, the engineers would go back to the engine and improve it for the next game. It was an active feedback loop that incrementally improved the engine bit by bit. The engines did not just improve their physics or their graphics; they also improved their tooling and user experience, since many users of the engine were active employees of the company, so they could just ask their programming co-worker to add a certain feature or fix an annoying bug that was prevalent during development. It was _dogfooding_ at its best. A game company is making a game engine. 

Epic Games, though? They don't have that feedback loop. At least not as tight as it was back then. 

Game engines like Unity and Unreal are created purely for profit. They are _software_ companies, not _game_ companies. And while that term is quite loose when it comes to Epic Games, it is especially true for Unity and Godot, where the only development that actively takes place is the engines themselves. 

Now, listen, there is absolutely no problem with that. I'm not claiming that these companies are predatory or whatnot. However, I am claiming that it is hurting the game development community since everyone thinks that this is the _peak_ of game development and this is how games _should_ be developed. 

Unreal was, and still is, the smartest company that exists in the game industry. The licensing model they had for years and the technological edge they had over their opponents made them the _kings_ of game engines for years. And, to some extent, they still are the kings. They discovered that making game _tools_ is way more profitable than making _games_. 

Think about it. The Havok Physics Engine that powered the physics of so many games in the past was profitable not because it made games, but because it made a very useful tool. I mean, why would I gamble my money on a game that might or might not reimburse my investment? What if people hate it? What if it bombs? What then? Especially nowadays, where the budget of certain AAA games can balloon to _hundreds of millions_ of dollars. Millions for a few pixels moving on a screen. Why should I risk it? Let's just make Unreal 6.5 or Assassin's Creed 79.

Unreal developed and improved its technology because it had to. If it wanted to survive and keep making money, then it _had_ to be the best. I am not taking away from the time, effort, and dedication the wonderful engineers at Epic Games put into the engine. However, the fundamental business model, or _just_ developing a tool without using it, is flawed. 

Again, I am not blaming Epic Games as much as I am blaming the developers at Unity, Godot, or any software company that doesn't put itself in its customer's position. After all, the most important thing for a tool to do is to _help_ people create more stuff. Blender was created to ease the creation of 3D models and make it a more enjoyable process while having the low, low price of _free_. Game engines, as well, were created to ease the process of game creation. 

Now, you tell me, have you ever used any of these engines? Has it truly been a _great_ experience, or was it the only way you could make a game, so you just accepted your fate and moved along? Is the game creation process when using Unity, Godot, or Unreal really that easy, simple, and enjoyable? I do understand that games are not easy to create, but that doesn't mean that it should be a drudge to create them, because the UI programmers at Unity Inc. never created a game with their engine, so they don't understand how awful their UI is.

This is where I'll return to my previous categorization of the two people who get into making game engines: _Those who wish to make a game from scratch_ and _those who wish to make game engines_. 

I am not saying that an Entity Component System is not useful or fun to add, but is it really necessary to add it to a 2D platformer? Can I not make this 2D platformer unless I use an ECS? Do I really need to add a monounmental physics engine that will increase the compile time of my game just because I _might_ need it? And this is the problem I see with half, if not, _all_ the game engines that get created. They always lack the idea of a truly satisfying user experience. Go and use your engine to make a game for once. Does it hold up? Is it annoying? Is it fun to work with? Do the functionalities just work, or do you need to cut corners? Why should I get a script from the Unity asset store that adds functionality the engine itself should have in the first place? 

This is not how games should be created. If, say, I'm trying to create a level, I should have a _killer_ level editor that makes the process of level creation simple and easy. Level designers have to create and edit levels _constantly_. A lot of times, levels get thrown out completely after playtests because they suck. The editor should make up for that, where, perhaps, I can make a level in minutes, not hours. 

"But some games are really huge," I hear you say. And that's what I'm saying. Game engines claim to be "general-purpose," but you can only go so far with generalization. Look at level editors like DoomEd or Valve's Hammer. Or Diablo's level editor, even. They are all _fantastic_ tools to create levels for their respective games. Not only for developers to use them, but also for modders and normal folks. These tools are a work of art. They set out to solve an already-existing problem with known parameters, and they do it _very_ well. 

And this is where things get a little "fantastical". In this section, I'll get into more about what I wish game engines could be in the future, and a few ideas and paths I might be going towards. So if you don't agree with any of the crazy man rambalings from before, then you'll _definitely_ not like this one. But keep reading because I'll cry if you don't. 

## Do Not Generalize

One of the absolute _worst_ things you can do with your game engine is to _generalize_ it.

I always hoped that Unreal would share its rendering system as a component or framework that people can use. Instead of building an ecosystem of half-working systems, why not make the thing they're best at and distribute it to people? Just like Havok did with their physics engine. Of course, that doesn't work anymore since Unreal was built as software from the beginning, so separating the renderer, which can be tied to the resource manager and the animation system, can be quite a hassle. However, it can be done with other engines _if_ they were built like that from the beginning. For instance, look at an engine like [Ogre3D](https://www.ogre3d.org/), which is purely a _graphics_ engine and was used for plenty of game engines. 

What I'm saying here is that you shouldn't create a general-purpose game engine. Unless you truly _love_ general-purpose game engines, there is absolutely no reason to build them. Instead, build a game engine specific to a game genre.

And while it might not be as popular, RPG Maker is the epitome of this idea. It is a game engine only built for making story-based 2D RPG/JRPG games. And it is very successful as well. We can also take a look at the first iteration of Unreal, which was a game engine built purely for First-Person Shooters. So were the early Id Tech engines, the Build engine, and a bunch more. RenPy, too, is for creating Visual Novels.

If you're making a game engine for a specific genre, then you'll know what problems to solve. There are certain parameters or surface areas that you will operate in. Perhaps you do not need to build a resource manager that loads and caches resources on the fly. If I know that my game engine is going to only make level-based games, I can orient my resource manager around that idea. Maybe the resource manager will load the resources that are specific to this level. And if I do need to cache some resources (like the player's model, for example), I can add that functionality easily for the user to control. 

The same thing with physics. If I know that my game engine will only need support for cube colliders, then I only need to add collision detection and resolution for cube colliders and nothing else. Perhaps you don't even need a physics engine at all. Just a velocity and a direction vector.

As soon as you try to generalize, you run into problems that require complicated solutions. Solutions that might not be satisfactory or even doable at a smaller scale. Unless you wish to create the next Unreal engine, there is no need to make a general-purpose game engine. 

Some problems do require a general-purpose solution. For example, scenes and entities often do require a general-purpose solution. However, even then, you can get away with just having a god entity `struct` or `class` and making sure not to use the player's variables with a certain enemy. You'd be surprised how far you can go with such a solution. 

## I Could Not Think Of A Better Conclusion Title 

At the end of the day, you should carve your own path through this treacherous landscape that is game development. Game engines are not the end-all-be-all. They are not the final answer. They are not the final solution. They are only another iteration of a problem we have had since the inception of game development. Not one game engine is perfect. And not one solution solves all problems. 

Keep making games, and please, feed your dogs.
