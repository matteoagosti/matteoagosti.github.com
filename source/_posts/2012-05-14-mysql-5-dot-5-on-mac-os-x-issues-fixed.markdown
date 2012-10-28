---
layout: post
title: "MySQL 5.5 on Mac OS X issues fixed"
date: 2012-05-14 21:46
comments: true
categories: [Mac OS X, Development, Fixes]
---
If you have recently installed MySQL 5.5 on Mac OS X you have probably faced some issues with anything that relies on libmysqlclient as, for example, the handy `node.js db-mysql` or `gem mysql2` modules.

<!--more-->

The problem lies into the fact that the system doesn't know the full path to the library. To solve the issue, you simply have to setup the full path in your enviroment variables. Assuming you have a UNIX system with a Bash shell you could simply run the following command:

{% codeblock lang:bash %}
echo 'export DYLD_LIBRARY_PATH=/usr/local/mysql/lib:$DYLD_LIBRARY_PATH' >> ~/.bash_profile
{% endcodeblock %}
