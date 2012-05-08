---
layout: post
title: "Responsive design: forget about the media queries"
date: 2012-05-09 00:05
comments: true
categories: 
---

Let's just put it out front: responsive design is a marketing term. There, now that you've finished scoffing as I have deeply shocked your designer world, allow me to rephrase that: [a lot](http://fitandfinish.ironworks.com/2011/05/the-future-of-web-design-will-be-responsive.html) of discussion around responsive design gloriously [misses the point](http://mediaqueri.es/). 

<!-- more -->

{% pullquote %}
"Handheld devices don't show traditional websites well", that much is agreed - but beyond that, pretty much everybody has there own opinion. While [some](http://www.lukew.com/ff/entry.asp?1390) build separate sites for handhelds and others warn against [siloing content](http://www.alistapart.com/articles/responsive-web-design/) to a separate domains. So what's a company looking to go mobile to do?

Heck, there's even a case here in Finland where the CMS provider for a bank recommended simply to [increase the size of touch targets](http://www.ch5finland.com/ajankohtaista/artikkelit/fi_FI/mobiilioptimointi/) (only in Finnish, sorry)! Well, that's their process and what they deemed to be the most cost-effective solution for their customer and I won't go deeper into that. Another company found that [creating multiple native apps](http://twitter.com/#!/dumbstereo/status/198309896402378752) was more cost-effective than going fully responsive. And therein lies a seed of truth: *{"'best way to achieve responsiveness' depends on your strategy, process and competences"}*.
{% endpullquote %}

[Some people](http://www.netmagazine.com/opinions/separate-mobile-website-no-forking-way) lay the blame of rigidity on antiquated CMSes, using WordPress and Drupal as an example of a web CMS no longer cutting it in a multi-screen world. But what if you're a kick-ass PHP house (underlying factor here: competencies), and your _strategy_ does not mandate business rules to determine what to publish and where? I can't see a reason why you couldn't create a template good enough to cater for different experiences. Now then, if your needs go beyond that, one can question the choice of Wordpress or Drupal in the first place.

I have been working almost exclusively with mobile web from 2005, starting when we were creating a mobile site for a very high-profile customer that went on to win several Webbys. Back then we had to maintain a WML site for some browsers, but the main focus was still HTML. Nevertheless, the main drivers in choosing an implementation method were the three factors above. We had to go with pure server-side technologies since the devices couldn't handle anything else (RAZR, anyone?). For _process_ reasons the site couldn't be served off the same domain as the main site - a separate team building a separate codebase would not be allowed to touch the main web servers. Due to _competence_ reasons the customer was not building the site themselves as it was not part of their _strategy_ to acquire these competencies.

{% pullquote %}
[Stephanie Rieger](http://stephanierieger.com/responsiveness-is-a-characteristic/) raises excellent points to the discussion: to the user it does not matter whether or not your site is based on a different CSS, different HTML-files, different frontend codebase or even completely different programming language. What is relevant to the end user is a good experience, whereas what is relevant to the company is to get the best value for money and empower their own employees as much as possible. And for that reason, {" questions like "should I use responsive design or build a separate codebase" are both moronic and pointless"}. Instead, ask yourself:

   * How can I build the best experience with the skills that I have / can learn?
   * What will give me best ROI in both short and long run?

I can build an RIA app with media queries, client-side JavaScript or server-side Java, JavaScript, Python - you name it. What I will choose to build it with depends on scope of the app and competencies of the customer.
{% endpullquote %}

If the case is a corporate blog that needs to look good on the CEOs Galaxy S III, I'll gladly hack it together on Wordpress. If you want to maintain your current website with the skillsets that you have in place, and don't plan on expanding towards anywhere RIA-ish, I wholeheartedly recommend media queries and sprinkled-on JavaScript. But if you want a native app-like experience (damn it with these marketing terms already), most of the code will be straight-up JavaScript, and it's going to look and feel so different on a Galaxy Tab vs. iPhone - in a good way. And it's still responsive even though you can't resize your window on the fly. Heck, I'll even be nice enough not to cause your device to [download all the images twice](http://www.bram.us/2012/03/01/the-slow-elephant-in-the-responsive-images-room/).