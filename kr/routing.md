# Routing
# 라우팅

- [Basic Routing](#basic-routing)
- [기본적인 라우팅](#basic-routing)
    - [Redirect Routes](#redirect-routes)
    - [리다이렉트 라우트](#redirect-routes)
    - [View Routes](#view-routes)
    - [뷰-View 라우트](#view-routes)
- [Route Parameters](#route-parameters)
- [라우트 파라미터](#route-parameters)
    - [Required Parameters](#required-parameters)
    - [필수 파라미터](#required-parameters)
    - [Optional Parameters](#parameters-optional-parameters)
    - [선택적인 파라미터](#parameters-optional-parameters)
    - [Regular Expression Constraints](#parameters-regular-expression-constraints)
    - [정규표현식 제약](#parameters-regular-expression-constraints)
- [Named Routes](#named-routes)
- [이름이 지정된 라우트](#named-routes)
- [Route Groups](#route-groups)
- [라우트 그룹](#route-groups)
    - [Middleware](#route-group-middleware)
    - [미들웨어](#route-group-middleware)
    - [Namespaces](#route-group-namespaces)
    - [네임스페이스](#route-group-namespaces)
    - [Sub-Domain Routing](#route-group-sub-domain-routing)
    - [서브 도메인 라우팅](#route-group-sub-domain-routing)
    - [Route Prefixes](#route-group-prefixes)
    - [라우트 Prefixes](#route-group-prefixes)
- [Route Model Binding](#route-model-binding)
- [라우트 모델 바인딩](#route-model-binding)
    - [Implicit Binding](#implicit-binding)
    - [명시적 바인딩](#implicit-binding)
    - [Explicit Binding](#explicit-binding)
    - [묵시적 바인딩](#explicit-binding)
- [Form Method Spoofing](#form-method-spoofing)
- [Form Method Spoofing](#form-method-spoofing)
- [Accessing The Current Route](#accessing-the-current-route)
- [현재 라우트에 엑세스하기](#accessing-the-current-route)

<a name="basic-routing"></a>
## Basic Routing
## 기본적인 라우팅

The most basic Laravel routes simply accept a URI and a `Closure`, providing a very simple and expressive method of defining routes:

가장 기본적인 라라벨 라우트는 간단하게 URI 와 `클로저`를 전달 받아, 라우팅을 정의하는 간단하고 쉽게 이해할 수 있는 방법을 제공합니다:

    Route::get('foo', function () {
        return 'Hello World';
    });

#### The Default Route Files
#### 기본 라우트 파일

All Laravel routes are defined in your route files, which are located in the `routes` directory. These files are automatically loaded by the framework. The `routes/web.php` file defines routes that are for your web interface. These routes are assigned the `web` middleware group, which provides features like session state and CSRF protection. The routes in `routes/api.php` are stateless and are assigned the `api` middleware group.

모든 라라벨의 라우트는 `route` 디렉토리 안에 들어 있는 라우트 파일에 정의되어 있습니다. 이 파일들은 프레임워크에 의해서 자동으로 로드됩니다. `routes/web.php` 파일은 웹 인터페이스를 위한 라우트들을 정의합니다. 이 라우트들에는 세션 상태와 CSRF 보호와 같은 기능을 제공하는 `web` 미들웨어 그룹이 할당되어 있습니다. `routes/api.php` 안에 들어 있는 라우트들은 stateless 하고 `api` 미들웨어 그룹이 할당되어 있습니다.

For most applications, you will begin by defining routes in your `routes/web.php` file. The routes defined in `routes/web.php` may be accessed by entering the defined route's URL in your browser. For example, you may access the following route by navigating to `http://your-app.dev/user` in your browser:

대부분의 어플리케이션에서, 여러분은 `routes/web.php` 파일에 라우트를 정의하여 시작할 수 있습니다. `routes/web.php` 에 정의된 라우트는 브라우저를 통해서 유입되는 라우트 URL을 정의하는데 사용됩니다. 예를 들어 브라우저에서 `http://your-app.dev/user`와 같이 접속하기 위해서 다음의 라우트를 정의할 수 있습니다:

    Route::get('/user', 'UserController@index');

Routes defined in the `routes/api.php` file are nested within a route group by the `RouteServiceProvider`. Within this group, the `/api` URI prefix is automatically applied so you do not need to manually apply it to every route in the file. You may modify the prefix and other route group options by modifying your `RouteServiceProvider` class.

`routes/api.php` 파일은 `RouteServiceProvider`의 라우트 그룹안에 중첩되어 정의되어 있습니다. 이 그룹을에 의해서 `/api` URI가 자동으로 앞에 붙게 되므로, 이 파일에 정의한 모든 라우트에 일일이 적용하지 않아도 됩니다. `RouteServiceProvider` 파일의 라우트 그룹 옵션을 수정하면, 다른 prefix를 붙일 수도 있습니다.

#### Available Router Methods
#### 가능한 라우터 메소드

The router allows you to register routes that respond to any HTTP verb:

라우터는 다음의 HTTP 메소드에 해당하는 응답을 위한 라우트를 등록할 수 있습니다:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

Sometimes you may need to register a route that responds to multiple HTTP verbs. You may do so using the `match` method. Or, you may even register a route that responds to all HTTP verbs using the `any` method:

때로는 여러개의 HTTP 메소드에 응답하는 라우트를 등록해야 할 수도 있습니다. 이경우 `match` 메소드를 사용하면 됩니다. 또는 `any` 메소드를 사용하여 모든 HTTP 메소드에 응답하는 라우트를 등록할 수도 있습니다:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF Protection
#### CSRF 보호하기

Any HTML forms pointing to `POST`, `PUT`, or `DELETE` routes that are defined in the `web` routes file should include a CSRF token field. Otherwise, the request will be rejected. You can read more about CSRF protection in the [CSRF documentation](/docs/{{version}}/csrf):

`web` 라우트 파일 안에 정의된 `POST`, `PUT` 또는 `DELETE` 를 가리키는 라우트들은 모두 CSRF 토큰 필드를 포함해야합니다. 그렇지 않으면, 이 request들은 거부될 것입니다. CSRF 보호에 대해서 더 알아보려면 [CSRF 문서](/docs/{{version}}/csrf)를 읽어보십시오:

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

<a name="redirect-routes"></a>
### Redirect Routes
### 리다이렉트 라우트

If you are defining a route that redirects to another URI, you may use the `Route::redirect` method. This method provides a convenient shortcut so that you do not have to define a full route or controller for performing a simple redirect:

다른 URI로 리다이렉트 시키는 라우트를 정의하려면, `Route::redirect` 메소드를 사용할 수 있습니다. 이 메소드는 간단한 리다이렉트를 위해서 복잡한 라우트나 컨트롤러 전체를 정의하지 않아도 되는 편리한 방법을 제공합니다.

    Route::redirect('/here', '/there', 301);

<a name="view-routes"></a>
### View Routes
### 뷰-View 라우트

If your route only needs to return a view, you may use the `Route::view` method. Like the `redirect` method, this method provides a simple shortcut so that you do not have to define a full route or controller. The `view` method accepts a URI as its first argument and a view name as its second argument. In addition, you may provide an array of data to pass to the view as an optional third argument:

단지 뷰를 반환하기만 하는 라우트가 필요하다면, `Route::view` 메소드를 사용할 수 있습니다. `redirect` 메소드와 같이 이 메소드는 간단한 리다이렉트를 위해서 복잡한 라우트나 컨트롤러 전체를 정의하지 않아도 되는 편리한 방법을 제공합니다. `view` 메소드는 첫번째 인자로 URI를, 두번째 인자로 뷰 파일의 이름을 전달 받습니다. 또한 이에 더해, 세번째 인자로 view 에 제공할 데이터들의 배열을 제공할 수도 있습니다:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## Route Parameters
## 라우트 파라미터

<a name="required-parameters"></a>
### Required Parameters
### 필수 파라미터

Of course, sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You may do so by defining route parameters:

라우트중에 URI 세그먼트를 필요로 할 수도 있습니다. 다음과 같이 URL 에서 사용자의 ID를 확인하고자 하는 경우 입니다. 이 경우 라우트 파라미터를 정의할 수 있습니다:

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

You may define as many route parameters as required by your route:

라우트에서는 여러개의 라우트 파라미터를 정의할 수도 있습니다:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Route parameters are always encased within `{}` braces and should consist of alphabetic characters, and may not contain a `-` character. Instead of using the `-` character, use an underscore (`_`). Route parameters are injected into route callbacks / controllers based on their order - the names of the callback / controller arguments do not matter.

라우트 파라미터는 항상 "{}"(중괄호)로 쌓여져 있고, `-` 문자를 포함하지 않은 알파벳 문자로 구성되어 있어야합니다. `-` 문자는 사용하기 보다는 대신 (`_`) 언어스코어를 사용하십시오. 라우트 파라미터는 라우트 콜백 / 컨트롤러에 주입되는데 이때 사용되는 콜백 / 컨트롤러 인자에서 문제가 되지 않는 이름이어야 합니다.

<a name="parameters-optional-parameters"></a>
### Optional Parameters
### 선택적 파라미터

Occasionally you may need to specify a route parameter, but make the presence of that route parameter optional. You may do so by placing a `?` mark after the parameter name. Make sure to give the route's corresponding variable a default value:

때로는, 라우트 파라미터를 지정하긴 하지만, 파라미터가 선택적으로 존재하기를 원할수도 있습니다. 이 경우 파라미터 이름뒤에 `?` 를 표시하면 됩니다. 라우트 파라미터와 일치하는 변수가 기본값을 가지는지 확인하십시오:

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### Regular Expression Constraints
### 정규표현식 제약

You may constrain the format of your route parameters using the `where` method on a route instance. The `where` method accepts the name of the parameter and a regular expression defining how the parameter should be constrained:

라우트 인스턴스에 `where` 메소드를 사용하여 라우트 파라미터들의 포맷을 제한할 수 있습니다. `where` 메소드는 파라미터의 이름과 파라미터가 어떻게 규정되어야 하는지 나타내는 정규표현식을 인자로 전달 받습니다:

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### Global Constraints
#### 글로벌 제약

If you would like a route parameter to always be constrained by a given regular expression, you may use the `pattern` method. You should define these patterns in the `boot` method of your `RouteServiceProvider`:

라우트 파라미터가 항상 주어진 정규표현식으로 제약을 가지게 된다면, `pattern` 메소드를 사용할 수 있습니다. 이 패턴들은 `RouteServiceProvider`의 `boot` 메소드 안에서 사용해야 합니다:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

Once the pattern has been defined, it is automatically applied to all routes using that parameter name:

패턴을 한번 정의하고나면, 해당 파라미터 이름을 사용하는 모든 라우트들에 자동으로 적용됩니다:

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="named-routes"></a>
## Named Routes
## 이름이 지정된 라우트

Named routes allow the convenient generation of URLs or redirects for specific routes. You may specify a name for a route by chaining the `name` method onto the route definition:

이름이 지정된 라우트는 URL 이나 지정된 라우트로의 리다이렉션을 손쉽게 생성하기 편리하게 해줍니다. 라우트 정의에 `name` 메소드를 체이닝 하여 라우트에 이름을 지정할 수 있습니다:

    Route::get('user/profile', function () {
        //
    })->name('profile');

You may also specify route names for controller actions:

컨트롤러의 액션에도 라우트 이름을 지정할 수 있습니다:

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### Generating URLs To Named Routes
#### 이름이 지정된 라우트들에 대한 URL 생성하기

Once you have assigned a name to a given route, you may use the route's name when generating URLs or redirects via the global `route` function:

주어진 라우트에 대한 이름이 할당되면, 전역 `route` 함수를 통해서 URL 또는 리다이렉션을 생성할 때 라우트 이름을 사용할 수 있습니다:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

If the named route defines parameters, you may pass the parameters as the second argument to the `route` function. The given parameters will automatically be inserted into the URL in their correct positions:

이름이 지정된 라우트에 파라미터를 정의하였다면, `route` 메소드의 두번째 인자로 파라미터를 전달할 수 있습니다. 주어진 파라미터는 자동으로 올바른 위치에 있는 URL 에 삽입될 것입니다:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

#### Inspecting The Current Route

#### 현재의 라우트 검사하기

If you would like to determine if the current request was routed to a given named route, you may use the `named` method on a Route instance. For example, you may check the current route name from a route middleware:

현재의 request 가 주어진 이름의 라우트가 맞는지 확인하고자 한다면, Route 인스턴스의 `named` 메소드를 사용하면 됩니다. 예를 들어 라우트 미들웨어에서 현재 라우트의 이름을 확인할 수 있습니다:

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>
## Route Groups
## 라우트 그룹

Route groups allow you to share route attributes, such as middleware or namespaces, across a large number of routes without needing to define those attributes on each individual route. Shared attributes are specified in an array format as the first parameter to the `Route::group` method.

라우트 그룹을 사용하면 미들웨어나, 네임스페이스와 같은 라우트 속성을 공유할 수 있어, 많은 수의 라우트를 등록할 때 각각의 개별 라우트에 매번 속성들을 정의하지 않아도 되게 해줍니다. 공유하려는 속성은 배열 형식으로 지정되어 `Route::group` 메소드의 첫번째 인자로 전달됩니다.

<a name="route-group-middleware"></a>
### Middleware
### 미들웨어

To assign middleware to all routes within a group, you may use the `middleware` method before defining the group. Middleware are executed in the order they are listed in the array:

그룹 안의 모든 라우트에 미들웨어를 할당하기 위해서는, 그룹을 정의하기 전에 `middleware` 메소드를 사용하면 됩니다. 미들웨어는 배열에 나열된 순서대로 실행될 것입니다:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second Middleware
        });

        Route::get('user/profile', function () {
            // Uses first & second Middleware
        });
    });

<a name="route-group-namespaces"></a>
### Namespaces
### 네임스페이스

Another common use-case for route groups is assigning the same PHP namespace to a group of controllers using the `namespace` method:

라우트 그룹을 사용하는 또 다른 사용 예로는 `namespace` 메소드를 사용하는 컨트롤러들에 동일한 PHP 네임스페이스를 할당하는 경우 입니다:

    Route::namespace('Admin')->group(function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

Remember, by default, the `RouteServiceProvider` includes your route files within a namespace group, allowing you to register controller routes without specifying the full `App\Http\Controllers` namespace prefix. So, you only need to specify the portion of the namespace that comes after the base `App\Http\Controllers` namespace.

주의할점은, 기본적으로 `RouteServiceProvider` 는 `App\Http\Controllers` 네임스페이스를 접두사로 굳지 지정하지 않아도 컨트롤러가 등록되도록, 네임스페이스 그룹 안에서 라우트 파일을 로드한다는 것입니다. 따라서 여러분들이 네임스페이스에서 필요한 부분은 `App\Http\Controllers` 네임스페이스 뒷부분만 지정하면 됩니다.

<a name="route-group-sub-domain-routing"></a>
### Sub-Domain Routing
### 서브 도메인 라우팅

Route groups may also be used to handle sub-domain routing. Sub-domains may be assigned route parameters just like route URIs, allowing you to capture a portion of the sub-domain for usage in your route or controller. The sub-domain may be specified by calling the `domain` method before defining the group:

라우트 그룹은 또한 카드형태의 서브 도메인을 처리하는데 사용할 수도 있습니다. 서브 도메인은 라우트 URI와 같이 서브 도메인의 일부를 추출하여, 라우트 파라미터로 할당할 수 있습니다. 서브 도메인은 그룹을 정의하기 전에 `domain` 메소드를 호출하여 지정할 수 있습니다:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### Route Prefixes
### 라우트 Prefix

The `prefix` method may be used to prefix each route in the group with a given URI. For example, you may want to prefix all route URIs within the group with `admin`:

`prefix` 메소드는 그룹안의 라우트에 특정 URI을 접두어로 지정할 때 사용합니다. 그룹의 모든 라우트 URI 앞에 `admin` 을 붙이고 싶다면 다음과 같이 지정하면 됩니다:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-model-binding"></a>
## Route Model Binding
## 라우트 모델 바인딩

When injecting a model ID to a route or controller action, you will often query to retrieve the model that corresponds to that ID. Laravel route model binding provides a convenient way to automatically inject the model instances directly into your routes. For example, instead of injecting a user's ID, you can inject the entire `User` model instance that matches the given ID.

모델 ID 를 라우트나 컨트롤러 액션에 주입한 경우, 여러분은 곧잘 ID와 일치하는 모델을 조회하기 위해서 쿼리를 수행할 것입니다. 라라벨 라우트 모델 바인딩은 여러분의 라우트에 자동으로 모델 인스턴스를 직접 주입하는 편리한 방법을 제공합니다. 예를 들어 사용자의 ID를 주입하는 대신 주어진 ID와 매칭되는 전체 `User` 모델 인스턴스를 주입할 수 있습니다.

<a name="implicit-binding"></a>
### Implicit Binding
### 묵시적 바인딩

Laravel automatically resolves Eloquent models defined in routes or controller actions whose type-hinted variable names match a route segment name. For example:

라라벨은 자동으로 라우트나 컨트롤러 액션에서 정의되어 있는 라우트 세그먼트 이름과 일치하는 타입힌트된 변수에 해당하는 Eloquent 모델을 의존성 해결합니다. 예를 들어:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

Since the `$user` variable is type-hinted as the `App\User` Eloquent model and the variable name matches the `{user}` URI segment, Laravel will automatically inject the model instance that has an ID matching the corresponding value from the request URI. If a matching model instance is not found in the database, a 404 HTTP response will automatically be generated.

`App\User` Eloquent 모델로 타입힌트된 ``$user` 변수와 `{user}` 세그먼트가 일치하기 때문에, 라라벨은 자동으로 request URI 로 부터 일치하는 ID 값을 가진 모델 인스턴스를 주입할것입니다. 만약 데이터베이스에서 매칭되는 모델 인스턴스를 찾을 수 없으면, 자동으로 404 HTTP response 생성됩니다.

#### Customizing The Key Name
#### 키의 이름을 변경하기

If you would like model binding to use a database column other than `id` when retrieving a given model class, you may override the `getRouteKeyName` method on the Eloquent model:

주어진 모델을 클래스를 찾을 때 `id` 와는 다른 데이터베이스 컬럼을 사용하는 모델 바인딩을 하고자 한다면, Eloquent 모델의 `getRouteKeyName` 메소드를 재지정하면 됩니다:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### Explicit Binding
### 명시적 바인딩

To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings in the `boot` method of the `RouteServiceProvider` class:

명시적 바인딩을 등록하기 위해서, 주어진 파라미터에 대한 클래스를 지정하려면 라우터의 `model` 메소드를 사용하십시오. 여러분은 `RouteServiceProvider` 클래스의 `boot` 메소드 안에서 명시적 모델 바인딩을 정의해야 합니다:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

Next, define a route that contains a `{user}` parameter:

다음으로, `{user}` 파라미터를 포함한 라우트를 정의합니다:

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

Since we have bound all `{user}` parameters to the `App\User` model, a `User` instance will be injected into the route. So, for example, a request to `profile/1` will inject the `User` instance from the database which has an ID of `1`.

`{user}` 파라미터와 `App\User` 모델이 바인딩되어 있기 때문에 라우트에는 `User` 인스턴스가 주입 될것입니다. 예를 들어 `profile/1`으로 요청이 들어오면 데이터베이스로 부터 ID가 `1`인 `User`의 인스턴스가 주입됩니다.

If a matching model instance is not found in the database, a 404 HTTP response will be automatically generated.

만약 데이터베이스에서 일치하는 모델 인스턴스를 찾지 못하는 경우, 404 응답이 자동으로 생성될 것입니다.

#### Customizing The Resolution Logic
#### 의존성 해결 로직 커스터마이징하기

If you wish to use your own resolution logic, you may use the `Route::bind` method. The `Closure` you pass to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

만약 여러분의 고유한 의존성 해결 로직을 사용하려면 `Route::bind` 메소드를 사용할 수 있습니다. `bind` 메소드에 전달되는 `클로저`에는 URI 세그먼트에 해당하는 값이 전달되고 라우트에 주입되어야 하는 클래스의 인스턴스를 반환해야 합니다:

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first() ?? abort(404);
        });
    }

<a name="form-method-spoofing"></a>
## Form Method Spoofing
## Form 메소드 Spoofing-속이기

HTML forms do not support `PUT`, `PATCH` or `DELETE` actions. So, when defining `PUT`, `PATCH` or `DELETE` routes that are called from an HTML form, you will need to add a hidden `_method` field to the form. The value sent with the `_method` field will be used as the HTTP request method:

HTML form은 `PUT`, `PATCH` 와 `DELETE` 액션을 지원하지 않습니다. 따라서 `PUT`, `PATCH` 이나 `DELETE` 로 지정된 라우트를 호출하는 HTML form을 정의한다면 `_method` 의 숨겨진 필드를 지정해야합니다. `_method` 필드로 보내진 값은 HTTP request 메소드를 판별하는데 사용됩니다:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

You may use the `method_field` helper to generate the `_method` input:

`_method` 입력을 생성하기 위해서 `method_filed` 헬퍼 함수를 사용할 수 있습니다:

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## Accessing The Current Route
## 현재 라우트에 엑세스하기

You may use the `current`, `currentRouteName`, and `currentRouteAction` methods on the `Route` facade to access information about the route handling the incoming request:

`Route` 파사드의 `current`, `currentRouteName` 그리고 `currentRouteAction` 메소드를 사용하여, 유입된 request-요청을 처리하는 라우트에 대한 정보에 엑세스 할 수 있습니다:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Refer to the API documentation for both the [underlying class of the Route facade](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) and [Route instance](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) to review all accessible methods.

모든 메소드를 확인하고자 한다면 [Route 파사드 뒤에서 동작하는 클래스](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) 와 [Route 인스턴스](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) API 문서를 참고하십시오.
