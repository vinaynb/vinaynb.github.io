---
layout: post
title: Integerating Payumoney payment gateway with Sencha Touch Hybrid App
description: "This article goes through the process of using Cordova's In App Browser plugin for accepting payments in a hybrid app using Paymoney payment gateway."
modified: 2016-12-16
category: articles
tags: [web-dev,mobile-hybrid,sencha-touch,cordova]
comments: true
share: true
---

---

I'll be honest here. I started writing this article about 10 months back in February and its crazy that i took such a long time to complete this (who says blogging is easy ?), so it may be possible that there are better ways to do what i am doing here. I am all ears to those solutions. And yes I will be trying to blog a bit more regularly and probably on shorter topics in the future. Happy reading !
{: .notice}

---

This article is a result of endless tries of googling to find some help in setting up a payment gateway in a hybrid mobile app. I didn't find the exact kind of guide i was looking for and hence i had to do it myself. Here's how i did it.

So you've built a shiny new hybrid app (in half the cost of maybe even more less than a native one, hybrid 1-native 0) and want to add a feature to accept payments through a 3rd party payment gateway such as [Paytm](http://paywithpaytm.com/) or [Payumoney](https://www.payumoney.com/websiteintegration.html?f=#lrnMore1) or [Stripe](https://stripe.com/) etc.

In most of the cases wherein you provide users multiple options to make the payment such as Credit and Debit cards, Net Banking and 3rd Party Wallets, the payment process involves a browser inside the app which displays different web pages, step by step, to process the payments. Once the payment is done user is navigated to a screen showing success or failure of the transaction and the browser is closed.

