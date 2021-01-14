*[This post originally appeared on Medium](https://medium.com/livefront/swift-expected-nil-not-nil-27ac2eb8505d)*.

My unit test failed. It wanted a `nil` but got a `&lt;nil&gt;`. My initial reaction was to accuse my computer of being dumb. Are you expecting a different type of `nil` from me? There’s only one `nil`, and its name is `nil`.

But hold up. Computers aren’t sophisticated enough to prank us. What did I do wrong?

## The Dictionary Problem

{% highlight swift %}
var dict: [String: Any?] = ["first": nil, "second": 2]

if let val = dict["first"] {
    print(val)
}
// Output: nil

dict["first"] = nil
if let val = dict["first"] {
    print(val)
}
// No output
{% endhighlight %}

It’s standard in Swift to unwrap an optional with `if let`. Why did I receive a `nil` on line 3?

There’s a clue on line 8. When I explicitly set `first` to be `nil`, the unwrap operation behaves as expected. There really must be a different kind of `nil` here.

## How Dictionary fetches values

`Dictionary` wraps the value in an optional before handing it to you.

![](/assets/images/swift-expected-nil-not-nil/1LHvVrDoM-Zf_cRjcFwzk1g.png)

This is all good and expected. My problem came in because I was setting my value type as `Any?` instead of `Any`. Swift was double wrapping my value!

![](/assets/images/swift-expected-nil-not-nil/1o2JutK_fwV-KFf35sJ4QKA.png)

## Setting a key to nil

When you set a key to `nil` in a Dictionary it removes that key/value pair for you. That’s why line 8 above behaves as expected.

If you want to explicitly set your value to `nil` in a Dictionary with an optional value type, you must wrap it yourself first.

{% highlight swift %}
dict["first"] = .some(nil)
if let val = dict["first"] {
    print(val)
}
// prints ‘nil’
{% endhighlight %}

Swift was doing this extra step automatically when I first created my dictionary with `var dict: [String: Any?] = ["first", nil]` — it wrapped that nil for me behind the scenes.

## Don’t do this!

I ended up refactoring my Dictionary to not use an optional for its value. The fewer quirky rules one has to keep in mind the better. If I got confused from my own code, what would happen to another developer jumping in? For a longer meditation on this idea, [check out Tyler Johnson’s essay on code design](https://medium.com/livefront/applying-design-concepts-to-code-7cd65143b79a).

Code should present itself like a political cartoon — full of blunt metaphors with helpful labels and no subtlety.

*Sean judges what code looks like at [Livefront](http://www.livefront.com).*
