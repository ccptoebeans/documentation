---
title: Update Binaries
parent: Components
---
The binary artifacts required to create a working version of a carbon engine application are located in various places within a Perforce repository. For example, those built by TeamCity reside inside the `autobuild` folders, while those from external SDKs are placed in the `vendor` folder. Nota bene that nowadays the Perforce branches see carbon components as an external SDK as well. Furthermore, most binaries exist in multiple versions that depend on the operating system, CPU architecture, and compiler used. As such we need to have a process which assembles the artifacts from those different sources and places them inside a location from where an application can be run. By having a separate step that assembles the final execution environment we used to also gain the benefit that we can sync Perforce and compile C++ code while an EVE process is running, though these days this is much less of an issue as components are developed in isolation from any game itself.

We call this step the "Update Binaries" process. The one and only purpose of this is to fulfil the above task: assemble an execution environment for the various applications which live in EVE's monolithic Perforce repository.

## Requirements

- needs to work across different platforms

- needs to work without any major prerequisites

- needs to understand how different applications are copied

Clearly the first two requirements are at odds with each other due to inherent differences between different platforms - for example, Batch scripts which work out of the box on Windows cannot be run on on macOS. Since EVE is making heavy use of Python, and a Python version is pre-installed on most operating systems, then a simple python script which only requires standard library modules is a good candidate to satisfy the first two requirements.

Requirement number 3 is a question of providing a suitable data structure. This can be as trivial as maintaining a look up table of "application names" to "binaries required by platform, environment, and architecture".

All requirements are fulfilled by a simple Python script called "updateBinaries.py" which lives at the root of a branch of EVE's Perforce repository. The vast majority of the contents of that script are two look up tables: A look up table for applications, and a lookup table for packages. An `application` describes which binaries are required for an execution environment, and where they should be located. A `package` is a collection of files that belong together. Each entry in one of the look up tables needs to specify for which platform, architecture, and environment it should be available. This may seem overly verbose, but it does prevent accidentally leaking binary files into an environment where they do not belong.

## How to...

#### ... use the script

Passing the `--help` parameter to the script will print usage instructions and descriptions of available parameters.

#### ... add a new binary

Identify which applications require this binary

Provide a `CopyRule` for the binary for each identified application, platform, architecture, and configuration for which it is required.

#### ... add a new package

Simply add a new entry in the `PACKAGES` dictionary.

#### ... add a new environment / platform / architecture

Talk to platform - these are very fundamental properties which may take considerable effort to get done right.

#### ... add a new application

Simply add a new entry in the `APPLICATIONS` dictionary. See the comment in the code for details on its composition.

## Troubleshooting

Sometimes things can go wrong, and situations arise that weren't anticipated. The script was designed to be as verbose and helpful as possible in this case, but sometimes the information provided is not enough. In that scenario you can run the script with the `-v` / `--verbose` flag, which will print a lot  more information about what is going on in the background. Even if you cannot make sense of the output now it is a good idea to save this log for future reference. Should the verbose log not have provided a hint on what was going wrong then there is always the nuclear option of providing the `-d` / `--delete` flag, which will try to remove the destination folder before copying existing files.
