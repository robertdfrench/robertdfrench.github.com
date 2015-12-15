---
layout: post
title:  "WTF is CORS?"
date:   2015-12-14 10:53:24
categories: cors ajax api http header policy browser same-origin
---
You've heard of CORS right? It's that thing that lets you like... make an ajax call to some other site? Or something? Yeah. It can be a bit fuzzy. 

**tl;dr**: If you want a page hosted on `sunnyvale.ca` to call `api.sunnyvale.ca/lots/open`, just make sure the client specifies that it is making a CORS request, and have the backend application return `Access-Control-Allow-Origin: sunnyvale.ca` as part of its headers.

### Can't I just make HTTP requests to whatever host I want?
Well, yes. You can in curl, or some other basic http client. But modern browsers have all agreed to implement a rule that prevent scripts hosted on Site A from making ajax calls to Site B. Just like showing lock icons for sites with valid SSL certificates, this is so-called "same-origin policy" is something browsers do to make consumers safer.

### Why is this needed?
Suppose some asshole buys the domain `paypalloginhelper.com` and registers an SSL certificate for it. They copy all the image and css assets from paypal, but modify the client-side javascript so that it does the following when the user tries to log in:

1. Send the login credentials to `paypal.com/login/auth`
1. Upon success, send a copy of the login credentials to `paypalloginhelper.com/steal/password`
1. Request and display the content from `paypal.com/user/123/account_info`

This would allow the attacker to gather login credentials from anyone they could trick into going to their site. With additional measures (such as [CSRF Protection](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%28CSRF%29_Prevention_Cheat_Sheet)), PayPal could detect and reject such requests, but the same-origin policy allows browsers to prevent them altogether.

### So what is CORS, again?
Invoking `api.sunnyvale.ca/lots/open` from within the `sunnyvale.ca/index.html` document is an example of Cross-Origin Resource Sharing (in browser lingo, `sunnyvale.ca` and `api.sunnyvale.ca` are two separate *origins*). Since the browser normally stops such behavior, we need to ask for an exception. The browser will try to confirm this exception by double checking with our API server. If the API server says `sunnyvale.ca` is allowed to make CORS requests, the browser will give us a thumbs up and proceed with our request.

#### Step 1 -- Client tells browser it wants to make a CORS request
A jQuery example due to [Yuriy Dybskiy](https://github.com/html5cat/cors-wtf) shows how to initiate a CORS request from a document on `sunnyvale.ca`:

{% highlight javascript %}
$.ajax({
  type: 'GET',
  url: 'https://api.sunnyvale.ca/lots/open',
  crossDomain: true // <=== this is the important bit
}).done(function (data) {
  // print open lots to console
  console.log(data);
});
{% endhighlight %}

#### Step 2 -- Browser asks server if sunnyvale.ca is on its VIP list
Before sending our GET request, the browser will first issue what is called a *pre-flight* request in order to check whether our intended request is even allowed. To do this, the browser sends a request with the 'OPTIONS' verb to ask the server to confirm that we are able to send such a 'GET' request from our origin:

{% highlight http %}
OPTIONS /lots/open HTTP/1.1
Host: api.sunnyvale.ca
Origin: https://sunnyvale.ca
Access-Control-Request-Method: GET
{% endhighlight %}

Recall here that CORS and the same-origin policy are browser-level features. It does no good to write code to discriminate requests based solely on the Origin header; attackers will lie to you anyhow. We simply need to tell the browser what Origins we expect such calls from. These features are for the user's protection, not yours.

#### Step 3 -- Server confirms that it expects CORS requests from sunnyvale.ca
To allow CORS requests from documents on `sunnyvale.ca`, include the `Access-Control-Allow-Origin` header in your reply, like so:

{% highlight http %}
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://sunnyvale.ca
Some-Other-Headers: "Whatever"
{% endhighlight %}

That tells the browser that we know about `sunnyvale.ca`, and that we expect documents hosted there to make CORS requests to us. That will assuage the browser's concerns, and it will then proceed with the original request *like a normal ajax call*.

### Why can't browsers just assume that subdomains are safe?
In most cases, `whatever.com` and `api.whatever.com` will be managed by the same team. So why not just have browsers trust requests to `*.whatever.com`? 

Think about sites like GitHub that allow users to manage content on their own subdomains. Similar to the hypothetical PayPal example above, a malicious user could register the name `developerlogin` and then control content for the browser origin `developerlogin.github.com`. Explicitly specifying which origins are allowed to make CORS requests prevents this means of leveraging a users' browser in a data-sniffing attack.

### References
1. [HTTP Access Control (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) -- [Mozilla Developer Network](https://developer.mozilla.org/en-US/).
1. [CORS W.T.F.?! or 'What is the best way to play with cors locally?'](https://github.com/html5cat/cors-wtf) -- [Yuriy Dybskiy](http://dybskiy.com/).
1. [Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%28CSRF%29_Prevention_Cheat_Sheet) -- [Open Web Application Security Project](https://www.owasp.org/index.php/Main_Page).
