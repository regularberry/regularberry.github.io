*[This post originally appeared on Medium](https://medium.com/livefront/why-isnt-viewwillappear-getting-called-d02417b00396)*.

![](/assets/images/viewwillappear/1ruVxN4nS2ijZeVA0U1WPog.jpeg)

Why were you expecting it to be?

Most likely — you want to know that your user is about to see a screen.

There are all sorts of useful reasons to want that. Maybe you want to track an analytics screen view, or trigger a data fetch to keep your screen fresh. Whatever your aim, someone probably told you to check out `viewWillAppear` — Well, bad news. It’ll work for those goals, but only ***sometimes***.***

## The Simple Answer

The [technical reason](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621510-viewwillappear) for when `viewWillAppear` gets called is simple.
> Notifies the view controller that its view is about to be added to a view hierarchy.

It can’t be any view hierarchy — it has to be the one with a`UIWindow` at the root (not necessarily the visible window).

This is fine if you’re jumping from tab to tab or pushing and popping from a `UINavigationController`. iOS is removing your view from the view hierarchy and inserting it back in each time. The view remains in memory during all this so `viewDidLoad` won’t be called repeatedly.

However, you’ll get into trouble when you start presenting views over your current screen, because ***sometimes**** it will remove the underlying view, but ***sometimes**** it won’t.

## Presenting your View Controller

It all depends on how you’re presenting your modal view. iOS prefers to remove the underlying view if it can (memory is precious). But if your designers insist on keeping that underlying view around, you can ask iOS to keep it by setting the correct `.modalPresentationStyle`.

Here are you choices and whether or not it will trigger `viewWillAppear` on its parent view when dismissed:

{% highlight csv %}
UIModalPresentationStyle, iPhone, iPad
.fullScreen, YES, YES
.pageSheet, YES, NO
.formSheet, YES, NO
.currentContext, YES, YES
.custom, NO, NO
.overFullScreen, NO, NO
.overCurrentContext, NO, NO
.blurOverFullScreen, only on tvOS - N/A, N/A
.popover, YES, NO
.none, CRASH, CRASH
{% endhighlight %}

The [documentation](https://gist.github.com/regularberry/da8bc937fe5b7204ade75d64d94933f6) for `.none` says…
> “Do not use this style to present a view controller.”

They enforce this by crashing. Thanks Apple for keeping us on our toes!

## Backup Plan

When the view below remains, `viewWillAppear` will NOT be called when the overlaying view gets dismissed. To do your business, you’ll have to hook into the completion handler in your `dismiss` call.

{% highlight swift %}
dismiss(animated: true, completion: {
    referenceToUnderlyingViewController.doTheBusiness()
})
{% endhighlight %}

What’s the best way to pass a reference to presented view controller? That really depends on how you set up your app and is beyond the scope of this article.

## The Missing Link

When we add the view to a window’s view hierarchy, how does iOS know which `UIViewController` to notify? From the [`UIViewController` docs](https://developer.apple.com/documentation/uikit/uiviewcontroller), we can see that it has a reference to the [`view`](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621460-view) it owns, but nothing in the [ UIView  ](https://developer.apple.com/documentation/uikit/uiview) docs indicates it knows about which `UIViewController` owns it.

There’s nothing special happening in these `present` methods either — the `viewWillAppear` call happens even when you directly add the view.

Just because Apple doesn’t tell us what’s happening doesn’t mean we can’t make some educated guesses. Let’s get sneaky and [inspect the private APIs](http://developer.limneos.net/index.php?ios=12.1&framework=UIKitCore.framework&header=UIView.h)…

Look at all of those extra properties! And there’s only one that has the type we’re looking for… `UIViewController* _viewDelegate;`

Let’s test it out by creating 2 view controllers. We’ll inject one into the `_viewDelegate` of the other’s `view`, and when that `view` is added to the window’s view hierarchy, the injected view controller’s `viewWillAppear` should get called.

Try it out yourself!

{% highlight swift %}
import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    class InjectedViewController: UIViewController {
        override func viewWillAppear(_ animated: Bool) {
            super.viewWillAppear(animated)
            print("INJECTED VIEW WILL APPEAR")
        }
    }

    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        let injected = InjectedViewController()
        let simple = UIViewController()
        simple.view.setValue(injected, forKeyPath: "_viewDelegate")
        window?.addSubview(simple.view)
        return true
    }
}

{% endhighlight %}

**WARNING** — Do not ship an app using private APIs. This is a dangerous way to code. These properties are private and Apple reserves the right to change their underlying implementation at any time without telling you.

*Sean is living his best iOS life at [Livefront](http://www.livefront.com).*

****sometimes** is a very irritating word to a software developer*
