---
layout: post
title: "AngularJS and jQuery Dialogs" 
date: 2014-06-29 21:00 
comments: true
categories: AngularJS jQuery
---
<p>I'm using jQuery dialogs in an AngularJS application. I've struggled to find
a way to use them that follows angular 'best practices'. My first attempt used a 
directive for the DOM manipulation and a controller for binding to the scope.  I was 
uncomfortable with this approach because I was splitting the code between two
different artifacts.  I also ended up adding the open dialog method to the $rootScope
which is the same as making it global.</p>
{% codeblock lang:javascript %}
app.controller('DialogCtrl', function($scope) {
  $scope.message = 'Hello Dialog!';
  this.onSave = function() {
    alert('saved!');
  };
})
app.directive('myDialog', function() {
  return {
    controller: 'DialogCtrl',
    link: function(scope,elem,attrs,ctrl) {
      elem.dialog({
        autoOpen: false,
        buttons: { 
           'Save': function() {
              ctrl.onSave();
              elem.dialog('close');
            }
        }
      });
      scope.$root.openTestDialog = function() {
        elem.dialog('open');
      };
    }
  };
});
{% endcodeblock %}
<p>The other method I tried was using a single controller.  This was also not best
practice since I was manipulating DOM in a controller.  I still had to put the
open method on $rootScope.</p>
{% codeblock lang:javascript %}
app.controller('TestDialog', function($scope) {
  var self = this,
      $dialog = $('#test-dialog');
  
  $scope.message = "Hello!!!";
    
  this.onSave = function() {
    alert('saved!');
    $dialog.dialog('close');
  };

  $dialog.dialog({ 
    autoOpen: false,
    buttons: {
      'Save': self.onSave
    }
  });

  $scope.$root.openTestDialog = function() {
    $scope.message = "boom";
    $dialog.dialog('open');
  };
  
});
{% endcodeblock %}
<p>Others have suggested using a service, but I would be accessing the DOM in the
service and what would I do about scope?</p>
<p>Then I thought, what if I created a generic directive that would allow me to
set the dialog options in the markup?  Furthermore, what if I could register the
the dialog in a service so I could inject it into a controller and open the dialog
using that controller?  It turns out it's possible with a little angular black magic.
Meet the jqdialog directive.</p>
<p>Step one: create directive that provides the jQuery dialog options as scope variables.  I capture 
the options directly off the $.ui.dialog and then add them to the scope using '&' bindings.  This
allows any values to be entered without having to convert from strings with the downside of having to
place strings in quotes inside the attribute.  Then in the link function, I can loop thru the options,
and if they have values on the scope, I put them in the options array which is passed to the dialog
function.</p>
{% codeblock lang:javascript %}
app.directive('jqdialog', ['$injector', function($injector) {
  var options = Object.keys($.ui.dialog.prototype.options);
  return {
    scope: options.reduce(function(acc, val) {
      acc[val] = "&"; return acc;
    }, {
      onOpen: "&",
      onClose: "&",
      buttonClasses: "&"
    }),
    <...>
    compile: function(elem, attrs) {
      <...>        
      return function(scope, elem, attrs, ctrl, transclude) {
        var opts = options.reduce(function(acc, val) {
              var value = scope[val] ? scope[val]() : undefined;
              if (value !== undefined) {
                acc[val] = value;
              }
              return acc;
            }, {}),
            dialog = elem.dialog(opts);
            <...>
      };
    };
});
{% endcodeblock %}
<p>Step two: require a dialogName attribute and use that to create a service for the dialog with
open and close methods.  We'll need to create the service in the compile function so it will
be available when dependencies are injected.  Then we'll add the methods in the link function 
when the scope is available.  Finally, we must call the transclude method using the $parent scope
instead of ng-transclude so we get the correct scope for contents in the dialog.</p>
{% codeblock lang:javascript %}
app.directive('jqdialog', ['$injector', function($injector) {
  <...>
  return {
    <...>
    compile: function(elem, attrs) {
      app.provide.service(attrs.dialogName + 'DialogService',
                ['$q', function($q) { this.q = $q; }]);
      return function(scope, elem, attrs, ctrl, transclude) {
        var service = $injector.get(attrs.dialogName + 'DialogService');
        <...>
        
        service.openDialog = function(options) {
          var onOpen = scope.onOpen();
          this.dfd = this.q.defer();
          if (onOpen) {
            onOpen(options);
          }
          dialog.dialog('open');
          return this.dfd.promise;
        };

        service.closeDialog = function(data) {
          var onClose = scope.onClose();
          if (onClose) {
            onClose(data);
          }
          dialog.dialog('close');
          this.dfd.resolve(data);
        };

        transclude(scope.$parent, function(clone) {
          elem.append(clone);
        });

      };
    }
  };
}]);
{% endcodeblock %}
<p>The full version is available on 
<a href="https://github.com/jurberg/angular.jquery/blob/master/src/jqdialog/jqdialog.js">github</a>.  Here is an example 
of the tag usage.</p>
{% codeblock lang:html %}
<div ng-app="app" ng-controller="DemoCtrl">
  <button ng-click="testDialog();">Open Dialog</button>
  <jqdialog dialog-name="Test" title="'Test Dialog'"
            auto-open="false" width="400" height="300"
            on-open="onOpen" buttons="{'OK': onOk, 'Cancel': onCancel}"
            ng-controller="DialogCtrl">
    <h3>This is a test dialog</h3>
    <label for="fullName">Full Name:</label>
    <input id="fullName" ng-model="fullName"/>
  </jqdialog>
</div>
{% endcodeblock %}
{% codeblock lang:javascript %}
app.controller('DemoCtrl', ['$scope', 'TestDialogService', function($scope, TestDialogService) {
  $scope.testDialog = function() {
    TestDialogService.openDialog().then(function(result) {
      if (result.ok) {
        alert('You entered ' + result.name);
      }
    });
  };
}]);
app.controller('DialogCtrl', ['$scope', 'TestDialogService', function($scope, TestDialogService) {
  $scope.fullName = "Test";
  $scope.onOpen = function() {
    $scope.fullName = "";
  };
  $scope.onOk = function() {
    TestDialogService.closeDialog({ok: true, name: $scope.fullName});
  };
  $scope.onCancel = function() {
    TestDialogService.closeDialog({ok: false});
  };
}]);
{% endcodeblock %}
<p>I've started an <a href="https://github.com/jurberg/angular.jquery">angular.jquery</a> project on github to hold 
this and other jQuery directives.  If you have any directives, feel free to contribute</p>