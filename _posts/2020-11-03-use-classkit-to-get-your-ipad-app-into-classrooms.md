*[This post originally appeared on Medium](https://medium.com/livefront/how-to-get-featured-on-the-app-store-by-making-apple-look-good-6a200b5d1ec7)*.

There’s an unprecedented demand for digital education right now, and Apple has put millions of iPads into classrooms.

As an iOS developer, you can get your app into this huge platform by creating activities for [Schoolwork](https://apps.apple.com/us/app/schoolwork/id1355112526) — Apple’s assignment-management app.

The basic workflow is:

* The Teacher browses activities and selects one to assign to their students.

* Students receive the activity, and tap a button to launch them into the corresponding section of your app.

* Students complete said activity, which gets reported back to the Teacher.

* The Teacher reviews metrics and marks it as done.

In this article, I’m going to show you how to:

* Set up a test environment.

* Create a simple activity — define it, start it, and report on it.

* Tell ClassKit about your activities, so it can display them to the Teacher to choose from.

## Set up a test environment

Grab [Schoolwork](https://apps.apple.com/us/app/schoolwork/id1355112526) off the App Store. It’s iPad only. [Here’s a detailed intro](https://support.apple.com/guide/schoolwork-teacher/intro-to-schoolwork-phxe0979e998/ios).

When you open it, you’ll see a message that says:
> “*To get started, contact your school administrator or make sure you’re signed in with your school account.*”

Schoolwork is made for classrooms which use [Managed Apple IDs](https://www.apple.com/education/docs/overview_of_managed_appleid.pdf). These are special Apple IDs that help schools manage their set of iPads for different classrooms. They come with restrictions like the inability to download from the App Store, and benefits like bonus iCloud storage and [Shared iPad](https://developer.apple.com/education/shared-ipad/).

Let’s assume you don’t have access to that.

If we make our iPad a development device (by installing an app from Xcode), we get access to a ‘Developer’ set of options in Settings.app. There’s a ClassKit section in there that will allow us to emulate being a Teacher or a Student.

![Path to emulate roles in Schoolwork](/assets/images/classkit/first.png)

![Choose which role you want to emulate in Settings.app](/assets/images/classkit/second.png)

Try being a teacher and open up Schoolwork again. You’ll be able to create a Handout and assign activities to the ‘Dev Class’. If you don’t have any apps supporting Schoolwork activities, you’ll still see the ability to assign students to an app. Just without any specific activity for it.

Now go back to Settings.app and become a student. You’ll see your assignment appear in Schoolwork. Try marking it complete, and then become a teacher again to approve! Wow! You’re a great teacher/student!

Now let’s make our own…

## Create a simple activity

Before you do anything else, add the ClassKit capability in Xcode.

![Add the ClassKit capability in Xcode](/assets/images/classkit/third.png)

It’s real simple but real important. Also, ClassKit doesn’t work in the simulator, so you’ll need an actual iPad.

### Types of Activities

Your first job as an education developer is to figure out what activities your app supports. Is it a game? A reading? A quiz? [Here’s a list of types](https://developer.apple.com/documentation/classkit/clscontexttype).

You also need to decide what metrics to report. These are called [CLSActivityItems](https://developer.apple.com/documentation/classkit/clsactivityitem), and you can choose between [Quantity](https://developer.apple.com/documentation/classkit/clsquantityitem), [Binary](https://developer.apple.com/documentation/classkit/clsbinaryitem), or [Score](https://developer.apple.com/documentation/classkit/clsscoreitem).

Let’s keep it simple. Say we’re making an educational app called “Edu Jam” with 2 mini games: “Math Jam”, and “Word Jam” — and we’ll report the score for both.

Our CLSContextType will be .game, the CLSContextTopic will be either .math or .literacyAndWriting, and our CLSActivity will be CLSQuantityItem.

Why not CLSScoreItem? Because that one requires a maxScore. That makes it more appropriate for quizzes rather than high-score games.

### Activity Hierarchy

You need to define the hierarchical structure of your activities. Schoolwork displays it in their app for teachers to browse (this usually mirrors your app’s navigational hierarchy).

Each activity gets an identifier which is used for the construction of an identifierPath.

Our app is simple, so our activities will be direct children of the root context. The root context is called the mainAppContext and is automatically created by ClassKit. Its identifier is your app’s bundle id.

Let’s say our “Math Jam” game has an identifier of “MATH-JAM”. Its identifierPath would be:

**[“com.seanberry.edujam”, “MATH-JAM”]**

For more sophisticated hierarchies, [read up on Apple’s explanation](https://developer.apple.com/documentation/classkit/advertising_your_app_s_assignable_content).

### Creating CLSContexts

Apple needs to communicate from your app back to Schoolwork, which (in the normal use case) is on a different iPad. That means a network is in play.

Use the CLSDataStore.shared instance to create and fetch activities by theiridentifierPath. You need to provide a delegate, just like UICollectionViewDataStore, which is responsible for creating the CLSContext if it hasn’t been cached yet.

Here’s an example implementation of CLSDataStoreDelegate:

{% highlight swift %}
import ClassKit

class ActivityCreator: NSObject, CLSDataStoreDelegate {
    static let shared = ActivityCreator()

    override init() {
        super.init()
        CLSDataStore.shared.delegate = self
    }

    func createContext(forIdentifier identifier: String, parentContext: CLSContext, parentIdentifierPath: [String]) -> CLSContext? {

        if identifier == "MATH-JAM" {
            let context = CLSContext(type: .game, identifier: "MATH-JAM", title: "Math Jam")
            context.topic = .math
            return context
        } else if identifier == "WORD-JAM" {
            let context = CLSContext(type: .game, identifier: "WORD-JAM", title: "Word Jam")
            context.topic = .literacyAndWriting
            return context
        } else {
            return nil
        }
    }
}
{% endhighlight %}

In practice, your activities will most likely correspond to types that already exist in your app. [Check out Apple’s sample app for a way to extend your existing types to support ClassKit types.](https://developer.apple.com/documentation/classkit/incorporating_classkit_into_an_educational_app)

### Starting an Activity

The student will tap a button to open your app directly to the assigned activity.

We have two options: [NSUserActivity](https://developer.apple.com/documentation/foundation/nsuseractivity) or [Universal Links](https://developer.apple.com/ios/universal-links/).

If your app already supports Universal Links it’s a no brainer to piggyback onto them. All you’ll need to do is fill out the context’s.universalLinkURL property. [See more info here](https://developer.apple.com/documentation/classkit/clscontext/2953041-universallinkurl).

For NSUserActivity, you check if it’s a isClassKitDeepLink and grab the context’s identifierPath.

Here’s a quick example:

{% highlight swift %}
func application(_ application: UIApplication,
                   continue userActivity: NSUserActivity,
                   restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
  guard
      userActivity.isClassKitDeepLink,
      let identifierPath = userActivity.contextIdentifierPath,
      let gameID = identifierPath.last
      else { return false }

  let gameVC = GameViewController(gameID: gameID)
  navigationController.pushViewController(gameVC, animated: true)
  return true
}
{% endhighlight %}

Now that we’re in the correct part of the app, we tell ClassKit we’re starting the activity in that View Controller’s viewWillAppear.

First, fetch the context from CLSDataStore.shared (make sure you’ve set up its delegate first!) Then, check if an activity already exists on that context, and resume it if so. If not, create a new one. Finally, save. This will tell ClassKit to propagate what you’re doing out into the network.

{% highlight swift %}
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)

    CLSDataStore.shared.mainAppContext.descendant(matchingIdentifierPath: [game.identifier]) { context, _ in

        context?.becomeActive()
        if let activity = context?.currentActivity {
            activity.start()
        } else {
            context?.createNewActivity().start()
        }

        CLSDataStore.shared.save()
    }
}
{% endhighlight %}

Likewise, let’s stop the activity in viewWillDisappear.

{% highlight swift %}
override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)

    CLSDataStore.shared.mainAppContext.descendant(matchingIdentifierPath: [game.identifier]) { context, _ in

        guard let activity = context?.currentActivity else { return }

        activity.stop()
        context?.resignActive()

        CLSDataStore.shared.save()
    }
}
{% endhighlight %}

### Reporting Metrics

Now let’s report the score. Each activity has a primaryActivityItem and anadditionalActivityItems array we can add to.

Note: You can only add one type of each ActivityItem to the additionalActivityItems array. Subsequent additions of the same type will overwrite previous ones.

As always, we’re fetching the context from CLSDataStore.shared and saving afterwards. Here’s an example:

{% highlight swift %}
let quantity = CLSQuantityItem(identifier: "Score", title: "Score")
quantity.quantity = Double(score)

CLSDataStore.shared.mainAppContext.descendant(matchingIdentifierPath: identifierPath) { context, _ in
    guard let activity = context?.currentActivity else { return }
    activity.primaryActivityItem = quantity
    CLSDataStore.shared.save()
}
{% endhighlight %}

## Tell ClassKit about your activities

This has changed significantly in the new version of Schoolwork (2.1).

Previously, Schoolwork would display all the activities your app had created in the CLSDataStore. So for a teacher to see a list in Schoolwork, they would have to open your app first, and your app would have to create all of your activities on startup.

This is not ideal. Teachers get a bunch of apps installed on their iPads by administrators, and might not take the time to explore your app before using Schoolwork.

Schoolwork 2.1 will access the new [ClassKit Catalog API](https://developer.apple.com/documentation/classkitcatalogapi) — a web service run by Apple. You can declare and edit the activities your app supports, including keywords for easier discovery, all outside of your app’s code. This is the future and is something you should support.

I’ve also written a [quick-start guide on how to use the ClassKit Catalog API](https://medium.com/p/457cab29516d/edit).

If you want to test your implementation the old way, fetch each activity on app launch, and then open the app as a teacher. This will add each activity to the CLSDataStore and make them visible in Schoolwork. [See an example implementation from Apple](https://developer.apple.com/documentation/classkit/advertising_your_app_s_assignable_content/declaring_your_app_s_context_hierarchy).

**Bonus**: I’ve uploaded all of the above code into [a sample app on GitHub](https://github.com/regularberry/ClassKitExample) for you to tinker with.

## Conclusion

By implementing ClassKit, you’ll open your app to a large audience of teachers and students. They’re transitioning to a distance learning environment, so any additional resources will be welcomed! Help them out today.

*If you want to see my professional implementation, check out [Algebra Touch](http://www.algebratouch.com).*
