*[This post originally appeared on Medium](https://medium.com/livefront/how-to-add-a-dynamic-swift-framework-to-a-command-line-tool-bab6426d6c31)*.

Let’s walk through how I tried to add a dynamic framework to my Command Line Tool and discuss what went wrong each step of the way.

## Step 1: Add it to the ‘Linked Frameworks and Libraries’ section

![](/assets/images/how-to-add-a-dynamic-swift-framework-to-a-command-line-tool/1_eh-w7XAp3ej_Id9_rVjYA.png)

This is what happens when you run your app:

{% highlight powershell %}
**dyld: Library not loaded: @rpath/libswiftAppKit.dylib**

Referenced from: /Users/seanberry/Library/Developer/Xcode/DerivedData/TestCommandLineTool-fnrmhjvjmugvqueaqvbklzwhqvuv/Build/Products/Debug/ThirdParty.framework/Versions/A/ThirdParty

Reason: image not found
{% endhighlight %}

The `ThirdParty.framework` is trying to find `libswiftAppKit.dylib` (which is part of the Swift standard libraries) in the `@rpath` directory.

We can see how `ThirdParty.framework` defines `@rpath` by running

{% highlight powershell %}
$ oTool -l ThirdParty.framework/Versions/Current/ThirdParty

Load command 27
cmd LC_RPATH
cmdsize 48
path @executable_path/../Frameworks (offset 12)

Load command 28
cmd LC_RPATH
cmdsize 40
path @loader_path/Frameworks (offset 12)
{% endhighlight %}

Well shoot. Those aren’t relevant to our Command Line Tool. We don’t have any folders named /Frameworks or ../Frameworks. Why is it looking for the Swift standard libraries there?

Because it’s built for iOS and Mac apps. Here’s the directory structure inside a Mac app:

![](/assets/images/how-to-add-a-dynamic-swift-framework-to-a-command-line-tool/15CYsM3KN3Y0hgmlVlIHMJw.png)

![](/assets/images/how-to-add-a-dynamic-swift-framework-to-a-command-line-tool/15z5SNlgkJeyWCKUZeK7rFw.png)

That explains `path @executable_path/../Frameworks`

And for iOS, the directory structure inside the app is:

![@loader_path equals @executable_path when the framework is loaded from the executable](/assets/images/how-to-add-a-dynamic-swift-framework-to-a-command-line-tool/1wZOXeUuBQmb30f4Mu9m10A.png)

And that explains `path @loader_path/Frameworks (offset 12)`

But what about us, the humble command line tool developer? The Swift standard libraries are statically linked inside our executable, but our third party frameworks can’t find them. Unfortunately they’re not stored in a standard place on every Mac. Developers can get access to them buried inside `Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/swift/macosx` (but please don’t require your users to install Xcode).

So what do we do? The standard practice I’ve seen is to create a custom framework for your project, import all of your dependencies into it, then copy the swift standard libraries as well.

## Step 2: Create a custom framework and copy the standard Swift libraries

![This will copy/paste the Swift standard libraries inside FirstParty.framework/Versions/Current/Frameworks/](/assets/images/how-to-add-a-dynamic-swift-framework-to-a-command-line-tool/1MsF9fJQ3U9bAQQxf3HpDzw.png)

![Make sure to copy in your Third party dependency as well](/assets/images/how-to-add-a-dynamic-swift-framework-to-a-command-line-tool/1TTZXIeZiD8PGHdMXmY01KQ.png)

![Now your frameworks are all together!](/assets/images/how-to-add-a-dynamic-swift-framework-to-a-command-line-tool/1qMoEdH8-iCRGZICQpJJiog.png)

Perfect! Now let’s go ahead and run our Command Line Tool…

{% highlight powershell %}
**objc[2335]: Class _TtC8Dispatch16DispatchWorkItem is implemented in both /Users/seanberry/Library/Developer/Xcode/DerivedData/TestCommandLineTool-fnrmhjvjmugvqueaqvbklzwhqvuv/Build/Products/Debug/FirstParty.framework/Versions/A/Frameworks/libswiftDispatch.dylib (0x1016c8530) and /Users/seanberry/Library/Developer/Xcode/DerivedData/TestCommandLineTool-fnrmhjvjmugvqueaqvbklzwhqvuv/Build/Products/Debug/TestCommandLineTool (0x1005c7698). One of the two will be used. Which one is undefined.**

REPEAT THE ABOVE ERROR IN 20 DIFFERENT WAYS
{% endhighlight %}


Your app is now seeing two different copies of the Swift standard libraries: the ones statically linked inside your executable, and the ones inside the /Frameworks folder.

There are two solutions from here.

## Step 3 (option A): Make a Mac app instead and extract the executable

I investigated famous command line tools [Carthage](https://github.com/Carthage/Carthage) and [SwiftLint](https://github.com/realm/SwiftLint) to see how they handled this problem. Turns out they’re not set up as command line tools! They’re Mac apps! Why? Because a Mac app doesn’t statically link the standard libraries. They deploy themselves as command line tools by adding a run phase that extracts out the executable from the app package.

![](/assets/images/how-to-add-a-dynamic-swift-framework-to-a-command-line-tool/1_d-qG2cKZP64_Xgcs_r9JA.png)

{% highlight powershell %}
#!/bin/bash
## Extracts the carthage CLI tool from its application bundle. Meant to be run
# as part of an Xcode Run Script build phase.

cp -v "${BUILT_PRODUCTS_DIR}/${EXECUTABLE_PATH}" "${BUILT_PRODUCTS_DIR}/${EXECUTABLE_NAME}"
{% endhighlight %}


You can go ahead and do it that way, no problem. But I found another way around this issue.

## Step 3 (option B): Disable static linking

Add these into your User Defined Build Settings:

{% highlight powershell %}
SWIFT_FORCE_DYNAMIC_LINK_STDLIB YES
SWIFT_FORCE_STATIC_LINK_STDLIB NO
{% endhighlight %}


This will force your executable to dynamically link all libraries. Make sure to tell Xcode where to find them by adding the framework directory to the ‘Runpath Search Paths’

`@executable_path/FirstParty.framework/Versions/Current/Frameworks`

Good luck with your command line tool!

*Sean tries to get Xcode to compile at [Livefront](http://www.livefront.com)*