But since this article is context of [hybrid mobile apps](http://developer.telerik.com/featured/what-is-a-hybrid-mobile-app/) we are talking of a app which is already entirely running in a browser itself. So how do you open a browser inside a browser to process your payments. Well you guessed it correctly, we don't ! but still we got to do something that will help us navigate different pages of the payment gateway and process of our payments.

Before getting into knitty-gitty of integeration let me tell you a bit about the framework that has been used by me for building this App. [Sencha Touch](https://www.sencha.com/products/touch/#overview) has been around now for quite a long time on the hybrid scene and has been regularly listed among other frameworks as a popular choice for building hybrid mobile apps.I have been personally working on Sencha touch for more than a year now and based on my experience and the current scenario i would personally recommend other frameworks such as [Ionic](http://ionicframework.com/) or [Telerik](http://www.telerik.com/platform/appbuilder) as they are more flexible and easy to develop in compared to Sencha Touch.

**Nevertheless** of which UI framework you use, this tutorial still will be helpfull for you to integerate your payment gateway as the basic flow remains the same for all kinds of hybrid apps. Infact this article might give you a few pointers if you want to integerate payments into your website,web-app or your mobile website as all of these run in a browser just like our hybrid app.

Below is an image showing the exact process involved in processing a payment. I will be going in detail for each of the stages shown in the diagram :

![Payumoney PG process in browser]({{ site.url }}/images/posts/payumoneyPgProcess.svg){: .article-content-img }

So lets break down that ugly looking diagram into words.

Client makes API request to Payumoney
------------------------
In order to make an API request to Payumoney you will need certain credentials that uniquely identify your transactions among all other merchants of Payumoney. **API Key** and **Salt** are the two things that are unique to each account on payumoney to accept payments. These can be found in your **Merchant Dashboard** of Payumoney UI interface.

As i had mentioned below we will need something that will let us switch between the payment process pages and the page that our hybrid app is running in. For that we will use [InAppBrowserPlugin](https://github.com/apache/cordova-plugin-inappbrowser) from Cordova.

Below is the function that kicks off my payment process when user clicks on PAY button in my app.

<div class="github-gist-css">
<script src="https://gist.github.com/vinaynb/0e23822a8c053e00b12d3d0878ba7ddb.js"></script>
</div>

Initially i get my data which consists of data that i need for payments processing such as the amount to be paid, mobile no of the user, name of user etc. Now the important thing here is to realize that you will have to make a rest call to Payumoney servers with the required data, and if the data sent is accurate, Payumoney sends html web page in response which gets rendered on user's screen inside your app. Now as our app is a hybrid one we cant allow payuomoney's html to replace our html (i.e. our app) as then our app would be lost. Instead of that what we do is that we create another instance of webview by using `window.open()` function of javascript. When we call this our app will be still removed from screen but it will not be destroyed from DOM. It's just an new instance of webview that is rendered on to the screen instead of the old one which had our app running inside it.

So now we have an new instance of webview which will handle all our processing with payumoney. Hence the rest call with data to Payumoney will be made from new webview instance. As this new webview instance is a standalone thing, it has absolutely no access to our Sencha Touch stores, Controllers or any other stuff. So how do you go ahead and pass the data that you have in your controller to this webview instance and make a REST call ?

To do this firstly create an html file in your project similiar to the one below

<div class="github-gist-css">
<script src="https://gist.github.com/vinaynb/cf9d3adc7573c880867331723c2c5c22.js"></script>
</div>

This html basically contains just an form with hidden input tags corresponding to various required fields that are required by Payumoney while processing payments. More details about these fields can be found [here](https://blog.payumoney.com/?p=55) and [here](https://www.payumoney.com/pdf/PayUMoney-Technical-Integration-Document.pdf)(PDF). Inputs are hidden because we don't want user to view some ugly and raw html form which he does not need to interact with. The sole purpose of using a form here is that we can use form's submit method to send a POST request with all the input fields to Payumoney easily.

For security purposes and to verify authenticity of transactions, Payumoney requires us to send a hash of few fields along with the POST request. In order to derive that hash i use [this](https://github.com/Caligatio/jsSHA/) library which has been loaded in pgLoader.html

Few important fields that need to be understood here :

* *curl* - this field should contain an enpoint of your application server that can process payments cancellation. If due to any reason payment process is cancelled when in progress. Payumoney will make a POST request to this endpoint with neccessary details which you can extract and process.
* *surl* - this should contain endpoint of your app server to handle payment succcess. Payumoney will make POST to this endpoint with txn id, txn status and all that stuff.
* *furl* - endpoint to process failure. Same process as above.
^

Now we are all set to talk to Payumoney servers and proceed with our purchase. But we kind of need a specific time or event when to kick start this process. Also we cannot force the user to press a button to kick start the process, we need to do this automatically. In our case the best time to start this process will be when our InAppBrowser window finishes loading our pgLoader.html page. Fortunately window object in js emits different events during a lifecyle of a webpage and **loadstop** is the event we are looking for.

So i add an event listener for loadstop event and when the event occurs i make sure the event is fired for pgLoader.html as we will be having other pages (transaction success and error) being loaded into this same window in future. Once confirmed i call `w.executeScript({paymentGateway.js});` which kicks off the magic. The [executeScript](https://cordova.apache.org/docs/en/2.7.0/cordova/inappbrowser/inappbrowser.html#executescript) function allows you to inject and random javascript into the InAppBrowserWindow. Below is the source code for the javascript that prepares the data for payumoney and makes API call.

<div class="github-gist-css">
<script src="https://gist.github.com/vinaynb/fe01bc47b1bfeba84b2c17cc426316d7.js"></script>
</div>

The two functions written above are pretty simple. In `submitForm()` i extract the data that i need from the html via form's name parameter and few with id's. Then combining them with `salt` & `key` with all the data in a specified arrangement (note the serial arrangement is neccessary please follow the guidelines set by payumoney) i generate a correct hash value. After hash is generated i call the form's submit method from javascript itself which makes a POST request to payumoney servers with our data and if all's well and hash value was generated properly you should get a payuomoney page is response which tells you the amount you need to pay and other details. If you got this page (trust me you never will in the first attempt;) then you can celebrate, if not go grab a cup of coffee or something as you got some debugging to do.

The next 2 steps from the diagram i.e.

* User enters payumoney credentials and chooses mode of payment
* Payumoney redirects users to respective banks/wallets webpage to verify payments

are processed on paymoney's webpages which you do not need to take care of. All thats remaining now is to handle transaction error and success flows. We'll assume the user isn't broke has enough money to give us and hence we have success from Payumoney.

On successfull payment Payumoney redirects user to our app
------------------------
So how exactly this redirection happen. Do you remember those three fields that i talked about in the earlier steps above i.e. **curl,surl,furl**. surl indicates success url and this url is the one Payumoney servers will make a POST request too with all the transactional data once the transaction is successfull.

We cannnot simply handle a POST request using any arbitrary html page or js file in our InAppBrowser because browsers won't allow payumoney servers to access a file or make a request to it directly inside a device (security reasons) so we need some kind of request handling API on the server which can process the data sent by payumoney and then redirect our app back from InAppBrowser window to our Sencha Touch app running inside a cordova webview.

I create an API on my server, supply it in surl field and when the payumoney success request is recieved all i do is verify the hash sent by payumoney (you need to store the initial hash generated by you when intiating the transaction somewhere, possibly in a db) with your stored hash value to check if the transaction is authentic and data is tampered with on the network. Once hash is verified i create a small html page with inline javascript which has the code to close my InAppBrowser window and pass transaction details to my Sencha Touch App. I send this html code with embedded script in response and when it is runs on the mobile device it closes the InAppBrowser window and we are back to our Sencha Touch App !

Client processes the transaction details sent by Payumoney
------------------------

Of course we need some event in our sencha code to show some success information and activity to the user indicating that the transaction has been successfull. I use the same technique that i used earlier while kicking off the transaction API after loading pgLoader.html page.
The script injected inside the html sent from my API server redirects InAppBrowser window to `PaymentSuccess.html` page. It is basically a blank html page just used as a placeholder and a means to differentiate between error and success. Inside the `loadstop` event i now check for url to be PaymentSuccess.html, well aware that if this url gets called it means that a transaction has been completed successfully and now we need to show a success screen to the user.

To extract out the transaction details embedded in the html recieved from API server i call `getDetails()` by using `window.executeScript()`. This allows me to run getDetails() function which again is written inside the embedded javascript in the html code and gives me the transaction info.

PHEW ! So that's it, you have the success and the transaction data, show something in green to the user (a cartoon, a big tick, smiley whatever..) and you're done !

I hope this helps someone out there developing a hybrid app or working with accepting payments on the web in general. Feedback is appreciated. Please do share the post if you find it useful. Cheers !