# Angular Jokes App with Laravel5 Backend API 

In the Previous part of this series, we have seen how we can use Laravel5 to create an full fledged API and we checked the API response with the chrome extension called Postman. And in this second part of the two part series, we will see how we can interact with or use this API to make requests from the client side with Angularjs and make a simple jokes app.

So without further due, let’s jump right into the frontend part.

INSTALLING ANGULAR SEED
Let’s start by creating skeleton for the client side work, we will use angular seed for this.

First you need to clone the angular seed project from the git.

git clone https://github.com/angular/angular-seed.git
cd angular-seed
Next step is to install the dependencies for the angular seed project. So we will run following command.

npm install
This will fetch the dependencies for the angular seed project. Now you can see two new folders created in the directory structure named node_modules & app/bower_components.

Both Laravel 5 and Angularjs by default use port 8000 so we need to use other port for one of these. Let’s change port for angular seed project.

Open package.json file and update start property as,

"start": "http-server -a localhost -p 8080 -c-1"
Now you can run the project by running the following command.

npm start
Now browse to the app at

http://localhost:8080/app/index.html
Install ui-router Angular Seed project by defaults ships with angular’s default router. But i prefer ui-router (A third party router for angular). So we will modify our project to include the ui-router instead of default angular router.

We will use bower for downloaing ui-router to our project as

bower install angular-ui-router --save
Include angular-ui-router.js in your index.html

<script src="bower_components/angular-ui-router/release/angular-ui-router.js"></script>
Now add ui-router to list of dependenices in the app.js

angular.module('myApp', [
  // 'ngRoute',
  'ui.router',
  'myApp.view1',
  'myApp.view2'
  'myApp.version'
])
CREATING NEW ROUTES
Now we will create two routes, one for the auth and other for the jokes. Let’s start by creating two new directories in the app directory named view_auth & view_jokes. Your directory structure will look something like this.

null

Now open view_auth/auth.js, and update the code as follows:

angular.module('myApp.auth', [])

.config(['$stateProvider', '$urlRouterProvider', function($stateProvider, $urlRouterProvider) {
  $stateProvider
  .state('auth', {
    url: '/auth',
    views: {
      'jokesContent': {
        templateUrl: "view_auth/auth.html",
    	controller: 'AuthCtrl as auth'
      }
    }
  })
}])

.controller('AuthCtrl', ['$rootScope', function($rootScope){
 
}])
Now open view_jokes/jokes.js, and update the code as follows:

angular.module('myApp.jokes', [])

.config(['$stateProvider', '$urlRouterProvider', function($stateProvider, $urlRouterProvider) {
  $stateProvider
  .state('jokes', {
    url: '/jokes',
    views: {
      'jokesContent': {
        templateUrl: "view_jokes/jokes.html",
    	controller: 'JokesCtrl as jokes'
      }
    }
  })
}])

.controller('JokesCtrl', ['$rootScope', function($rootScope){
 
}])
Open view_auth/auth.html & view_jokes/jokes.html and add some content. These are view specific files.

Now include these view specific script files in the index.html as

<script src="view_auth/auth.js"></script>
<script src="view_jokes/jokes.js"></script>
Now update the list of dependencies in the app.js file as

// Declare app level module which depends on views, and components
angular.module('myApp', [
  // 'ngRoute',
  'ui.router',
  'myApp.jokes',
  'myApp.view2',
  'myApp.auth',
  'myApp.version'
])
Now you can view these routes in the browser as.

http://localhost:8080/app/#/auth
and

http://localhost:8080/app/#/jokes
CLIENT SIDE AUTH
Open the view_auth/auth.html and add the following code.

<div class="container">
    <div class="row">
        <div class="col-md-4 col-md-offset-4">
            <div class="panel panel-default">
                <div class="panel-heading"> <strong class="">Login</strong>

                </div>
                <div class="panel-body">
                    <form class="form-horizontal" role="form">
                        <div class="form-group">
                            <label for="inputEmail3" class="col-sm-3 control-label">Email</label>
                            <div class="col-sm-9">
                                <input type="email" class="form-control" id="inputEmail3" placeholder="Email" required="" ng-model="auth.email">
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="inputPassword3" class="col-sm-3 control-label">Password</label>
                            <div class="col-sm-9">
                                <input type="password" class="form-control" id="inputPassword3" placeholder="Password" required="" ng-model="auth.password">
                            </div>
                        </div>
                        <div class="form-group last">
                            <div class="col-sm-offset-3 col-sm-9">
                                <button type="submit" class="btn btn-success btn-sm"  ng-click="auth.login()">Sign in</button>
                                <button type="reset" class="btn btn-default btn-sm">Reset</button>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
