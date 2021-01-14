*[This post originally appeared on Medium](https://medium.com/livefront/unit-testing-race-conditions-by-creating-chaos-swift-512a55e09806)*.

![](/assets/images/unit-testing-race-conditions-by-creating-chaos-swift/1b1Yueox84YZb3BtqgGSUHg.jpeg)
Containing Chaos — Source: https://fineartamerica.com/featured/--containing-chaos--michael-lang.html

Multithreaded race conditions don’t happen consistently, which makes them difficult to reproduce. On the other hand, best software practices demand we write automated tests to validate our changes. I’m going to share my technique for creating enough chaos that those rare crashes become reliable enough to test against.

## What’s a race condition?

Bad things can happen if you have multiple threads trying to mutate and access data at the same time. One thread may change a value while another is using it!

The best way to avoid this scenario is to structure your code in such a way that it’s not even possible. Swift’s structs are a good starting point because they get copied on write (for extra credit: [think functionally!](https://medium.com/livefront/thinking-functionally-in-swift-832efc8f7141)).

But we can’t always do the ideal thing. We’re developing in the real world with constraints outside of our control. Maybe we’re stuck in a legacy code base. Maybe we don’t have the time or money to refactor everything. Sometimes all we can do is implement best practices going forward.

And one of those best practices is: ***When fixing a bug, write a unit test that reproduces it to prevent regressions.*** Normally that’s straight forward, but these race condition crashes can be tricky. They’re rarely showing up in your crash reporter, and you’ve never seen it happen on your device.

![[https://blog.codinghorror.com/the-works-on-my-machine-certification-program/](https://blog.codinghorror.com/the-works-on-my-machine-certification-program/)](/assets/images/unit-testing-race-conditions-by-creating-chaos-swift/1gbqUzxvyrYF2y8KSi8jSZA.png)

## Let’s Make it Crash

First, set up a simple method that has some unsafe code:

{% highlight swift %}
class MutableClass {
    var val: Int?

    init(val: Int = 0) {
        self.val = val
    }

    func update(newVal: Int) {
        val = nil
        val = newVal
        let f = val! + 5
    }
}
{% endhighlight %}

This is not going to crash in normal conditions. The value is always set before being forcefully unwrapped. We can even spawn 10 threads to attack it at once and it’ll probably be ok. But if we **use our imagination** we can see the possibilities. What if one thread sets the value to nil while another is about to unwrap it? Disaster.

To create that scenario, let’s make as much chaos as possible (without destroying our machines).

{% highlight swift %}
func testMutableClass() {
    let group = DispatchGroup()
    let obj = MutableClass()

    for _ in 0...1000 {
        group.enter()

        DispatchQueue.global().async {
            let sleepVal = arc4random() % 1000
            usleep(sleepVal)
            obj.update(newVal: 42)
            group.leave()
        }
    }

    let result = group.wait(timeout: DispatchTime.now() + 5)

    XCTAssert(result == .success)
}
{% endhighlight %}

* **Step 1:** Spawn a legion of threads.

* **Step 2:** Check that they all finish within a certain frame of time. (We want to give all of the threads a chance to crash before moving on to the next unit test, otherwise we could assert ‘true is true’.)

* **Step 3:** Delay the threads by random intervals. This is our chaos creator, and pushed this test from “ kinda consistent” to “consistent” for me.

* **Step 4:** Run your unit tests and watch them crash.

![](/assets/images/unit-testing-race-conditions-by-creating-chaos-swift/1an-GjopFC0J6Wndn5Kf53A.png)

Unexpectedly? More like expectedly. Now you can fix it and be reassured future changes won’t bring it back.

*Sean battles chaos daily at [Livefront](https://livefront.com).*
