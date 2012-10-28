---
layout: post
title: "Add spaces to Mac OS X dock"
date: 2012-05-14 18:30
comments: true
categories: [Mac OS X, Tricks]
---
I recently discovered that you can easily add spaces between Mac OS X Dock icons. Here's how.

<!--more-->

Simply open the Terminal and run the following command:

{% codeblock lang:bash %}
defaults write com.apple.dock persistent-apps -array-add '{tile-data={}; tile-type="spacer-tile";}'
{% endcodeblock %}

To see changes you need to restart the Dock process:

{% codeblock lang:bash %}
killall Dock
{% endcodeblock %}

You can drag and move the spacer between icons as you would do with app icons. If you want to add another spacer simple re-run the first command. An example of what you could obtain

![Screenshot of a Mac OS X Dock with spaces](http://www.matteoagosti.com/images/posts/dock-with-spaces.png)

For geeks: if you want to add the spacer to the right side of the Dock (documents section), run the following command:

{% codeblock lang:bash %}
defaults write com.apple.dock persistent-others -array-add '{tile-data={}; tile-type="spacer-tile";}'
{% endcodeblock %}