---
layout: post
title: "Complexity and Angular DI" 
date: 2014-06-22 21:00 
comments: true
categories: AngularJS
---
<p>We chose AngularJS as our primary JavaScript framework and I've had the opportunity to use it heavily in a project.  
While controllers and directives are cool, I find the dependency injection to be overly complicated.  There are many 
different ways to provide objects to inject: provider, factory, value, service and constant.  In the end, most of 
these are shortcuts that do the same thing.  The following example is modified from the 
<a href="https://github.com/angular/angular.js/wiki/Understanding-Dependency-Injection">angular documentation</a> 
and creates a simple alert function that can be injected through angular DI.</p>
{% codeblock lang:javascript %}
var app = angular.module('app', []);
// Using a provider:
app.provider('greeting', ['$window', function($window) {
  this.$get = function() {
    return {
      alert: function(name) {
        $window.alert("Hello, " + name);
      }
    };
  };
}]);
// Using a factory:
app.factory('greeting', ['$window', function($window) {
  return {
    alert: function(name) {
      $window.alert("Hello, " + name);
    }
  };
}]);
// Using a service:
app.service('greeting', ['$window', function($window) {
  this.alert = function(name) {
    $window.alert("Hello, " + name);
  };
});
// Using a value:
app.value('greeting', {
  alert: function(name) {
    alert("Hello, " + name);
  }
});
{% endcodeblock %}
<p>There are a few minor differences: the service takes a constructor to create the object, while provider and factory 
take the results of calling a function for the object.  Value takes the object and provides no dependency injection.  Otherwise,
these are all creating exactly the same module.</p>
<p>Contrast this to creating modules using AMD.  In the past I've used either <a href="http://requirejs.org/">RequireJS</a> 
or <a href="https://github.com/jurberg/define.js/tree/master">define.js</a> as libraries to add AMD to my projects.  I use
 them mostly as a way to organize my code, but I get a form of dependency injection also.  When using RequireJS, I can 
 add <a href="https://github.com/iammerrick/Squire.js/">Squire.js</a> to mock module dependencies. define.js includes 
 this ability in the library.  AMD has the dependencies "injected" into the module. The only 
 difference from AngularJS is that they are injected when the module is loaded.  Here's the same module created
 using AMD:</p>
{% codeblock lang:javascript %}
define('greeting', ['window'], function($window) {
  return {
    alert: function(name) {
      $window.alert("Hello, " + name);
    }
  };
});
{% endcodeblock %}
<p>  Here's a test for the angular code using the angular mock library:</p>
{% codeblock lang:javascript %}
define("Greeting module", function() {

  var mockWindow, service;

  beforeEach(function() {
    mockWindow.alert = jasmine.createSpy();
    module('app', function($provide) {
      provide.value('$window', mockWindow);
    });
    inject(function($injector) {
      service = $injector.get('greeting');
    });
  });
  
  it("should show alert", function() {
     service.alert('everyone');
     expect(mockWindow.alert).toHaveBeenCalledWith("Hello, everyone");
  });

});
{% endcodeblock %}
<p>Here is the same test with define.js and no additional libraries:</p>
{% codeblock lang:javascript %}
define("Greeting module", function() {

  var mockWindow, service;

  beforeEach(function() {
    mockWindow.alert = jasmine.createSpy();
    redefine('greeting', { '$window': mockWindow });
    require(['greeting'], function(greeting) { service = greeting; });
  });
  
  afterEach(function() {
     redefine('greeting');
  });
  
  it("should show alert", function() {
     service.alert('everyone');
     expect(mockWindow.alert).toHaveBeenCalledWith("Hello, everyone");
  });

});
{% endcodeblock %}
<p>So AMD gives you most of the ability provided by Angular DI with only one way to do it.  If you are using RequireJS,
you have the added ability to automatically resolve dependencies and load them asynchronously.</p>
<p>Of course, you can always use both.  If you treat Angular as a dependency container and build your 
modules in AMD, you can get to a place were you can have more fine grain organization of your code and not have to 
force everything that's not a controller or directive into a service.  Here's a quick example that is setup to
use no globals other than the define and require methods of AMD.</p>
{% codeblock lang:javascript %}
define('app', ['angular'], function(angular) {
  return angular.module('app', []);
});

// A utility module that doesn't need to be a service
define('app/util', function() {
  return {
    upper: function(value) {
      value.toUpperCase();
    }
  }
});

define('app/service', ['app', 'app/util'], function(app, util) {

  function Service() {
    this.name = 'service';
  }
    
  Service.prototype = {
    getName: function() {
      return util.upper(this.name);
    },
    setName: function(name) {
        this.name = name;
    }
  };
    
  app.service('MyService', Service);
    
  return {};
});

define('app/MyCtrl', ['app'], function(app) {

  function MyCtrl($scope, $window, MyService) {
     $scope.click = function() {
       $window.alert(MyService.getName());
     };
  }

  app.controller('MyCtrl', ['$scope', '$window', 'MyService', MyCtrl]);
  
  return {};
});
{% endcodeblock %}