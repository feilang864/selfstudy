# How JavaScript works: The building blocks of Web Workers + 5 cases when you should use them

Alexander Zlatkov

Alexander Zlatkov

Following

Jan 26, 2018 · 11 min read

[How JavaScript works: The building blocks of Web Workers + 5 cases when you should use them](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a)

This is post # 7 of the series dedicated to exploring JavaScript and its building components. In the process of identifying and describing the core elements, we also share some rules of thumb we use when building SessionStack, a lightweight JavaScript application that has to be robust and highly-performant to help users see and reproduce their web app defects real-time.


If you missed the previous chapters, you can find them here:


An overview of the engine, the runtime, and the call stack


Inside Google’s V8 engine + 5 tips on how to write optimized code


Memory management + how to handle 4 common memory leaks


The event loop and the rise of Async programming + 5 ways to better coding with async/await


Deep dive into WebSockets and HTTP/2 with SSE + how to pick the right path


How JavaScript works: A comparison with WebAssembly + why in certain cases it’s better to use it over JavaScript


This time we’ll be taking apart Web Workers: we’ll offer an overview, discuss the different types of workers, how their building components come to play together, and what advantages and limitations they offer in different scenarios. Finally, we’ll provide 5 use cases in which Web Workers will be the right choice.


You should already be familiar with the fact that JavaScript runs on a single thread as we have discussed it previously in great detail. JavaScript, however, gives developers the opportunity to write asynchronous code too.


Limitations of Async programming


We have discussed async programming previously and when it should be used.


Async programming enables your app UI to be responsive, by「scheduling」parts of the code to be executed a bit later in the event loop, thus allowing the UI rendering to be performed first.


A good use case for async programming is making AJAX requests. Since requests can take a lot of time, they can be made asynchronously, and while the client is waiting for a response, other code can be executed.


This, however, poses a problem — requests are handled by the WEB API of the browser, but how can other code be made asynchronous? For example, what if the code that is inside the success callback is very CPU intensive:


If the performCPUIntensiveCalculation is not an HTTP request but a blocking code (e.g. a huge for loop), there is no way to free up the event loop and unblock the UI of the browser — it will freeze and be unresponsive to the user.


This means that asynchronous functions solve only a small part of the single-thread limitations of the JavaScript language.


In some cases, you can achieve good results in unblocking the UI from longer-running computations by using setTimeout. For example, by batching a complex computation in separate setTimeout calls, you can put them on separate「locations」in the event loop and this way buy time for the UI rendering/responsiveness to be performed.


Let’s take a look at a simple function that calculates the average of a numeric array:


This is how you can rewrite the code above and「emulate」asynchronicity:


This will make use of the setTimeout function which will add each step of the calculation further down the event loop. Between each calculation, there will be enough time for other calculations to take place, necessary to unfreeze the browser.


Web Workers will save the day


HTML5 has brought us lots of great things out of the box, including:


SSE (which we have described and compared to WebSockets in a previous post)


Geolocation


Application cache


Local Storage


Drag and Drop


Web Workers


Web Workers are in-browser threads that can be used to execute JavaScript code without blocking the event loop.


This is truly amazing. The whole paradigm of JavaScript is based on the idea of single-threaded environment but here come Web Workers which remove (partially) this limitation.


Web Workers allow developers to put long-running and computationally intensive tasks on the background without blocking the UI, making your app even more responsive. What’s more, no tricks with the setTimeout are needed in order to hack your way around the event loop.


Here is a simple demo that shows the difference between sorting an array with and without Web Workers.


Overview of Web Workers


Web Workers allow you to do things like firing up long-running scripts to handle computationally intensive tasks, but without blocking the UI. In fact, it all takes place in parallel . Web Workers are truly multi-threaded.


You might say —「Wasn’t JavaScript a single-threaded language?」.


This should be your ‘aha!’ moment when you realize that JavaScript is a language, which doesn’t define a threading model. Web Workers are not part of JavaScript, they’re a browser feature which can be accessed through JavaScript. Most browsers have historically been single-threaded (this has, of course, changed), and most JavaScript implementations happen in the browser. Web Workers are not implemented in Node.JS — it has a concept of「cluster」or「child_process」which is a bit different.


