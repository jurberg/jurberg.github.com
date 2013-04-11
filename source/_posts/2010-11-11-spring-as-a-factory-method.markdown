---
layout: post
title: "Using Spring as a Factory Method"
date: 2011-11-11 19:00
comments: true
categories: Java 
---
One common strategy in object-oriented program is to use the <a href="http://en.wikipedia.org/wiki/Dependency_inversion_principle">dependency inversion principle</a> to decouple high level code from the low level details. To demonstrate this, let's use the movie example from <a href="http://www.amazon.com/Refactoring-Improving-Design-Existing-Code/dp/0201485672">Martin Fowler's Refactoring book</a>.
<br/><br/>
After some initial refactoring, we have a Movie class that can calculate it's charge based on days rented:
<pre><span style="color:#000000;">// </span><span style="color:#b22222;">From book: 'Refactoring' by Martin Fowler
</span><span style="color:#a020f0;">public</span> <span style="color:#a020f0;">class</span> <span style="color:#228b22;">Movie</span> {

   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">static</span> <span style="color:#a020f0;">final</span> <span style="color:#a020f0;">int</span> <span style="color:#228b22;">CHILDRENS</span> = 2;
   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">static</span> <span style="color:#a020f0;">final</span> <span style="color:#a020f0;">int</span> <span style="color:#228b22;">NEW_RELEASE</span> = 1;
   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">static</span> <span style="color:#a020f0;">final</span> <span style="color:#a020f0;">int</span> <span style="color:#228b22;">REGULAR</span> = 0;

   <span style="color:#a020f0;">double</span> charge(<span style="color:#a020f0;">int</span> daysRented) {
      <span style="color:#a020f0;">double</span> result = 0;
      <span style="color:#a020f0;">switch</span> (getPriceCode()) {
	<span style="color:#a020f0;">case</span> <span style="color:#228b22;">Movie</span>.<span style="color:#228b22;">REGULAR</span>:
	   result += 2;
	   <span style="color:#a020f0;">if</span> (daysRented &gt; 2)
	      result += (daysRented - 2) * 1.5;
	   <span style="color:#a020f0;">break</span>;
	<span style="color:#a020f0;">case</span> <span style="color:#228b22;">Movie</span>.<span style="color:#228b22;">NEW_RELEASE</span>:
	   result += daysRented * 3;
	   <span style="color:#a020f0;">break</span>;
	<span style="color:#a020f0;">case</span> <span style="color:#228b22;">Movie</span>.<span style="color:#228b22;">CHILDRENS</span>:
	   result += 1.5;
	   <span style="color:#a020f0;">if</span> (daysRented &gt; 3)
	      result += (daysRented - 3) * 1.5;
	   <span style="color:#a020f0;">break</span>;
	}
	<span style="color:#a020f0;">return</span> result;
   }

}</pre><p>Next we replace the switch statement with polymorphism using the state pattern:</p>
<pre><span style="color:#000000;">// </span><span style="color:#b22222;">From book: 'Refactoring' by Martin Fowler
</span><span style="color:#a020f0;">public</span> <span style="color:#a020f0;">class</span> <span style="color:#228b22;">Movie</span> {

   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">static</span> <span style="color:#a020f0;">final</span> <span style="color:#a020f0;">int</span> <span style="color:#228b22;">CHILDRENS</span> = 2;
   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">static</span> <span style="color:#a020f0;">final</span> <span style="color:#a020f0;">int</span> <span style="color:#228b22;">NEW_RELEASE</span> = 1;
   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">static</span> <span style="color:#a020f0;">final</span> <span style="color:#a020f0;">int</span> <span style="color:#228b22;">REGULAR</span> = 0;

   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">static</span> <span style="color:#228b22;">Price</span> newPrice(<span style="color:#a020f0;">int</span> priceCode) {
      <span style="color:#a020f0;">switch</span> (priceCode) {
         <span style="color:#a020f0;">case</span> <span style="color:#228b22;">REGULAR</span>:
	    <span style="color:#a020f0;">return</span> <span style="color:#a020f0;">new</span> <span style="color:#228b22;">RegularPrice</span>();
	 <span style="color:#a020f0;">case</span> <span style="color:#228b22;">NEW_RELEASE</span>:
	    <span style="color:#a020f0;">return</span> <span style="color:#a020f0;">new</span> <span style="color:#228b22;">NewReleasePrice</span>();
	 <span style="color:#a020f0;">case</span> <span style="color:#228b22;">CHILDRENS</span>:
	    <span style="color:#a020f0;">return</span> <span style="color:#a020f0;">new</span> <span style="color:#228b22;">ChildrensPrice</span>();
	 <span style="color:#a020f0;">default</span>:
	    <span style="color:#a020f0;">throw</span> <span style="color:#a020f0;">new</span> <span style="color:#228b22;">IllegalArgumentException</span>();
	}
   }

   <span style="color:#a020f0;">double</span> charge(<span style="color:#a020f0;">int</span> daysRented) {
      <span style="color:#a020f0;">return</span> newPrice(getPriceCode()).charge(daysRented);
   }

}

