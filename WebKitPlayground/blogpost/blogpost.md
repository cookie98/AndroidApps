# Android WebKit Performance

This article discusses the use of JavaScript and HTML5 to author mobile apps, and provides hands-on examples and performance tests for Android.

Which mechanism is the fastest for making calls from JavaScript to Java? How can Native UI components be controlled from JavaScript? How does the HTML5 Canvas compare to drawing on a Native Android Canvas?

## Introduction

Authoring mobile apps in JavaScript is compelling for several reasons:

* Portability
* High-level language
* Fast turn-around (Live coding)

### Portability

JavaScript is available on all modern mobile platforms (Android, iOS, Firefox OS, Windows Phone, etc). An app can run JavaScript code in a WebView and make full use of HTML5 functionality, and as needed invoke native code using mechanisms such as JavaScriptInterface on Android.

Furthermore, apps can run JavaScript inside a hidden WebView, and call platform code to display a fully Native UI. Cross-platform development tools like MoSync SDK and Cordova provide such capabilities out of the box.

Thus, using JavaScript can make your apps easier to port and to maintain, while still providing the possibility to access unique native features of each target platform.

You can also target desktop apps for Windows, OS X, and Linux can be made, using frameworks such as node-wekbit.

### High-level language

JavaScript is a high-level dynamic language, that enables the use of powerful programming techniques such as [higher-order functions](http://en.wikipedia.org/wiki/Higher-order_function), closures and prototyp objects. While developers make disagree on the suitability of JavaScript and other dynamic languages for large applications, it is a fact that JavaScript is widely adopted and appreciated by its users.

### Fast-turn around

With a dynamic language comes the prospect of speeding up the development cycle. It is a well established pattern to edit code and then reload on a web browser to quickly see the result. For mobile development, tools such as MoSync Reload provide similar capabilities.

There is also the possibility of updating a program while running, using technologies similar to a REPL or Smalltalk Workspace. Furthermore, there is the prospect of "live coding", where the program is updated while edited, and can be edited in real-time while running. The [MoLive project](http://www.youtube.com/watch?v=uqsxRTx0Iv8) is an example of live coding for mobile devices.

## Android WebView

Android provides the WebView class, which encapsulates the WebKit browser and a JavaScript engine as a widget. You can even run JavaScript in a hidden WebView, and this is a handy way to script apps, or write the entire application logic in JavaScript.

You can use the WebView to display the UI in HTML/CSS, or use bindings to Java to call methods to create a Native UI. This provides developers with a toolbox of flexible solutions.

Now, the question is, how fast is JavaScript running in a WebView? And what is the performance when it comes to calling into the native layer from JavaScript?

## WebView Performance Test App

To study the performance of running JavaScript in a WebView, we will use a hands on example. The Android app [WekKit Playground](https://github.com/divineprog/AndroidApps/tree/master/WebKitPlayground), demonstrates the programming techniques involved, and also provides performance tests. The full source code of the app is found on [GitHub](https://github.com/divineprog/AndroidApps/tree/master/WebKitPlayground), and there is also an [APK-file](https://github.com/divineprog/AndroidApps/blob/master/WebKitPlayground/blogpost/WebKitPlayground.apk) (debug signed) you can download and install.

TODO: Insert screenshots.

## Tests

Here is a description of the tests made by the app.

### Calling tests

The following JavaScript to Java calling mechanisms are tested by the app:

* **One-way calls:** Performance of calling from JavaScript to Java using the [JavaScriptInterface binding mechanism](http://developer.android.com/guide/webapps/webview.html#BindingJavaScript) on Android.
* **Two-way calls:** Performance of calling from JavaScript to Java and back to JavaScript. Calling from Java to JavaScript is done by invoking a "javascript:" URL on the WebView.
* **Prompt hook:** There are various alternatives to using JavaScriptInterface, but they are less elegant. One such mechanism is the hook provided by the WebView class for the JavaScript prompt function. This is tested to illustrate an additional mechanism. Also in this case two-way calls are used. Thus the results should be compared to the two-way JavaScriptInterface test.

The call tests use 10000 iterations for each test.

### Canvas tests

The app also has interactive tests of painting on a canvas:

* **HTML5 Canvas:** Performance test of painting on an HTML5 canvas.
* **Android Canvas controlled via JavaScript:** Test of painting on an Android canvas where events are routed through JavaScript and drawing is made using JavaScript calls to Java.
* **Android Canvas in Java:** Painting on an Android canvas natively from Java (no JavaScript used).

The interactive tests work as follows. A full screen canvas is displayed. On the first touch down event, a counter is set to zero and a time stamp is made. On subsequent touch drag events, the counter is incremented by one for each event, and when the number of touch strokes is 100, and the time from the touch down event is measured.

To run the canvas tests, touch the screen with a finger, and then rapidly move the finger back and forth, painting with the finger. When the test is done, the result screen is presented.

While this is not an ideal test, since it depends on how fast the tester moves the finger, it gives a hint at the performance of the different canvas options.

Now, it is time to see how the tests score. Which canvas is the fastest?

## Test Results

The test app was run on Nexus 7 (Android 4.2.2) and on Nexus One (Android 2.3.6). Below are the results.

### JavaScript to Java calling tests

<table>
<tr><td><b>Test</b></td><td><b>Nexus 7</b></td><td><b>Nexus One</b></td></tr>
<tr><td>One-Way Call (10000 calls)</td><td>400 ms</td><td>360 ms</td></tr>
<tr><td>Two-Way Call (10000 round trips)</td><td>3200 ms</td><td>4100 ms</td></tr>
<tr><td>Prompt Call (10000 round trips)</td><td>5500 ms</td><td>6000 ms</td></tr>
</table>

Tests results show that the JavaScriptInterface mechanism is faster than the prompt hook. Results also indicate that calling from Java to JavaScript much slower than the other way around. Even though not tested explicitly by the app, we can see that the one-way call test performs much faster than the two-way round trip test.

### Interactive canvas tests

<table>
<tr><td><b>Test</b></td><td><b>Nexus 7</b></td><td><b>Nexus One</b></td></tr>
<tr><td>HTML5 Canvas (100 strokes)</td><td>2000 ms</td><td>5800 ms</td></tr>
<tr><td>Android Canvas JavaScript (100 strokes)</td><td>3600 ms</td><td>2200 ms</td></tr>
<tr><td>Android Canvas Native (100 strokes)</td><td>2800 ms</td><td>1700 ms</td></tr>
</table>

Here we can see that on a modern Android device (Nexus 7), the HTML5 Canvas is very fast. It is much slower on the older device (Nexus One). This is likely because of improvements in the WebKit implementation in later Android versions. The Native Canvas rendering is likely faster on Nexus One than on Nexus 7 because of the smaller screen of this device.

## Conclusion

The Android JavaScriptInterface binding mechanism is a performant technique for making JavaScript to Java calls. Calling JavaScript from Java is much slower. To synchronously return data from Java to JavaScript using JavaScriptInterface would be a faster option than returning data asynchronously, unless it takes time to retrieve/process the data (e.g. downloading from the network).

It is unfortunate that Android does not provide a fast way to call into JavaScript from Java, and handle the result. This is available on iOS in the form of the method **stringByEvaluatingJavaScriptFromString** in class [UIWebView](http://developer.apple.com/library/ios/#documentation/uikit/reference/UIWebView_Class/Reference/Reference.html). On the other hand, iOS does not have a mechanism as handy as JavaScriptInterface to call Objective-C from JavaScript.

In general, the mechanisms used to invoke native code from JavaScript varies across platforms. It would be welcome to see a standardization of this, as we can expect apps written in JavaScript accessing native functionality to be a common pattern.

Depending on the HTML5 Canvas performance of the Android versions and devices targeted by your app, it may pay off to use a Native Canvas for rendering. The trend is likely that HTML5 Canvas performance will continue to improve, which reduces the need to go native.

To write your mobile application is JavaScript is a viable option, in particular if one considers ease of porting and the prospect of fast turn-around development using "live" tools. You can also develop parts of (or most of) the code using a web browser, which is a quick way of testing and debugging code.

You can use the WebKit Playground app as a starting point for your own hybrid apps. The way the app uses JavaScript to handle events from and draw on a native canvas serves as an illustration of how JavaScript can be extended beyond its traditional role as a web browser language.

You are most welcome to contact me to provide feedback, suggest improvements, or discuss mobile app development. I can be reached at mikael.kindborg@gmail.com.


