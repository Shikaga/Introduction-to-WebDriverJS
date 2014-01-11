---
layout: main
---

For the last few weeks I have been developing my tests in WebdriverJS.

WebdriverJS is a yet another implementation of the Selenium/Webdriver API, this time in Javascript, and is typically expected to run in Node.

However, while I have grown to like WebdriverJS a great deal, I don't think that it has been very well documented at all. In fact, I think it is very poorly documented. 

There are no or at least very few, tutorials, apis, stack exchanges and so forth for people to get up to speed on the library.

So, after a few weeks of smashing my head against it (largely due to my own ignorance), I will be writing a few blogs to hopefully steer others around the mistakes I made.

PROMISES
--------

Before you can get started using WebdriverJS, you need to understand the Promises Pattern, as Promises are the core upon which WebdriverJS is written.

I am sure that in a few years time Promises will be so fundamental and obvious as to not need an introduction, but it did take me a significant amount of time to get my head around them, and in my opinion, there are no really good descriptions of Promises, so let me try and describe them as best I can.

If you want to get a professional description, see Q.JS (XXX) or (XXX) ???

All that a Promise is, is an Object, which represents a result that exists or will exist at some point.

What do I mean by this? Consider a typical Ajax request function, it might look something like this:

getAjax('http://www.example.com', function(data) { console.log('Data Received', data) });

This is a perfectly legitimate way of dealing with such a thing. However, it can become a little unwieldy and lead to 'Callback Hell', although, some might argue that isn't the worst thing in the world: http://blog.caplin.com/2013/03/13/callback-hell-is-a-design-choice/

However, there is another we could implement an Ajax call, with Promises:

var promise = getAjax('http://www.example.com');
promise.then(function(data) {console.log('Data Received', data) });

So, what is going on here? Well quite simply, the getAjax() function, instead of taking a function to invoke, returns a Promise for the user to deal with.

As stated before, this promise represents the value or the future value of the result. As this Promise is an object, it can be passed anywhere in your code, assigned to variables, placed in an array or a map, or anything you like.

While Promises are a generic pattern, there is a mostly (I am not going to get into it) agreed spec (http://promises-aplus.github.io/promises-spec/) which defines how Promises should be implemented in Javascript.

According to this spec, you can use the then() function to provide the Promise with a function to invoke after successfully receiving the value. If the value has already been received, your function will be invoked immediately, otherwise the function will be invoked when it is ready.

So, as you can see in the above example, the exact same result occurs in either case. So why use Promises?

Well, that is a bit of a religious question, and I am not going to get into it really, but for me Promises allow you to avoid Callback Hell, and make your code much more readable than the available alternatives. But I accept that there are plenty of other answers to that question.

But, on a practical note, you have to use Promises because they are the foundation upon which WebdriverJS is built, and it is important to understand them fully.

Getting Started Code:

var webdriver = require('selenium-webdriver');

var driver = new webdriver.Builder().
	withCapabilities(webdriver.Capabilities.chrome()).
	build();

driver.get('http://www.google.com');
var searchBox = driver.findElement(webdriver.By.name('q'));
searchBox.sendKeys('webdriver');
searchBox.sendKeys('JS');
searchBox.sendKeys(' ');
searchBox.sendKeys('Rocks');

var btnG = driver.findElement(webdriver.By.name('btnG'));
btnG.click();

driver.wait(function() {
	return driver.getTitle().then(function(title) {
		return title.indexOf('webdriver') !== -1;
	});
}, 1000);

driver.quit();

Above is the example code given on the WebdriverJS website.

Now, something that might not be obvious, is that *driver.findElement(webdriver.By.name('q'));* returns a Promise. In the code above, searchBox is a Promise. Indeed, it is a very specific kind of promise, it is a WebDriver Promise, which provides a lot more features than a typical Promise, but still supports the Promises/A+ spec.

So, when *searchBox.sendKeys('webdriver');* is invoked, searchBox is still unresolved, and the Promise simply records that the user would like to "sendKeys" when "driver.findElement" returns.

Interestingly enough, sendKeys() also returns a promise, although it will never resolve to a value (sendKeys obviously doesn't have a return value), however for the purposes of chaining Promises, it is still important that it does, although we do not use that functionality in this example.

Also, note, that sendKeys() is invoked multiple times on searchBox, yet not only are they run in the order invoked.

This may seem like a arbitrary distinction, but it is very important, and I will get back to this in a moment.

Now, I must admit, I am not fully sure what all these clever functions like 'sendKeys()' do, but I can make a guess, and it may help to think of things in the following way:

Before, I mentioned that the 'then()' function was the function that all Promise implementations to allow users to define what they wanted to occur when the Promises result is resolved. From what I can determine, 'sendKeys()' is simply semantic sugar to us handling the result ourselves. As such, we could do it manually as follows:


searchBox.then(function(elementPromise) {
	elementPromise.sendKeys('webdriver');
})

This is a little more verbose, but it is functionally synonymous with  '
searchBox.sendKeys('webdriver');' and hopefully helps you see what is going on.

Notice also, that the parameter passed, is a Promise. If this seems surprising to you, get used to it, almost everything in WebdriverJS is.


