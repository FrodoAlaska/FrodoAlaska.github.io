---
layout: post
title: Starting Up The Engine
subtitle: Nikola Engine Devlog 1
comments: true
tags: [gamedev, game-development, opengl, game-engine, nikola-engine, nikola, glfw, devlog, discussion]
---

## Rev The Engine 

Game engines are _huge_ beasts of complexity and intricacies. It is often hard to know _which_ system to start working on first. Maybe start off with the window system? Input? Logging? Perhaps begin to work on your custom data structures? So, essentially, _where to start?_ This is a question that vexes me every time I start on a new game engine. If you read the last [devlog](https://frodoalaska.github.io/2025-02-24-new-year-new-game-engine/), you'd know that my journey with game engines has not been the most favorable, let's just say. I always truly "fucked it up". Especially at the beginning. And the beginning, I feel, is as important as anything else. 

Mind you, I did not want to go whip up the whiteboard and start to design my system like that. I feel that the method of creating systems--often called "white-boarding"--is somewhat contrived. I do understand it was meant as a way to expect the unexpected, essentially. But, for me, it never sufficed. So, instead, I decided to just _outline_ the systems that I would create and then go from there. I did not want to put too much thought into how a system would be created, used, and coincide with other systems. I just wanted to know what are the purposes of the system, what I'm planning to achieve with it, and go from there. So far, that way of thinking about systems has worked for me. 

In my last devlog, I outlined exactly how the engine's architecture is going to look. Basically, I split the engine into three modules: *Core*, *Engine*, and *UI*. The *Core* part of this project is where I'll start, obviously. It is the core after all. I wanted this part of the engine to be as barebones as possible. I did not want to include any external libraries in the header file at all. In fact, I was pretty successful in achieving that. With the exception of the ol' `#include <cstddef>` for `size_t`, I did not include any third-party libraries at all. Any custom data structures or engine-specific systems like scenes, resources, or renderers will be instead in the sister *Engine* side of the aisle. 

So, what does this *Core* part of the engine include, then? Glad you asked, person who is totally real. Mainly, this *Core* section is separated into six distinct areas: _Logger_, _Window_, _Events_, _Input_, _Clock_, and a section I call _Gfx_ which is just a nice wrapper around OpenGL. I will go in-depth about the first five areas of *Core* in this article, while I'll leave the _Gfx_ area for a later article.

As you will later see, my code is pretty much as C-style as it gets. I will not go in-depth about why I write code this way (maybe in the future I will), but it is the code style I adapted over the years. It is one that I enjoy very much and find fairly intuitive. So you won't see any `class`es, `template<typename T>`, or `boost::lexical_cast<int>(no)`. I rely heavily upon `struct`s and functions. I do not use constructors or destructors. Again, not any particular reason other than I like this style. That does not mean I never use any C++ features, though. I do. Plenty, as well. In the *Engine* section, as you will see, I use a lot of the features in the `STL` library, as well as some function overloads here and there. So don't be surprised when you see a stray `struct` here and there.  

Okay, with that out of the way, let's start with some printing, shall we?

## Log, Assert, And Watch Your Step 

Whenever I am presented with quite an annoying bug, I often `printf` my way into the solution. Having had very little experience with debuggers early on in my programming journey, I printed out many variables and results of functions to overcome bugs. You can say it is an old habit of mine. I never grew out of it. And while I do often use debuggers for the occasional stubborn segmentation fault, my first intuition when I'm presented with a problem is to print everything. And so, with that in mind, my first call to action was to create a helpful logger.

Well, I wouldn't call it much of a "logger", per se. It is more like a collection of useful macros that print to the console using some pretty colors. I did have in mind to use the [spdlog](https://github.com/gabime/spdlog.git) library since I do know that it is quite fast and has a lot of useful functionalities. But if you have read my previous devlog, you would know I do _not_ like to use a lot of dependencies. I felt like logging was something that I could handle on my own without adding another dependency. And so I did. With a lot of help, though. 

This "logging system" I adapted to this engine was influenced a whole _ton_ by the [kohi](https://github.com/travisvroman/kohi) engine. If you don't know, the Kohi Engine is quite a robust game engine built in C. The guy who created it also logs his progress on his [YouTube](https://www.youtube.com/@TravisVroman) channel. Check it out. The series is very fun to watch and very informative. 

Either way, in The Kohi Engine, there is a list of macros he uses to log into the console. Now, he has a more sophisticated system for logging. But I, on the other hand, decided to create something simple instead. I _did_ just use a bunch of macros to log into the console. Each had its own cool color associated with it. It is nuanced, as well. If the logger were to, say, log a "fail" message status, for example, it would send a "Quit" event to the event manager which someone would hopefully be listening to. That's where the "nuance" stops, though. 

My reason for going for something so simple initially is quite obvious: I, generally, do not care about logging. Here's the thing, logging has, quite often, saved my ass. Excuse my French. It is truly a very important system that will, theoretically, be very crucial to my development. However, that is only what it is: A useful tool for _development_. Only a tool. When I do eventually ship a game with this engine, logging will be cut out completely. I sincerely did not see the reason to build an entire robust logging system. 

But, controversial takes aside, let's take a look at the logging system. 

The logging system has a few "statuses" that attach to every message logged to the console. 

```c++
enum LogLevel {
  LOG_LEVEL_TRACE,
  LOG_LEVEL_DEBUG, 
  LOG_LEVEL_INFO,
  LOG_LEVEL_WARN,
  LOG_LEVEL_ERROR, 
  LOG_LEVEL_FATAL,
};
```

As you can see with beautiful `enum`, there are _six_ levels to logging. The first two are _only_ present in debug builds. The two after that can be turned on or off as the user pleases. And as for the last two, they are _always_ turned on, no matter the build configuration or the user's pleasure. With each level, there's an associated macro: 

```c++
NIKOLA_LOG_TRACE("TRACE");

NIKOLA_LOG_DEBUG("DEBUG");

NIKOLA_LOG_INFO("INFO");

NIKOLA_LOG_WARN("WARN");

NIKOLA_LOG_ERROR("ERROR");

NIKOLA_LOG_FATAL("FATAL");
```

Each macro is really just a `printf` under the hood. Meaning, using variadic arguments (it's the `...` you usually find with `printf` functions), we can insert variables into each log statement like such: 


```c++
NIKOLA_LOG_DEBUG("FPS = %i", fps);
```

As you can see, it is _very_ much like a `printf` statement, but with slightly more variation. Besides the fact that it associates each log level with a pretty color in the console, the macros call a logging function that does a little bit more work. 

```c++
// The `i8`, by the way, is just a `typedef`ed `char`. 
NIKOLA_API void logger_log(const LogLevel lvl, const i8* msg, ...);
```

The function takes a log level, a message, and the variadic argument I was just talking about. Under the hood, the function will extract the variadic argument using the old C `va_start` and `va_end`, format the string using `vsprintf`, pick an appropriate prefix (`NIKOLA-FATAL` for fatal errors, for example), pick a nice color, and then output the final message to the console. While it _may_ sound complicated, it really isn't. It is only these three functions to extract variadic arguments and then a single `printf` at the end. It is very simple, in fact. 

You might wonder how I changed the colors for the output. Well, my friend, I went complete "Unix-mode" on the matter. I used the good ol' [escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code) to print the colors. The Wikipedia page goes more in-depth about that, but, essentially, the console already has preset colors you can use by using _specific_ sequence of characters like `\033[1;91m`. Go read the Wikipedia page if you are more interested in it. 

I did think that escape codes would not work on Windows. But, to my surprise, they did! Pretty well, too. Although, I would think using the Win32 API to print out to the console would give you more benefits than a simple `printf`. But, nonetheless, it is good enough for me. 

Another technique I use to root out bugs is the _asserts_. Now, there is a pretty nice assert library already in the C standard. The one that exists in `<cassert>`. However, I wanted to have my own control over the `assert`s, so I decided to use my own implementation. Once again, I took a lot of liberty from The Kohi Engine. 

Asserts are a way to completely halt the code from any further execution. It is a useful way to know _exactly_ where and what failed. I usually use asserts to catch the stupid bugs I always end up causing like passing an invalid pointer or going past the current size of an array. If you ever get to see my code (god forbid), you will see that asserts populate my codebase quite a bit. And for good reason, too. Asserts have helped me root out some very stubborn bugs before. 

Now, much like logging, asserts have their own set of macros. Well, not a _set_, but just _one_ macro.


```c++
NIKOLA_ASSERT((window != nullptr), "Wrong window, stupid");
```

The messages are usually more useful than that, but you get the point. As you can see, the first argument is a condition. This condition _must_ be true for the assertion not to trigger. Otherwise, you will have a bad time. The output will look something like this: 


```bash
[NIKOLA ASSERTION FAILED]: Cannot initialize GfxShader with an invalid context 
    [EXPR]: gfx != nullptr 
    [FILE]: gl_backend.cpp 
    [LINE]: 420 
```

Under the hood, the macro calls a function and then another macro. The function,


```c++
NIKOLA_API void logger_log_assert(const i8* expr, const i8* msg, const i8* file, const u32 line_num);
```

will simply propagate into multiple `printf` statements that log the given information. No colors involved or anything. I wanted to make it serious to really show how disappointed the engine is with you. Besides the function, `NIKOLA_ASSERT` will also call `DEBUG_BREAK()`. That macro will be specific to the current operating system running the engine. On Windows, the macro is really just a call to the intrinsic `__debugbreak()`. While on Linux, it will call `__builtin_trap()`. These intrinsics will halt the code completely at a specific point, just as if you were using a debugger and it halts at a breakpoint.

Well, that was a bore. Let's get some fresh air, don't you think?

## Open A Window To Get Some Fresh Air 

As I discussed in my previous devlog, I decided to go with [glfw](https://github.com/glfw/glfw.git) for handling the window creation for this engine. It is a library I used a _ton_ before. And while I do fancy replacing it later, it will suffice for now. 

In my previous engines, I was actively trying to make the window a global variable that would only have _one_ instance throughout the whole application. The reason I went for this approach in the past was for the "ease of use". And while it was pretty simple, I wanted to go for something different in this engine. Basically, I did not want to force the user (me, in this case) to work with a global variable. Even though I do not mind global variables, I still did not want to force myself to use that kind of system _every time_ I would make a game. What if I woke up someday and decided that global variables were bad? What then? Should I change the _whole_ API? No. I did not want this to happen. But that would mean that I would have a `Window` structure out there somewhere with public variables the user can access like the width and height of the window. While these variables can be changed, they need to be changed through an _API call_ to the specific operating system. Not through using `+=` on the variable, for example. Besides that, each operating system has a state that needs to be kept. On Win32, for example, I would have to keep a `HWND` variable somewhere. And in the case of GLFW, I would have to keep a `GLFWwindow` lying around somewhere. And so, here's my solution: 

```c++
struct Window;
```

Beautiful, isn't it? What's that? What is it, you're asking? Well, in the old C lexicon, we call this an "opaque struct". An "opaque struct" is essentially a _promise_ made by you to the compiler that, while the implementation is not yet defined, you will _eventually_, in some `.c` or `.cpp` file somewhere, will implement the type. The compiler can pass this "safely" to any function that needs it without any errors. However, if this type were not implemented somewhere you _will_ have a _linker_ error. Since the linker will be looking for an exact implementation but won't find it. 

This is actually how GLFW is implemented. It has a main `glfw3.h` which will declare an opaque `GLFWwindow` struct the user code can use. The _true_ implementation of this struct lies somewhere in one of the translation units. Either in `win32_window.c` on Windows or `x11_window.c` on Linux. As long as there is only _one_ implementation somewhere of the opaque struct, then you're all good. And, seeing how I want to move away from GLFW eventually, I decided to go with this approach. And, you must admit, it is quite beautiful, right? 

Either way, any window-related function will need a pointer reference of this opaque struct in order to do anything useful. 

```c++
NIKOLA_API const bool window_is_open(const Window* window);
```

But how do you open a window? 

```c++
NIKOLA_API Window* window_open(const i8* title, const i32 width, const i32 height, i32 flags);
```

The function looks pretty self-explanatory, right? The `title` will set the name of the window. The `width` and the `height` will be the total size of the window. The function will return a `Window` pointer if everything goes well. If there's anything wrong with the internal window API, the function will actually assert. So you will know for sure if there's something wrong that happened. But, what about the `flags` parameter? 

Windows are, like engines, quite complicated. I want to open a window, yes, but what kind? A fullscreen window? Maximized? Minimized? Do you want decorations (the borders and widgets around the window)? Do you want it to gain focus initially? Do you want the mouse hidden? So, in order to make the code cleaner and my life easier, I put all of these possibilities into one enum: 

```c++
// There are _a lot_ more flags than these, but I don't want to pollute the article any further.
enum WindowFlags {
  WINDOW_FLAGS_NONE
  WINDOW_FLAGS_RESIZABLE
  WINDOW_FLAGS_FOCUS_ON_CREATE
  WINDOW_FLAGS_FOCUS_ON_SHOW
  WINDOW_FLAGS_MINIMIZE
  WINDOW_FLAGS_MAXMIZE
  ...
};
```

These flags can be `OR`ed together into one `i32` and then passed to the function. In the function `window_open`, it will check which flags are set and act accordingly. 

```c++
i32 win_flags = WINDOW_FLAGS_RESIZABLE | WINDOW_FLAGS_HIDE_CURSOR | WINDOW_FLAGS_GFX_HARDWARE;
```

Now, of course, if the window is opened, it needs to be closed as well. Pretty obvious, I would say.

```c++
NIKOLA_API void window_close(Window* window);
```

While I do hope to improve upon the window system in the future and perhaps move away from GLFW to reduce any dependencies, this system works pretty well for me currently. It has served me thus far without any hiccups or issues. I do not claim that it is the best system out there, but it is one that worked fairly well for me. 

Besides, the windows, I decided to also use GLFW for any timing-related work. GLFW has a function called `glfwGetTime()`, which retrieves the elapsed time since the GLFW library has been initialized. So, in a sense, it also measures the amount of time since the _application_ started. This is a good way to query for time, and we will be using this a lot. 

I made a clock section as well which includes four functions:

```c++
//Every function has a "ni" prefix to avoid any confusion with the C clock library.

NIKOLA_API void niclock_update();

// `f64` is just a `typedef`ed `double`
NIKOLA_API const f64 niclock_get_time(); 

NIKOLA_API const f64 niclock_get_fps();

NIKOLA_API const f64 niclock_get_delta_time();
```

The first function--`niclock_update()`--is to be called internally and therefore has no purpose to be so out in the open. However, since it is working as it is currently, I have no plans to change that. As for the other functions, they are pretty self-explanatory. 

Inside the `nikola_clock.cpp` translation unit, there is a secret data structure called `ClockState`, which keeps track of various variables. For example, `frame_count`, which keeps track of the frames to calculate the FPS later, and so on. The `niclock_get_time()` is just a convenient wrapper around the `glfwGetTime()` I talked about earlier. Since I plan to have separate delta times for various reasons, it would be nice to have that function available when needed. 

Hang on. I just got an event that you pressed a key. Did you?

## On Key Pressed

Cute, isn't it? 

Either way, I want to talk about the last two areas of the *Core* section: the event and input system. As it goes with the rest of this section, these systems are fairly simple in principle. Once again, they are not the _best_ systems ever. Neither are they efficient. Yet, they have served me very well over the past few months. With only a few minor changes, these systems were taken from the last engine I created. 

The event system is quite simple. 

```c++
NIKOLA_API void event_listen(const EventType type, const EventFireFn& func, const void* listener = nullptr);

NIKOLA_API const bool event_dispatch(const Event& event, const void* dispatcher = nullptr);
```

Now the `EventType` is just an enum that will tell the event system what kind of event to listen to. The `EventType` enum includes values like, `EVENT_WINDOW_MOVED`, `EVENT_KEY_PRESSED`, `EVENT_JOYSTICK_CONNECTED`, and so on. Besides that, the `Event` is just a data structure I created to hold _all_ the possible variables that can be passed around from dispatcher to listener. I took inspiration from the SDL and SFML libraries. They have something similar in their even systems. Except, with their implementation, there lies an _event queue_, while mine is more of a _fire-and-forget_ system. Any event that gets dispatched, will make the system go through all of the events with the same `EventType` found in `Event` and call the associated callback `func`, making sure to pass the given `Event` parameter. The event does _not_ get removed from the system, as it would with an event queue. Rather, it stays there until otherwise instructed. Once again, pretty inefficient, but stable and functional. 

So what is the callback? 

```c++
using EventFireFn = bool(*)(const Event&, const void* dispatcher, const void* listener);
```

Every time someone listens to an event, this function prototype gets passed in, bundled up with the event type as well as a scary `const void*` _listener_. On the other hand, whenever an event gets _dispatched_, meaning, fired, the function expects an `Event` with the `.type` member filled and with the appropriate variables set. Besides that, another horrifying `const void*` _dispatcher_ is passed. This is done so we can pass the state around on the event callback. Of course, these scary `const void*` parameters can be left to be invalid with a `nullptr`. That's the default behavior, in fact. But, if otherwise, we can use these pointers and convert them to any type we may know has been passed in. Of course, there has to be some bookkeeping within the listener's callback in order to know for sure if either pointer is valid and if they are the type the listener expects them to be. 


```c++
// Type, function callback, and the listener
event_listen(EVENT_KEY_PRESSED, key_callback, input_state);

Event evnt = {
	.type              = EVENT_KEY_PRESSED, 
	.key_pressed = KEY_Q, // This would be more automated
}; 
// Event and dispatcher
event_dispatch(evnt, window_state);
```


Now, this event system, for the time being, is only used internally by the window system and the input system in order to communicate. I am planning to add a similar event system but to be used by the _user_ code instead. But that is in the future, and I like to live in the present. 

The input system uses the events extensively in order to communicate with the window. You see, in GLFW, there are two ways to query for input. You can use `glfwGetKey` and check if the result is `GLFW_PRESS` or `GLFW_RELEASE`, depending on what you may want. The other way is to use the various callbacks GLFW provides you. The first method, while quite simple, is not efficient at all in this case. If I used it, I would have to go through a loop of some kind and check if there is a key that has been pressed _every frame_. Awfully inefficient. And you know me, I'm all about efficiency. And so, I used the second method. 

In the function, 

```c++
NIKOLA_API void input_init();
```

the input system would _listen_ to various different input events, passing in the internal `InputState` data structure of the `input.cpp` translation unit in the process. The window system, for its part, would _dispatch_ any equivalent input events if any key was pressed or released, if the mouse was moved, if the mouse button was pressed or released, and so on. Using the callbacks provided by GLFW, of course. The same story would go for the joystick input events as well. 

I did all of that in order to disassociate the window from the input system completely. I did not want the end user to query for input while passing the window around everywhere. In theory, I would be asking for input from various places in the codebase. I do not know if they could reach the window at all. So, by doing it this way, I can at least be sure that the window and input system can communicate while keeping them effectively separated. It is like a long-distance relationship that is not quite meant to be.

The input system provides some useful functions as well, of course:

```c++
NIKOLA_API const bool input_key_pressed(const Key key);

NIKOLA_API const bool input_key_released(const Key key);

NIKOLA_API const bool input_button_pressed(const MouseButton button);

NIKOLA_API const bool input_button_released(const MouseButton button);

NIKOLA_API const bool input_gamepad_button_pressed(const JoystickID id, const GamepadButton button);
```

There are plenty more functions, obviously, but these are some examples. The `Key`, `MouseButton`, and `GamepadButton` enums are self-explanatory. They are just enums with all the valid keys/buttons that could be queried for. As for the `JoystickID`, it is also an enum that goes through all of the possible ids for connected joystick controllers. It goes from 0 to 15, for 16 in total. It is quite standard.

Now, internally, the input system keeps a data structure to hold both the previous state and the current state of every button. In the `InputState` data structure, these states are arrays of `bool`s. There are two states in particular: the _previous_ state and the _current_ state. The _current_ state is manipulated inside the event callbacks the `InputState` provides to the event system. Depending on which key was pressed or released, we can turn the switch on or off for _that_ specific key. The values in the `Key` enums are actually more akin to _indices_ that can be used in order to index into the _current_ state array. Very convenient. The same goes for the `MouseButton` and `GamepadButton` enums. 

Now, with functions like `input_key_down`, we can just return the current state of the given `Key`. However, with the function `input_key_pressed`, we cannot simply do that. In the first function, we are effectively asking if the given `Key` is _held down_ or not. While with the second function, we are asking if the given `Key` has been _pressed before_. 

For a more applicable example, think about shooting a gun in games. In that case, I want to query the input system to see if the key was _pressed_ before. Meaning, if, on the previous frame, it was released and on the current frame it was _just_ pressed. That way, I can fire the gun and then start a cool down. In the next frame, the button would be already pressed and, therefore, it is now being _held down_, which does not interest me at the moment. If I were to, instead, check if the shoot key is being _held down_, the behavior would not be controlled. These key presses happen almost instantly. There is not much control over that. 

But, on the other hand, let us say we are trying to move a player to the left or the right. If we were to use something like `input_key_pressed`, we would have to press the key _every time_ we want to move the player even in an inch towards one direction (not the band). In that case, we can use `input_key_down` to make the player consistently move in one direction. 

That is why the `InputState` keeps track of a _previous_ state and a _current_ state. Every frame, internally, we update these states so that the _current_ becomes the _previous_. 

I never said I could explain things easily. 

## Let's Bitshift Left Out Of Here

Honestly, this part of the engine is not really all that fun. Interesting, perhaps. But not very fun. This section always bores me and I always just try to move on away from it. Nothing wrong with some nice logging and input handling, but there are way more fascinating parts of game engine development than these. I did not get into game engine development because I liked making event systems so much. Thankfully, however, the next part is _way_ more fun. Orders of magnitude more fun, actually. In the next article, we're going to talk about OpenGL, graphics APIs, and making a whole wrapper around OpenGL. Stay tuned. 

Thanks for reading and have a good day/night.
