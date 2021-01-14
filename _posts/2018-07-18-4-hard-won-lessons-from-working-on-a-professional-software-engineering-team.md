*[This post originally appeared on Medium](https://medium.com/livefront/4-hard-won-lessons-from-working-on-a-professional-software-engineering-team-26cc9adaeee5)*.

![](/assets/images/4-hard-won-lessons/1fjaMhp5bTWUk-HCOBivtOQ.jpeg)

I’ve been a software developer for a long time (15 years), but only in the past few have I been in a professional collaborative engineering environment. Before that I was a solo developer working on my own projects. When I was solo, I became great at making the computer do what I wanted, but I didn’t have to deal with the complexities (nor gain the benefits) that come from being on a team. In this article I will highlight some of the biggest mindset shifts I’ve experienced and values I’ve come to embrace over the past year.

## Code Reviews

After joining a professional team I initially felt dumb, like dang, they keep finding things to call out in my code. I need to not mess up next time. They would hit me with questions I wasn’t prepared for.

Turns out, this is actually fantastic. They’re trying to poke at your code to see how strong it is. The dialogue will make the code better, either by forcing you to make appropriate changes or by surfacing its limitations (which may be acceptable).

I like to think of it as throwing my code into a gladiator arena. It then gets challenged by powerful beings. People with a fresh set of eyes can imagine all sorts of horrible circumstances that will defeat my code. Possibilities that I was blind to. Now I try to beat up my code too. It’s fun.

However, this can be frustrating if the standards of the project aren’t set or aren’t consistent. Do we value unit testing? How do we feel about code comments? If you don’t know which standards will apply to your PR, how can you effectively pre-evaluate your code? Sometimes it will be knocked for the very same reasons it passed the previous time. You might find that there aren’t standards set for your project, or that they’re sporadically applied.

I’m sorry if that’s your team, but it’s not the fault of code reviews. That’s a project leadership and communication problem, which can infect all aspects of software development. Code reviews, when done right, strengthen your code by infusing it with everyone’s expertise.

*(Note: A lot of time can be wasted on code style. I’ve settled on the idea that if you want to enforce a code style rule, you should be able to express that in a [linter](https://github.com/realm/SwiftLint). If you can’t, let it go.)*

## Act Now

During the course of my day, I notice little things I don’t 100% understand that aren’t relevant to what I’m working on. Like hey, why did that value appear like that? Or hey, why is this named `NonyaBeeswax`?

There’s also little things I could clean up that I don’t *need* to right now, but it sure would be nice in the future. Let’s add another unit test here, or extract that functionality there. but since they’re not part of my ticket I put them on the ‘[back burner](https://goo.gl/images/8orDxV)’.

Guess what my friend. Don’t bet on some time in the future for cleanup. When I worked by myself I could prioritize such days, but you sacrifice your ability to set priorities when working on a team. There are deadlines and constraints outside of your control.

I found it’s much better to be a little slower and call out any peculiarities of my environment when I first notice them. Worst case scenario it’s nothing and I got to learn something about the codebase. Best case scenario I spotted something that could have caused a lot of damage in the future. Either way it’s a win.

There have been multiple times where I noticed something I didn’t understand (that wasn’t relevant to what I was working on), and said to myself “it’s probably for a good reason” — and then later it was revealed that no, it wasn’t done for a good reason. There was a miscommunication about requirements or how things actually work and someone made a guess. It was only a potential problem back then, but an actual problem now. They’re like little pockets of chaos slowly growing over time.

Don’t assume anyone else notices it either. If you deal with things right when you first see them, your project will be much healthier and you’ll have saved time in the long run . Think of it as an investment. It’s ok to question things and ‘appear dumb’. In fact, people will respect you for having the integrity to admit when you don’t know something, and they’ll trust you more when you say you do know something.

## Unit Tests

I already knew how to write a unit test, I just didn’t appreciate how valuable they were until I was on a team. Now I love them. They grant so much peace of mind.

There are inherently a number of assumptions built into your code. It’s not reasonable to expect everyone to take the time to suss them all out. Other developers will need to make changes. To do so, they’re going to make reasonable assumptions and jump in.

Unit tests declare the assumptions about your code *explicitly*. More importantly, they make a lot of red blocks appear in Xcode when they fail (guaranteed to catch a developer’s attention, as opposed to the yellow variety).

It can be awkward at first to adjust to writing unit tests. You’ll end up restructuring your code to make it testable, but trust me, it’s worth it. You’ll even help your future self, because not even you can remember all of the assumptions you made in the past.

Software is made to be changed. It will change. That’s ok. But declare how you expect it to work right now, and then when you’re changing things, declare your changes.

*(Note: if you want to get on the Unit Test hype train I recommend reading [Jon Reid’s blog](https://qualitycoding.org/).)*

## Defensive Coding

This took the longest for me to appreciate. When you’re designing code, think of the most messed up way it could be abused. Not, like, intentionally, but innocently. Some of my more cynical colleagues assume incompetence or worse, but I like to assume people have limited time and jump to the first reasonable conclusion they come across. No one wants to question everything all the time.

When people add to your code, they’re going to copy whatever pattern you’ve established and go on with their day. Be sure it’s a pattern that leads to good code.

One time I had a protocol with a bunch of methods, and to save time I gave each a default implementation (that didn’t do much). However, months later another developer came in and changed a parameter for one of them. Do you see the horrible potential here?

He saw my pattern of having default implementations, added his changed method to the defaults (and to the class he cared about), and went on his way. Everything compiled fine, but we lost some functionality because the various other classes weren’t updated **because they didn’t need to be**. This was preventable had I not given myself a shortcut that could be abused. That was my bad.

By accepting this, you’ll have to impose some limits on what you can do in your code for the sake of defensiveness. I believe those limits are worth it. Rather than adding some documentation saying “*only use it this way or else!*” — why not build those limits directly into your code?

It’s much easier to change your actions than enforce restrictions on everyone else.

## At the end of the day it’s great

I had no idea how much I’d be able to learn by getting access to brilliant developers. They have so much knowledge built up from their unique struggles and successes.

As a kid I had always heard horror stories about working for the man and how it should be avoided. I guess it depends on who you’re working with.

*Sean works as an iOS developer at [Livefront](http://www.livefront.com).*
