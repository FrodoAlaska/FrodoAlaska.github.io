---
layout: post
title: Making An OpenGL Wrapper
subtitle: Nikola Engine Devlog 2
comments: false
tags: [gamedev, game-development, opengl, game-engine, nikola-engine, nikola, glfw, devlog, discussion]
---

## To OpenGL Or Not To OpenGL

Whenever I am confronted with the question of which graphics API to use, I'm almost always hesitant to use OpenGL.

OpenGL is an API that was used extensively for drawing pixels to a screen. Or, what we experts like to call: _rendering_. Specifically, it worked extremely well with consoles and even proved useful for Linux machines. However, times have changed. There is more demand for high-resolution and realistic graphics with ray tracing, all of that with little impact on performance. Seeing how OpenGL was created in _1992_ (yeah, you're old), it lacks the features needed to implement such demands. And while it has changed plenty in the last 30 years or so, making it somewhat possible (I use the word "possible" very lightly here) to achieve those features, it still lacks a ton when it comes to performance. Trying to make multi-threading work with OpenGL, for example, is quite difficult. Ray-tracing is always a pain with OpenGL. I would even go as far as to say it is impossible. And it makes sense. It was not designed for that. That is why newer graphics APIs like DirectX12, Vulkan, and Metal were created. 

Moreover, OpenGL's API design is, for lack of a better term, awfully _shit_. The newer versions (specifically version 4.5) proved to have a better design. Yet, still, OpenGL hides a lot of implementation details from you. Crucial details that would otherwise be important to possess in order to apply better optimizations by the user. OpenGL does apply a lot of optimizations, of course. But, for the modern systems at least, they are not enough. An API like Vulkan (which was made by the same people who made OpenGL, by the way) does not hide all these details for you. Instead, it opts to give you a _very_ low-level API that, while complex and difficult to learn, gives you the ability to apply any kind of optimizations that _your_ application may need. 

OpenGL is definitely a product of its time. It is an old API that has a terrible design and even worse debug capabilities. So why, then? _Why_ am I using it? Well, simplicity, my friend. 

Despite OpenGL's flaws, it is still, without a doubt, the simplest graphics API out there. Will it be fast? No. Is it modern? By no means. But it gets the job done without much headaches or heartaches. You can still send data to the GPU to draw things like quads (rectangles), circles, triangles, meshes, 3D models, and even create interesting shader effects.


<img src="https://frodoalaska.github.io/assets/img/screenshots/engine-thingy-2.gif" alt="Nikola" class="project-image">


The reason I made this game engine in the first place was to make _my_ kind of game. The games I always wanted to make. And, at least for my purposes, OpenGL is more than enough to get the ball rolling. Besides that, I'm well-versed in the OpenGL world. That's to say, it won't take me much time to set it up or discover any faulty behavior if (when) it arises. 

Now, having said that, I decided to give myself a little advantage (I guess it can be called an "advantage") by using the newer OpenGL API introduced in version 4.5. The _Direct State Acess_ API (DSA, for short) is, by all standards, a _way_ better API than the older versions. You do not have to "bind" a resource and then use it. You can just _create_ a resource, which returns an ID that can be passed to any function associated with the resource. The newer OpenGL versions also introduce better debug capabilities, using callbacks you provide to OpenGL that get called whenever a problem arises. That, in my opinion, is way better than checking for an error code every time you call a function. I won't get into the details of how different the DSA API is from the older versions, but you can check out [this repository](https://github.com/fendevel/Guide-to-Modern-OpenGL-Functions) if you are interested. 

Don't get me wrong, Vulkan is definitely an API I would love to learn in the future. It is complex, perhaps, yes. But it is marginally superior to OpenGL. It very much interests me with its capabilities. But, for the time being, at least, OpenGL will suffice. 

## Wrapped Up Like A Burrito 

When I was still thinking about the design of this game engine, I wanted the *Core* part to be something more akin to [SDL2](https://www.libsdl.org/) or [sokol](https://github.com/floooh/sokol) coupled with [GLFW](https://www.glfw.org/). I wanted the *Core* section to manage input and window creation while having an abstracted API that, no matter the platform, will render whatever data you give it to the screen. I did not want to create a renderer, per se. The concept of a renderer is much more "involved" than what I had in mind. A renderer is way more than an interface that sends data to the GPU. I just wanted a thin wrapper around graphics APIs without much setup or headache. In that sense, I can create all sorts of renderers without having to worry about retrieving OpenGL libraries or writing hundreds of lines of code to create a texture. That is why, in my infinite wisdom, I decided to have a _graphics_ wrapper around OpenGL _and_ DirectX11. Mind you, at the time, I did not know _anything_ about DirectX11. I might have seen some snippets of code here and there but nothing that would make me an expert. I thought I could learn the magnificent creation of Microsoft which is DirectX11 in a week or so. Write the OpenGL backend and then quickly whip up a similar DirectX11 backend. That way, I would have the best of both worlds. If I were to run the game engine on any [Windows](https://www.google.com/search?q=a+window&oq=a+window&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIKCAEQABixAxiABDINCAIQABiRAhiABBiKBTINCAMQABiDARixAxiABDIGCAQQRRg9MgYIBRBFGD0yBggGEEUYPDIGCAcQLhhA0gEIMjEyNmowajGoAgCwAgA&sourceid=chrome&ie=UTF-8) machines, it would render using DirectX11. Otherwise, OpenGL would be used. Genius! What a great idea _no one_ ever thought of it before. I was expecting my Nobel prize at any moment now... 

Safe to say that I failed. Miserably at that. 

The problem with DirectX11 is that--and I'm sorry for the OpenGL folks for saying this--it is, fundamentally, a _better_ API than OpenGL's. Now, hear me out. I do not like having to write `D3D11_DEPTH_STENCIL_DESC`. Can you see how hideous this looks? And I have my ring finger on the shift key the whole time! Why don't I just turn on caps, write it down, and then turn off caps again? I don't know. But maybe you should keep reading without making a snarky comment. Either way, despite DirectX's awful notation style, I still think it is a better _graphics_ API than OpenGL. It is straightforward and does not hide as much implementation details as OpenGL does. Granted it still does have _some_ details hidden. But, on the other hand, it also has basic things like changing the texture format of the swap chain fairly easily. Something that cannot be done easily with OpenGL. Render targets are, similarly, much easier to manage and create than in OpenGL. And the list goes on and on. So why didn't I go with DirectX11 then and not OpenGL? I, well, frankly, don't know. 

DirectX11 is a _Windows-only_ API. That would have been a great choice if I were planning to _only_ support Windows (and perhaps the Xbox). And, to be fair, Windows has the biggest piece of the pie when it comes to PC games. I'm not surprised either. So, essentially, I wouldn't be throwing anyone out of the... window (huh? No?) if I were to only support Windows. But, alas, Windows is not the only operating system I'm planning to support. In the first [devlog](https://frodoalaska.github.io/2025-02-24-new-year-new-game-engine/) I made, I said that I was planning to support both Windows and Linux. For the simple fact that my main development environment is on Linux and, like I said, Windows has the biggest market share when it comes to PC gamers. Hence, Windows and Linux. Besides that, the cost of cross-platform support is inexpensive nowadays, thanks to libraries like SDL and GLFW, making our lives much easier. That is why I decided in the first place to support both OpenGL and DirectX11 since it would have been the [best of both worlds](https://www.youtube.com/watch?v=uVjRe8QXFHY)(sorry about that). 

However, given the amount of time and effort such a task needed, I, well, threw it out of the window (still no? Wow). There are still some elements of DirectX11 support in my engine here and there. I even kept the API quite ambiguous since I do want to, perhaps in the future, add DirectX11 support. But, for the time being, OpenGL is the only graphics API this engine supports. 

Now that I have told you this very sad story of mine and mentioned a few bad jokes, we can *direct*ly (Wow, you're a hard sell) rush into the API design. 


## A Description And A Pointer

Having served some time in the DirectX prison, I was really influenced by the way its API is designed. You see, in DirectX11, the way resources are created is through a description and pointer method. This means, that every time you want to, say, create a texture, you would call `CreateTexture2D`, and an accompanying `D3D11_TEXTURE2D_DESC` would be supplied to the function. That "description" carries various information about the texture itself. The width and height of the texture, for example, the mipmap levels, the format of the pixels, and so on. Bar a few annoying Microsoft-isims here and there, my API is basically the same. 

The reason I wanted to pursue this design is because the _core_ resources of my graphics API are supposed to be hidden. The _implementation_ of these resources that is. So you cannot simply fetch a `GfxTexture` and edit the `width` member to change its width. Instead, you would need to keep a `GfxTextureDesc` laying around somewhere (or retrieve it from the texture itself), change the width member of the description, and then call `gfx_texture_update`, passing the texture you'd like to update and the `GfxTextureDesc` as well. A roundabout way of doing it, for sure. But it is only done this way so as to ensure the security and the independence of any graphics API's interpretation of a "texture". For example, in OpenGL, a "texture" is just an `unsigned int` id. While in DirectX, a texture is a type, `D3D11Texture2D`, that needs to be allocated and deallocated. Therefore, keeping the implementation a "secret" effectively ensures cross-platform support.

Any type of resource can be allocated by passing in its own equivalent `Gfx*Desc` type and getting back an opaque pointer of the type. For example, to create a buffer of data, you can do:

```c++
GfxBufferDesc buff_desc {
    .data  = vertices, 
    .size  = sizeof(vertices), 
    .type  = GFX_BUFFER_VERTEX, 
    .usage = GFX_BUFFER_USAGE_STATIC_DRAW,
};

GfxBuffer* buff = gfx_buffer_create(gfx_ctx, buff_desc);
```

Of course, if you wish to be a good boy programmer, you need to _de-allocate_ the buffer so as to not leak memory all over the place. 

```c++
gfx_buffer_destroy(buff);
```

Besides buffers and textures, there are also other types as well. `GfxShader`, for instance, `GfxFramebuffer`, `GfxCubemap`, and so on. 

Now in order to use any of these types, a `GfxContext` is _required_ to be initialized at the start of the application. This is the mastermind behind the whole section. Now, back when I still had DirectX11 as an option, `GfxContext` used to have way more duties. In DirectX11, there are the `D3D11Device` and `D3D11DeviceContext`. Both are very crucial for carrying _any_ task in that library. Hence, `GfxContext` was way more important. However, in OpenGL, `GfxContext` does not really do that much. Essentially, its only use case is to check if OpenGL is initialized in the first place. Otherwise, it is somewhat useless. Not entirely useless, of course. It still carries out important functionalities like clearing the screen and turning on or off a certain rendering state.


```c++
/// @NOTE: That `nullptr` is actually supposed to be a pointer to a `GfxFramebuffer`. 
///
/// In OpenGL, "framebuffers" are essentially the render targets. You would use them to apply post-processing effects, for example. 
/// The OpenGL context itself has a default framebuffer that it uses. We as the developers cannot touch that default framebuffer in any way (not to my knowledge at least). 
/// Hence, if you pass a `nullptr` to this function, the default OpenGL framebuffer will be used. 
/// 
/// This will be explained more in-depth in later devlogs when I talk about the renderer. 
gfx_context_clear(gfx_ctx, nullptr);

gfx_context_set_state(gfx_ctx, GFX_STATE_DEPTH, true);
```

But, again, if I were to add any other graphics API like DirectX11 or Vulkan, `GfxContext` would be _crucial_ for creating resources and drawing them, for example. Therefore, it is still alive for now.

Now, having a buffer or a texture created is not enough. You need to somehow draw these resources to the screen. So, if I were a simpler man, I would have just let the `GfxContext` have the ability to draw any resource it is given. In retrospect that would have been way easier. But (since I like to torture myself) I decided against that. There are plenty of operations that need to be carried out in order to draw even a single pixel to the screen. In OpenGL, for instance, you need to have a vertex array which requires a vertex buffer and perhaps an index buffer. Besides that, you need a shader that perhaps needs some uniforms uploaded. Maybe if you want a texture to render you need to bind said texture. So, in my defense, having all of these operations carried out in a single function would have been lackluster at best.  And so, here comes the `GfxPipeline`.

## Go Through This Pipeline, Please

A "pipeline" is really just the preparation of a draw call. It has all the necessary elements in order to draw something to the screen. And, like many other types in this graphics section, the pipeline has a description and a pointer. 

```c++
GfxPipelineDesc pipe_desc = {
    .vertex_buffer  = vert_buff, 
    .vertices_count = 36, 

    .index_buffer  = index_buff, 
    .indices_count = 24,

    .shader = effects_shader, 
    
    .depth_mask = false,

    ...
};

GfxPipeline* pipe = gfx_pipeline_create(gfx_ctx, pipe_desc);
```
There is actually a whole lot more to the `GfxPipelineDesc` than the members shown above. But, in reality, not _every_ member of the structure will be used. In fact, not all the members are _supposed_ to be used. This pipeline type is supposed to be used in a "general-purpose" way. And even though that word makes me want to throw up, it strongly fits the purpose of the API. I could have very much made a `GfxCubemapPipeline` that would only render cubemaps. But then I would have _restricted_ the end user (usually me) to a certain path that they might not like. Or, perhaps, they might like it _for now_ but then decide that they might want to try it a different way. Once again, these design decisions might not--and are not--the best out there, but they do work for me now. Everything is subject to change, after all.    

Now, before anything is to be drawn, the pipeline will first have to be "conditioned" by the graphics context to set any relevant state. This step is present in order to "ready up" the context for _this_ pipeline to be dispatched. In particular, this is done so as to a) ensure that all the "front-end" types are converted into the relevant "back-end" types to remove any extra overhead when we do eventually get to drawing and b) to set any flags or certain state-specific configuration the pipeline might have (like changing the depth mask or the stencil value, for example). This _has_ to be done prior to dispatching the pipeline. Otherwise, you will just be left with the previous pipeline's state. Which, perhaps, is what you want to do. 

```c++
gfx_context_apply_pipeline(gfx_ctx, pipeline, pipe_desc);
gfx_pipeline_draw_index(pipeline);
```

This is typically how a "render call" is carried out in the engine. For example, a `Mesh` might have a `GfxPipeline` and an accompanying `GfxPipelineDesc` that both will be created when the mesh is created and then used to draw the contents of the mesh. This design is _extremely_ influenced by the sokol library I discussed earlier. And also [this talk](https://www.youtube.com/watch?v=qx1c190aGhs&t=1093s) gave me some ideas as well.

## The Last Bite 

Listen, this API is not the best. It is not perfect by any means. It is missing a lot of features. The biggest is instancing. In fact, I was just done implementing framebuffers yesterday! It is by no means "complete". And, to be honest, it may never be "complete". I still have big plans for it moving forward. I might get around to finally implementing the DirectX11 backend. Who knows at this point? However, what I do know is that the design and the layout of the API are going to stay roughly the same. Some function names might change here or there. Some new `struct`s may be added, too. But, despite any _new_ additions, the library will have the same _spirit_. 

Once again, if anything intrigues you and you want to see the full implementation of everything, you can check out the whole project [here](https://github.com/FrodoAlaska/Nikola). I'm warning you, though, it might be the prettiest code you've ever seen. 

Thanks for reading and have a good day/night.
