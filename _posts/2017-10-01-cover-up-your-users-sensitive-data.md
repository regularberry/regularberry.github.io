*[This post originally appeared on Medium](https://medium.com/livefront/cover-up-your-users-sensitive-data-it-s-private-309c0173dffd)*.

As an app developer, I check my sales using Apple’s iTunes Connect. When you background the app it blurs out all of the sales data (pretty cool!)

![*Blurred and blackout techniques for hiding sensitive data.*](/assets/images/cover-up-your-users-sensitive-data-it-s-private/1_f5zfvHWyx3vhtuvywpWfw.gif)

Why do they do that? Well, [Apple recommends](https://developer.apple.com/library/content/qa/qa1838/_index.html) it as a best practice because iOS takes a snapshot of your app and stores it in a cache on the filesystem.

You can see this for yourself by [downloading iExplorer](https://macroplant.com/iexplorer) .

![[.ktx](https://www.khronos.org/opengles/sdk/tools/KTX/file_format_spec/) is a picture format](/assets/images/cover-up-your-users-sensitive-data-it-s-private/1XIJCCwdI5iQkJb_kXNh5Iw.png)

This only works for apps you’ve installed outside of the App Store (e.g. through Xcode). Otherwise it’s hidden unless someone jailbreaks your phone.

And you might say, hey, if someone has my phone and knows how to jailbreak it… them having access to the last visual state of my apps is not going to be high on my list of things to worry about. I get you. But on the other hand, if certain things are preventable it’s still a good idea to prevent them. It’s like deciding whether or not to eat that last cookie even though you ate the rest of the box in one sitting. You still have the choice to not eat that last cookie, and you’ll be better off for it.

In fact, [security firms recommend](https://books.nowsecure.com/secure-mobile-development/en/ios/avoid-cached-application-snapshots.html) hiding sensitive data from iOS’ snapshot feature as part of their audits. You’ll notice all banking apps do it. (See the gif at the top.) Their solution is a bit ham-fisted. Regardless of where you are in the app, they cover everything up with their company’s splash screen.

I don’t blame them though. They probably copied that technique [from Apple’s example](https://developer.apple.com/library/content/qa/qa1838/_index.html), which shows you how to cover everything with a solid black view controller using Objective-C. Let’s do better.

{% highlight swift %}
protocol DisplaysSensitiveData {
    func hideSensitiveData()
    func showSensitiveData() // we make a mess, we clean it up
}
{% endhighlight %}


Throw that protocol on any view controller you want to protect, as well as their container view controllers. E.g. for UINavigationController:

{% highlight swift %}
extension UINavigationController: DisplaysSensitiveData {
    func hideSensitiveData() {
        if let vc = topViewController as? DisplaysSensitiveData {
            vc.hideSensitiveData()
        }
    }

    func showSensitiveData() {
        if let vc = topViewController as? DisplaysSensitiveData {
            vc.showSensitiveData()
        }
    }
}
{% endhighlight %}


Finally, hook into the UIApplicationDelegate to make the appropriate calls when the app gets backgrounded and when it comes back.

{% highlight swift %}
func applicationDidEnterBackground(_ application: UIApplication) {
  if let vc = window?.rootViewController as? DisplaysSensitiveData {
    vc.hideSensitiveData()
  }
}

func applicationWillEnterForeground(_ application: UIApplication) {
  if let vc = window?.rootViewController as? DisplaysSensitiveData {
    vc.showSensitiveData()
  }
}
{% endhighlight %}


For a more sophisticated example, [check out my selfie app](https://github.com/regularberry/selfieapp). Since [faces are now sensitive data](https://www.apple.com/iphone-x/#face-id), I decided to black out your eyes when the app gets backgrounded.

![](/assets/images/cover-up-your-users-sensitive-data-it-s-private/1M9Z6yvSpyWvXg_PmeXVTtQ.gif)

Face sensitivity is no joke! Apple does their blurring technique in the default camera app, check for yourself.

*Sean draws rectangles at [Livefront](http://www.livefront.com).*
