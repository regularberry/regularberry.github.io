*[This post originally appeared on Medium](https://medium.com/livefront/alarming-technique-for-letting-your-user-rearrange-objects-3072b7edbc9f)*.

This is a common gesture you will recognize from the OSX dock and many other places…

![](/assets/images/alarming-technique-for-letting-your-user-rearrange-objects/1gaJSxYgLiLM9cPT1Li-uRw.gif)

I’ve had first hand experience implementing this in my app [Algebra Touch](http://www.algebratouch.com):

![Rearranging parts of an expression in Algebra Touch](/assets/images/alarming-technique-for-letting-your-user-rearrange-objects/1KycWRqxbIakTV9_9K5W1Xw.gif)

*Side note: if you’re using a UICollectionView, forget this article and use[ Apple’s built-in API](https://hackernoon.com/swift-reorder-cells-in-uicollectionview-using-drag-drop-ff7eb5131052). For everyone else, please continue.*

The key to my technique is something I call “**Proximity Alarms**” — invisible rectangles that trigger an associated action.

![Proximity Alarms](/assets/images/alarming-technique-for-letting-your-user-rearrange-objects/1rdWO5BAMb-sRgpbcdCKlKw.png)

## Using Proximity Alarms to direct model updates

1. When an object is picked up, calculate the size and position of all proximity alarms.

1. When the user triggers an alarm, execute the associated action that manipulates the underlying model. (e.g. “3 + x + 7" becomes “x + 3 + 7”)

1. Tell all objects to animate to their new home (except the one being held by the user).

1. Re-calculate all proximity alarms.

Here’s an example of the alarms generated when the user picks up the ’3’.

I experimented a lot with their placement and found that putting them right on the other objects felt the best.

Here’s an example of alarms getting recalculated on-the-move:

![Recalculating proximity alarms](/assets/images/alarming-technique-for-letting-your-user-rearrange-objects/1amrfh4Exlj_UjE18baUNTg.gif)

What about those skinny alarms on either end? This is probably only relevant to my app, but I also require the object to be going in the correct direction before setting off the alarm (e.g. to the right or left) This becomes important for complex equations that are dense with objects. The skinny ones are for when the user comes at the expression from the side but the opposite direction.

![Coming at the expression from the side.](/assets/images/alarming-technique-for-letting-your-user-rearrange-objects/176PW2N3Jc_mvFDALzlX7Zw.gif)

That’s all there is to it! Obviously you have to architect your app in a way that cleanly handles model and view state updates, but that’s beyond the scope of this article. ([*Here’s a meditation on that by Collin Flynn*](https://medium.com/livefront/stop-putting-state-in-your-view-models-aa27d0a68b39).)

Got any article ideas you’d like to see? Hit me up [@regularberry](http://twitter.com/regularberry) or [seanb@livefront.com](mailto:seanb@livefront.com)

*Sean never sets off the proximity alarm at [Livefront](http://www.livefront.com).*
