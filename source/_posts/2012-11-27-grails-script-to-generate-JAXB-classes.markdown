---
layout: post
title: "Grails script to generate JAXB classes"
date: 2012-11-27 19:00
comments: true
categories: Grails
---
<p>I think Groovy's <a href="http://groovy.codehaus.org/api/groovy/xml/MarkupBuilder.html">MarkupBuilder</a> is great and I use it often.  Sometimes though, I need to generate a complex XML document that conforms to an XSD.  MarkupBuilder often leads to markup and business logic being mixed together much like HTML and Java code gets mixed in complex JSP pages. In cases like this, I like to use JAXB instead.  I can generate classes from the XSD, load up an object model and let JAXB generate the markup.</p>
<p>It's fairly easy to create a script to handle generating the JAXB classes for you.  JAXB comes with a handy Ant task that takes your XSD and generates classes. The trick is passing the build classpath to the task.  There is a "grailsSettings" variable available in scripts that contains <a href="https://github.com/grails/grails-core/blob/master/grails-bootstrap/src/main/groovy/grails/util/BuildSettings.groovy">BuildSettings</a>.  We can get the list of build dependencies files from this class to generate the classpath.  Here is a simple example that takes the Microsoft <a href="http://msdn.microsoft.com/en-us/library/ms256485.aspx">books.xsd</a> in the etc/schema directory and generates the code to the src/java directory:</p>
<script src="https://gist.github.com/4158702.js"> </script>
<p>UPDATE: Sometimes having the JAXB jars on your path can cause problems with your application.  In those cases, you call the generation class directly:</p>
<script src="https://gist.github.com/4186366.js"> </script>

