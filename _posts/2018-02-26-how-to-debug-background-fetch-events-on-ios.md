*[This post originally appeared on Medium](https://medium.com/livefront/how-to-debug-background-fetch-events-on-ios-29540b043adf)*.

Registering your app for background fetch enables you to do all sorts of things for your users… like update your icon’s badge count! But how are you supposed to test it? The fetch happens at the discretion of the operating system…

I’m going to show you how to debug your background fetch solution and the pitfalls you’ll encounter.

First, set a breakpoint in `performFetchWithCompletionHandler` so we can validate our test.

![](/assets/images/how-to-debug-background-fetch-events-on-ios/1-Lj75KXY4LCBXR3L4uFC7g.png)

Next, trigger a fetch from Xcode’s Debug menu:

![](/assets/images/how-to-debug-background-fetch-events-on-ios/1IWX6L7JbNYNsXSBR9dkoGw.png)

That should work fine, but it’s not the most helpful scenario. You could have simply invoked your `performFetchWithCompletionHandler` method directly if you wanted to debug like that. The tricky part is recreating the scenario in which your app isn’t loaded at all and iOS wakes it up to execute in the background.

This is where we hit [#xcodeproblems](https://twitter.com/search?q=xcodeproblems&src=typd). Edit your app target’s scheme and you’ll see an option to simulate launching from a background fetch event.

![Launch due to a background fetch event](/assets/images/how-to-debug-background-fetch-events-on-ios/1j_Qy3trAQBocXycmc2jdqw.png)

Try this in the simulator.

![Launching…](/assets/images/how-to-debug-background-fetch-events-on-ios/1IxRYGNYMIdnw6JDxUgI7kw.png)

You’ll be stuck waiting forever with no error. Apparently, this used to work in the simulator. But no longer. Fine. Let’s try the device instead.

![Your phone has denied the launch request.](/assets/images/how-to-debug-background-fetch-events-on-ios/1tkfcJGKu4BkdQCv7NlMlWw.png)

You might think that this error is meaningful. Maybe you’ll need to go into the Device Management settings on your phone to verify you’ve trusted your developer certificate. I recommend doing that in general, but it won’t help here. From what I can tell this is a generic “timed-out” error.

What gives? Well, [it’s a bug](https://forums.developer.apple.com/thread/92241#279414). After iOS 11.0.3 this functionality simply broke. If you keep your device up to date (11.2.6 as of this writing) you won’t be able to test background fetches on your device. So now what? (Edit: as of iOS 11.3 and Xcode 9.3 this is now working! Thanks [Marco Mussini](https://medium.com/@ggould75?source=post_header_lockup))

Hopefully you have an older test device to use until Apple fixes this. If you’re in the Twin Cities, you can stop by [Livefront](http://www.livefront.com) and use our iPhone 7 Plus running 11.0.3 (register at [FinalPass](http://finalpass.com/) — the first hour is free!).

Let this be a warning — always have a spare test device with an older version of iOS lying around!

*Sean has a strictly professional relationship with Xcode at [Livefront](http://www.livefront.com).*

p.s. If you’re stuck with no device but want to verify the launch sequence, I’ve updated my [AppLaunchSequence](https://github.com/regularberry/AppLaunchSequence) documentation to include background fetch.
