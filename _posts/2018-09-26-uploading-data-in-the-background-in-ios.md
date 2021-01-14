*[This post originally appeared on Medium](https://medium.com/livefront/uploading-data-in-the-background-in-ios-f93722013c6a)*.

The docs for properly uploading data in the background are spread over several documents. Some limitations aren’t enforced by the API. Instead you’ll be notified about the proper usage with a crash. This article will take you on a journey of the many wrong ways to upload in the background before showing you the right way.

Let’s start with the quick and dirty solution you might stumble upon when you first search for how to upload data:

{% highlight swift %}
let url = URL(string: "http://0.0.0.0")!
let data = "Secret message".data(using: .utf8)
var request = URLRequest(url: url)
request.httpBody = data
let task = URLSession.shared.dataTask(with: request) { data, response, error in
    print("We're all done here")
}
task.resume()
{% endhighlight %}

Notice how we used `dataTask` there? You can certainly use that for uploading data. The key difference Apple describes between `dataTask` and `uploadTask` is that `dataTask` is intended for short requests, and `uploadTask` allows you to continue execution in the background.

Great, let’s change our example to use an upload task and configure our session to be in the background.

{% highlight swift %}
let url = URL(string: "http://0.0.0.0")!
let data = "Secret Message".data(using: .utf8)
let request = URLRequest(url: url)
let config = URLSessionConfiguration.background(withIdentifier: "uniqueId")
let session = URLSession(configuration: config, delegate: nil, delegateQueue: nil)
let task = session.uploadTask(with: request, from: data) { responseData, response, error in
    print("We're done here")
}
task.resume()
{% endhighlight %}

Xcode will let us write this code and it will compile fine, but if you run it you’ll get this crash:

{% highlight powershell %}
***** Terminating app due to uncaught exception 'NSGenericException', reason: 'Completion handler blocks are not supported in background sessions. Use a delegate instead.'*****
{% endhighlight %}

Dang it, I’ve been spoiled by how Swift-y completion handlers are. Delegates feel so 2010. Should I synthesize my properties as well?

Enough sass, let’s add a `URLSessionTaskDelegate`:

{% highlight swift %}
class ViewController: UIViewController, URLSessionTaskDelegate {
    override func viewDidLoad() {
        super.viewDidLoad()

        let url = URL(string: "http://0.0.0.0")!
        let data = "Secret Message".data(using: .utf8)!
        let request = URLRequest(url: url)
        let config = URLSessionConfiguration.background(withIdentifier: "uniqueId")
        let session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
        let task = session.uploadTask(with: request, from: data)
        task.resume()
    }

    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
        print("We're done here")
    }
}
{% endhighlight %}

Once again, Xcode will successfully compile this code but then crash on us 😞.

{% highlight powershell %}
***** Terminating app due to uncaught exception 'NSGenericException', reason: 'Upload tasks from NSData are not supported in background sessions.'*****
{% endhighlight %}

We need to first write our data to a local file, and then tell iOS where to find it. The operating system will copy it and manage the upload, so we’re safe to delete our copy after handing it off.

{% highlight swift %}
class ViewController: UIViewController, URLSessionTaskDelegate {
    override func viewDidLoad() {
        super.viewDidLoad()

        let url = URL(string: "http://0.0.0.0")!
        let data = "Secret Message".data(using: .utf8)!

        let tempDir = FileManager.default.temporaryDirectory
        let localURL = tempDir.appendingPathComponent("throwaway")
        try? data.write(to: localURL)

        let request = URLRequest(url: url)
        let config = URLSessionConfiguration.background(withIdentifier: "uniqueId")
        let session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
        let task = session.uploadTask(with: request, fromFile: localURL)
        task.resume()
    }

    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
        print("We're done here")
    }
}
{% endhighlight %}

Finally, it won’t crash!

There’s one final point to consider. What if our app closes and gets purged from memory before the upload is complete? The default behavior is that iOS will wake up your app and call a method in your AppDelegate:

{% highlight swift %}
**func** application(**_** application: UIApplication,
handleEventsForBackgroundURLSession identifier: String,
completionHandler: **@escaping** () -> Void)
{% endhighlight %}


You’ll have to recreate your background session with the given identifier (it will be the same as the one you used to create your original background session). Be sure to give it a delegate as well! Your delegate will then be notified with the relevant response information.

To avoid all of that business, you can tell iOS to not wake up your app by setting `sessionSendsLaunchEvents` to false in your `URLSessionConfiguration`. Instead of your app being woken up, iOS will leave it up to you to recreate that same background session. Whenever you end up doing that, like, say, the next time your app is launched, you’ll be given the response info through your delegate.

I want to reiterate: when you’re recreating these sessions, be sure to use the same unique identifier, or you’ll miss out on the responses.

Final note: if for some reason you’re regularly uploading data in your app, you can achieve some significant energy savings by deferring those uploads to a single task in the background — for more information see: [https://developer.apple.com/videos/play/wwdc2018/228/](https://developer.apple.com/videos/play/wwdc2018/228/)

That’s all there is to it! I hope I saved you some time.

*Sean deals with a lot of data at [Livefront](http://www.livefront.com).*
