---
layout: main
---

If you are experienced with using Webdriver in other languages, some things may seem particularly difficult in WebdriverJS.
One such example is the concept of looping.

Let us consider that we want to write a script that sees the option text available
in the [BBC News](http://www.bbc.co.uk/news/) navbar.

Now, the Navbar is logically enough, implemented as a Unorded List, which has a header which has an identifier.

	<ul id='nav'>
		<li>Text1</li>
		<li>Text2</li>
		...
	</ul>

So, in a typical impelemntation of Webdriver, we might want to do something like this.

<a name="typical"></a>
[Typical Implementation](#typical)
----------------------

	var ul = driver.findElement(By.id('nav'));
	var children = ul.findElements(By.tagName('li')); //Search for sub-elements of the ul
	var childText = [];
	for (var child in children) {
		childText.push(child.getText());
	}
	console.log(childText());

This is a perfectly reasonable way of doing things in a synchronous language, but is of course completely
unworkable in WebdriverJS, as it uses Promises. When first using WebdriverJS, one may
mistakenly try to do the following:

<a name="wrong"></a>
[Wrong WebdriverJS Implementation](#wrong)
--------------------------------

	var webdriver = require('selenium-webdriver');

	var driver = new webdriver.Builder().
		withCapabilities(webdriver.Capabilities.chrome()).
		build();

	driver.get('http://www.bbc.co.uk/news');
	var ul = driver.findElement(webdriver.By.id('nav'));
	var lis = ul.findElements(webdriver.By.tagName('li')); //findElements returns a Promise which will resolve
		//to an Array of WebElements.

	var liText = [];
	lis.then(function(liArray) {
		liArray.forEach(function(li) {
			liText.push(li.getText());
		})
		console.log(liText);
	});

	driver.quit();

But sadly, even this won't work. And if you run this code you will notice that what gets logged, is, logically enough,
and array of Promises.

But now we are in a bit of trouble. We are only really interested in all the value simultaneously. While we could of
course apply a *then()* method to each promise in *liText*, this will become very messy: perhaps there is a better way.

<a name="correct"></a>
[Correct Implementation](#correct)
----------------------

	var webdriver = require('selenium-webdriver');
	var Q = require('q');

	var driver = new webdriver.Builder().
		withCapabilities(webdriver.Capabilities.chrome()).
		build();

	driver.get('http://www.bbc.co.uk/news');
	var ul = driver.findElement(webdriver.By.id('nav'));
	var lis = ul.findElements(webdriver.By.tagName('li'));

	lis.then(function(liArray) {

		var liTextPromises = [];
		liArray.forEach(function(li) {
			liTextPromises.push(li.getText());
		})
		Q.all(liTextPromises).then(function(liText) {
			console.log(liText);
		})
	});

	driver.quit();

And here you can see how we can use *Q.all()* to combine all these individual promises into a single promise.
*Q.all()* takes our array of promises, waits for all of them to resolve to a value, and then calls us back
with their resolved value in an Array.

Hopefully that helps if you are trying to loop over values!

This is where a certain external Promise library becomes useful: Q.js

Q.js provides a feature, [*.all*](https://github.com/kriskowal/q#combination), which will allow us to
deal with multiple promises which will resolve at an arbitrary future time.

First, remember to install it: *npm install q*

Then we can use q as follows

