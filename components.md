---
title: Components
has_toc: false
---

The carbon engine is organized as a set of components, each providing specific functionality. The following table provides an overview of available components and their purpose.

| Component | Description                                                                                                                                                               |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| audio | Audio engine                                                                                                                                                                 |
| blue | Historically a kitchen sink of functionality that's slowly broken apart. Functionality includes the main game loop, resource loading, but also various other bits and pieces |
| blue-exposure | Static library providing macros for exposing C++ code to Python                                                                                                              |
| core | Generic low-level functionality and cross-platform abstractions for system calls                                                                                             |
| db | Optimized access to the MS SQL Server database                                                                                                                               |
| destiny | Core game world simulation engine for MMOs whose titles start with Eve                                                                                                       |
| exefile | Entrylevel executable that bootstraps the embedded Python environment                                                                                                        |
| fsd | Optimized access to file static data                                                                                                                                         |
| geo2 | Python exposed Vector types and operations                                                                                                                                   |
| imageio | Support for reading and writing different image formats                                                                                                                      |
| ime | Python exposed Input Method Editor functionality                                                                                                                             |
| io | Provides low-level networking functionality                                                                                                                                  |
| localization | Localization framework                                                                                                                                                       |
| math | Math functionality for vectors, planes, quaternions, etc.                                                                                                                    |
| pathfinder | Pathfinding functionality                                                                                                                                                    |
| pdm | Platform Detection Module, an OS agnostic library for data collection                                                                                                        |
| pdm-proto-wrapper | C++ library to wrap PDM output in protobuf                                                                                                                                   |
| scheduler | Scheduling functionality for Greenlets                                                                                                                                       |
| spacemouse | Support for the SpaceMouse input device                                                                                                                                      |
| trinity | Rendering engine                                                                                                                                                             |
| videoplayer | Video playback functionality                                                                                                                                                 |
| webbrowser | Python exposure of the Chromium Embedded Framework                                                                                                                           |
