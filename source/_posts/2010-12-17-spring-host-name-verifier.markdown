---
layout: post
title: "Using a HostnameVerifier with Spring Web Services"
date: 2010-12-17 19:00
comments: true
categories: Java 
---
<p>I was working with a web service from a site that uses SSL with a certificate that was self-signed.  When attempting to make the calls, I received the error:</p>
<pre>
    javax.net.ssl.SSLHandshakeException: java.security.cert.CertificateException: No name matching {web address} found
</pre>
<p>The workaround for this issue is to provide a HostnameVerifer that skips the host name verification process.  There is an example of how to do this <a href="http://www.jroller.com/hasant/entry/no_subject_alternative_names_matching">here.</a></p>
<p>I ran into this issue after I had a nice clean codebase using Spring's WebServiceTemplate and RestTemplate.  It took some digging, but I was able to find the spots where I had to install the verifier.  We start with a NullHostnameVerifier that returns true for everything:</p>
<pre>
<span style="color:#a020f0">public</span> <span style="color:#a020f0">class</span> <span style="color:#228b22">NullHostnameVerifier</span> <span style="color:#a020f0">implements</span> <span style="color:#228b22">HostnameVerifier</span> {
   <span style="color:#a020f0">public</span> <span style="color:#a020f0">boolean</span> verify(<span style="color:#228b22">String</span> hostname, <span style="color:#228b22">SSLSession</span> session) {
      <span style="color:#a020f0">return</span> <span style="color:#b8860b">true</span>;
   }
}
</pre>
<p>The <b>org.springframework.ws.client.core.WebServiceTempate</b> extends <b>org.springframework.ws.client.support.WebServiceAccesor</b> which uses a <b>org.springframework.ws.client.support.WebServiceMessageSender</b>.  The WebServiceMessage sender for HTTPS is <b>org.springframework.ws.transport.http.HttpsUrlConnectionMessageSender</b> which is found in the spring-ws-support-1.5.9.jar. We need to create one of these and set the HostnameVerifier into it and pass it along to the WebServicesTemplate:</p>
<pre>
<span style="color:#a020f0">public</span> <span style="color:#a020f0">void</span> setWebServicesTempalate(<span style="color:#228b22">WebServicesTemplate</span> template) {
   <span style="color:#228b22">HostnameVerifier</span> verifier = <span style="color:#a020f0">new</span> <span style="color:#228b22">NullHostnameVerifier</span>();
   <span style="color:#228b22">HttpsUrlConnectionMessageSender</span> sender = <span style="color:#a020f0">new</span> <span style="color:#228b22">HttpsUrlConnectionMessageSender</span>();
   sender.setHostnameVerifier(verifier);
   template.setMessageSender(sender);
   <span style="color:#a020f0">this</span>.template = template;
}
</pre>
<p>The <b>org.springframework.web.client.RestTemplate</b> is setup differently.  It uses a <b>org.springframework.http.client.ClientHttpRequestFactory</b> to handle the connections.  Stepping thru the code, I found the RestTemplate using <b>org.springframework.http.client.SimpleClientHttpRequestFactory</b> which has a protected prepareConnection(...) method I could override and catch the HttpsURLConnection which I could set the verifier in.  First we need  our own ClientHttpRequestFactory:</p>
<pre> 
<span style="color:#a020f0">public</span> <span style="color:#a020f0">class</span> <span style="color:#228b22">MySimpleClientHttpRequestFactory</span> <span style="color:#a020f0">extends</span> <span style="color:#228b22">SimpleClientHttpRequestFactory</span> {

   <span style="color:#a020f0">private</span> <span style="color:#a020f0">final</span> <span style="color:#228b22">HostnameVerifier</span> verifier;

   <span style="color:#a020f0">public</span> <span style="color:#228b22">MySimpleClientHttpRequestFactory</span>(<span style="color:#228b22">HostnameVerifier</span> verifier) {
      <span style="color:#a020f0">this</span>.verifier = verifier;
   }

   <span style="color:#b8860b">@Override</span>
   <span style="color:#a020f0">protected</span> <span style="color:#a020f0">void</span> prepareConnection(<span style="color:#228b22">HttpURLConnection</span> connection, <span style="color:#228b22">String</span> httpMethod) throws <span style="color:#228b22">IOException</span> {
      <span style="color:#a020f0">if</span> (connection <span style="color:#a020f0">instanceof</span> <span style="color:#228b22">HttpsURLConnection</span>) {
         ((<span style="color:#228b22">HttpsURLConnection</span>) connection).setHostnameVerifier(verifier);
      }
      <span style="color:#a020f0">super</span>.prepareConnection(connection, httpMethod);
   }

}
</pre>
<p>Now we can use that as the request factory in our RestTemplate:</p>
<pre>
<span style="color:#a020f0">public</span> <span style="color:#a020f0">void</span> setRestTemplate(<span style="color:#228b22">RestTemplate</span> template) {
   <span style="color:#228b22">HostnameVerifier</span> verifier = <span style="color:#a020f0">new</span> <span style="color:#228b22">NullHostnameVerifier</span>();
   <span style="color:#228b22">MySimpleClientHttpRequestFactory</span> factory = <span style="color:#a020f0">new</span> <span style="color:#228b22">MySimpleClientHttpRequestFactory</span>(verifier);
   template.setRequestFactory(factory);
   <span style="color:#a020f0">this</span>.template = template;
}
</pre>
<p>Once the NullHostnameVerifer is in place in the WebServiceTemplate and the RestTemplate, we will no longer see the error</p>
