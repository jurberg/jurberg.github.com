---
layout: post
title: "Converting XML to Flat Files with Groovy"
date: 2012-11-11 19:00
comments: true
categories: Grails
---
<p>When integrating systems, you often need to convert data from one format to another.  For example, you may receive data from a web service in XML and need to convert it to a flat file for a legacy system.  It's usually not as simple as just moving data from a tag to a column either.  There's often calculations and formatting required.  Most often the code has the work of parsing, converting and formatting all mixed together.  With a dynamic language like Groovy which has closures, we can abstract much of that work.  For our example, we'll take a subset of the <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/ms762271(v=vs.85).aspx">Microsoft sample books.xml file</a>:</p>
<script src="https://gist.github.com/4056093.js"> </script>
<p>We would like to provided a mapping for each column that allows us to pull data out of the XML file and define the width of the column.  We'll keep it simple to start with by assuming a fixed width flat file with all columns left justified.  Following is a simple mapping using an array of maps:</p>
<script src="https://gist.github.com/4056280.js"> </script>
<p>Each closure should get a GPathResult to a "row" in the XML file.  The closure should return the text that should go in the column.  We'll need to supply the tag that identifies the tag in the XML file to pass to the closure.  Following is an example test using our books data:</p>
<script src="https://gist.github.com/4056102.js"> </script>
<p>The implementation is fairly simple.  We process each "tag" in the XML.  For each mapping, we set the delegate for the closure to a map with the tag name and the GPathResults for that tag.  The closure can then be executed to get the text for the column:</p>
<script src="https://gist.github.com/4056079.js"> </script>
<p>The above piece of code abstracts the processing into 8 lines of code.  It is a testament to the power of a dynamic language that provides closures.</p>