It’s worth noting that the specification mentions three types of Web Workers:


Dedicated Workers


Shared Workers


Service workers


Dedicated Workers


Dedicated Web Workers are instantiated by the main process and can only communicate with it.


Dedicated Workers browser support


Shared Workers


Shared workers can be reached by all processes running on the same origin (different browser tabs, iframes or other shared workers).


Shared Workers browser support


Service Workers


A Service Worker is an event-driven worker registered against an origin and a path. It can control the web page/site it is associated with, intercepting and modifying the navigation and resource requests, and caching resources in a very granular fashion to give you great control over how your app behaves in certain situations (e.g. when the network is not available.)


Service Workers browser support


In this post, we’ll focus on Dedicated Workers and refer to them as「Web Workers」or「Workers」.


How Web Workers work


Web Workers are implemented as .js files which are included via asynchronous HTTP requests in your page. These requests are completely hidden from you by the Web Worker API.


Workers utilize thread-like message passing to achieve parallelism. They’re perfect for keeping your UI up-to-date, performant, and responsive for users.


Web Workers run in an isolated thread in the browser. As a result, the code that they execute needs to be contained in a separate file. That’s very important to remember.


Let’s see how a basic worker is created:


If the「task.js」file exists and is accessible, the browser will spawn a new thread which downloads the file asynchronously. Right after the download is completed, it will be executed and the worker will begin.


In case the provided path to the file returns a 404, the worker will fail silently.


In order to start the created worker, you need to invoke the postMessage method:


Web Worker communication


In order to communicate between a Web Worker and the page that created it, you need to use the postMessage method or a Broadcast Channel.


The postMessage method


Newer browsers support a JSON object as a first parameter to the method while older browsers support just a string.


Let’s see an example of how the page that creates a worker can communicate back and forth with it, by passing a JSON object as a more「complicated」example. Passing a string is quite the same.


Let’s take a look at the following HTML page (or part of it to be more precise):


And this is how our worker script will look like:


When the button is clicked, postMessage will be called from the main page. The worker.postMessage line passes the JSON object to the worker, adding cmd and data keys with their respective values. The worker will handle that message through the defined message handler.


When the message arrives, the actual computing is being performed in the worker, without blocking the event loop. The worker is checking the passed event e and executes just like a standard JavaScript function. When it’s done, the result is passed back to the main page.


In the context of a worker, both the self and this reference the global scope for the worker.


There are two ways to stop a worker: by calling worker.terminate() from the main page or by calling self.close() inside of the worker itself.


Broadcast Channel


The Broadcast Channel is a more general API for communication. It lets us broadcast messages to all contexts sharing the same origin. All browser tabs, iframes, or workers served from the same origin can emit and receive messages:


And visually, you can see what Broadcast Channels look like to make it more clear:


Broadcast Channel has more limited browser support though:


The size of messages


There are 2 ways to send messages to Web Workers:


Copying the message: the message is serialized, copied, sent over, and then de-serialized at the other end. The page and worker do not share the same instance, so the end result is that a duplicate is created on each pass. Most browsers implement this feature by automatically JSON encoding/decoding the value at either end. As expected, these data operations add significant overhead to the message transmission. The bigger the message, the longer it takes to be sent.


Transferring the message: this means that the original sender can no longer use it once sent. Transferring data is almost instantaneous. The limitation is that only ArrayBuffer is transferable.


Features available to Web Workers


Web Workers have access only to a subset of JavaScript features due to their multi-threaded nature. Here’s the list of features:


The navigator object


The location object (read-only)


XMLHttpRequest


setTimeout()/clearTimeout() and setInterval()/clearInterval()


The Application Cache


Importing external scripts using importScripts()


Creating other web workers


Web Worker limitations


Sadly, Web Workers don’t have access to some very crucial JavaScript features:


The DOM (it’s not thread-safe)


The window object


The document object


The parent object


