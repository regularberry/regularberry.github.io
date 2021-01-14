*[This post originally appeared on Medium](https://medium.com/livefront/4-tips-for-using-carthage-38604f257b54)*.

I’ve been using Carthage as my dependency manager for about 6 months now, and here are some tips I’ve learned the hard way.

## #1 The Repository Name

As shown in the [Carthage setup guide](https://github.com/Carthage/Carthage#upgrading-frameworks), to update a particular repository out of a group use the following command:

{% highlight powershell %}
carthage update <repository name> --platform ios
{% endhighlight %}


Great, no problem — but what’s the format of that repository name? Check out this example `Cartfile`. Can you guess the correct repository name for each dependency below?

{% highlight powershell %}
github "swift/Sugar" "e081f48892c1234dcgujk5a61892e5088fc544ff"
github "airbnb/lottie-ios" == 1.5.2
github "moretap/PrettyBorders" "master"
git "[git@git.nasa.com](mailto:git@git.comp.com):SeanBerry/launch-ios.git" == 1.2.3
{% endhighlight %}


The correct format is the last part of the filename, excluding the ‘.git’ suffix. So for the above it’s `Sugar`, `lottie-ios`, `PrettyBorders` and `launch-ios`, respectively.

## #2 Clear the Cache

If you’re working on your own SDK, you’re probably pushing experimental changes to experimental branches:

{% highlight powershell %}
git "git@git.company.com/test.git" "86753098675309abcdefg"
{% endhighlight %}


Sometimes Carthage will come at you with an error that it can’t find your commit. What happened? As a senior developer, when something goes wrong my first instinct is to blame myself and question all of my assumptions. In this case, that’s a great way to waste a couple hours. Turns out the cache sometimes doesn’t update [(as reported back in 2015)](https://github.com/Carthage/Carthage/issues/443). Clear it and Carthage will be forced to rebuild its knowledge of your dependency:

{% highlight powershell %}
rm -rf ~/Library/Caches/org.carthage.CarthageKit
{% endhighlight %}


## #3 Xcode Command Line Version

When using Carthage with source control, you have to decide whether to check in your `Build` folder or not. There’s no correct answer. If you check it in, people using your project can pull and run your app without running `carthage bootstrap` — but you’ll be adding to the size of your repo. (Github’s [default .gitignore for Swift](https://github.com/github/gitignore/blob/master/Swift.gitignore) includes Carthage/Build.)

Say you check it in because you’re using CI and your build server needs those files. A few weeks later, Apple releases a new version of Xcode and Swift. But the coworker in charge of the SDK hasn’t updated yet! When you try to use their dependency, you’ll see an error in Xcode like:

{% highlight powershell %}
Module compiled with Swift 4.0 cannot be imported in Swift 4.0.2
{% endhighlight %}


The version of Swift that your Xcode.app is using might be different than the version the Xcode command line tools are using. To check what version Carthage is using to build your dependencies, go into Xcode-&gt;Preferences-&gt;Locations

![](/assets/images/4-tips-for-using-carthage/1MPP9T4swJAnkaFLuYj4llg.png)

Make sure that’s set to the correct version, then run:

{% highlight powershell %}
carthage update coworker-repo --platform ios
{% endhighlight %}

The dependency will be good to go (assuming the new version of Swift didn’t break it of course).

## #4 Specify a particular release or commit

If you don’t check in your /Build folder and ask your colleagues to install the dependencies through Carthage, they might use `carthage update` or `carthage bootstrap` Sure, they could look at the documentation for your project and follow your instructions there, but let’s say someone decides to jump right in.

`update` will look at your `Cartfile` and fetch the latest version of everything that fulfills the spec, e.g. if you specify `master` it will go and see what the new master commit is and write it to `Cartfile.resolved.`

`bootstrap` will simply look at `Cartfile.resolved` and fetch those particular commits.

This can be a headache if everyone isn’t on the same page. To side-step this entirely, mark a particular release or a commit, not a branch.

{% highlight powershell %}
github "Quick/Nimble" == [7.0.3](https://github.com/Quick/Nimble/releases/tag/v7.0.3)   # not "master"
{% endhighlight %}


**Warning**: You will miss out on incremental updates by not using `~&gt;` in your `Cartfile`. Ultimately you have to figure out what works best for your project.

*Sean works at [Livefront](http://www.livefront.com) and uses Carthage almost every day.*
