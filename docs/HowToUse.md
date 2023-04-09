---
title: How to Use This Tutorial
layout: default
nav_order: 2
---

# How to Use This Tutorial

This tutorial is intended to be followed sequentially. We will be building up our codebase, and most chapters require the previous chapters to work. There is space for you to apply your own creativity to your implementation, especially on the GUI side. Feel free to experiment and make it your own.

I wrote this very casually. I use contractions. I use first person pronouns. It was actually a struggle to not revert to writing style of a grad school publication.

Whenever possible I try to include screenshots to help explain things on the Unity side. If you look carefully, you may spot inconsistencies with things like the project name, or may notice that I have a `GameObject`in the scene that has not been added in the tutorial. I am not going to fix these, but the examples should still be easy enough to follow.

{: .warning }
The P3 SDK is very good and does a lot to protect your machine. However, it is possible to write code that could damage your machine. For example by requesting the crane to extended into physical objects. This tutorial does not use any of these features, but you should use caution when running this code on your physical machine and take responsibility for what happens.

## Text Convention

  * When refering to a filename, I will use `filename.cs`. 
  * When talking about a class, variable, or other code objects we will use `ClassName`
  * Shell commands and code blocks will be as described in the next section  

## Code Blocks

The convention for this tutorial for code examples will always follow the same format shown below. There are 4 cases:

  * New files.
  * Complete methods or member variables added to a class.
  * Code blocks inserted / changed into an existing method.
  * PowerShell commands

In all cases, the complete filename relative to the project root directory will be displayed at the top. With this convention, I will often use just the basename instead of full paths in the inline description.

### New File

Here you should be able to create this file and copy and paste the entire content into the file.

{: .filename }
Filename

```csharp
using UnityEngine;
using Multimorphic.P3App.GUI;

namespace Gammagoat.BWT.GUI
{

    public class SomeClass : P3Aware
    {
       
        public string SomeMember;
        
        public override void Start ()
        {
            base.Start ();
        }

        protected override void CreateEventHandlers()
        {
            base.CreateEventHandlers ();
        }

        // Update is called once per frame
        public override void Update ()
        {
            base.Update ();
        }
    }
}
```

### Adding or Changing Members

These types of edits happen at the scope of the class associated with the file (except optional new `using` declarations). These changes should be made within that scope.

{: .filename }
Filename

```csharp
// May contain new using statements.
using Some.New.Scope;

        // May contain member variables scoped to the main class.
        public string SomeMember;
        
        // May coontain functions scoped to the main class.
        public void SomeMethod ()
        {
        }

```

### Adding or Changing code in a Method

Sometimes we will insert code blocks or edit code blocks in existing methods without reproducing all the code around it. The scope of the change will be shown in a comment at the top of the block. Find that method and copy or perform the edit described.

{: .filename }
Filename

```csharp
// Gammagoat.BWT.Modes.SomeMode.SomeMethod

        for (int i=0; i< 10; i++)
        {
            // do something
        }
```

### PowerShell Commands

I will ocastionally show some commands to run. They are intended to be run from the root project directory. I have not at all been careful about my toolchain, and you might not be able to run these commands because you are missing things I have installed on my machine. I should probably create a docker container for the toolchain and then we could all be consistent, but that isn't going to happen.

{: .filename }
Powershell

```powershell
echo "Hello World"
```