The output will look similar to this.

null

Using Satellizer Now for client side authentication, we will use a token based authentication moude for angular called satellizer, let’s first start by including satellizer to our project by running,

bower install satellizer --save
Add satellizer to list of dependencies in the app.js,

angular.module('myApp', [
  // 'ngRoute',
  'ui.router',
  'myApp.jokes',
  'myApp.view2',
  'myApp.auth',
  'myApp.version',
   'satellizer'
])
Now add the login end point in the app.js from the api by updating the config block,

.config(['$stateProvider', '$urlRouterProvider', '$authProvider', function($stateProvider, $urlRouterProvider, $authProvider) {

	$authProvider.loginUrl = 'http://localhost:8000/api/v1/authenticate';
  // $urlRouterProvider.otherwise('/view1');
  $urlRouterProvider.otherwise('/auth');
}]);
Satellizer gives us access to the $authProvider, which we used to specify the login url. The benefit of Satellizer is that when the user is logged in, a token is saved to the local storage. We know that to make request to our laravel api we need token every time, Satellizer makes sure to send the token with each request. So we don’t have to send the token manually.

Now open the view_auth/auth.js and update the controller code as follows:

.controller('AuthCtrl', ['$auth', '$state', '$http', '$rootScope', function($auth, $state, $http, $rootScope) {

      var vm = this;

        vm.loginError = false;
        vm.loginErrorText;

        vm.login = function() {

            var credentials = {
                email: vm.email,
                password: vm.password
            }

            $auth.login(credentials).then(function() {
                $http.get('http://localhost:8000/api/v1/authenticate/user').success(function(response){
                    var user = JSON.stringify(response.user);
                    localStorage.setItem('user', user);
                    $rootScope.currentUser = response.user;                   
                    $state.go('jokes');
                })
                .error(function(){
                    vm.loginError = true;
                    vm.loginErrorText = error.data.error;
                    console.log(vm.loginErrorText);
                })
            });
        }

}]);
Now after starting the laravel server, hit the auth route and enter the correct credentials and hit submit. If everything works, you will be redirected to the jokes route and you will see following items added to the local storage.

null

This satellizer_token will be sent with the each request that we will make from the client side from now on.

GET ALL JOKES
Open view_jokes/jokes.html and update the code as follows,

<div class="col-sm-6 col-sm-offset-3">
        <div>
        <ul class="list-group">
            <li class="list-group-item" ng-repeat="joke in jokes.jokes">
                <h5>{{joke.joke}}</h5>
                <h5>{{joke.submitted_by}}</h5>
            </li>
        </ul>
        <div class="alert alert-danger" ng-if="jokes.error">
            <strong>There was an error: </strong> {{jokes.error}}
            <br>Please go back and login again...
        </div>
    </div>
</div>
Now open view_jokes/jokes.js and update the controller code as follows:

.controller('JokesCtrl', ['$http', '$auth', '$rootScope','$state', '$q' , function($http, $auth, $rootScope, $state, $q) {

  var vm = this;
        
  vm.jokes = [];
  vm.error;
  vm.joke;

  $http.get('http://localhost:8000/api/v1/jokes').success(function(jokes){    
    console.log(jokes);
    vm.jokes = jokes.data;
    }).error(function(error){
      vm.error = error;
    })

  vm.init();

}]);
I know, i can move most of the code to the factory or service, but for the sake of simplicity, i am not going to do this.

Now Open the jokes route, you will see a output similar to this,

null

PRETTIFY APP & LOGOUT PART
Let’s start by adding nav to the app, open index.html and update the container code as follows:

<div class="container">
        <nav class="navbar navbar-default">
          <div class="container-fluid">
            <!-- Brand and toggle get grouped for better mobile display -->
            <div class="navbar-header">
              <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
              </button>
              <a class="navbar-brand" href="">Jokes App</a>
            </div>

            <!-- Collect the nav links, forms, and other content for toggling -->
            <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
              <ul class="nav navbar-nav">
                <li><a ui-sref="auth">Auth</a></li>
                <li><a ui-sref="jokes">Jokes</a></li>
              </ul>
              <ul class="nav navbar-nav navbar-right" ng-show="currentUser != null">
              <li><a href="">Welcome, {{currentUser.name}}</a></li>
                <li class="dropdown">
                  <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">My Profile <span class="caret"></span></a>
                  <ul class="dropdown-menu">
                    <li><a ng-click="logout()" style="cursor: pointer;">Logout</a></li>
                  </ul>
                </li>
              </ul>
            </div><!-- /.navbar-collapse -->
          </div><!-- /.container-fluid -->
        </nav>


        <div ui-view="jokesContent"></div>
    </div>
