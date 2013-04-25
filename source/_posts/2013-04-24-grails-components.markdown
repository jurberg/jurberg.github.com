---
layout: post
title: "Creating Grails UI 'Components'" 
date: 2013-04-24 21:00 
comments: true
categories: Grails JavaScript  
---
<p>I'm working on a Grails application with a large amount of JavaScript.  I often have UI 'components' that have markup, CSS and JavaScript associated with them that are used on several pages.  A couple of examples are shopping carts and dialogs.  I've found a good way to organize this code is to make use of the resources plugin to put the pieces together into modules that I can then place into the pages.  As an example, I made a simple dialog that has a GSP for the layout, a CSS file with some styling and JavaScript to open the dialog.  The first step is to create some markup for the dialog in a template:</p>
<script src="https://gist.github.com/jurberg/5457059.js"></script>
<p>I added a require at the top to pull in the module definition from the ApplicationResources file.  This will pull the CSS and JavaScript for the component in each page that uses the template.</p> 
<script src="https://gist.github.com/jurberg/5457071.js"></script>
<p>A little CSS and JavaScript will bring the component to life:</p>
<script src="https://gist.github.com/jurberg/5457099.js"></script>
<script src="https://gist.github.com/jurberg/5457104.js"></script>
<p>I can now include the template on the page and all the pieces of my component are pulled in.  I can now open the dialog from my page.</p>
<script src="https://gist.github.com/jurberg/5457112.js"></script>

