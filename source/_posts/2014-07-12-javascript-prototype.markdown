---
layout: post
title: "Prototypal JavaScript" 
date: 2014-07-12 21:00 
comments: true
categories: JavaScript
---
## Intro
I was watching the [Advanced JavaScript course on Frontend Masters](https://frontendmasters.com/courses/advanced-javascript/) 
and Kyle brought up his concept of “OLOO” (objects-linked-to-other-object).  It reminded me of a blog post by Keith Peters
a few years back on [Learning to live without "new"](http://www.adobe.com/devnet/html5/articles/javascript-object-creation.html)
where he explains using prototypal inheritance instead of constructor functions.  Both are examples pure prototypal coding.  

## The Standard Way
The way we were taught to create objects in JavaScript is to create a constructor function and add methods to the
function's prototype object.

{% codeblock lang:javascript %}
function Animal(name) {
  this.name = name;
}
Animal.prototype.getName = function() {
  return this.name;
};
{% endcodeblock %}

To create a sublcass, we create a new constructor function and set it's prototype to the parent prototype. To call the
parent constructor, we need to call it passing in this as the context object.

{% codeblock lang:javascript %}
function Dog(name) {
  Animal.call(this, name);
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.speak = function() {
  return "woof";
};

var dog = new Dog("Scamp");
console.log(dog.getName() + ' says ' + dog.speak());
{% endcodeblock %}

## The Prototypal Way
If you've had any exposure to prototypal languages, the above example will look strange.  I've tried out the 
[IO language](http://iolanguage.org/) which is a prototype based language.  In a prototypal language, you create a 
prototype by cloning Object and adding methods and properties to it.  You can then clone that prototype to create
instances to use or you can clone it to create a prototype that you can extend.  This is how the above example would look in IO:

{% codeblock lang:io %}
Animal := Object clone
Animal getName := method(name)

Dog := Animal clone
Dog speak := method("woof")

dog := Dog clone
dog name := "Scamp"
writeln(dog getName(), " says ", dog speak())
{% endcodeblock %}

## The Good News
We can code this way in JavaScript!  The Object.create function is similar to IO's clone.  Here's
a pure prototypal implementation in JavaScript.  Other than syntax, it's the same as the IO version.

{% codeblock lang:javascript %}
Animal = Object.create(Object);
Animal.getName = function() {
  return this.name;
};

Dog = Object.create(Animal);
Dog.speak = function() {
  return "woof";
};

var dog = Object.create(Dog);
dog.name = "Scamp";
console.log(dog.getName() + ' says ' + dog.speak());
{% endcodeblock %}

## The Bad News
JavaScript engines have optimizations when you use constructor functions.  Testing the two
different options on [JSPerf](http://jsperf.com/proto-vs-ctor/3) shows the prototypal implementation to be up to 90 
times slower than using constuctors.

![Perf Graph](/images/proto-vs-ctor.png)

Also, if you are using frameworks such as [Angular](https://angularjs.org/), you have to use constructor functions when
you are creating controllers and services.

## Enter Classes
With ES6 we have a new class syntax.  This syntax is just sugar for the standard constructor way to produce objects.
It looks like we are creating classes like one would in Java or C#, but it's still creating prototype objects under
the covers.  This will be confusing for people coming from class based languages as they will expect it to have the 
same properties as a class in their language when it's really creating prototypes.

{% codeblock lang:javascript %}
class Animal {
  constructor(name) {
    this.name = name;
  }
  getName() {
    return this.name;
  }
}

class Dog extends Animal {
  constructor(name) {
    super(name);
  }
  speak() {
    return "woof";
  }
}

var dog = new Dog("Scamp");
console.log(dog.getName() + ' says ' + dog.speak());
{% endcodeblock %}

## Conclusion
If I had a choice, I would always write using a pure prototypal style.  It's more expressive, dynamic and fun.
Because of the way the way the virtual machines optimize for constructor functions and the way frameworks use them, 
production code I write will still use constructor functions.  Once ES6 becomes common, I expect I'll be using
the class syntax instead as it's easier than using constructor functions.