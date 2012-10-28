---
layout: post
title: "Compiling Ruby extensions with Xcode 4.3"
date: 2012-05-14 21:12
comments: true
categories: [Mac OS X, Development, Fixes]
---
Today I decided to try out [Octopress](http://octopress.org/ "Visit Octopress website if you are looking for a blogging framework for hackers") for handling my personal website. As always, things never go as smooth as it should be, otherwise where is the fun?  

<!--more-->

The major flaw I had to face was the failure in compiling some Ruby extensions, apparently due to missing header files and binaries.

My system currently runs OS X 10.7.4 with Xcode 4.3 and the [standalone command line tools](http://adcdownload.apple.com/Developer_Tools/command_line_tools_for_xcode__february_2012/command_line_tools_for_xcode_.dmg "Download the standard command line tools for Xcode. Warning, requires an Apple Developer account") installed (the latter is not any longer packaged into Xcode and has to be installed manually).

After having checked that all required libs and tools were installed on my system I started to think on a path-related issue. Before Xcode 4.3, in fact, all dev related stuff was stored in `/Developer`, whereas since Xcode 4.3 everything is packed inside the application folder itself.

I found out the handy command line tool `xcode-select` whose responsability is to manage the path for Xcode and UNIX tools and decided to force the new path:

{% codeblock lang:bash %}
sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/
{% endcodeblock %}

That did the trick and the fact that you are reading this on Octopress proves its success :P