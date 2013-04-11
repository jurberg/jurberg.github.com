---
layout: post
title: "Running the Require.JS optimizer in a Grails build"
date: 2013-02-02 22:06
comments: true
categories: Grails
---
<p>The <a href="http://grails.org/plugin/resources">Grails Resource Plugin</a> provides some dependency management for your JavaScript resources by allowing you to define modules of scripts, including their dependencies.  This works fine for applications with small numbers of JavaScript.  If you're writing an application with a large amount of JavaScript, having to keep those dependencies in sync in a separate Groovy config file can get difficult.  Enter <a href="http://requirejs.org/">Require.JS</a>.  Require.JS is an <a href="https://github.com/amdjs/amdjs-api/wiki/AMD">Asynchronous Module Definition (AMD) API</a> for JavaScript.  It allows you to define modules in JavaScript, including what modules it depends on.  The module loader will then load the dependencies when needed.  This is easier to manage in a large JavaScript project since you define the dependencies in each JavaScript file.</p>
<p>Require.JS comes with an <a href="http://requirejs.org/docs/optimization.html">optimizer</a> that will crawl thru all your code, evaluate the dependencies, merge them into one file and minimize it.  The optimizer code is in a JavaScript file that can be run using <a href="http://nodejs.org/">Node.JS</a> or <a href="https://developer.mozilla.org/en-US/docs/Rhino">Rhino</a>.  We would like to run this as part of the Grails build during the war command.</p>
<p>The Grails war command will copy the resources for the war file into a staging directory.  Before the war file is built, we will get a "CreateWarStart" event which gets the war name and the staging directory.  This is where we can call the optimizer command. Following is an example that can be placed in our _Events.groovy script.   It assumes you've placed the Rhino js.jar and the Require.JS r.js optimizer script in the Scripts folder.</p>
<script src="https://gist.github.com/4699237.js"></script>
<p>We run the script using the Java command instead of ant.java to avoid any issues from code loaded by Grails.  The scripts are run from the project root, so we reference the js.jar and r.js files in the .\Scripts directory.  The JavaScript files will be copied to the stagingDir in the js folder.  We'll set the baseUrl to that directory and output the build file there also.  While the example does not show it, you can also pass removeCombined=true to remove the JavaScript files that were combined.</p>
<p>The final step is to update our GSP file so it uses the -build version of the main script.  Following is an example of the script line adds -build if we are running in a war</p>
<script src="https://gist.github.com/4699505.js"></script>
