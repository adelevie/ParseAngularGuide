ParseAngularGuide
=================

Collecting what I've learned building an Angular + Parse App

### Setup

Include the Parse JS SDK. In this example, I'm just using Parse's CDN.

`index.html`:

```html
<!-- snip -->

<!-- inclue Parse JS SDK-->
<script src="http://www.parsecdn.com/js/parse-1.2.9.min.js"></script>

<!-- snip -->
```

Create an Angular Service to initialize the Parse JS SDK:

```javascript
skeletonApp.service('ParseService', [function() {
  var app_id = "1234";
  var js_key = "5678";
  Parse.initialize(app_id, js_key);
}]);
```

Now you have access to the JS SDK anywhere you inject `ParseService`.

### Queries

Inside of the `success` callback, you must call `$scope.$apply` in order for the new data to be bound to the `$scope`. The same step is needed for `$rootScope`.

```javascript
skeletonApp.controller('GameScoreListController', 
  ['$scope', 'ParseService', function($scope, ParseService) {

  var GameScore = Parse.Object.extend("GameScore");

  var query = new Parse.Query(GameScore);

  query.find({
    success: function(results) {
      $scope.$apply(function() {
        $scope.gameScores = results;
      });
    },
    error: function(error) {
      console.log(error);
    }
  });

}]);
```

### Inserting Data

Just as with querying data, you'll need to add `$scope.$apply` inside of your `success` callback:

```javascript
'use strict';

/**
 * RegistrationController
 */
  
skeletonApp.controller('RegistrationController', 
  ['$scope', 'ParseService', function($scope, $location, ParseService) {

  $scope.submit = function() {

    var GameScore = Parse.Object.extend("GameScore");

    var gameScope = new GameScore();

    gameScore.set("score", 1337);
    gameScore.set("playerName", "Sean Plott");
    gameScore.set("cheatMode", false);
     
    gameScore.save(null, {
      success: function(gameScoreAgain) {
        $scope.$apply(function() {
          $scope.gameScore = gameScoreAgain;
        });
      },
      error: function(gameScore, error) {
        alert('Failed to create new object, with error code: ' + error.description);
      }
    }); 

  }


}]);
```

### Persisting logged in users with localStorage

So far my favorite part of using the Parse JS SDK is that you don't need to touch `localStorage` to locally persist a user.

```javascript
skeletonApp.controller('LoginController', 
  ['$scope', 'ParseService', '$location', '$rootScope', function($scope, ParseService, $location, $rootScope) {

  // redirect to "/" if user is already logged in
  if ($rootScope.currentUser !== null) {
    $location.path("/");
  }

  function loginSuccessful(user) {
    $rootScope.$apply(function() {
      // set the current user
      $rootScope.currentUser = Parse.User.current();
      // redirect
      $location.path("/");
    });
  }

  function loginUnsuccessful(user, error) {
    alert("Error: " + error.message + " (" + error.code + ")");
  }

  $rootScope.loggedIn = function() {
    if ($rootScope.currentUser === null) {
      return false;
    } else {
      return true;
    }
  };

  $scope.login = function() {    
    var username = $scope.login.username;
    var password = $scope.login.password;

    Parse.User.logIn(username, password, {
      success: loginSuccessful,
      error: loginUnsuccessful
    });
  };

  $scope.logout = funciton() {
    $rootScope.currentUser = null;
    Parse.User.logOut();
  }

}]);
```

In your views, you can hide and display different elements using the `ng-show` and `ng-hide` directives:

```html
<div ng-controller="LoginController" ng-cloak>

  <div ng-show="loggedIn()">
    Welcome, <a href="#">{{ currentUser.get("username") }}</a> (<a ng-click="logout()">logout</a>)
  </div>

  <div ng-hide="loggedIn()">
    <a href="#login">Login</a> or <a href="#register">Create new account</a>
  </div>

</div>
```