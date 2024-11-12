---
layout: post
title: I Made My Own Standard Template Library
subtitle: And Here's How You Can Too
comments: true
tags: [api, gamedev, discussion, tutorial, c++]
---

Check out the library if you're interested: [Ishtar](https://github.com/FrodoAlaska/Ishtar)

# The Story Of Steve 
Once upon a time, there was a man named Steve. Steve--like many of us--was a programmer. He specifically programmed in the lord's language C++. He was obsessed with C++, in fact. He woke up every day just to kiss a photo of Bjarne's very shiny scalp. Now even though I like to take it up the ass (as some would say), I still think that's kind of homo not gonna lie. However, that was all to change. One day Steve was programming in his favorite language, and his curiosity took over him. He was using the best-named container of all time `std::vector`. He wanted to know what the `push_back` function _actually_ did. And even though his mind was telling him no, his body, was indeed, telling him hell yeah. And, since Steve is a genius and used the best editor to ever exist NeoVim, he went to see the definition of the `push_back` function... what followed was a cry for help. 

Steve was not the same after that. He was devastated. Beaten. He felt betrayed. On a rainy and dark day, Steve decided to end it all. He brought a buffer of memory with him to the park. He found a nice tree and... thankfully, the memory was segmented and he survived... so anyways, I replaced C++'s horrendous Standard Template Library. 

# But Why? 
Now if you're an intellectual like myself, you'd probably be asking, "Why do this?". Well my dear... ah... dude, there are a few reasons that I decided to jump into this venture. I have 3 reasons in particular for starting this project. 

The first is that I think this is _extremely_ fun and interesting. I learned programming because it's fun in the first place. It piqued my interest and gave me an unbelievable sense of enjoyment. Yes, even more than sleeping with your mom. So, why not keep on that tradition and make projects that I find enjoyable and intriguing? And also, tell your mom to call me back. I didn't mean to call her fat. 

The second reason is that I think doing such a thing is a _great_ learning experience overall. Even though this library of mine is not efficient or performant by any means, I still think that it gave more appreciation and knowledge when it comes to data structures. I'm not oblivious to what the function `insert` does on a `std::unordered_map`. I know the costs of calling that function. I know how to make that function faster. Now, of course, every library has a different implementation of these data structures. Though, still, at least I have an idea of _how_ this might be implemented. 

The third and final reason is control. I love to control every byte of memory that is given to me by my computer. I vied to control every piece of data ever since I can remember. I want to control the world and the people and make them do... oh wait. No sorry. Wrong script. Excuse me. 

# The Rules 
So before I get into my beautiful piece of code, I need to go over the rules and the inspiration of this library. I decided to set a few small rules for *thyself* before starting this library. Essentially I was not to use _any_ piece of code from the standard template library. So no `std::string`, no `std::array`, no `aids::uh_uh_ah_push_into_butt`, and so on. Any of the code of the data structures that will be implemented will be _my_ code and only my code. Now, I did permit myself to use things like `malloc`, `free`, `realloc`, `memcpy`, and so on. However, those are mandatory at this point since I _will_ need to allocate memory and I can't do that... unless I write my own operating system. And no that would be silly. I would never do that. Hey, google, how do I go to Finland? 

# Inspirations And Architecture
Now another thing we'll need to talk about is the architecture of this library. Even though I can go all `boost` style with this library and make `dlls` that are the size of your mom's left elbow, I will refrain from that. Instead, I'll take inspiration from the `stb_` collection of libraries. If you don't know, the `stb_` libraries are single-file-only libraries. Meaning, you don't have to link them or take cocaine in order to use them. All you have to do is copy the header file, make a `.cpp` or `.c` equivalent put this definition before the include statement `STB_IMAGE_IMPLEMENTATION`, make sure to build that translation unit with your project, and voila. You're using the library now. Obviously, there might be some cons with this approach but I think it's the simplest and most headache-free solution to the mess that is the C++ dependency hell. Now, by default, my library will be templated, meaning I'll be forced to use a header file-only approach anyway. However, not _everything_ in the library is going to be templated and we'll see that shortly. So, in the name of simplicity and ease of use, I'll be going with the `stb_` single-file approach. 

