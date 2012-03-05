---
layout: post
title: "Best way to kill your HTML5 tablet app performance"
date: 2012-03-05 20:28
comments: true
published: true
categories:
- Javascript
- HTML5
- RIA
- iPhone
- Android 
---

As I was developing a cool new HTML5 (in the marketing sense) RIA tablet app, I noticed that it was acting a little slow. All the CSS transforms that were supposed to be hardware accelerated were stuttering slightly on the iPad 1, and quite a lot on Android devices. At first I just chalked it all up to the iPad 2:s beefier CPU, and the notoriously bad transformation performance on Android devices.

That is until I found this piece of code. <!-- more -->Buried inside a _reset.css_ file was **this declaration**:

{% codeblock lang:css %}
* {
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0.1);
    -webkit-transition: all 0ms;
    -webkit-transform: translateZ(0);
    -webkit-font-smoothing: antialiased;
    -webkit-backface-visibility: hidden;

    -moz-transition: all 0ms;
    -moz-font-smoothing: antialiased;
    -moz-backface-visibility: hidden;

    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -o-user-select: none;
    user-select: none;
}
{% endcodeblock %}

Quite an eyeful, isn't it? The highlight here is our tiny little _tranlateZ()_ transform. I guess the original author intended to [enable hardware acceleration](http://www.html5rocks.com/en/tutorials/speed/html5/) on the page. So what's the problem with this selector? Why is it causing this huge hit on performance? It's elementary, dear Watson! The answer you seek is _the C in CSS_: it stands for _cascading_. Allow me to demonstrate with this piece of CSS.


{% codeblock lang:css %}
.rotating-example * {
	-webkit-transform: rotate(60deg);
	-moz-transform: rotate(60deg);
	transform: rotate(60deg);
}
{% endcodeblock %}

Yes, it's just an example. No, I don't think descendant selectors are cool. But we'll use this HTML as an example.

{% codeblock lang:html %}
<div class="rotating-example">
	<div>
		<div>
			<div>
				Upside down!
			</div>
		</div>	
	</div>	
</div>
{% endcodeblock %}

<div class="rotating-example">
	<div>
		<div>
			<div>
				Upside down!
			</div>
		</div>	
	</div>	
</div>

As you'll likely see (with a sufficiently modern browser), our div is upside down. Since the selector applies to all elements, the rotations cascade down the DOM, being applied one at a time at each descendant separately. In a similar manner the CSS star selector applies to **every single element** in your DOM **individually**. Now, usually the _translate3d()_ would be applied rather quickly, without any consequences, but with a dense enough DOM tree, it all starts to add up - and the app slows down to a crawl.

I removed the rule, committed, and performance was back up. Everything was nice and smooth on iPhone, Android and all the usual friends. In retrospect, I'm actually surprised the performance wasn't even worse!