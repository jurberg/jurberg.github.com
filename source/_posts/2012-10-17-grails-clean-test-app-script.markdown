---
layout: post
title: "Grails clean-test-app script"
date: 2012-10-17 19:00
comments: true
categories: Grails
---
I've been developing Grails apps in Spring Tool Suite. STS will compile your Groovy code as you save it just as Eclipse does for Java code.  This is great during coding, but can cause issues when it's time to check in.  If you deleted a class and then run <span style="font-family:Monospace;">test-app</span>, that class may not be cleared from your target/classes and tests may pass that shouldn't.  Before checking in, I now need to run <span style="font-family:Monospace;">clean</span>, then <span style="font-family:Monospace;">refresh-dependencies</span>, then <span style="font-family:Monospace;">test-app</span>.  To save some time, I created the following script that will do it all at once.  Now I can just run <span style="font-family:Monospace;">clean-test-app</span>.
<br/>
<script src="https://gist.github.com/3909243.js?file=CleanTestApp.groovy"></script>
