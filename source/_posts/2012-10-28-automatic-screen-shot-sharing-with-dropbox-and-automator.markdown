---
layout: post
title: "Automatic screen shot sharing with Dropbox and Automator"
date: 2012-10-28 22:31
comments: true
categories: [Mac OS X, Tricks]
---

After long time using various utilities to automatically share my screen shots when I updated to Montain Lion I had to find another solution as many of them stopped working. It came to my mind that OS X is bundled with Automator, an extremely powerful utility that I always relegated to thumbnails generation. So I decided to give it a try and I eventually made it. That's how I did.

<!-- more-->

First, fire up Automator, create a new `folder action` and associate it to the Desktop, as that is the place where screen shots captured with the 3 fingers key combo are stored.

![Screenshot of a folder action for desktop](http://www.matteoagosti.com/images/posts/Screen Shot 2012-10-28 at 10.42.55 PM.png)

Add the `Filter Finder Items` action to select only images whose file name starts with `Screen Shot`

![Screenshot of a folder action for desktop](http://www.matteoagosti.com/images/posts/Screen Shot 2012-10-28 at 10.47.00 PM.png)

Add the `Move Finder Items` action to move previously selected files to your Dropbox public folder

![Screenshot of a folder action for desktop](http://www.matteoagosti.com/images/posts/Screen Shot 2012-10-28 at 10.49.01 PM.png)

Now the tricky part, add a `Run Shell Script` action, selecting `as argument` in the `Pass input` option and adding the following code:

```
dropbox="http://dl.dropbox.com/u/XXXXXX/"
url=${dropbox}$(basename $1)
encodedUrl=`echo $url | sed 's/ /%20/g'`
echo -ne ${encodedUrl} | pbcopy
afplay /System/Library/Sounds/Glass.aiff 
```

Replace `XXXXXX` with your Dropbox user id. To find it, just copy the URL of one of your public files.

![Screenshot of a folder action for desktop](http://www.matteoagosti.com/images/posts/Screen Shot 2012-10-28 at 11.04.54 PM.png)

Now save the folder action and have fun!