<span style="color:#a020f0;">public</span> <span style="color:#a020f0;">interface</span> <span style="color:#228b22;">Price</span> {
   <span style="color:#a020f0;">double</span> charge(<span style="color:#a020f0;">int</span> daysRented);
}

<span style="color:#a020f0;">public</span> <span style="color:#a020f0;">class</span> <span style="color:#228b22;">RegularPrice</span> <span style="color:#a020f0;">implements</span> <span style="color:#228b22;">Price</span> {
   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">double</span> charge(<span style="color:#a020f0;">int</span> daysRented) {
      <span style="color:#a020f0;">double</span> result = 2;
      <span style="color:#a020f0;">if</span> (daysRented &gt; 2)
         result += (daysRented - 2) * 1.5;
      <span style="color:#a020f0;">return</span> result;
   }
}

<span style="color:#a020f0;">public</span> <span style="color:#a020f0;">class</span> <span style="color:#228b22;">NewReleasePrice</span> <span style="color:#a020f0;">implements</span> <span style="color:#228b22;">Price</span> {
   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">double</span> charge(<span style="color:#a020f0;">int</span> daysRented) {
      <span style="color:#a020f0;">return</span> daysRented * 3;
   }
}

<span style="color:#a020f0;">public</span> <span style="color:#a020f0;">class</span> <span style="color:#228b22;">ChildrensPrice</span> <span style="color:#a020f0;">implements</span> <span style="color:#228b22;">Price</span> {
   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">double</span> charge(<span style="color:#a020f0;">int</span> daysRented) {
      <span style="color:#a020f0;">double</span> result = 1.5;
      <span style="color:#a020f0;">if</span> (daysRented &gt; 3)
         result += (daysRented - 3) * 1.5;
      <span style="color:#a020f0;">return</span> result;
   }
}</pre><p>This requires us to create some sort of factory method to decide which Price to create based on the Movie type (such as newPrice in the example).  Usually this occurs in a static method with a switch statement on type.  In a large program where Price is used multiple times, this is an advantage because we only have to change the one switch statement.  But what if we could get rid of that switch statement also?
<br/><br/>
If you are using Spring, you can take advantage of the replace-method tag.  You can pass it a class implementing MethodReplacer to replace a method in the class.  That method could be an abstract factory method.  Let's modify our Movie class so that newPrice is an abstract method:</p>
<pre><span style="color:#000000;">// </span><span style="color:#b22222;">From book: 'Refactoring' by Martin Fowler
</span><span style="color:#a020f0;">public</span> <span style="color:#a020f0;">abstract</span> <span style="color:#a020f0;">class</span> <span style="color:#228b22;">Movie</span> {

   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">static</span> <span style="color:#a020f0;">final</span> <span style="color:#a020f0;">int</span> <span style="color:#228b22;">CHILDRENS</span> = 2;
   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">static</span> <span style="color:#a020f0;">final</span> <span style="color:#a020f0;">int</span> <span style="color:#228b22;">NEW_RELEASE</span> = 1;
   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">static</span> <span style="color:#a020f0;">final</span> <span style="color:#a020f0;">int</span> <span style="color:#228b22;">REGULAR</span> = 0;

   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">abstract</span> <span style="color:#228b22;">Price</span> newPrice(<span style="color:#a020f0;">int</span> priceCode);

   <span style="color:#a020f0;">double</span> charge(<span style="color:#a020f0;">int</span> daysRented) {
      <span style="color:#a020f0;">return</span> newPrice(getPriceCode()).charge(daysRented);
   }

}</pre><p>Now we create a handy method replacer utility class that joins a prefix with the first argument to create a bean name and looks up the bean in the context:</p><pre><span style="color:#a020f0;">public</span> <span style="color:#a020f0;">class</span> <span style="color:#228b22;">BeanFactoryMethodReplacer</span> <span style="color:#a020f0;">implements</span> <span style="color:#228b22;">MethodReplacer</span>, <span style="color:#228b22;">ApplicationContextAware</span> {

   <span style="color:#a020f0;">private</span> <span style="color:#228b22;">ApplicationContext</span> context;

   <span style="color:#a020f0;">private</span> <span style="color:#228b22;">String</span> prefix;

   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">void</span> setApplicationContext(<span style="color:#228b22;">ApplicationContext</span> context) throws <span style="color:#228b22;">BeansException</span> {
      <span style="color:#a020f0;">this</span>.context = context;
   }

   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">void</span> setPrefix(<span style="color:#228b22;">String</span> prefix) {
      <span style="color:#a020f0;">this</span>.prefix = prefix;
   }

   <span style="color:#a020f0;">public</span> <span style="color:#228b22;">Object</span> reimplement(<span style="color:#228b22;">Object</span> obj, <span style="color:#228b22;">Method</span> method, <span style="color:#228b22;">Object</span>[] args) throws <span style="color:#228b22;">Throwable</span> {
      <span style="color:#228b22;">String</span> bean = (prefix != null) ? prefix : <span style="color:#bc8f8f;">""</span>;
      bean += args[0];
      <span style="color:#a020f0;">return</span> context.getBean(bean);
   }

}</pre><p>Finally, we register the method replacer, the movie with the newPrice method replaced and beans for each Price.  We use "price" as a prefix.  The name of each price bean should be "price" + the number code for it's type:</p><pre>&lt;<span style="color:#0000ff;">bean</span> <span style="color:#b8860b;">id</span>=<span style="color:#bc8f8f;">"movie"</span> <span style="color:#b8860b;">class</span>=<span style="color:#bc8f8f;">"Movie"</span>&gt;
   &lt;<span style="color:#0000ff;">replaced-method</span> <span style="color:#b8860b;">name</span>=<span style="color:#bc8f8f;">"newPrice"</span> <span style="color:#b8860b;">replacer</span>=<span style="color:#bc8f8f;">"factoryMethodReplacer"</span>&gt;
      &lt;<span style="color:#0000ff;">arg-type</span>&gt;int&lt;/<span style="color:#0000ff;">arg-type</span>&gt;
   &lt;/<span style="color:#0000ff;">replaced-method</span>&gt;
