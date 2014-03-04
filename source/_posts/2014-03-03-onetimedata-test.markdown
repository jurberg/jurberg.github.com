---
layout: post
title: "Unit Testing with the OneTimeData Plugin" 
date: 2014-03-03 21:00 
comments: true
categories: Grails  
---
<p>The <a href="http://grails.org/plugin/one-time-data">OneTimeData</a> plugin is handy when flash doesn't work for you. I recently had to use it in an application that required an extra redirect between controller actions.  The OneTimeData plugin does not come with any unit testing assistance, so I created a simple mixin that adds the OneTimeData methods to a controller under test and backs them with a mock map on the spec to make it easy to access what the controller passed. The following class creates the mixin.</p>
<script src="https://gist.github.com/jurberg/9338727.js"></script>
<p>Using it requires two steps: 1) Add the mixin using the @TestMixn annotation and 2) call mockOneTimeData passing in the spec and the controller.</p>
<script src="https://gist.github.com/jurberg/9338807.js"></script>
<p>If anyone knows of a way to capture the spec and controller inside the mixin, please comment. Thanks!</p>