Now open app.js and add the logout code in the run function as,

.run(function ($rootScope, $state, $auth) {

      $rootScope.logout = function() {
        $auth.logout().then(function() {
            localStorage.removeItem('user');
            $rootScope.currentUser = null;
            $state.go('auth');
            });
           }
         })
$rootScope.currentUser = JSON.parse(localStorage.getItem('user'));
}
Now the app will look similar to this and logout will work.

null

Securing Routes At that point, we are authenticating the user from the server and fetching and showing the jokes from the server. But we can see that, if the user is not authenticated, he/she can still go to jokes route and if the user is authenticated, he/she can still go to the auth route. We don’t want this to happen in our app. So we need to secure our routes.

For securing routes we will use a angular package called angular-permission, so let’s start by installing the package.

bower install angular-permission --save
Include the package to the app.js list of dependencies,

angular.module('myApp', [
  // 'ngRoute',
  'ui.router',
  'myApp.jokes',
  'myApp.view2',
  'myApp.auth',
  'myApp.version',
   'satellizer',
   'permission'
])
Now first open the view_auth/auth.js and update the config block as,

.config(['$stateProvider', '$urlRouterProvider', function($stateProvider, $urlRouterProvider) {
  $stateProvider
  .state('auth', {
    url: '/auth',
    data: {
        permissions: {
          except: ['isloggedin'],
          redirectTo: 'jokes'
        }
      },
    views: {
      'jokesContent': {
        templateUrl: "view_auth/auth.html",
    	controller: 'AuthCtrl as auth'
      }
    }
  })
}])
Here you can see we are using the new field data, where we have defined permissions, it is available by the angular-permission package. Here we have defined permission role (isloggedin) as, this route is available all the users who are not logged in.

So now we need to define that role in the app.js run funciton,

Permission 
     .defineRole('isloggedin', function (stateParams) {
        // If the returned value is *truthy* then the user has the role, otherwise they don't
        // console.log("isloggedin ", $auth.isAuthenticated()); 
        if ($auth.isAuthenticated()) {
          return true; // Is loggedin
        }
        return false;
      })
Here we are checking if the user is authenticated, then he will not be allowed to enter this route.

Similarly update the view_jokes/jokes.js config function as,

.config(['$stateProvider', '$urlRouterProvider', function($stateProvider, $urlRouterProvider) {
  $stateProvider
  .state('jokes', {
    url: '/jokes',
    data: {
        permissions: {
          except: ['anonymous'],
          redirectTo: 'auth'
        }
      },
    views: {
      'jokesContent': {
        templateUrl: "view_jokes/jokes.html",
    	controller: 'JokesCtrl as jokes'
      }
    }
  })
}])
Now we need to define the anonymous role in the app.js run function, so the complete permission block will look like this.

// Define anonymous role
      Permission
      .defineRole('anonymous', function (stateParams) {
        // If the returned value is *truthy* then the user has the role, otherwise they don't
        // var User = JSON.parse(localStorage.getItem('user')); 
        // console.log("anonymous ", $auth.isAuthenticated()); 
        if (!$auth.isAuthenticated()) {
          return true; // Is anonymous
        }
        return false;
      })

      .defineRole('isloggedin', function (stateParams) {
        // If the returned value is *truthy* then the user has the role, otherwise they don't
        // console.log("isloggedin ", $auth.isAuthenticated()); 
        if ($auth.isAuthenticated()) {
          return true; // Is loggedin
        }
        return false;
      })
      ;
Now Try going to the jokes route without login, you will be redirected to the auth route also login and then try to open the auth route you will be redirected to the jokes route.

POST, UPDATE, DELETE REQUEST
For using Post, Update and Delete Request. Update the view_jokes/jokes.html code as follows.

