*[This post originally appeared on Medium](https://medium.com/livefront/uploading-data-in-the-background-in-ios-f93722013c6a)*.

![](/assets/images/code-liability/1TeSlq04CtYDQguAA-b_0Cw.jpeg)
Credit: www.instagram.com/miss_caitlin_rose (NSFW)

My journey towards creating code looks like this:

* Research the details of the goal (Learn)

* Come up with a plan to implement (Create)

* Code (Clicking and typing)

* Return to step 1 when I realize I had a false assumption (Regret)

Naming the types correctly is the final obstacle during planning before I can get into the (fun 🤓) labor of coding. It’s like, ugh, I know the computer doesn’t care what I name my type. It’s all going to be stripped away by the time the machine is interpreting the instructions. My functionality won’t be affected. Why am I spending as much time naming as I did figuring out roles and how things fit together?

Because code isn’t just for the machine. It’s for developers who will have to understand it well enough to manipulate it correctly. “Future you” is included in that category.

To do this right requires clarity about your problem set, and enough empathy for your audience to anticipate all of the implications of your chosen name.

## A True Name

There’s this old idea that if you know something’s “[true name](https://en.wikipedia.org/wiki/True_name)” then you have power over it. It’s a [common trope in fantasy](https://tvtropes.org/pmwiki/pmwiki.php/Main/IKnowYourTrueName) — and part of religious stories.

Take the [story of Isis and Ra](https://www.ancientegyptonline.co.uk/isisra.html). Isis was a great healer, but could only help Ra if she knew his real name. He tried to satisfy her request with ‘lesser’ names, but that only caused Ra to not get at the core issue. Once he gave up his true name, she was able to cure him.

That sounds like debugging to me. You can’t fix the problem if you don’t know the true meaning of the things you’re dealing with. Lazy labels will be deceiving. The more mental gymnastics you’re making people do while reading your code the more cognitively draining it will be to work with.

*“Yes that’s called a `Provider` but it actually just transforms data.”*

*“That `modelId` isn’t for this model, it’s for the other generic model.”*

But if you truly know what something is you can manipulate it effortlessly. That’s important because code will have to be changed in the future. My ideal is absolute oneness between functionality and the label representing it. Since it’s an ideal, I never get there. There is such a thing as diminishing returns while thinking up names, but it’s worth putting in some time.

## Balancing Clarity with Readability

The most precise name for something might be…

{% highlight swift %}
struct SwiftStructForiOSThatContainsUserAgeData {
  let age: Int
}
{% endhighlight %}

That name blatantly ignores all available context. Obviously this example is ridiculous, but it can be tempting to add in unnecessary qualifiers.

Even if you feel the name isn’t the best it could be, it’s better to be internally consistent within your project rather than waffle back and forth between two conventions. Once people are tuned into your project’s standards, they’ll know what you’re talking about. A consistent message is better than a conflicted one, even if it’s ultimately subpar.

Also, don’t be afraid to rename old elements when new elements are introduced, as a way to clarify their differences. We’re not coding in stone tablets. Some people have a dream of future-proofing their code and anticipating future changes. I’d rather make the code cleanly represent the present day situation, and make changes in the future that reflect the future. This saves me a lot of worrying about things outside of my control.

## We Control Both Sides

Maybe you’re not taking the time to accurately name something because it’s doing multiple things…

{% highlight swift %}
protocol UserStoreController {
    func getAllUsers() -> [User]
    func getUser(_ id: String) -> User
    func openProfile(_ user: User)
}
{% endhighlight %}

Sometimes the right answer might be to refactor the underlying implementation to enable a better name. We have control over both. We’re not stuck trying to name something that can’t be changed. We can rip things apart and be blunt.

It’s common in politics to strive to get your chosen label onto an initiative, because all of the implications of your label will now be associated with said initiative. Something similar happens with your name, especially combined with the context of the project.

*“Ah, this was named `chosenAddress` as opposed to other one named `address` — there must be some functionality about selecting an address without changing the official record…”*

*“This enum value is named `notification`, while another is `push`— what’s the difference here? There must be a non-push notification, so is there polling? Maybe — or maybe I don’t trust the developer now and I’m going to dig into the code to see if they’re using these terms correctly.”*

It’s worth the time investment to pause and properly name your types. Code isn’t just for the computer, it’s for developers as well. By naming things properly, you can take advantage of that name’s implications to guide your reader into understanding things correctly.

*Sean never creates inappropriate acronyms in his code at [Livefront](http://www.livefront.com).*
