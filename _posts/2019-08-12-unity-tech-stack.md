I’m coming to Unity after working in native iOS. My goal is to get my head around Unity’s tech stack  with links to reference documentation. Apple spoiled me by having everything [in one place](http://developer.apple.com/).

Disclaimer: This information was derived using Unity 2019.2 on a Mac.

You write scripts in C#, currently [version 7.3](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7-3).

Great, C# executes in a runtime. Which .NET runtime does Unity use? I’m on a Mac, so, Mono?

Yep - but it’s their [fork of Mono](https://github.com/Unity-Technologies/mono).

When you build your game, it will embed their version of Mono called `MonoBleedingEdge` and package all of the DLLs you’ve included. Your custom scripts will be packaged into `Assembly-CSharp.dll`. (You can open that DLL in Visual Studio to see your objects!)

Running in a managed runtime will impact performance because those DLLs will have to be loaded when they’re needed.

Another option (which is required for iOS) is to use [IL2CPP](https://docs.unity3d.com/Manual/IL2CPP.html). This will convert your scripts and assemblies to C++ so they will run natively - no managed runtime. This is better for performance! However, the build time for this is significantly longer. (Try it yourself, seriously)

Also, word of warning. While you get to use Visual Studio, it will ship with its own C# compiler. This is not the one [Unity is using in its Editor](https://github.com/dotnet/roslyn). So if a script works inside Visual Studio it might not necessarily work in the Editor (it probably will).

You also get to choose which API level to target: [.NET Standard 2.0](https://docs.microsoft.com/en-us/dotnet/api/?view=netstandard-2.0), or [.NET 4.x (currently 4.6)](https://docs.microsoft.com/en-us/dotnet/api/?view=netframework-4.6) Unity says that [they support .NET Standard 2.0 for all platforms](https://blogs.unity3d.com/2018/03/28/updated-scripting-runtime-in-unity-2018-1-what-does-the-future-hold/) but not necessarily .NET 4.x.

That’s the gist. Good luck!

p.s. [Source for current C#/compiler/.NET support versions](https://docs.unity3d.com/Manual/CSharpCompiler.html)