&lt;/<span style="color:#0000ff;">bean</span>&gt;

&lt;<span style="color:#0000ff;">bean</span> <span style="color:#b8860b;">id</span>=<span style="color:#bc8f8f;">"factoryMethodReplacer"</span> <span style="color:#b8860b;">class</span>=<span style="color:#bc8f8f;">"BeanFactoryMethodReplacer"</span>&gt;
   &lt;<span style="color:#0000ff;">property</span> <span style="color:#b8860b;">name</span>=<span style="color:#bc8f8f;">"prefix"</span> <span style="color:#b8860b;">value</span>=<span style="color:#bc8f8f;">"price"</span>/&gt;
&lt;/<span style="color:#0000ff;">bean</span>&gt;

&lt;<span style="color:#0000ff;">bean</span> <span style="color:#b8860b;">id</span>=<span style="color:#bc8f8f;">"price0"</span> <span style="color:#b8860b;">class</span>=<span style="color:#bc8f8f;">"RegularPrice"</span>/&gt;

&lt;<span style="color:#0000ff;">bean</span> <span style="color:#b8860b;">id</span>=<span style="color:#bc8f8f;">"price1"</span> <span style="color:#b8860b;">class</span>=<span style="color:#bc8f8f;">"NewReleasePrice"</span>/&gt;

&lt;<span style="color:#0000ff;">bean</span> <span style="color:#b8860b;">id</span>=<span style="color:#bc8f8f;">"price2"</span> <span style="color:#b8860b;">class</span>=<span style="color:#bc8f8f;">"ChildrensPrice"</span>/&gt;</pre><p>
Spring has now become our factory method.  As new price codes are added, we can just register the Price class as a bean without having to manually change a factory method.</p>
