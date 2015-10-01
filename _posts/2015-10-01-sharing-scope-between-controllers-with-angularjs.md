---
layout: post
permalink: /sharing-scope-between-controllers-with-angularjs
title: Sharing $scope between controllers in AngularJS
path: 2015-10-01-sharing-scope-between-controllers-with-angularjs.md
---

If our application needs access to another controller’s `$scope`, we probably are doing something wrong and we need to re-think our problem. But anyway it’s possible to access another controller’s `$scope` if we store it within a `service`.

A `service` provides an easy way for us to share data and functionality throughout our app. The services we create are singletons that can be injected into controllers and other services, making them the ideal place for writing reusable code.

###### share-service.js
{% highlight javascript %}
  angular
    .module('myApp', [])
    .factory('shareService', shareService)

    function shareService() {
      var memory = {};
      return {
          store: function (key, value) {
              memory[key] = value;
          },
          get: function (key) {
              return memory[key];
          }
      };
    }
{% endhighlight %}

And now, in order to store the `$scope` in the service:

###### controller.js
{% highlight javascript %}
  function FirstCtrl ($scope, shareService) {

    shareService.store('FirstCtrl', $scope)
    $scope.variableOne = 'First Controller Variable';

  }
{% endhighlight %}

Alternatively, if we are using `controller as` we can:

###### controller.js
{% highlight javascript %}
  function FirstCtrl (shareService) {

    var vm = this;

    shareService.store('FirstCtrl', vm)
    vm.variableOne = 'First Controller Variable';

  }
{% endhighlight %}

Now, to access this `$scope` from another controller:

###### another-controller.js
{% highlight javascript %}
  function SecondCtrl ($scope, shareService) {

    $scope.getOne = function() {
      $scope.resOne = shareService.get('FirstCtrl').variableOne;
    }

  }
{% endhighlight %}



### See it in action

<iframe width="100%" height="550" src="http://embed.plnkr.co/vhvg8463vP5qRCptzZ5h" frameborder="0" allowfullscren="allowfullscren"></iframe>