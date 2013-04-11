---
layout: post
title: "Transmitting Unicode data between C++ and Java"
date: 2009-10-09 19:00
comments: true
categories: Java C++
---
We ran into an issue this week were we had to figure out how to transmit Unicode data between a C++ program and a Java program using TCP.  We have a working program that is sending ASCII just fine.  We expected the C++ Unicode work would be difficult and the Java work would be easy.  It turns out the C++ side was not as difficult as we thought.  The Java side turned out to be easy, but it took some time to figure out how to make it work.  Here's a quick explanation of what we found:

<h3>C++</h3>
<p>Most of our C++ code is using the standard library (i.e. std::string) and boost::asio for the networking.  After a bit if research, I created a tstring.hpp file using code I found <a href="http://www.rioki.org/2008/02/18/stdtstring/">here</a> and <a href="http://www.codeproject.com/KB/stl/upgradingstlappstounicode.aspx">here</a>.  I changed the networking code to use the new std::tstring instead of std::string and was able to deal with Unicode data.  Here's an example of the asio code to read data sent from Java.  Java sends a header with "file0000000000" where the 10 digits contain the data length, followed by the data itself.  Notice the buffer size is the size * sizeof(TCHAR) to account for the fact that wide char are twice the size of normal char.</p><pre><font color="#a020f0">class</font> <font color="#228b22">session</font> : <font color="#a020f0">public</font> <font color="#228b22">boost</font>::enable_shared_from_this&lt;session&gt;
{
<font color="#5f9ea0">public</font>:
  <font color="#228b22">void</font> <font color="#0000ff">read_header</font>()
  {
    buffer_.resize(14);
    boost::asio::async_read(
      socket_,
      boost::asio::buffer(buffer_, buffer_.size() * <font color="#a020f0">sizeof</font>(TCHAR)),
      boost::bind(
	&amp;session::handle_read_header, 
	shared_from_this(),
	boost::asio::placeholders::error,
	boost::asio::placeholders::bytes_transferred));
  }

  <font color="#228b22">void</font> <font color="#0000ff">handle_read_header</font>(<font color="#a020f0">const</font> boost::system::error_code&amp; error, <font color="#228b22">size_t</font> <font color="#b8860b">bytes_transferred</font>)
  {
    <font color="#a020f0">if</font> (error) <font color="#a020f0">return</font>;

    std::tstring data(buffer_.begin(), buffer_.end());
    <font color="#228b22">short</font> <font color="#b8860b">length</font> = boost::lexical_cast&lt;<font color="#228b22">short</font>&gt;(data.substr(4));

    buffer_.resize(length);
    boost::asio::async_read(
      socket_,
      boost::asio::buffer(buffer_, buffer_.size() * <font color="#a020f0">sizeof</font>(TCHAR)),
      boost::bind(
	&amp;session::handle_read_data, 
	shared_from_this(),
	boost::asio::placeholders::error,
	boost::asio::placeholders::bytes_transferred));
  }

  <font color="#228b22">void</font> <font color="#0000ff">handle_read_data</font>(<font color="#a020f0">const</font> boost::system::error_code&amp; error, <font color="#228b22">size_t</font> <font color="#b8860b">bytes_transferred</font>)
  {
    <font color="#a020f0">if</font> (error &amp;&amp; error.value() != 2) <font color="#a020f0">return</font>;
    std::tstring data(buffer_.begin(), buffer_.end());
    <font color="#b22222">// process data
</font>    read_header();
  }

<font color="#5f9ea0">private</font>:
  tcp::socket socket_;
  std::<font color="#228b22">vector</font>&lt;<font color="#228b22">TCHAR</font>&gt; <font color="#b8860b">buffer_</font>;
  <font color="#228b22">int</font> <font color="#b8860b">count_</font>;

};
</pre>
<h3>Java</h3>
<p>For an example, we created a simple Java program that would load in a Unicode file, append a header and send it to the C++ code above.  What took a great deal of time was figuring out how to get the data in the correct format.  It turns out the way the C++ program was looking for data was in UTF-16LE format.  We took our data string and called getBytes("UTF-16LE") to get the data in the correct format to send.</p><pre><font color="#228b22">String</font> <font color="#b8860b">data</font> = getFileContents();
<font color="#228b22">Formatter</font> <font color="#b8860b">f</font> = <font color="#a020f0">new</font> <font color="#228b22">Formatter</font>();
<font color="#228b22">String</font> <font color="#b8860b">header</font> = f.format(<font color="#bc8f8f">&quot;file%010d&quot;</font>, data.length() * 2).toString(); 
data = header + data;
<font color="#228b22">byte</font>[] <font color="#b8860b">buffer</font> = data.getBytes(<font color="#bc8f8f">&quot;UTF-16LE&quot;</font>);
<font color="#228b22">Socket</font> <font color="#b8860b">socket</font> = <font color="#a020f0">new</font> <font color="#228b22">Socket</font>(address, port);
<font color="#228b22">BufferedOutputStream</font> <font color="#b8860b">bis</font> = <font color="#a020f0">new</font> <font color="#228b22">BufferedOutputStream</font>(socket.getOutputStream());
<font color="#a020f0">try</font> {
   bis.write(buffer);
   bis.flush();
} <font color="#a020f0">finally</font> {
   bis.close();
}
</pre><p>In the end, this turns out to be easy to do.  It just takes a bit of time to figure out the settings.  We had to spend time searching for bits and pieces of info to put this together.  If you happen to know any good resources to help others with this, feel free to add a comment.</p>
