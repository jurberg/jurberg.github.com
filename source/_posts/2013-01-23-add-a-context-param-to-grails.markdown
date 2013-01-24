---
layout: post
title: "Add a context-param to Grails"
date: 2013-01-23 22:06
comments: true
categories: Grails
---
With the recent <a href="https://www.aspectsecurity.com/uploads/downloads/2012/12/Remote-Code-with-Expression-Language-Injection.pdf">Spring vulnerability</a>, we thought it would be a good idea to disable EL in our Grails application.  To do this, we need to add a context-param to the beginning of the generated web.xml file.  Rather than using install-templates, I used the eventWebXmlEnd event to insert it.  XMLSlurper would have been nice, but it does not have a simple way to insert a node.  So instead I used the XmlParser and came up with the following.

<script src="https://gist.github.com/4617534.js"></script>
