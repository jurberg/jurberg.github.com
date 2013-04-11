---
layout: post
title: "Hosting a Maven internal repository on a file share"
date: 2011-02-17 19:00
comments: true
categories: Java 
---
<p>I spent some time today setting up an internal repository for my Maven build.  I wanted to host just the artifacts that are used by my projects and not allow the project build to go outside of that repository.  I first setup my repositories like this:</p>
<pre>
&lt;<span style="color:#0000ff">repositories</span>a&gt;
  &lt;<span style="color:#0000ff">repository</span>&gt;
    &lt;<span style="color:#0000ff">id</span>&gt;central&lt;/<span style="color:#0000ff">id</span>&gt;
    &lt;<span style="color:#0000ff">name</span>&gt;My Company's Internal Repository&lt;/<span style="color:#0000ff">name</span>&gt;
    &lt;<span style="color:#0000ff">url</span>&gt;file:///server/Maven/repo/&lt;/<span style="color:#0000ff">url</span>&gt;
  &lt;/<span style="color:#0000ff">repository</span>&gt;
&lt;/<span style="color:#0000ff">repositories</span>&gt;
</pre>
<p>The idea behind this setup is that I overrode the "central" repository and pointed it to my internal repository.  This way Maven would not try to go out to repo1.maven.org if it did not find something.</p>
<p>And it did not work.  After much research and trying many things, it turns out my URL needs to look like this:</p>
<pre>
&lt;<span style="color:#0000ff">url</span>&gt;file://\\server\Maven\repo\&lt;/<span style="color:#0000ff">url</span>&gt;
</pre>
<p>Hope this helps someone in the future trying to setup a repository on a share.</p>
