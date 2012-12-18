---
layout: post
title: "The rationale behind andthewinner.is, a platform for viral contests on Twitter"
date: 2012-11-16 06:22
comments: true
categories: [Development, JavaScript]
---

On November 10-11th 2012 I attended the [Node.js conference](http://nodejsconf.it/ "Node.js conference Italy homepage") in Brescia (Italy) and participated in the hackathon to experiment and have fun with JavaScript on the server side. Together with [@nickbalestra](https://twitter.com/nickbalestra "Nick Balestra's Twitter profile") we decided to experiment the realtime aspect of Node.js and as we already had background on dealing with social media we decided to build a platform for viral contests on Twitter. We came up with [andthewinner.is](http://www.andthewinner.is "andthewinner.is homepage") and with this post I'd like to share the details on how we built it.

<!-- more-->

![Screenshot of a andthewinner.is homepage](http://www.matteoagosti.com/images/posts/andthewinneris-homepage-screenshot.png)

## Introduction

Before digging into technical details I'd like to quickly explain the reason why we came up with this project. As [Nick mentioned in his post](http://feathe.rs/201211111353 "Nick Balestra's post on the creation of andthewinner.is") when we attended TechCrunch 2012 event in Rome we were so shocked by the voting done using [pieces of paper](http://www.flickr.com/photos/michele_ficara_manganelli/8043905616/in/set-72157631667552679 "A picture of people voting for startup ideas using pieces of paper at TechCrunch 2012 event in Rome") that we immediately decided to came up with a solution that should be both tech and social.

We started to think about how voting is done in general (written voting, tele voting, etc…) and decided to focus on a public voting system that should be both quick and viral: Twitter! As of late 2011 there are approximately 250 million Tweets and most of them are part of a discussion identified by an hashtag and will likely include a mention to one or more Twitter users. In addition, a Tweet can be also be done via SMS, avoiding 3G/WiFi network issues (many complained about our point for TechCrunch saying that there was no Internet). If this is not an argument strong enough, you just have to look at the recent 2012 US presidential campaign whose result was totally driven by social media. For these reasons we decided to base andthewinner.is on Twitter and leverage on its sharing mechanism to make contests viral.

The general idea is fairly simple: set an #hashtag to identify the contest and use @mention as identifier for contestants. The point of having an hashtag for the contest was introduced to restrict the scope of voting: counting only mentions would simply lead to the top mentioned person overall on Twitter, whereas having an hashtag could actually set up a context. One nice aspect of this approach is that every time a vote is tweeted, anybody tuned on the hashtag will see the vote and could potentially retweet it, thus leading to another vote! In addition, the  tweet text can be anything, it is enough to include the hashtag and one (or even more than one) mention; this means that I can express my vote not just by picking up an option but also attaching a comment to it. This mechanism wouldn't probably fit strict rules of political elections, but it could definitely foster promotion of public events, TV programs and so on.

We created a couple of contests to try it out, you can check them out here:

* [Node.js Conference Speakers](http://www.andthewinner.is/nodejsconfit "Node.js Conference Speakers contest") where conference attendance could vote their favourite speaker.
* [X Factor 2012 Italia Judges](http://www.andthewinner.is/xf6italia "X Factor 2012 Italia Judges contest") where X Factor TV program fans could vote their favourite judge.

At the moment andthewinner.is is not yet opened to the public in terms of contest creation, but only for contest participation. This has to deal with limitations in the free Twitter API that I am going to analyse later on. However, we are looking forward to run popular contests and if you feel you could have one please contact us via [Twitter](https://twitter.com/and_thewinneris "andthewinner.is' Twitter profile").

## Architecture

When designing the architecture of andthewinner.is we started to think about splitting it into different components as to keep it maintainable and easily scalable. Dealing with Twitter, a potential source for a huge amount of data, means planning ahead something that is dedicated just to it as when traffic grows this could be spread across different machines. We came up with two macro blocks of components: a *crawler* for fetching tweets and a *server app* to provide the website experience. A *MySQL* database is also used for storing contests and votes. Everything has been developed using node.js and relying on the following modules:

* [request](https://github.com/mikeal/request "Homepage of request module for node.js") so far one of the most complete module for handling HTTP/HTTPS requests with support for OAuth and streams; basically with this module we can handle any request to Twitter
* [express](https://github.com/visionmedia/express "Homepage of express module for node.js") the web framework we use for the server app, handling routes, views rendering, etc…
* [connect-mysql-session](https://github.com/CarnegieLearning/connect-mysql-session "Homepage of connect-mysql-session module for node.js") a MySQL session store for express
* [less-middleware](https://github.com/emberfeather/less.js-middleware "Homepage of less-middleware module for node.js") a LESS middleware for express
* [hbs](https://github.com/donpark/hbs "Homepage of hbs module for node.js") a handlebars template engine for express
* [mysql](https://github.com/felixge/node-mysql "Homepage of mysql module for node.js") so far the best module (pure JavaScript) to handle MySQL from node.js; the [author](http://felixge.de/ "Felix Geisendörfer personal website") is mad about performances and his talk at the conference just confirmed that this module is the way to go (an old [talk](http://www.youtube.com/watch?v=Kdwwvps4J9A "Video of Felix Geisendörfer talking about benchmarking") but on the same topic)
* [socket.io](https://github.com/LearnBoost/socket.io "Homepage of socket.io module for node.js") handling web sockets from the server point of view
* [socket.io-client](https://github.com/LearnBoost/socket.io-client "Homepage of socket.io-client module for node.js") handling web sockets from the client point of view

### Crawler

The crawler is responsible for handling running contests and deal with the setup / processing of Twitter stream.

As the filtering happens on all tweets and is not restricted to an authenticated user's timeline, the [public stream API](https://dev.twitter.com/docs/api/1.1/post/statuses/filter "Twitter public stream API documentation") has been chosen as the only viable option. However, there are some limitations to bear in mind 

* The number of messages sent to the client is limited to a very small fraction of the total volume of tweets (firehose). If there are more tweets that would match the criteria, they'll get discarded.
* The stream can be filtered with up to 400 keywords, 5000 mentions, 25 location boxes.
* In theory no more than one connection per IP is allowed, even though we went stable up to 11 connections (but this is likely to get banned!). If you try to open two connections to the stream for the same authenticated user the oldest one will get closed, however multiple streams for different users works well (again, until you get banned!).

Obviously for the hackathon we could cope with these limitations, but in a real scenario you are forced to switch to [Twitter certified data resellers](https://dev.twitter.com/programs/twitter-certified-products/products "Twitter certified products page"). That is the main reason why we are not yet opening andthewinner.is to the public.

The logic behind the crawler is fairly simple: it opens a single connection to the stream API using an internal authenticated user and wraps all hashtags into a comma separated values query. Resulting tweets are then processed to find out the related contest and matching participants. This would potentially allow for 400 simultaneously running contests, but depending on their "popularity" the stream cap could be reached quite rapidly. So far, we tested it out with popular hashtags and haven't reached the cap yet.

Every time a new vote is matched, the database gets updated by increasing contest counters; we decided not to store the originating tweet, but that could be easily done and used for some sort of restrictions (e.g. max number of tweets from the same user) or for statistics (e.g. geo distribution of voters). In any case, when dealing with Twitter contents, it is important to know that if you store data you can't display them publicly, but only in a private dashboard.

As to prevent polling the database to find out when a new vote is counted, the crawler provides a socket access to the server app and broadcast the contest data whenever a new update occurs. 

### Server app

The server app is responsible for providing the overall website experience. It is built as an express application and relies on handlebars for rendering page layouts. Page styling is done using a LESS file that is compiled every time the app is started and it includes a responsive layout for a better mobile experience. When the server app is started, it immediately connects to the crawler waiting for contest updates. In addition, it creates a server socket for updating connected clients on specific contest updates; for every contest, in fact,  a channel is created and only its related votes are broadcasted.

Whenever a contest page is requested, the database is queried for fetching data, the handlebars template is compiled and the full page is sent to the client browser; until this point it is not necessary to have support for JavaScript on the client and this allows for fast rendering on mobile devices.

Immediately after the page is loaded the client opens a socket connection with the server app by tuning on the contest channel, thus receiving updates only for the requested contest.

So far everything is pretty standard, there is however a technical solution that in my opinion is worth mentioning as it proves how effective it is to work with JavaScript on both server and client side.

The socket communication exchanges JSON data, so every time a contest receives a vote, a JSON representation of contest and participants is sent to the client. This data structure is normalised from the server as to prevent calculations on the client side; what the server does is basically sorting out rankings, find out if there are ties and so on. As we had mobile devices in mind, we wanted to minimise the time needed for calculus and if you think about contests with a rate of 10 votes per second, unless you rate limit the updates on socket, this would lead to an unusable page. So, the first point of contest normalisation is solved on the server, but what about page rendering on the client? The trick here was to rely on handlebars on the client side too: we precompiled the template we use on the server side (yes, exactly the same template!) as to speed up any operation on the client, and whenever a new update is sent over socket the template is applied to the data structure and the contest HTML replaced with the new one. This gave us an amazing result as we could easily handle 10 votes per second on a mobile device, whereas with the standard solution of processing JSON and rendering the page we could reach a max of 2.7 votes per second.

To sum up, the server app provides a mechanism for normalising contests and the resulting JSON structure is used for compiling the same handlebars file on both server and client. Access to the database is done only on page reload whereas in page updates are done using per-contest channeled socket communication. Did I say already that everything is done in JavaScript? Isn't that awesome? :-)

## Conclusions

With this post I tried to explain the overall architecture of andthewinner.is, a platform for creating viral contests on Twitter. It is an hackathon project whose goal was to play with node.js and realtime communication; despite being a 24 hours coding / designing result, a lot of efforts went in the overall concept and we strongly believe it has a remarkable growing potential. Our goal was also to show the unexplored capabilities of social media platforms and, in some way, to tease traditional voting systems that in our opinion are not effective and viral.

Due to limitations of free APIs the platform is opened only for the voting part, this means that if you want to try out a contest you'll have to [contact us](mailto:andthewinner.is@beyounic.com "Email address of andthewinner.is team"); however, by subscribing commercial data plans any limitation could be bypassed.