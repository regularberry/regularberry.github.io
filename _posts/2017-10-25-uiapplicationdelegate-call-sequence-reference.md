*[This post originally appeared on Medium](https://medium.com/livefront/uiapplicationdelegate-call-sequence-reference-65a5fe1ee634)*.

In the course of making iOS apps you can’t avoid dealing with the UIApplicationDelegate. It’s where you load your root view controller. It’s where you process incoming links and handle push notifications. It’s where all of your 3rd party libraries compete for the privilege of being initialized first.

I’ve spent a lot of time in there trying to accurately track app launches for analytics. This requires an understanding of the exact sequence of all of those method calls. I was hoping for a reference that laid it all out for me. But there wasn’t one. [So I made it](https://github.com/regularberry/AppLaunchSequence).

It’s a simple app that implements some common scenarios and prints out the order in which the `UIApplicationDelegate` methods are called. Feel free to customize it for your own use cases!

Fun facts from my investigation:

* `applicationDidBecomeActive` gets called last which makes it a much better choice than `applicationWillEnterForeground` to record app launches because you’ll have your launch data processed by then. However, `applicationDidBecomeActive` also gets called after a system alert dialogue gets dismissed. So don’t blindly hook into it and say you’re done tracking app launches.

* There are 38 methods in `UIApplicationDelegate`.
37 of them use `_ application: UIApplication` as the first parameter,
 while one uses `_ app: UIApplication`. That doesn’t irritate me at all.

* `applicationSupportedInterfaceOrientationsFor` gets called 7 times during a fresh app launch.

Check out my [UIApplicationDelegate launch sequence reference](https://github.com/regularberry/AppLaunchSequence) on GitHub, and please send me PRs with additional scenarios!

*-Sean writes documentation for UIKit at [Livefront](http://www.livefront.com).*
