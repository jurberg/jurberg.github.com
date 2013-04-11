---
layout: post
title: "An easier way to exclude methods from Grails JMX plugin services"
date: 2012-10-29 19:00
comments: true
categories: Grails
---
The <a href="http://grails.org/plugin/jmx">Grails JMX plugin</a> is a great way to quickly make a service available through JMX.  Things get trickier when you want to only expose a few methods on the service.  The plugin provides the option to list methods to exclude, but you need to list all of them including the ones Grails added to the service.  Fortunately, we can use the metaclass to help us out.  We can add a list of the few methods we want exposed and then use to that to generate the list of exclude method as follows:
<script src="https://gist.github.com/3977988.js"> </script>
