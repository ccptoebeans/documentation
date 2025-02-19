---
title: Overview
layout: default
---

What is the carbon engine? First of all, it is the game engine that powers EVE Online and EVE Frontier. But that doesn‘t tell us much now, does it? No, so let‘s take a closer look.

What the carbon engine really is, is a lot of components written mostly in C++, but also with a lot of Python in the mix. In fact, at it‘s very heart – and unlike other game engines – the central piece of the carbon engine is an embedded Python interpreter.

I say unlike other game engines, because usually game engines allow writing some scripts, or store some data files, in a script language. Those other game engines are all held together by a very firm core of C / C++ code, and maybe they expose a plugin interface for writing some extensions.

Carbon, on the other hand, builds on top of the embedded Python interpreter. There is a Python module that acts as an entry point which then ties all the components together. Components like the rendering engine, or the audio engine, or the database driver, or the network RPC interface, and many, many more. This has many advantages, like in theory it would be possible to switch out the physics simulation component. In practice that‘s not so trivial, because there are many informal interfaces that need to be adhered to. But it also allows rapid prototyping of new game and engine features. And yes, I know what question this raises: isn‘t Python too slow for this? Yes, it certainly is! But that‘s where the C++ components come into play. When a system is deemed to slow, it gets rewritten in C++.

Which components are available is completely controlled by the Python environment that is prepared. For example, the environment for the game server does not contain the graphics or audio engine, while the client does not get to talk to the database, and therefore does not contain the database driver. It is very much an engine where „you only pay for what you use“.

OK, that‘s a lot to take in, and it is all very abstract. So, let‘s look at something more concrete.



Most developers on the game teams interact with the engine without having to think about the things I just mentioned. What they do instead is to simply run some internal tool that starts a client or server for them.

And that’s fine, really, nobody should have to think about “preparing an environment” when they are trying to add a new spaceship or tweak some in-game UI. It should just work.

But sometimes things go wrong: a game developer writes server code that accidentally attempts to import the graphics engine and then wonders why their server doesn’t start up. Or, more important for us on platform, an engine developer writes this cool new component and then needs to know how to make it available to game developers.

In those moments, as a curious developer, you do want to know what happens underneath the hood, even if only to be able to ask for help from your co-workers. So, let’s take a look.

What you see in this picture is what happens whenever one of our internal tools used to start a client, or server, and so on. The first step – called “updateBinaries.py” – is a simple Python script that copies files from some place to another. This step decides which components are available in the process environment – after all, a component is just a set of files.

The next thing that then happens is that the entrypoint executable, aptly named “exefile”, gets run. “Exefile” utilizes a component called” blue” to retrieve information from the process environment – e.g. read command-line switches and environment variables, map a virtual file system to a real file system.

It then uses all this information to initialize the embedded Python interpreter. Once that is done, the Python interpreter imports a module called “autoexec”, and from there onwards it’s effectively game code. If this were a game server, it would start opening a port to listen on for connecting clients. On the client-side, this would go and create some UI or other graphics scene for the player to interact with.

All in all, it’s very straight-forward, and not very complicated.



The first step in the start-up sequence – preparing the environment – is rather unique to Carbon. Because of the engine’s modular nature, and because it was developed for a game that runs different kinds of processes which share some Python code between them.

This is done through the `updateBinaries.py` script that contains some lookup tables and a simple loop that then copies the requested contents in an expected location.

And it warns (or errors) when it fails to find a file that should be copied.



The flow you’ve just seen is what we commonly call “game engine mode”. It is how the Carbon engine traditionally always worked. Let’s quickly recap it.

Engine and product specific environment configuration, like making sure components are in the right place

A very locked down Python execution environment: This is a critical part, because by default Python allows a lot of configuration through environment variables and such, and if we wouldn’t lockdown the environment in game mode then on some user’s machines the engine may not run as fast as we’d like it to, or it would end up using libraries we don’t want it to, and so on.

It then looks up the autoexec module, and after importing the module the game loop runs.

Now, remember. A lot of the code for the games is written in Python. The nature of modern Python code is that many developers want to practice test-driven development. Or someone just wants to play around with a new component in sandbox environment. All of this is certainly possible when running in game engine mode, but it comes with a lot of constraints and often awkward ways to go about things. Leave alone attaching debuggers from external IDEs can be really difficult when your Python code is running in the locked down Python interpreter that the game engine mode provides. To make all this simpler, the Carbon engine can also be run in what we call “interpreter mode”.

Interpreter mode starts very similar, but deviates quite quickly in its behaviour. Notably, it provides an almost standard Python execution environment – the major difference being that the PYTHONPATH contains the requested engine components and game code. And then it simply runs a standard python REPL: if you pass it a script or a module on the command-line, it will run that. Otherwise, it will simply prompt you with the interactive shell. So you can see, it doesn’t run the game loop. One will have to do that themselves, should it be required.

Let’s look at something concrete to understand the difference.