This means that a Web Worker can’t manipulate the DOM (and thus the UI). It can be tricky at times, but once you learn how to properly use Web Workers, you’ll start using them as separate「computing machines」while all the UI changes will take place in your page code. The Workers will do all the heavy lifting for you and once the jobs are done, you’ll pass the results to the page which makes the necessary changes to the UI.


Handling errors


As with any JavaScript code, you’ll want to handle any errors that are thrown in your Web Workers. If an error occurs while a worker is executing, the ErrorEvent is fired. The interface contains three useful properties for figuring out what went wrong:


filename - the name of the worker script that caused the error


lineno - the line number where the error occurred


message - a description of the error


This is an example:


Here, you can see that we created a worker and started listening for the error event.


Inside the worker (in workerWithError.js) we create an intentional exception by multiplying x by 2 while x is not defined in that scope. The exception is propagated to the initial script and onError is being invoked with information about the error.


Good use cases for Web Workers


So far we’ve listed the strengths and limitations of Web Workers. Let’s see now what are the strongest use-cases for them:


Ray tracing: ray tracing is a rendering technique for generating an image by tracing the path of light as pixels. Ray tracing uses very CPU-intensive mathematical computations in order to simulate the path of light. The idea is to simulate some effects like reflection, refraction, materials, etc. All this computational logic can be added to a Web Worker to avoid blocking the UI thread. Even better — you can easily split the image rendering between several workers (and respectively between several CPUs). Here is a simple demo of ray tracing using Web Workers — https://nerget.com/rayjs-mt/rayjs.html.


Encryption: end-to-end encryption is getting more and more popular due to the increasing rigorousness of regulations on personal and sensitive data. Encryption can be a something quite time-consuming, especially if there’s a lot of data that has to be frequently encrypted (before sending it to the server, for example). This is a very good scenario in which a Web Worker can be used since it doesn’t require any access to the DOM or anything fancy — it’s pure algorithms doing their job. Once in the worker, it is seamless to the end user and doesn’t impact thеir experience.


Prefetching data: in order to optimize your website or web application and improve data loading time, you can leverage Web Workers to load and store some data in advance so that you can use it later when needed. Web Workers are amazing in this case because they won’t impact your app’s UI, unlike when this is done without workers.


Progressive Web Apps: they have to load quickly even when the network connection is shaky. This means that data has to be stored locally in the browser. This is where IndexDB or similar APIs comes into play. Basically, a client-side storage is needed. In order to be used without blocking the UI thread, the work has to be done in Web Workers. Well, in the case of IndexDB, there is an asynchronous API that allows you to do this even without workers, but there was a synchronous API before (it might be introduced again) which should only be used inside workers.


Spell checking: a basic spell checker works in the following way — the program reads a dictionary file with a list of correctly spelled words. The dictionary is being parsed as a search tree to make the actual text search-efficient. When a word is provided to the checker, the program checks whether it exists in the pre-built search tree. If the word is not found in the tree, the user can be provided with alternate spellings, by substituting alternate characters and test if it’s a valid word — if it’s the word that the user wanted to write. All this processing can easily be offloaded to a Web Worker so that the user can just type words and sentences without any blocking of the UI, while the worker performs all the searching and providing of suggestions.


Performance and reliability are very critical for us at SessionStack. The reason why they’re so important is that once SessionStack is integrated into your web app, it starts recording everything from DOM changes and user interaction to network requests, unhandled exceptions and debug messages. All this data is transmitted to our servers in real-time which allows you to replay issues from your web apps as videos and see everything that happened to your users. This all takes place with minimum latency and no performance overhead for your app.


This is why we’re offloading (wherever it makes sense) logic from both our monitoring library and our player to Web Workers that are handling very CPU-intensive tasks like hashing to validate data integrity, rendering, etc.


Web technologies constantly change and develop so We go the extra mile to ensure SessionStack is very lightweight and has zero performance impact on our users’ apps.


There is a free plan if you’d like to give SessionStack a try.


Resources


https://www.html5rocks.com/en/tutorials/workers/basics/


https://hacks.mozilla.org/2015/07/how-fast-are-web-workers/


https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API