Okay, so, without further ado, let's get riiiiiight into the code... here's a formal letter of apology for that. I'm very sorry.

# Lists
The simplest kind of data structure is the `Linked List`. It is probably the most primitive of data structures. If you ever want to learn a programming language, linked lists are probably the first thing you'll implement since it's very simple. They aren't really hard to understand either. It is just a series of nodes that are connected via links--or pointers in this case. A list of linked nodes. Linked list. Now, there are two types of linked lists: Singly linked and doubly linked. Singly-linked lists mean that each node is only connected to the node next to it. While--as you would have guessed--in a doubly linked list, a node is connected to the next _and_ previous node to it. Here's how I implemented nodes:

```c++
template<typename T>
class Node {
  public:
    Node() = default;
    Node(const T& val, Node<T>* next = nullptr, Node<T>* previous = nullptr) {
      value          = val; 
      this->next     = next;
      this->previous = previous;
    }

  public:
    T value; 
    Node<T>* next;
    Node<T>* previous;
};
```

As you can see, each node has a `next` and `previous` pointer. Now, even though a linked list is easy to conceptualize, it is _very_ annoying to implement. Not because it is difficult, no. Because there are so many small edge cases that you have to account for. Still, they aren't very hard to implement even then. 

There are some basic operations a linked list has. Like `append` which pushes to the back of the list, `prepend` which pushes to the front of the array, `pop_back`, and `pop_front` which _removes_ a node either from the front or the back of the list. Believe me, I can make a lot of sexual jokes here but I won't because I'm a changed person now. There are other operations obviously like 'remove', `insert` (I'm a changed person now), and `get_at` which are all self-explanatory.

Here's how these roughly look in the library:

```c++
    Node<T>* get_at(const sizei index);
    void append(Node<T>* node);
    void prepend(Node<T>* node);
    void insert(Node<T>* node, Node<T>* prev, Node<T>* next);
    void remove(Node<T>* node);
    void remove_at(const sizei index); 
```

There are also `emplace_` operations as well which are equivalent to the `append` and `prepend` operations except that they take in a value of type `T` instead of a `Node` structure.

Now, the next two data structures are essentially just linked lists but with constraints. 

Oh and here's a fun fact, the doom engine used a linked list to link the menu scenes in the game together. You know, just a cool fact from a pretty cool guy I would say. 

Here's a working example:

```c++
ishtar::LinkedList<int> list(100); // First default value for the head

list.emplace_back(42); // Pushes the value '42' to the tail
list.emplace_front(69); // Pushes the value '69' to the head 
```

# Queues 
Imagine you're at the only operational toilet in a Billie Eilish concert. You stand in line, trying to hold in your poop and wonder why you ate that obviously suspicious-looking burrito earlier. In that situation, the first person in the line will go to the toilet, use it, and then leave. The person behind them will be next in line so they go to the toilet, use it, and then leave. This is what we, definitely big pp programmers call a `queue`. And queues are a "FIFO" or a First In First Out data structure. 

Queues have two very useful operations: `pop` and `push`. Other programmers call them differently. But I'm not like other programmers. I'm extremely in debt. 

When we _push_ to a queue, we push to the back. When we _pop_, we pop from the front. Pretty simple. But don't let their simplicity fool you. Queues are a _very_ powerful data structure that was implemented in _so_ many applications at this point.

Here's an example of an event Queue in action:

```c++
ishtar::Queue<EventType> event_queue;

event_queue.emplace(EVENT_KEY_PRESSED);
event_queue.emplace(EVENT_KEY_RELEASED);  event_queue.emplace(EVENT_APP_QUIT);
event_queue.emplace(EVENT_WINDOW_RESIZED);

event_queue.pop(); // Will pop `EVENT_KEY_PRESSED`  event_queue.pop(); // Will pop `EVENT_KEY_RELEASED`
```

# Stacks
Something similar to a Queue is the `Stack`. Stacks are the opposite of queues. They are a "LIFO" or a Last In First Out data structure. Imagine if you were a loser dweeb (which you are) who had a lot of anime pillowcases. Since you're a degenerate, you _finally_ decided to wash these cases that you have definitely slept on and nothing else. You put the first case down and then the second on top of it and then the third on top of that. Now, since you were dropped on your head as a kid and barely use your brain, you sit and think about how you can take out the pillowcases one by one to wash them. "Aha", you exclaim. "I'll just take the last one I placed and remove it from the stack". You _definitely_ need some help. I don't even know how you're still alive. But yes. That's what we call a `Stack`. 

Much like a queue, the stack also has the `push` and `pop` operations. However, this time, when we _pop_ from the stack, we'll pop from the back and not the front. Pushing is still the same. 

Here's a simple example:

```c++
ishtar::Stack<int> num_stack; 
  
num_stack.emplace(101);
num_stack.emplace(202);
num_stack.emplace(303);
num_stack.emplace(404);

num_stack.pop(); // Will pop '404'
num_stack.pop(); // Will pop '303'
```

# Dynamic Arrays 
If I had to name two of the greatest inventions known to men (and women, too), I would immediately name sandwiches and `Dynamic Arrays`. Mostly sandwiches, though. 

Unlike you're ever-mounting depression, regular arrays are fixed-size. They can't grow any larger than the initially agreed-upon size. In fact, it is illegal to go past that point. You'll get a segmentation fault and probably a slap on the face by me. However, there are many instances where the size of the array is not known priori. And yes, I did use a fancy word there. And so, dynamic arrays were invented... in some guy's moldy dungeon probably.

The basic idea of dynamic arrays, or if you're retarded, `std::vector`, is that they grow as the elements are pushed into them. Dynamic arrays have two very important properties: size and capacity. The size (or more like the count) signifies the number of elements in the array currently. While capacity is the... ah... well, capacity of the array. The number of elements the array can hold before it needs to allocate more memory essentially.

Now in the default constructor of my dynamic array, the size gets set to 0 and the capacity gets set to 5. This means the array will need to have 5 elements before it can be resized again. 

```c++
ishtar::DynamicArray<float> arr; // Default constructor will have a capacity of 5
```

And this is where _truly_ understanding data structure makes you a better programmer. Say you decide to push 5 elements into the array. When the array sees that the size is now equal to the capacity, it will grow the array by half the current size of the array. Meaning, that it will grow by 2 since this will be an `int` and we'll ignore the decimal part. Now our capacity is 7. But then you push 2 extra elements which means the array has to grow again. You can see how this can be very terrible on performance. Allocating memory is not as easy as watching that hentai that you kids watch nowadays. Back in my day, we watched real porn. Like gay porn.

```c++
// Must grow the array if the size is greater than the capacity 
if(size >= capacity) {
	// Re-allocate the buffer of data by the new capacity
        capacity = size + (size / 2);
	data = realloc(data, sizeof(T) * capacity);
}
```

So what would you do? Simple. There is another kind of constructor the dynamic array has. One that takes in an initial capacity. You don't have to set this initial capacity to a big number. But you just set it to something respectfully sizeable so that you don't have to allocate memory and move the pointers all the time. 

```c++
ishtar::DynamicArray<int> arr(24); // The array's initial capacity will be '24'
```

There are many operations that a dynamic array possesses. They include but are not limited to `push_back' and `push_front', 'pop_front' and 'pop_back', `remove`, `insert`, `slice`, and many more. 

The most expensive operation out of all of these is probably the `remove` and `insert` functions. Since they do require to shuffle the whole array in order to account for the change in elements. And again, I would have never known the _true_ precautions of such operations until I implemented them myself. 

```c++
ishtar::DynamicArray<int> arr;
arr.reserve(50);  // Reserves 50 elements in the array. This is an alternative to passing an initial capacity to the constructor
  
arr.append(0);
arr.append(1);
arr.append(2);
// ...
  
  // We can also take a slice of the array and copy it to another array
isthar::DynamicArray<int> slice_arr = arr.slice(0, 4); // Slice from the first index till the 4th index inclusive. 
```

# Strings 
Strings are probably the most complicated data structure out there. In theory, it is simple. An array of characters that you can push into. Essentially a dynamic array of `chars`. However, it's not so simple. There are so many operations associated with strings. Not to mention they can potentially be very large and therefore expensive to operate on. A simple comparison of two strings can, potentially, be _very_ expensive. I haven't even talked about localization. Where you'll need to account for various languages with different writing styles and notations. 

In general, there are a few types of strings. And they all are concerned with the size of each character in a string. For instance, a UTF-8 string can only hold 8 bits per character. And it doesn't take a bald genius with glasses to know that 8 bits is not big. However, UTF-8 strings are still sufficient enough for European languages and English of course. There are also UTF-16 and UTF-32 strings. Obviously, each holds either 16 bits or 32 bits per character in a string.

Despite having a wide array of choices (huh array), I decided to use what's known as "ASCII" or Unicode strings. Which can _only_ support the English language. And, unless you're some guy from Montana, that's not gonna suffice. However, for the sake of my sanity, I decided to use ASCII strings for now with plans to implement UTF-8, UTF-16, and UTF-32 in the future. Hopefully. If I don't get blown by that guy from Montana.

Besides the complexity of localization, strings also hold a massive arsenal of operations. Even larger of an arsenal than the arsenal of guns that guy from Montana has. Things like `reverse`, `append`, `find`, `replace`, `equality`, and so much more. You can do infinite things with strings. I just touched the surface. 

Now, strings are actually the only data structure in the library that isn't templated. It is just a simple class with two properties: length and data. I _could_ have made it a template class, but I didn't see any benefit from that so I decided to keep it simple. 

You should expect these at this point:

```c++
ishtar::String str1 = "hello"; // Implicit constructor
ishtar::String str2 = ", world";

// We can append the strings together
str1.append(str2);

// Wow, how original
ishtar::String palindrome = "racecar :emordnilap";
palindrome.reverse();
```

Speaking of length, the strings in this library are length-based and not null-terminated. You see, in C ("see" what I did there. Okay), each string has a "null" character (which is just a 0) appended to it. That's how the compiler will know _where_ the string ends. However, that is a stupid approach. Why? Because... fu.. ah... fuck you. So, instead, I went with the length approach. We don't have to go through the entire string until we find a null-terminator character. We can just store the length of the string right there in the data structure. This obviously has a lot of performance benefits since I don't have to calculate the length of the string _every time_ I need to do something with it. I can just recall it immediately.

I don't necessarily like my approach as I still use `strlen` to calculate the length of my string on _some_ occasions. However, it will do for now. I never claimed this is the best library ever. So leave me alone. Now wait. Don't leave me alone. I was just joking come on.

# Hash Tables 
Now, this is probably my favorite data structure ever. It's so useful for many different reasons. I have used it plenty of times and it never let me down. Unlike you. You always let me down. 

Hash tables work in a very magical way. They use a key-value pair to store each entry. Meaning, that in order to access any value in the table, you'll need to use its key. So essentially it's like indexing into an array except you can use any kind of primitive data types as indices, not just `int`s. This works by using a process called `hashing`. You basically scramble the key into an integer that can then be "modded" (`%`) with the size of the table to retrieve the index of the value in the array. You can, for example, use strings. This is, in fact, how many game engines create their resource managers. Or how YouTube stores its users. Everything is a hash table. Seriously. You're a hash table right now. So it's very useful to know them. 

Hash tables are also _extremely_ efficient. Their access time is almost as close as a regular array. Which is constant time. Which is, like, _extremely_ fathst. Now I say "almost as" because access time on hash tables is not _always_ constant and I'll get into that in a bit.

So, before we start looking at the implementation of the hash table, we need to start talking about hash functions. Now there are so many hashing functions out there. But, we only care about the hash functions that are a) fast and b) deterministic. Fast as in they should never be the bottleneck. They should be _lighting_ fast. And deterministic as in they should always produce the _same_ output _every time_ we give it the _same_ input. So if I decide to hash the number 0 and it produces the number 100 then it should _always_ produce the number 100, for example. I picked out a hashing function from the [Crafting Interpreters](https://craftinginterpreters.com/) book. It's not the best but it'll do for now. It looks something like this:

```c++
template<typename K> 
const u64 hash_key(const K& key) {
  u32 hash = 2166136261u;

  hash ^= key;
  hash *= 1677719;

  return hash;
}
``` 

I don't understand it. You don't understand it. Jared over there doesn't understand. But it doesn't matter. What matters is that it works. 

And once we hash a specific key, we can `%`ed it with the capacity of the table (and yes tables are just arrays under the hood). That should produce the index with which we can retrieve our value. However, there's a problem here. You smell like shit. But there's another problem. 

With hash tables, there's quite an infamous problem. When we hash a key and retrieve the index, we can potentially have two values that take up the same spot in the array. Meaning, they will produce the same index. This is called a _collision_. And it happens more times than you think, especially if you have a terrible hashing function. So a bunch of old white people cuddled together in a room and found a solution to this problem. Well, actually, multiple solutions but we'll look into two of them. 

The first is called an "open table" solution. In this method, we can use a linked list to store all of the values that produce the same index. Anytime we want to retrieve a value, we just get the index and if that index doesn't match the key, then we'll walk the linked list until we find our key. While this approach might _seem_ good, it's actually not the best. It does take up a lot of memory since every node in that linked list is a _required_ allocation. And traversing a linked list is not constant time so it kind of defeats the purpose. However, it is still very much faster than a regular linked list and those lists won't be that long anyway. 

The second solution to collisions is something called a "close table". Sometimes, this method, confusingly, is called "open addressing". In this method, something called "probing" is used. Essentially, whenever we encounter a collision, we just look for the next index. If that spot is empty, we'll just place the value there. But, if it's not empty, we just look to the next index and then the next index and then the next index and so on until we find an empty slot. Of course, if we do reach the end of the table we will wrap around to the beginning. 

In my opinion, the second solution _is_ better. However, it is much more headache-inducing to implement. Trust me, I tried. And so, even though I think it's an inferior solution, I used the open table method for resolving collisions. If you think that's stupid, then... I swear I'll break up with you. Like, I'm not even joking.

`HashTable` in action:

```c++
// Creating a table with a capacity of '12'
ishtar::HashTable<int, ishtar::String> names(12); 

names.set(32, "Buddy Guy Jones");
names.set(83, "Abigail Jack JR.");
names.set(128, "Salem Yunich Euler");
names.set(256, "Jackson Free Darnell");
names.set(512, "Ian Long Bundy");
names.set(1024, "Randall Axel Jones");
 
// Retrieve the name at index `1024`
ishtar::String randall = names.get(1024);

// Now that I think about it, using ints as keys wasn't the best idea. 
// But you can imagine the key being a string, for example. I'm dumb sometimes. On occasion. 
```

# Conclusion
So what are you supposed to take away from all this? Well, let me ask you a counter-question. Why are you... why are you dumb? Like. 

No, but seriously, the reason I made this library in the first place was to show you how simple and easy all of this is. I _highly_ encourage you to try out this on your own. You'll be a better programmer overall. Trust me, dude. Like, come on. 

Data structures are a _crucial_ part of any application. Understanding them will be of benefit to you. Are you gonna make a library that is better than C++'s STL? Obviously no. Is it gonna be more efficient or performant? Well, you never know maybe. But that's not the point. The point is that you'll learn a _ton_ of things. You'll appreciate these libraries more. You'll even have an idea of how things work under the hood so you can use them more efficiently in the future. 

Now, as for me, I plan to use this library for some time. My plan is to grow it and expand upon it. Fix some of the bugs and make it more performant. I'll see how far I can go with this library without losing faith in humanity. 

Well, actually, the joke's on you. I already lost faith in humanity.

But seriously though, tell your mom to call me back because like...