<div class="col-sm-6 col-sm-offset-3">
        <div class="row">
            <div class="col-sm-12">
                <div class="input-group">
                  <input type="text" class="form-control" placeholder="Add new joke.." ng-model="jokes.joke">
                  <span class="input-group-btn">
                    <button class="btn btn-default" type="button" ng-click="jokes.addJoke()">Add!</button>
                  </span>
                </div><!-- /input-group -->
              </div><!-- /.col-lg-6 -->
        </div>
        <br>
        <div>
        <ul class="list-group">
            <li class="list-group-item" ng-repeat="joke in jokes.jokes">
                <span ng-hide="editEnabled">
                            {{joke.joke}}
                                <a href="#" ng-click="editEnabled=!editEnabled"><span class="glyphicon glyphicon-pencil" ></span></a>
                </span>
                <span ng-show="editEnabled">
                            <input ng-model="joke.joke">
                            <a href="#" ng-click="editEnabled=!editEnabled; jokes.updateJoke(joke)"><span class="glyphicon glyphicon-ok" ></span></a>
                </span>
                <!-- <h4>{{joke.joke}}</h4> -->
                <span style="cursor: pointer;" class="glyphicon glyphicon-trash" ng-click="jokes.deleteJoke($index, joke.joke_id)"></span>
                <h5>{{joke.submitted_by}}</h5>
            </li>
        </ul>
        <div class="alert alert-danger" ng-if="jokes.error">
            <strong>There was an error: </strong> {{jokes.error}}
            <br>Please go back and login again...
        </div>
    </div>
</div>
Now open the view_jokes/jokes.js and update the controller code as follows:

.controller('JokesCtrl', ['$http', '$auth', '$rootScope','$state', '$q' , function($http, $auth, $rootScope, $state, $q) {

  var vm = this;
        
  vm.jokes = [];
  vm.error;
  vm.joke;

  $http.get('http://localhost:8000/api/v1/jokes').success(function(jokes){    
    console.log(jokes);
    vm.jokes = jokes.data;
    }).error(function(error){
      vm.error = error;
    })            


    vm.addJoke = function() {

        $http.post('http://localhost:8000/api/v1/jokes', {
            body: vm.joke,
            user_id: $rootScope.currentUser.id
        }).success(function(response) {
            // console.log(vm.jokes);
            // vm.jokes.push(response.data);
            vm.jokes.unshift(response.data);
            console.log(vm.jokes);
            vm.joke = '';
            // alert(data.message);
            // alert("Joke Created Successfully");
        }).error(function(){
          console.log("error");
        });
    };

    vm.updateJoke = function(joke){
      console.log(joke);
      $http.put('http://localhost:8000/api/v1/jokes/' + joke.joke_id, {
            body: joke.joke,
            user_id: $rootScope.currentUser.id
        }).success(function(response) {
            // alert("Joke Updated Successfully");
        }).error(function(){
          console.log("error");
        });
    }


    vm.deleteJoke = function(index, jokeId){
      console.log(index, jokeId);

        $http.delete('http://localhost:8000/api/v1/jokes/' + jokeId)
            .success(function() {
                vm.jokes.splice(index, 1);
            });;
    }
 

}]);
Add Joke

null

Update Joke

null

Delete Joke

null

TACKLING PAGINATION
Now the last step before completing our app is to add the pagination. We will use a load more button. When we click this button, It will load more jokes. First start by adding the load more button in the view_jokes/jokes.html as,

<button class="btn btn-success" ng-click="jokes.loadMore()" style="float:right">Load More....</button>
Now open the view_jokes/jokes.js and add the following code in the controller,

// $http.get('http://localhost:8000/api/v1/jokes').success(function(jokes){    
  //   console.log(jokes);
  //   vm.jokes = jokes.data;
  //   }).error(function(error){
  //     vm.error = error;
  //   })
   
  vm.lastpage=1;
  vm.init = function() {
                vm.lastpage=1;
                $http({
                    url: 'http://localhost:8000/api/v1/jokes',
                    method: "GET",
                    params: {page:  vm.lastpage}
                }).success(function(jokes, status, headers, config) {
                    vm.jokes = jokes.data;
                    vm.currentpage = jokes.current_page;
                });
            };
  vm.init();

  vm.loadMore = function() {
                vm.lastpage +=1;
                $http({
                    url: 'http://localhost:8000/api/v1/jokes',
                    method: "GET",
                    params: {page:  vm.lastpage}
                }).success(function (jokes, status, headers, config) {
 
                    vm.jokes = vm.jokes.concat(jokes.data);
 
                });
            };
Now open the jokes routes and click load more button, you can see the new data jokes loaded in the view.



Wrapping Up Part 2 I hope at the end of this part, you have successfully created you angular app that interact with the Laravel API that we have created in the Part 1 of the series.

If there are any doubts, you can ask in the comments section.

Thanks.

## Getting Started

To get you started you can simply clone the repository and install the dependencies:


### Install Dependencies

npm install 
bower install

### Starting Project
npm start