---
layout: post
title: "A simple JavaScript module library"
date: 2013-02-26 21:00 
comments: true
categories: JavaScript Grails
---
<p>When developing JavaScript, I like to break my code into modules and define what modules the depend on.  <a href="http://requirejs.org/">Require.JS</a> is a great tool for this, but it also means bringing in module loading.  I'm on a project using <a href="http://grails.org/">Grails</a> that has a moderate amount of JavaScript.  Grails provides a resource plugin that handles bundling of resources.  I would still like to use the Require.JS style of defining modules and their dependencies while still loading scripts the normal way so Grails is happy.</p>
<p>I ended up creating <a href="https://github.com/jurberg/define.js">define.js</a>.  This libary provides a simple 'define' and 'require' method.  The 'define' uses the global object to hold the modules so they are also available in html handlers.  Using the global object as the module list also allows us to treat third party libraries such as jQuery as dependencies.</p>
<p>The following example creates a module that depends on the domain.person module, jQuery and the window object.  Since we are using the global to store modules, we can treat jQuery and window just like any other dependency we have defined.</p>
<script src="https://gist.github.com/jurberg/5044824.js"></script>
<p>Using the global object to hold modules also allows us to access them in html</p>
<script src="https://gist.github.com/jurberg/5044843.js"></script>
