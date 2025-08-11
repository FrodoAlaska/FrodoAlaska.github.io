---
layout: post
title: All About Resources
subtitle: Nikola Engine Devlog 5
comments: true
tags: [gamedev, game-development, resources, game-engine, nikola-engine, nikola, devlog, discussion]
---

## The Structure 

Besides the multitude of different systems a game engine may include, a resource manager is possibly one of the more complex systems to implement. A resource manager can be the backbone of your engine. It certainly cannot be a system that you can ignore initially. It has to be implemented very early on, and it has to be implemented _well_. 

Now there are a thousand different ways to structure a resource manager. There are resource managers that are persistent only throughout one level. Some resource managers are omniscient, and they manage the lifetime of resources throughout the whole runtime, making sure to safely (and very carefully) deallocate the resources that are not needed. Other resource managers are nothing more than simple databases where all the resources a game may need get allocated at initialization. 

I'm not here to vow for one resource manager and make fun of the other. I'm just here to tell you how _I_ created my resource manager in the [Nikola Game Engine](https://github.com/FrodoAlaska/Nikola). It is by no means a "fully-featured" resource manager (it does not support asynchronous loading of resources, for example). Yet, it's still the best resource manager I've made so far. And I've made some terrible ones. And, don't worry, I'll tell you all about my amazing and totally fantastic resource manager. But, before we dive deep into the schematics of my resource manager, let's first talk about what resources are. 

## Resources With An Identity Crisis

In the simplest of terms, resources are "things" that will be consumed by the engine's various systems. An audio file, for example, will be taken and used by the audio system. Fonts and textures will be used by the UI system. 3D models, animations, and shaders are all to be consumed by the renderer. So we can assume that all resources can be categorized as files on the disk that then get loaded by the engine, right? Well, not exactly. 

What about meshes? Meshes are just a combination of different buffers. Buffers in themselves are allocated on the GPU, and they usually consist of vertices and/or indices. Can we save meshes into files? Well, sure. Many engines do exactly that. Generally speaking, though, a mesh on its own is useless unless it's a piece of geometry like a pre-constructed cube or terrain. And the engine can calculate the vertices and indices of these meshes in code. No need to waste space by putting them on disk. Besides that, there are plenty of components that _use_ a resource, but they are not resources themselves. An animation, for example, is pretty useless on its own. It has all the necessary data, yes, but we want to see the animation _playing_. And for that, we need some kind of _animator_ or _animation player_. We cannot constitute the animator as a resource since it really isn't. It's a component that acts on resources, as I've said before. 

And so, we need to first decide what is and isn't a resource. In my case, I've chosen to constitute a resource as any resource that is loaded from disk, and any sub-resource of that resource is, in itself, a resource. That sounds complicated, but it really isn't. Say we want to load a 3D model. That 3D model consists of a list of meshes and materials. A mesh, as discussed before, is just a bunch of buffers. A material has a few textures and then a few variables for lighting, rendering, and whatnot. Therefore, including the 3D model itself, meshes, buffers, materials, and textures are all constituted as resources. Any level above a resource is _not_ a resource, however. So, an animator, even though it does include an animation, is not a resource. To become a resource, you'll have to be loaded from disk, and then we can go from there. 

A pretty big exception to this is a "scene". I do not have a concept of a "scene" in my game engine. I'm not planning to have it either. A "scene" for me is just a collection of entities referencing resources. I do save scenes to custom binary formats and then load them when needed. But I do that in the _game_ not the engine. Perhaps I'll talk about scenes in a future devlog. 

Now, it's important to note here that not all engines jumble all of their resource types into a data structure and call it a resource manager. Instead, there's usually a sophisticated set of systems, each with its own internal array of resources. For example, you might have a texture system that manages textures specifically. The same can be said for materials, animations, shaders, and so on. However, for my own case, I decided to just lump all resources into one data structure and just call it a resource manager. As you have noticed, I am quite the lazy fella. 

Either way, I still need a unique way to identify resources. If I were to push a certain texture into the resource manager, I'd need to get back a unique value identifying said texture so that I can retrieve it later for my own needs. In the past, I took the "easy" approach and had each resource map into a human-readable string. It would look something like this: 

```c++
resources_push_texture("player_texture", "path/to/texture.png");
```

And when I want to retrieve the texture, I can just look it up with the name it was given on initialization.

```c++
GfxTexture* texture = resources_get_texture("player_texture");
```

This system was very simple in principle. It was also easy for the human (mostly me) to read the name of the resource and what it might be used for. However, that became tiresome as soon as I started adding resources like buffers and meshes. A 3D model can potentially have many meshes. And they might or might not have a name. And I don't want to name each individual buffer. Besides that, even though the string ID is hashed for a lookup in a hash map, it's still expensive to pass around strings all the time. And we humans are prone to error. What if I wrote `"plyer_texture"` instead? Then I would have to make sure to track down where I requested that texture and fix it. It is _great_ for humans, but awful for production. 

But fear not, for there is an alternative solution. UUIDs. 

UUIDs stand for **U**niversal **U**nique **I**dentifier. There are, in my engine at least, a randomly generated 64-bit integer value that can easily identify a resource. The problem here, however, is that I, as a human, will probably not be able to remember a 64-bit randomly-generated number. Otherwise, I won't be human. I would be a robot. Yet, this way of referencing resources is _much_ more valuable in production. It's fast, it's efficient, and it's error-prone (compared to the string ID). 

Now, there are a few ways to combine both worlds of resource handles. The way I decided to implement this is that I give unique names only to the resources that are loaded from disk. Well, the resource manager is the system that gives the resource a name. In particular, it takes the file name with the extension stripped off as the unique name. Then, this name gets mapped to a UUID, which can then be used to retrieve the resource internally. It would look something like this: 


```c++
// Load the texture into the resource manager 
ResourceID tex_id = resources_push_texture("path/to/player_texture.png");

// Or, you can do this if you don't have the ResourceID. 
// You just have to make sure it was loaded prior to this call.
ResourceID tex_id = resources_get_id("player_texture");

// Retrieve the texture for extra cool processing
GfxTexture* texture = resources_get_texture(tex_id);
```

Internally, the resource manager will do some extra work to ensure that the given `tex_id` is an actual valid texture resource and not some bogus value. Sometimes (and I've done this a lot), you might be retrieving the wrong resource in the first place. Perhaps the resource was loaded as a cubemap, but you were retrieving it as a shader. These edge cases are unfortunately prevalent with this method, but I still think it's way better than the string IDs.

Now, you might think that the type `ResourceID` is just `unsigned long` under the hood. And, in a way, you are right, but not really. 

You see, if we were to just `typedef unsigned long ResourceID;` and call it a day, we would be required to store all of the resources in a hash map where the key is the UUID and the value is the actual resource. That would be fine, of course, but I'm a person who still prefers arrays over hashmaps 90% of the time. I love hash maps, don't get me wrong, but an array is just way more representative of the underlying hardware. It's way more efficient to store values sequentially than it is to spread them across memory. And in the case of resources, we can potentially have hundreds of textures, shaders, and 3D models being managed at a time. Arrays are just way more favorable in this scenario. But I can't just index into an array of resources with a UUID. The UUID would surely go out of bounds of the array. Besides, I enjoy the benefits of a strongly-typed language like C++. I want to have the compiler tell me if I passed a `ResourceID` to the function or an `unsigned int`. That's very important. And so, instead of a `typedef`, I decided to create a structure out of the `ResourceID`. 


```c++
struct ResourceID {
  ResourceType _type; 
  u16 _id;

  ResourceGroupID group = RESOURCE_GROUP_INVALID;
};
```

Now, I know what you are thinking... ew! But hear me out. 

The first value is just an `enum` that describes the type of the resource. This could be anything such as `RESOURCE_TYPE_TEXTURE`, `RESOURCE_TYPE_FONT`, `RESOURCE_TYPE_CUBEMAP`, and so on. When we retrieve the resource, we check for this value to make sure that we're retrieving the correct type. Moving on, the `_id` is a little misleading. You might think it's just a UUID like we talked about, but it's actually not. It's just an increasing index. This index is used to look up the value from the associated resource array. You can tell I value security and safety by prefixing both members with an `_`. I did not want to use `private` since I'm trying to keep the C-like aesthetic. Either way, every `ResourceID` structure also has its own associated "resource group". We'll get to what those are, but just keep that in mind as we move along.

Now here's the kicker: this structure is probably the most unsafe way to represent a resource. And there are several reasons. Besides the fact that variables like `_type` and `_id` that are supposed to be left untouched by ANY code, I also only represent the resource as an index into an array. What's the problem with that? 

Let's suppose that we decided to add async functionality to the resource manager. When given a directory, the resource manager can recursively iterate through it and push each resource into its internal space. Let's suppose further that we have a job system with 8 jobs as the maximum number of worker threads that could be instantiated at a time. Let's say that all worker threads are retrieving a different resource from the disk. Now, one of the worker threads is done with its job. It was successfully able to retrieve a texture. Yay! The push function for the textures allocates space for the texture, increases the number of textures, and adds the texture to the array. Now, the number of textures in the array is 1. 

But wait a second, now we have two worker threads also done with their work. And look at that, both of them were also retrieving textures. Okay, that's fine. Let's push one texture into the array and... oh wait, hang on. The other texture was pushed at the same time. Actually, both textures were pushed to the textures array at the same time. So, which one is the second element and which one is the third element?

Yeah, having a variable like that with no thread safety in mind is awfully unsafe. Besides that, if the resource were ever to be erased, we would have a problem. What if there is a system in the engine that is interested in a specific texture with a specific index? That texture gets erased, would that system refer to a different texture, or would it assert and crash the game? 

You can tell that this kind of system has a lot of pitfalls. However, the only problem I'm currently concerned about is the thread safety. I do not allow resources to be erased (we'll get to that, hang on), so the second problem is not a concern. And, frankly, I could easily wrap the `_id` member in an atomic variable that will be locked by a certain thread when being increamented. So it's not an "unsolvable" problem. 

And as for the resources themselves, each one is stored as a pointer in its own respective array. For example, the resource manager's structure might look something like this: 

```c++
struct ResourceManager {
    // 
    // ...
    // 
 
    DynamicArray<GfxTexture*> textures;
    DynamicArray<GfxBuffer*> buffers;
    DynamicArray<GfxCubemap*> cubemaps;
    DynamicArray<GfxShader*> shaders;

    DynamicArray<Model*> models;
    DynamicArray<Font*> fonts;

    // 
    // ...
    // 
};
```

Each resource gets allocated by the system that handles it. For example, I talked a lot about the graphics backend in a previous [devlog](https://frodoalaska.github.io/2025-03-09-making-an-opengl-wrapper/). The backend will handle the allocation of these "core" resources. As for audio buffers, the audio system will handle allocating that (I talked about audio [here](https://frodoalaska.github.io/2025-05-13-implementing-audio-with-openal/)). And the rest of the resources like `Model`, `Mesh`, `Font`, and so on are allocated by the renderer--which I am yet to talk about. Whenever we request a certain resource, we can use the `ResourceID` that was given to us upon loading the resource initially, and get back a _valid_ pointer to the resource.

I believe we talked about resources individually enough. Let's talk about them in the whole context of the engine, shall we? And we'll get into some resource manager lore as well.

## A Manager For My Needs 

As I've said before, game engines have their own idea of what a "resource manager" is. Some engines don't even call it a _resource_ manager. Instead, opting to call it an "asset" manager. But that is only a difference in semantics. Just keep that in mind when I'm talking about resources. Assets and resources--in my mind at least--are the same. The real difference here is in the implementation. 

Let's say that, for simplicity's sake, there are two types of resource managers that interest us: The "level-based" resource manager and the "lifetime-based" resource manager. Keep in mind, these names are not "official". They are just the names that made sense to me. So let's just go with it. 

Firstly, the level-based is, as the name implies, a resource manager that only manages the resources of a certain level. Let's suppose that we have three resources that will be used in this level. We need a 3D player model, a sound bite for when the player dies, and another 3D model for the enemy. In a level-based resource manager, the resource manager will query the level for all the resources it requires and load them accordingly. This way, throughout the runtime of the level, these resources are always valid. No matter what. When we are done with this level, the resource manager is prompted to be destroyed. Therefore, it will de-allocate all of the resources currently in its possession. That means that throughout the duration of the level, there are _no_ resources that get de-allocated. They only get booted out of memory when the whole level is destroyed. 

Simple enough, right? You can think of this approach as a database of resources only valid for the runtime of the level. Thus the name. Level-based. However, problems start to arise when one or more levels share the same resource. 

Let's say we have the same resources we had in our previous example: A player model, a sound bite, and an enemy model. It's safe to assume that the player's 3D model needs to be persistent throughout the lifetime of the _game_, not just per-level. We will need the player to be rendered throughout the game, after all. Alright, that's fine. We can create a special level resource manager that will act as a cache for the rest of the game. Think of it as a level that will last the whole game. This is great for adding shaders that will be used throughout the game, a "master" font, and, of course, any resources that will be used throughout the game, such as the player's 3D model. 

Problem solved, right? Well, not exactly. The player's 3D model was a special case. Let's say that we have a game that consists of 10 levels. Each level has some kind of enemy attacking the player. Suppose that we have an enemy that will only be present in level 1, level 2, level 4, and level 9. The enemy will not be present in 6 of the 10 levels. So the enemy is only present in 40% of the game. We can put him in the cache, sure. But it is very wasteful to leave that enemy in the memory and use it only 40% percent of the time. Maybe we won't even use him at all if the player never plays a level that has that enemy in it. Alternatively, we can just load that enemy every time a level that needs him is loaded. So we load him when we are at levels 1, 2, 4, and 9. Of course, we will be eating the runtime costs for loading the same resource every time. 

The only "solution" to this problem (that I've found at least) is to simply bite the bullet and accept the runtime cost when loading the enemy in every level that needs him. It is not a huge cost anyway, especially if it's only one or two resources. On top of that, if the resource is somewhat simple to load or you implement a multi-threaded resource manager, then the cost won't be that "massive". So, like everything in life, this method has its pros and cons. It's certainly an "old-school" method. Games back then were always level-based in one way or another. There was no reason to load a texture that will be used in level 9 when the player is currently on level 5.  

As for the second kind of resource managers, it is a much more complicated beast, but it is much more favorable, especially if you're creating a massive world with thousands of resources. 

The "lifetime-based" resource manager is a global resource manager that is persistent throughout the lifetime of the game. It is omniscient, as I said before. It sees and knows everything. This resource manager will not load a resource into memory right away. Not until said resource is needed from multiple refrences. Now, to be honest, I'm not all that versed in the lifetime-based resource manager, but I'll try my best to explain it. 

Essentially, every resource acts like a reference-counted pointer. Think of it like an `std::shared_ptr`. A resource will increment its reference count when it is, well, referenced by any other resource or system in the engine. Usually, there is a threshold that the resource has to reach in order to be brought into memory. Let's suppose that the threshold is set to 5. The resource will need to satisfy the `resource.reference_count >= 5` condition in order to be brought into memory. Now this works in the opposite direction as well. If the reference count of the resource reaches 0, then said resource will have to be de-allocated from memory. But, and this is important, we _have_ to make sure that no other resource in the game is expecting this resource to be "alive". Otherwise, a resource in the game will be referencing another resource that has left the game. It's very crucial to implement the resource reference counter as an atomic variable, since systems on one or more threads can increment the reference counter. 

As you can tell, this is a very complex system to implement. Perhaps not in the direct implementation, but it's certainly difficult to get right and reap its benefits. As I said, this system is _perfect_ for open world games where each chunk of the world is loaded dynamically as the player traverses the world. We cannot possibly keep all the resources of Elden Ring, for example, in memory at the same time. Not even modern systems have enough memory to handle such a load. Besides that, Elden Ring doesn't really have a concept of a "level". It's just a big, huge world with a bunch of resources. That's why managing the lifetimes of each resource is crucial in such games. 

However, in my case, such a system is _overkill_. I'm not planning to make an open-world RPG with Dark Souls mechanics. If I do, slap me in the face to bring me back to reality. It's a difficult task to create such a game on my own or even with a small team. Therefore, I've concluded that the best system for my games is a level-based resource manager. Okay, so let's look at some C++, then. 

The best way to think about these level-based resource managers is to visualize them as a group. A group of resources that gets initialized and then deinitialized when needed. It has a beginning and an end, essentially. However, we still need an omniscient resource manager to manage these groups. Then, these groups manage their own resources. Complicated, I know. So let's visualize it with some code. 

```c++

struct ResourceGroup {
    ResourceGroupID id; 
    String name;

    // All of the resources here...
};

struct ResourceManager {
    HashMap<ResourceGroupID, ResourceGroup> groups;
};

```

Every resource group is given a name for easier management. But, in the `ResourceGroup` structure, is where the real action happens. We collect all of our resources in that structure. If you think back to how the `ResourceID` was structured, you'd see that every `ResourceID` structure had an associated `ResourceGroupID`. This value gets assigned when the resources get loaded. In the engine, I have the `RESOURCE_GROUP_INVALID` constant that, well, defines a value for an invalid group. When we decide to retrieve a resource, we also make sure the given `ResourceID` does not have an invalid resource group. Another edge case we have to test for, sadly. 

Now, the "master" resource manager itself has an initialization and de-initialization step. At initialization, the master resource manager will create a "cache" resource group with an associated ID that can be retrieved with the preset value in the `nikola_resources.h` header file, `RESOURCE_CACHE_ID`. Listen, I never said this resource manager was perfect. 

Anyway, the resource manager includes functions such as: 


```c++

/// Initialize the global resource manager as well as the global cache.
NIKOLA_API void resource_manager_init();

/// Free/reclaim any memory consumed by the global resource manager.
NIKOLA_API void resource_manager_shutdown();

/// Create and return a new resource group (a.k.a `unsigned short`) with `name` and `parent_dir`. 
///
/// @NOTE: Any `_push` function that takes a `path` will be prefixed with the given `parent_dir`.
NIKOLA_API ResourceGroupID resources_create_group(const String& name, const FilePath& parent_dir);

```

The functions are very self-explanatory, but I've left the comments just in case. As I said, the `resource_manager_init` function just creates a cache resource group. Currently, that function is quite sparse. However, later on when we add custom allocators, I'll add some preset maximum values for each of the various resource types, so that we can at least have a better memory coherency when loading resources. As for the `resource_manager_shutdown` function. It just destroys the cache resource group and any other resource groups that are still alive. Naturally, this function gets called at the very end of a game.

And as for the `resources_create_group`, it will generate a new resource group with a valid ID and return it to you. The given `name` is just for debug purposes currently. You can leave it empty for all I care. Just good luck with debugging your problems. As for the `parent_dir`, it's a path that gets prepended to any subsequent path given to this resource group. So if we have the parent directory as "level0_res", and we decide to push a texture at "textures/player_texture.png", the _full_ path will be "level0_res/textures/player_texture.png"

This is where the global resource manager's job ends, though. It does handle a lot of functionality internally, of course, but to the user, the resource manager is not needed anymore. What _is_ needed, however, are the resource groups. 

As I said before, resource groups are represented as the `ResourceGroupID` type. Under the hood, this type is just a classic `typedef unsigned short ResourceGroupID`. I do not like the way it is currently declared. Again, having the compiler scream at me if I passed the wrong type is pretty useful to protect me against my own stupidity. However, for now, we'll just keep using it as an alias. 

Now, besides creating a group, we can also both clear it and destroy it. The difference here is that clearing the group will not de-allocate the internal resource group, while _destroying_ the group completely eradicates any mention of the resource group. To be honest with you, I'm not sure why I did it this way, but the `resources_clear_group` gets called inside `resources_destroy_group` anyway. 

Either way, once a resource group is created, we can finally start pushing resources to it. And for that, there are plenty of functions. The common syntax between all pushing functions is as follows: 


```c++
ResourceID resources_push_texture(const ResourceGroupID& group_id, const GfxTextureDesc& desc);

ResourceID resources_push_texture(const ResourceGroupID& group_id, 
                                  const FilePath& nbr_path,
                                  const GfxTextureFormat format = GFX_TEXTURE_FORMAT_RGBA8, 
                                  const GfxTextureFilter filter = GFX_TEXTURE_FILTER_MIN_MAG_NEAREST, 
                                  const GfxTextureWrap wrap     = GFX_TEXTURE_WRAP_CLAMP);

ResourceID resources_push_cubemap(const ResourceGroupID& group_id, const GfxCubemapDesc& desc);

ResourceID resources_push_mesh(const ResourceGroupID& group_id, const GeometryType type);

ResourceID resources_push_model(const ResourceGroupID& group_id, const FilePath& nbr_path);
```

And you get the point... 

Every resource has two ways to be loaded into the group: through a file path or programmatically. This way, we can use any resources loaded from the disk, and/or any resources we might need to dynamically load, like meshes with a specific geometry or a default texture, for example. 

One thing I do regret doing, though, is creating the resources and then immediately loading them with data. That might sound weird, but if you're planning to support async resource loading, then you'd have to separate the _create_ and _load_ functionality. 

Firstly, we can allocate all the resource IDs that we may need. Then, we can defer the loading to a few worker threads to load the resources themselves. There are other ways to handle the same things, though, so I'm not "locked" by any means. Async resource loading is certainly on my TODO list. It is _very_ crucial, especially if you have multiple resources being loaded at every level. While creating my last game [Crossing The Line](https://frodoalaska.itch.io/crossing-the-line), I did not have any problems with resource loading. It was not a bottleneck. However, that game barely had any resources. In the next game, I might need double or even triple the resources I used there. Besides, it is a pretty fun feature to add. 

Either way, if you're an eagle-eyed reader, you might have noticed that the push functions that take a path require an "nbr_path" as a parameter. The path part is self-explanatory. We need a path to the file on the desk. But, "nbr"? What is that? Well...

## No Formats, Just Binary

Every resource that the engine consumes from disk is usually saved into some intermediary format, probably exported from a content authoring software or downloaded from the internet. You have plenty of these "intermediary" formats across multiple resource types. You might have formats like MP3, OGG, WAV, and FLAC for audio. While textures may have formats such as PNG, JPEG, BMP, and so on. Fonts, 3D models, and animations, too, have their own formats. In the past, I just used whatever format decoder to retrieve the necessary data in order to be consumed by the engine. These are, obviously, _fantastic_ formats in their own right. Each comes with its own list of pros and cons, of course, but they are formats that have stood the test of time, with years and years of battle-tested experience in various fields. Some of these formats also compress their data so that it does not consume all the memory on disk, which is also great. However, in that greatness lies a terrible curse: they are _very_ slow to parse/decode. 

Because of the fact that these formats compress their data, they also have to spend time decompressing that data. For some formats, this is trivial, but for others, not so much. In my experience, image formats are the worst offenders of this slowness in decoding. Image files are usually very large. Especially if you want a high-quality image. Decoding said image might take 5 to 10 seconds. That might not sound like a lot, but 5 to 10 seconds in games is a _huge_ miss. Furthermore, we might have literal _hundreds_ of images that need to be loaded. That adds up to, potentially, _minutes_. Horrendous. 

Besides that, plenty of these formats may include junk we may never need. The best example of this is the 3D scene formats. GLTF, FBX, Collada, and what have you are what we call "3D scene formats". They describe a scene and all of its features. Cameras, lights, meshes, models, animations, AABB collision boxes, and much more. Even for game engines like Unity and Unreal, that information is useless, since most of the world editing will be done in the engine's editor. They are useful, of course. No doubt about it. But, as far as I know, these settings are great for exporting the formats from one content authoring tool to another. Perhaps even for model viewers. But information like the camera's position and the amount of lights in the scene is best left configured in the actual engine. Besides that, these formats are usually only used at the _production_ stage. When the game is distributed, we may want to use _our_ own format that adheres to the engine's needs. If we do not need the camera settings of the 3D scene format, then we ignore it. We won't spend extra time retrieving all the lights and their properties. Instead, what we _truly_ care about in formats like these is the _data_ itself. The vertices, the indices, the joints, the animation keyframes, and so on. That data will be used in the engine during runtime.

The same thing goes for the rest of the resource types. There is a lot of junk in audio file formats as well as image formats that we simply do not care about. But, at the same time, we still need to load in that data somehow from the formats. So, what do we do? Well, make our own custom binary format for resources, of course. 

This is how Unreal, Unity, and plenty of other game engines do it. And I'm not just copying what the big boys are doing. This method is by far _way_ faster to load and use than these intermediary file formats. Once again, nothing inherently wrong with these formats. It's just they are not _configured_ to work with _our_ engine. And that's their point. These formats aim to distribute data in a non-pervasive and efficient way. Frankly, not all of them are efficient or useful, but that's their goal at least. But we generally do not care. We will use our own format for every resource that is made specifically for our engine's needs. And thus comes "NBR". 

I have plenty of regrets about this engine. For example, I regret starting this engine and writing the graphics backend in OpenGL. Vulkan would have been better. I regret using C++ in the first place. And, most importantly, I regret calling my binary resource format "NBR". But it stuck now. I can't change it. 

"NBR" stands for **N**ikola **B**inary **R**esource. It's a binary format specifically made for the Nikola game engine. Now I could have also used a text-based resource format, but these are usually way slower to parse, and very easy to mess up. Binary formats, while hard to debug, are _extremely_ fast to read into memory and they have the added benefit of being "safe". Not that I care about people messing with the resources, but still. It's a plus. 

If you've ever worked with binary formats before, you'd know that they are usually broken up into sections (this is not a standard for every binary format to follow, but it helps my argument): 

- **Header**: This section will define certain attributes of the binary format, like a unique identifier, a major and minor version, the offset between each element, and perhaps how big the file is in bytes. 
- **Data**: This is where the actual data lives, described by the header section. In order to traverse this section and retrieve the required data, you'll have to know what each byte does and how to use it. 

This is my first time making a binary format, though, so it's not the most perfect ever. With that being said, now that we know how binary formats work, let's dive deep into how the NBR format is structured...

## The Nikola Binary Resource Format

At the top of each NBR file lies the _header_ section. As discussed above, this section is supposed to give us an idea of the rest of the file. In my case, the header of an NBR file is structured as such:

`
- Unique identifier   = 1 byte (unsigned)
- Major version       = 2 bytes (signed)
- Minor version       = 2 bytes (signed)
- Resource type flag  = 2 bytes (unsigned)
`

The unique identifier is there for "safety" purposes. Not every file with the `.nbr` extension will be an actual NBR file. Think of this as checking for the file's age before letting it into our very cool 21+ bar. This is usually called a "magic number", and it is called that because it is. There's no rhyme or reason (usually) to having this identifier be something helpful. Not in resource formats, anyway. Nonetheless, the unique identifier is only one byte, and it's just the average of the three ASCII values of the 'n', 'b', and 'r' characters. That would always be the value `107` no matter what. For extra safety reasons, I could probably have a more unique value with a certain pattern, perhaps. But I'm not making spaceships. I'm just making games where you can shoot zombies. That's it. 

Moving on to the next four bytes, we have both the major and minor versions. Each version number equates to two bytes. And, frankly, this is another decision that I regret. I wish I had just kept one version number and not two. I really wanted that extra detail, I guess. But, the version members are actually _very_ important. This binary format is bound to change. There might be something I did not think of, or something that came up, and I am forced to update the format. These two version numbers will gatekeep the older versions and basically deprecate them. Since I'm working on my own and I'm not guessing that this binary format will be of any use to anyone outside of this engine, this is somewhat "useless". If I updated the binary format, I just need to remember to update all the resource files that I have. But I might forget to update them, or I might even ignore them altogether, which can cause problems in the future. And you never know, when I do start working with an artist, these two version numbers will come in handy. 

And, finally, the two bytes of the header section are the resource type flag. And, by far, this is the _most_ important member out of all the rest. This flag will define the way we'll read the rest of the data. When this flag is read, we can compare it with the `ResourceType` enumerator we already looked at earlier. Different resources need different sets of data. A texture will need its width, height, channels, and then the actual pixels in order to be created. A 3D model will need the number of meshes, which include the number of vertices, and maybe a material, which includes a texture, and so on. Again, this data member is _very_ crucial to the reading process. 

And, in order to make my life easier, I created a `struct` that represents the data members of the header section. 


```c++
struct NBRHeader {
  u8 identifier;                 

  i16 major_version;
  i16 minor_version; 

  u16 resource_type;                
};
```

With every file we read and write, the header file has to be taken into account. Every NBR file out there _must_ include this header section. But now that we have the header in check, we need to talk about the data.

As I said before, every resource type is read differently. In order to read and write the correct binary data, we'll have to query for the resource type in the header. After that, it's just all about writing the resources. However, we do not exactly write _everything_ the resource will need. Remember, the only thing we care about is the actual data. Anything else is just filler that we will ignore. We cannot go overboard with the amount of members we have in the binary format since we do _not_ compress the data in any way. Again, the purpose of this format is to reduce the overhead that many of the intermediary formats introduce. We just want to read the data, and that's it. That has the benefit of being fast, but it also means that, since we don't compress the data, the files are going to be _huge_. Or at least the size will be much bigger than the actual intermediary format. 

For example, let's look at this beautiful JPG image. 

![paviment](https://frodoalaska.github.io/assets/img/paviment.jpg) 

Very interesting. On disk, this 1024x1024 image takes around 6.87MB of space. Pretty compact. With our NBR format, however, this bloats to 34MB! That's what? Almost 6 times the original size? However, the load time on that image is _massively_ different. Our NBR format loads much faster than the original JPG. With that being said, though, there are a lot of low-hanging fruits we can reap in order to compress images. We can compress the images slightly so that it doesn't take up as much memory, but we still keep the faster load times. And that goes with the rest of the resource formats. Larger memory, but faster load times. However, that is something to do in the future. Currently, the resource format is so fast that it outweighs the cons of a bigger memory size. Even though it makes my inner programmer scream, we have to just ignore it, and move to more pressing issues. Keep tuning back to this blog since perhaps in 12 years or so, I'll finally implement a compression algorithm on the resource formats.

Either way, let's take a look at how a texture is represented in memory. And, much like the header of the NBR format, I have a special `struct` that represents the texture in NBR format. 

```c++
struct NBRTexture {
  u32 width, height; 
  i8 channels; 

  void* pixels = nullptr;
};
```

Pretty simple, eh? The structure is self-explanatory. The first 8 bytes represent the size of the texture. The `channels` member (the bytes per pixel) of the texture is in the subsequent byte. As for the rest of the file, it's just the literal pixels of the texture. Now, if you go back to the devlog where I talked about the [graphics backend](https://frodoalaska.github.io/2025-03-09-making-an-opengl-wrapper/), you'd see that a texture is much more than its width, height, channels, and pixels. It includes other attributes like the format of the pixels, the texture filtering, the addressing mode, and more. However, I choose to keep the texture pretty simple. Keep in mind that the resources in the NBR format are supposed to be the "compressed" version of the actual engine-side resources. That way, we don't take up as much memory on disk. However, I do feel like I could have at least kept the addressing mode and filtering mode of the texture on the NBR format. Many intermediary formats don't include such details. However, there are ones that do. Again, we'll leave it for the future. I haven't come across a use case for keeping these two properties in the NBR format. 

I won't go over all the resource types and how they are structured in the NBR format since they all follow the same principles. I won't go over the implementation of the reading and writing of the binary data, either. It's just a simple file operation. Nothing more to it. When we load the resources using the various `resources_push_*` functions, we'll first load the NBR file and then convert it to the engine-side resource type. For example, for textures, I do something like this: 


```c++
// Hypothetical function to read all of the NBR file 
NBRTexture nbr_tex; 
nbr_file_read(nbr_path, &nbr_tex);

nbr_file_check_validity(nbr_path);

GfxTextureDesc tex_desc = {
    .width    = nbr_tex.width, 
    .height   = nbr_tex.height, 
    .channels = nbr_tex.channels, 

    // Other settings.. 

    .data     = nbr_tex.pixels,
};

GfxTexture* texture = gfx_texture_create(gfx_ctx, tex_desc);
```

I don't exactly have the `nbr_file_read` and `nbr_file_check_validity` functions, but they are steps that are carried out when we load an NBR resource. The actual code is much more verbose and annoying to go through. But, I do hope you get the general gist. The engine reads in the NBR resource and converts it into an "engine-coherent" format. Afterwards, the resource can be used in whatever way is needed. Besides that, we won't need the NBR file anymore, so we deallocate any possible memory it allocated and close the file. However, that all happens on the engine side. The writing part of the operation takes place at runtime when we're playing the game. But, in that case, who actually _writes_ the data?

## That's Not All Folks

The Nikola game engine does not include any resource importers for intermediary file formats. The engine does not know, nor does it care about PNGs, JPEGs, MP3s, or GLTF files. It has no clue what these are. All it cares about are the files with the `.nbr` extension, and that's it. It only knows about NBR and cares about NBR. But we have to get the data from these intermediary file formats somehow, right? Well, yes, but that's not something that the engine is supposed to concern itself with. Remember, with Nikola, the engine _is_ the runtime. And, as we said before, the runtime should only care about NBR. But, in order to convert any intermediary file format into an NBR file, we have to use a tool created by yours truly called the--wait for it--[NBR Tool](https://github.com/FrodoAlaska/Nikola/tree/master/NBR). Yes, I was feeling particularly creative that winter. 

The NBR tool is a sub-project of the actual engine. It's a CLI tool that accepts a few arguments to convert any resource type from its intermediary format into our beloved NBR format. However, when we're creating a game with the engine, we do not necessarily care about the NBR tool. We only care about the engine. Converting any resource into the NBR format is supposed to be a one-time thing. And so, you have the option to either include or exclude the NBR tool from the engine's compilation process. And that's what I usually do. I would build the NBR tool separately and install it as a global CLI command. When making a game, I just build the engine and not the NBR tool. That way, I won't have to include things like `stb_image`, `stb_truetype`, or `Assimp`, for example. Since, again, I do not give a _squat_ about them. I'll use the CLI tool to convert the game's resources, and then go on with my day. 

However, I'm afraid I'll have to leave you on a cliffhanger here. The NBR tool is filled with so many functionalities that it will take me about two thousand more words to explain it to you. It's the only system that is multi-threaded. I created a custom programming language just for it. It uses a lot of third-party libraries for images, audio files, fonts, 3D models, and even animations. It's by no means a massive project, but it's complicated. 

And so, I'll have to leave it for another devlog. Don't worry, though, you don't have to pay a subscription fee for it. You just have to wait.

With that being said, if anything intrigues you and you want to see the full implementation of everything, you can check out the whole project over [here](https://github.com/FrodoAlaska/Nikola). You can also check out the NBR tool over [here](https://github.com/FrodoAlaska/Nikola/tree/master/NBR).

Thanks for reading and have a good day/night.
