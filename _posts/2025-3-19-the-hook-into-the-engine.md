---
layout: post
title: The Hook Into The Engine
subtitle: Nikola Engine Devlog 3
comments: true
tags: [gamedev, game-development, cmake, make, game-engine, nikola-engine, nikola, devlog, discussion]
---

## Unorthodox At Best 

Whenever I stumbled upon other game engines, I always saw the main "attractions", so to say, the renderer, the performance, the cool lighting effects, or what have you. While those features are indeed interesting and surely are very useful, they are not the _only_ thing that makes a game engine. 

After all, "game engines" are just a way for us developers to make a game. A neat set of tools we can use to make cool games. Yet, I often see many game engines neglecting user experience and instead opting to focus on more, in my opinion, unnecessary aspects of games. Once again, I would love for my games to have physically-based rendering and nice post-processing effects, with fast loading times and all else. But that is not _important_ in the grand scheme of things. 

When the game engine developers only focus on technical aspects and not much else, you get a complex beast that is hard to work with, clunky, and extremely annoying to do even the most trivial of tasks. But it has hot-reloadable resources, though! You get my point.

In previous devlogs, I talked in detail about what I wanted to do with this engine. A specific point that fits perfectly with this devlog is the user experience. Or, rather, _my_ experience. I admit, it is very easy to get bogged down in all the technical details and the immense complexity engines have. A lot of these game engines are just hobby projects and their creators are very passionate engineers who _love_ that sort of thing. And all power to them. However, I want to stay on track with my initial goals for this game engine. One such goal is to make the creation of games with this game engine as bloat-free, versatile, and comforting as possible. I shouldn't fight with complex systems just to get a simple pixel rendering on the screen. I shouldn't tinker for hours with the build system in order to get the engine up and running. So, with that in mind, I wanted a fairly simple way of _hooking_ a game into the engine. Or more like using the engine in some way to make a demo at least. 

The engine I have right now is barebones at best. It lacks many features and it will probably be like that for a bit, as I have discussed in previous devlogs. But, in order to test new features, I still want a way to use the engine as I would if it were finished. I wanted to have a simple system where I could easily use the engine's various functionalities without having to set up any annoying build environments or whatnot. If I were to create many different applications using the engine, I would easily be able to do that without much friction. 

Now, there are two parts to this: the build system and the application hook, as I call it. My method of implementing both is, well, unorthodox at best. Let's take a look, shall we?

## Project And Engine 

So, where does the _engine_ end and where does the _game_ begin? Well, it is quite simple, actually. In my case, at least.

I structured this engine in such a way that makes it fairly simple to use it as a library. Well, every game engine is just that, really. A framework/library of some sort that a project can supplement its functionality. I did not veer off much from this path. Though, as I said, my implementation of this "project and engine" feature is quite, well, "different".

There are two ways to use this game engine: the hard way and the simple way. The hard way has you managing the state of the window, graphics context, initializing the library, spinning up a loop, and then, of course, shutting everything down once you are done. There is no renderer, there is no camera, and while you can still work with resources such as a `nikola::Mesh` or a `nikola::Material`, these types are designed to be used with the built-in renderer. And, if you don't want to use that, you will have to whip up your own renderer. This method exists "organically" within the engine if you will. As I said in previous devlogs, it was just a way for me to abstract any platform-specific functionalities while still keeping the codebase fairly clean. Moreover, if I were to, say, get annoyed at the current state of the engine, I can just use these _core_ features to make another engine. Not entirely throwing out the code, basically, but just the _engine_ part.

As for the simple way, there exists the "application hook". 

There is a module in _Nikola_ (that's the name of the engine. Didn't I tell you that before?) called `engine.cpp`. How convenient. This "engine" module will handle everything that I just mentioned above for you. On top of that, it will handle the initialization as well as update, render, and shut down all engine systems and, most importantly, destroy a user-level `nikola::App` structure. What's that? Well, let me explain. 

In the source code, this `nikola::App` is just an opaque struct. Ah yes, my favorite. Accompanying this `struct` are a few function callbacks that accept a `nikola::App` parameter, naturally. It is up to the user to _implement_ this `nikola::App` structure and the callbacks as well. Honestly, this is a very "C-style" way of handling such a feature. In the future, I might make a devlog specifically going over why I love this style of programming so much. But, for the time being, just know that I _adore_ this old C-style procedural "paradigm", if you can call it that. I essentially have to read the entire Quake codebase before sleep every day. But, idolization aside, let's take a deeper look. 

