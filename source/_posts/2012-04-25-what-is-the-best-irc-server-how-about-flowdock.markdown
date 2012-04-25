---
layout: post
title: "What is the best IRC Server? How about Flowdock?"
date: 2012-04-25 17:07
comments: true
categories: irc, flowdock
---

Being an avid IRC user, ever since our company started using [Flowdock](http://www.flowdock.com/) I've been looking for ways to bridge it with IRC. Without an 
[XMPP integration](http://flowdock.uservoice.com/forums/36827-general/suggestions/635399-xmpp-jabber-integration), bitlbee was out of the question, and pretty
much the only way of integrating seemed to be over the AJAX API. That is, until now...

<!--more-->

Today Flowdock released built-in [IRC integration](https://www.flowdock.com/help/irc) and boy does it work amazingly!

{% img /images/posts/flowdock-irc.png %}

Above you can see an example of an irssi session. Influx shows our Github pushes and JIRA activity pretty nicely. We haven't enabled many other streams to prevent information overload. Also our [Hubot](http://hubot.github.com/) 'Botney Spears' works seamlessly!

{% img /images/posts/flowdock-irc-hubot.png %}

It was relatively painless to set up Flowdock in irssi. First you need to set up a new network and (if you want to connect and login automatically).

	/network add -autosendcmd "/^msg NickServ identify [your-e-mail] [your-password]" Flowdock
	/server add -ssl -network Flowdock -auto irc.flowdock.com 6697

And finally you need to connect to the server:

	/connect Flowdock

BAM! You're on Flowdock! You might want to _/window close_ any channels you're not participating in (in our case we had quite a few).

