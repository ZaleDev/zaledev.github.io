---
layout: post
title: Useful Functions
date: 2025-02-10 12:21 +0000
description: A collection of general-purpose useful functions for Unreal Engine C++.
categories: [Unreal, Misc]
tag: [unreal, utility, misc]
toc: false
---

### World Context from Anywhere
There are situations in which we may need to get a reference to the world context from a class that has no valid 'GetWorld()' implementation.  
A workaround is to fetch it through the GEngine.
```cpp
UWorld* UMyFunctionLibrary::GetWorldFromAnywhere()
{
    if (!GEngine || !GEngine->GameViewport)
    {
        // Handle error
        return nullptr;
    }
    UWorld* World = GEngine->GetWorldFromContextObject(GEngine->GameViewport, EGetWorldErrorMode::LogAndReturnNull);
    if (!World)
    {
        // Handle error
        return nullptr;
    }
    return World;
}
```
This function has never failed for me, but it is important to note that 'GEngine' is not guaranteed to always be valid. 'GEngine' should remain unchanged for the duration of the program after initialization, so care should be taken when trying to use it very early on during initialization, or in scenarios where GEngine is never initialized (maybe in dedicated server builds).