When you start a new project using the _Nikola_ engine, the process looks a bit like this: 
    1. Create a `main.cpp` file.
    2. Call the `nikola::engine_init` function, passing in a `nikola::AppDesc` that, well, _describes_ the application to be created (we'll take a closer look in a second). 
    3. Follow that by calling the `nikola::engine_update` function, which, as the name implies, will run a loop until some condition is met, after which it will exit. 
    4. After the previous function exits, the `nikola::engine_shutdown` function is called, cleaning up any used memory, closing any opened files, and the like. 
    5. After following all these steps, you should have a million-dollar game at hand. Now give me my cut. 

The implementation of these functions is not all that interesting. They basically go over the systems of the engine, initializing them and passing any user-level flags to any systems that require it. The engine receives these flags and specific customizations from the `nikola::AppDesc` structure.

```c++
struct AppDesc {
  AppInitFn init_fn         = nullptr;
  AppShutdownFn shutdown_fn = nullptr;
  AppUpdateFn update_fn     = nullptr;
  AppRenderPassFn render_fn = nullptr;

  // Pretty obvious what these are. 
  //
  // The `window_flags` are specific functionalities the window system can have. I have talked 
  // about this feature in great detail in a previous devlog.
  String window_title;
  i32 window_width, window_height;
  i32 window_flags;

  // This is an "under maintenance" side of the engine. The engine 
  // does accept command line arguments and even puts them in a convenient array of strings. 
  // But aside from that, we don't do anything special with them. For now at least.
  char** args_values = nullptr; 
  i32 args_count     = 0;
};
```

The callbacks shown above can either have a valid value or a `nullptr`. Aside from the `init_fn` callback, any of these functions can essentially be "ignored", in a sense. The engine is smart enough not to call any function with a `nullptr` value.

If you have read any of my previous devlogs, you will recognize this pattern. A _description_ structure that handles the behavior of an opaque `struct`. I use it all over the place [for handling graphics, for example](https://frodoalaska.github.io/2025-03-09-making-an-opengl-wrapper/) However, for this structure specifically, the _description_ does not have to be kept alive at all. It is just a way to keep all of the initial behavior of an application in one place. It worked pretty well for me in the past and it still works for me. 

Now, as for the function callbacks themselves, they are fairly simple.

```c++
using AppInitFn       = App*(*)(const Args& args, Window* window);
using AppShutdownFn   = void(*)(App* app);
using AppUpdateFn     = void(*)(App* app, const f64 delta_time);
using AppRenderPassFn = void(*)(App* app);
```

These functions need to be implemented by the user. It is kind of like having inheritance without having inheritance. As I said, aside from the `AppInitFn`, any of these functions can be "nulled". Meaning, they don't all have to be used. This way, you can have an application that does not render, for example. Pretty unlikely, seeing how this is a _game_ engine. But, for things like CLI applications, this approach _could_ be used.

As you have noticed, two of these functions look very similar yet they are intended for very different purposes. The `AppShutdownFn` and `AppRenderPassFn` are, essentially, the same exact function signature. However, they are not supposed to be like this for long. Once I set up an actual renderer, the rendering function callback will look _very_ different. For now, though, this is fine.

Now let's briefly take a look at an example of how to implement the `nikola::App` structure. 

```c++
struct nikola::App {
  nikola::Window* window;
  nikola::Camera camera;
  nikola::u16 res_group_id;

  nikola::ResourceID player_model_id, skybox_id;
};
```

This might look weird if you have never seen this syntax before, but it works _so_ perfectly. This `nikola::App` structure is basically your own playground. You can have whatever you might like in there. Now, in the initialization step, you are _required_ to allocate this structure. Since, as you know, opaque `struct`s are, in the compiler's eyes at least, empty. That is one potential problem I might fix in the future. But, for me at least, this works fairly well. Since this specific structure is passed to the function callbacks, you can use any members of this `struct` inside those callbacks. Once again, it mimics having `virtual` functions tied with inharetince without actually using any of these features. 

Now that we made an application with the engine, how do we actually _build_ it? I mean, we do have to find a way to _make_ it, right? I really thought that joke was funnier than it actually was. Sorry.

## Make All The Makes

Nobody really likes CMake. I don't either. I hate CMake more than I hate C++. Well, actually, I don't really _hate_ CMake. In my opinion, CMake is the best answer we could get from the awful situation that is the C++ "build system". But rather, I hate the C++ _environment_ as a whole. Honestly, anytime I try to set up a C++ project I get a brain aneurysm that sets me back to the middle ages. The tooling around C++ is virtually non-existent. And I'm not talking about package managers here. I don't think the problem with C++'s environment is the lack of a package manager. Instead, I feel at least, the problem is with the toxic wasteland that is the C++ build systems. There are _several_ ways of creating, building, and managing a C++ project. _None_ of them are fun or convenient to use. I can see that these build systems _try_ to make good of a terrible situation, but all they achieve is just a complex and annoying result that everyone has to deal with. You don't believe me? Well..

First of all, you have the command line. You can, theoretically, use MSVC on Windows and GCC (I prefer Clang, though) on Linux directly on the command line. That's what these build systems do behind the scenes, anyway. You _can_ just create build scripts that adhere to each environment. And while that is possible, it is tedious at best, annoying at most, and, I would even say, the cause of my balding (in the future I presume). Some folks did it somewhat successfully but I've been a contractor on some projects that had such a system and it made me want to vomit. It is very hard to maintain besides that. If, say, a compiler is to be updated or the user does not have the specific version of the compiler you are using, then you will have to handle that, somehow. Perhaps the user does not have the compiler _at all_. It's just a headache, honestly.

A step above that, there exists the build systems. There are several. Make, CMake, PreMake, Autotools, Ninja, and many more. I don't care about 90% of them, honestly. They all, at the end of the day, have the same problems. Once again, it's not their fault. It is the language's fault in the first place. Nonetheless, it is what it is. You either work with these tedious build systems or go program with Javascript or something.

I won't go into detail about all of the build systems that exist currently and which one is better or which one is worse (this [article already does](https://julienjorge.medium.com/an-overview-of-build-systems-mostly-for-c-projects-ac9931494444)). It is an interesting topic but it does not concern us for the time being. Seeing how I already have lots of experience using CMake, I decided to just use it. Every dependency I use for this game engine has a `CMakeLists.txt` file somewhere. Meaning, it will be _marginally_ easier to add them to the engine. 

I won't really bore you with all the details of my `CMakeLists.txt` file. There's really nothing interesting about it. It just lists all the source files, manages the build artifacts, and so on. Nothing you have not seen before. However, for me, using CMake alone is not just enough. Why? 

You see, I don't use a "conventional" IDE. Even when I'm on Windows, I don't really use Visual Studio. It is _very_ slow and it boggs me down a ton. When it comes to debugging, however, Visual Studio is kind of the only "free" C++ debugger on Windows. One of the better ones, at least. But, when it comes to editing the text, I use NeoVim. Yes. NeoVim on Windows. Cursed, I know. That means anything I do is almost always on the command line. Anytime I want to build the project, run the examples, or even create new files, I do it on the command line. If I were using something like Visual Studio, this would not necessarily be a problem (although it is still something to handle). I could have just used `Ctrl + B` to build the project and `Ctrl + F5` to run the project. But, that is not the case. 

Since I spend most of my time on the command line, _commands_, naturally, are my best friends (are you my best friend? No). And for that, I am in luck, for CMake is full of commands. I even memorized them by heart. Not out of love but out of frustration. Again, I won't bore you with the specific commands I use or how I use them. All you need to know is that there are two important commands: `cmake ..`, which generates the project in the first place, and `cmake --build .`, which, obviously, builds the project. Here comes the problem, however. 

Usually, when I want to test something, I, well, code it, compile the engine, change the application I'm using for testing if needed, compile the application, and then run the application. Sure, I could just use `cmake --build .` every time I have to go through that. But that is fairly tedious. Besides that, it is not versatile whatsoever. What if I wanted to build a different application? What if I wanted to add a specific build flag? What if I wanted to compile for the release build? What if I wanted to _just_ compile the application and not the engine? There are so many little bits and pieces that are added as the project grows. So, there must be a better way, right?

My goal with this build pipeline is to make it as versatile and simple as possible. Not just for me during development, but, perhaps, for anyone who wishes to use the engine. Having had some _real_ experience with shell scripts before, I decided to take this route. I would essentially have several _build scripts_ for different functionalities. Yet, there would still be one "master" build script, if you will, that will handle the compilation and execution part that I am so concerned with. These build scripts will handle every detail associated with the build and execution process without standing in the way of development. In addition, they shouldn't be too complex or annoying to use. 

Once again, the details are not fun at all. I made scripts for both Windows and Linux. I used Powershell for Windows and Shell script for Linux. I could have used Bash for Windows, but the scripts got a bit too complicated and Bash is quite simple at heart. And besides that, I knew Powershell pretty well so why not?

So the _main_ build script is called `build-nikola`. The script takes in _several_ flags, actually. 

```bash
   --clean          = Have a new fresh build              
   --debug          = Build for the debug configuration   
   --rel            = Build for the release configuration 
   --jobs [threads] = Threads to use when building        
   --run-testbed    = Run the testbed examples            
   --reload-res     = Reload the resources cache          
   --help           = Display this help message           
```

The `testbed` is just a collection of "sandbox" examples that I use for testing functionality. It has nothing to do with unit tests. Absolutely not. In the future, once I have more tests, I plan to make the `--run-testbed` flag take in a specific test to run. But, for now, there is only one test so there is no need for it. 

As you can see, the build script looks like a whole build system. That was my goal at least. I can, fairly comfortably, run a command like `./build-nikola.sh --debug --jobs 8 --run-testbed` and then watch as the whole project compiles and runs. I absolutely _love_ it. It has been a crucial time saver for me. Without it, I think, the lifespan of this project would have doubled. It just goes to show how important tooling is for a programming language. I did not need to set this up if I were, say, using a language like Odin or Jai. But that is a discussion for another day. For now, C++ all the way. Plus, it was actually fun to set up. 

Once again, if anything intrigues you and you want to see the full implementation of everything, you can check out the whole project [here](https://github.com/FrodoAlaska/Nikola). Specifically, you can check out the `scripts` directory for the build scripts I mentioned. 

Thanks for reading and have a good day/night.
