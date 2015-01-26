---
layout: post
title: "Angular Routers in Large Projects" 
date: 2015-01-25 21:00 
comments: true
categories: JavaScript
---
## Intro
If you read the [Angular Tutorial section on the router](https://docs.angularjs.org/tutorial/step_07) you will find what amounts to a large switch statement.  Here's the example code from the tutoral:

{% codeblock lang:javascript %}
phonecatApp.config(['$routeProvider', function($routeProvider) {
  $routeProvider.
    when('/phones', {
      templateUrl: 'partials/phone-list.html',
      controller: 'PhoneListCtrl'
    }).
    when('/phones/:phoneId', {
      templateUrl: 'partials/phone-detail.html',
      controller: 'PhoneDetailCtrl'
    }).
    otherwise({
      redirectTo: '/phones'
    });
}]);
{% endcodeblock %}

A diligent programmer might build a service around this to provide methods to switch routes or to provide constants for the routes.  

{% codeblock lang:javascript %}
(function(app) {
  
  var PHONE_ROUTE  = '/phones';

  app.config(['$routeProvider', function($routeProvider) {
    // config routes (see previous example)
  }]);

  app.service('RouteService', ['$location', function($location) {
    this.goToPhoneList = function() {
      $location.path(PHONE_ROUTE);
    };
    this.goToPhoneDetails = function(phoneId) {
      $location.path(PHONE_ROUTE + '/' + phoneId);
    };
  }]);

}(phonecatApp));
{% endcodeblock %}

This is not a big deal for a small project with a couple of routes.  As your application grows, you'll find this file needs to be updated everytime a route is added.  These methods to navigate to a route will start to contain logic that should be in a controller.  Image what this file would look like with 30 routes?

Let's apply some Object oriented design to our problem.  One way to manage our dependencies is to use the [Open Closed Principle](http://en.wikipedia.org/wiki/Open/closed_principle); we want to make our RouterService open for extension but closed for modification.  The $routeProvider gives us some help here.  We don't need to add all our routes in one block of code.  Since routes tend to match up to controllers, why not add it in the code file with it's controller?

{% codeblock lang:javascript %}
(function(app) {

  var PHONE_LIST_ROUTE = '/phones';

  app.controller('PhoneListCtrl', function() {
    // controller code here
  });

  app.config(['$routeProvider', function($routeProvider) {
    $routeProvider.
      when(PHONE_LIST_ROUTE, {
        templateUrl: 'partials/phone-list.html',
        controller: 'PhoneListCtrl'
      }).
      otherwise({
        redirectTo: PHONE_LIST_ROUTE
      });
  }]);

}(phonecatApp))
{% endcodeblock %}

We still have the issue of those methods on the RouterService itself. We should move those to the file with the controller.  This keeps the logic concerning what happens when you try to route to a controller together with the controller.  We can take advantage of that fact that JavaScript is a dynamic language by defining a simple service and adding methods on the fly.

First start with a simple service.  Notice it is just an empty object.  We'll extend it as we go.

{% codeblock lang:javascript %}
(function(app) {
  app.value('RouterService', {});
}(phonecatApp))
{% endcodeblock %}

Now after we've setup our controller and $routeProvider, we can add the goto method.

{% codeblock lang:javascript %}
(function(app) {

  // controller def

  // route config

  app.run(['RouterService', '$location', function(RouterService, $location) {
    Router.goToPhoneList = function() { $location.path(PHONE_LIST_ROUTE); };
  }]);

}(phonecatApp));
{% endcodeblock %}

Using this pattern, I can now add or change code for routes in a single file.  The route details, the controller and a go to method are all together.  More importantly, multiple developers can now work on a large application with dozens of routes without constantly having conflicts on that RouterService.
