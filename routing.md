# 라우팅

- [기본적인 라우팅](#basic-routing)
    - [리다이렉트 라우트](#redirect-routes)
    - [뷰-View 라우트](#view-routes)
- [라우트 파라미터](#route-parameters)
    - [필수 파라미터](#required-parameters)
    - [선택적인 파라미터](#parameters-optional-parameters)
    - [정규표현식 제약](#parameters-regular-expression-constraints)
- [이름이 지정된 라우트](#named-routes)
- [라우트 그룹](#route-groups)
    - [미들웨어](#route-group-middleware)
    - [네임스페이스](#route-group-namespaces)
    - [서브 도메인 라우팅](#route-group-sub-domain-routing)
    - [라우트 Prefixes](#route-group-prefixes)
    - [라우트 이름 접두사](#route-group-name-prefixes)
- [라우트 모델 바인딩](#route-model-binding)
    - [명시적 바인딩](#implicit-binding)
    - [묵시적 바인딩](#explicit-binding)
- [대체 라우트](#fallback-routes)
- [Rate 제한](#rate-limiting)
- [Form-폼 메소드 Sppring-스푸핑](#form-method-spoofing)
- [현재 라우트에 엑세스하기](#accessing-the-current-route)

<a name="basic-routing"></a>
## 기본적인 라우팅

가장 기본적인 라라벨 라우트는 URI와 `클로저`를 전달 받아, 라우팅을 정의하는 간단하고 쉽게 이해할 수 있는 방법을 제공합니다:

    Route::get('foo', function () {
        return 'Hello World';
    });

#### 기본 라우트 파일

모든 라라벨의 라우트는 `route` 디렉토리 안에 들어 있는 라우트 파일에 정의되어 있습니다. 이 파일들은 프레임워크에 의해서 자동으로 로드됩니다. `routes/web.php` 파일은 웹 인터페이스를 위한 라우트들을 정의합니다. 이 라우트들에는 세션 상태와 CSRF 보호와 같은 기능을 제공하는 `web` 미들웨어 그룹이 할당되어 있습니다. `routes/api.php` 안에 들어 있는 라우트들은 stateless 하고 `api` 미들웨어 그룹이 할당되어 있습니다.

대부분의 애플리케이션에서, 여러분은 `routes/web.php` 파일에 라우트를 정의하여 시작할 수 있습니다. `routes/web.php` 에 정의된 라우트는 브라우저를 통해서 유입되는 라우트 URL을 정의하는데 사용됩니다. 예를 들어 브라우저에서 `http://your-app.test/user`와 같이 접속하기 위해서 다음의 라우트를 정의할 수 있습니다:

    Route::get('/user', 'UserController@index');

`routes/api.php` 파일은 `RouteServiceProvider`의 라우트 그룹안에 중첩되어 정의되어 있습니다. 이 그룹을에 의해서 `/api` URI가 자동으로 앞에 붙게 되므로, 이 파일에 정의한 모든 라우트에 일일이 적용하지 않아도 됩니다. `RouteServiceProvider` 파일의 라우트 그룹 옵션을 수정하면, 다른 prefix를 붙일 수도 있습니다.

#### 가능한 라우터 메소드

라우터는 다음의 HTTP 메소드에 해당하는 응답을 위한 라우트를 등록할 수 있습니다:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

때로는 여러개의 HTTP 메소드에 응답하는 라우트를 등록해야 할 수도 있습니다. 이경우 `match` 메소드를 사용하면 됩니다. 또는 `any` 메소드를 사용하여 모든 HTTP 메소드에 응답하는 라우트를 등록할 수도 있습니다:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF 보호하기

`web` 라우트 파일 안에 정의된 `POST`, `PUT` 또는 `DELETE` 를 가리키는 라우트들은 모두 CSRF 토큰 필드를 포함해야합니다. 그렇지 않으면, 이 request들은 거부될 것입니다. CSRF 보호에 대해서 더 알아보려면 [CSRF 문서](/docs/{{version}}/csrf)를 읽어보십시오:

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### 리다이렉트 라우트

다른 URI로 리다이렉트 시키는 라우트를 정의하려면, `Route::redirect` 메소드를 사용할 수 있습니다. 이 메소드는 간단한 리다이렉트를 위해서 복잡한 라우트나 컨트롤러 전체를 정의하지 않아도 되는 편리한 방법을 제공합니다.

    Route::redirect('/here', '/there');

기본적으로, `Route::redirect` 는 `302` 상태코드를 반환합니다. 만약 다른 상태코드를 반환하도록 커스터마이징 하고자 한다면, 세번째 인자를 옵션으로 전달하십시오:

    Route::redirect('/here', '/there', 301);

`Route::permananentRedirect` 메소드는 `301` 상태코드를 반환합니다:

    Route::permanentRedirect('/here', '/there');

<a name="view-routes"></a>
### 뷰-View 라우트

단지 뷰를 반환하기만 하는 라우트가 필요하다면, `Route::view` 메소드를 사용할 수 있습니다. `redirect` 메소드와 같이 이 메소드는 간단한 리다이렉트를 위해서 복잡한 라우트나 컨트롤러 전체를 정의하지 않아도 되는 편리한 방법을 제공합니다. `view` 메소드는 첫번째 인자로 URI를, 두번째 인자로 뷰 파일의 이름을 전달 받습니다. 또한 이에 더해, 세번째 인자로 view 에 제공할 데이터들의 배열을 제공할 수도 있습니다:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## 라우트 파라미터

<a name="required-parameters"></a>
### 필수 파라미터

라우트중에 URI 세그먼트를 필요로 할 수도 있습니다. 다음과 같이 URL 에서 사용자의 ID를 확인하고자 하는 경우 입니다. 이 경우 라우트 파라미터를 정의할 수 있습니다:

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

라우트에서는 여러개의 라우트 파라미터를 정의할 수도 있습니다:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

라우트 파라미터는 항상 "{}"(중괄호)로 싸여져 있고, `-` 문자를 포함하지 않은 알파벳 문자로 구성되어 있어야합니다. `-` 문자를 사용하기 보다는 언더스코어 (`_`) 를 사용하십시오. 라우트 파라미터는 라우트 콜백 / 컨트롤러에 주입되는데 이때 사용되는 콜백 / 컨트롤러 인자에서 문제가 되지 않는 이름이어야 합니다.

<a name="parameters-optional-parameters"></a>
### 선택적 파라미터

때로는, 라우트 파라미터를 지정하긴 하지만, 파라미터가 선택적으로 존재하기를 원할수도 있습니다. 이 경우 파라미터 이름뒤에 `?` 를 표시하면 됩니다. 라우트 파라미터와 일치하는 변수가 기본값을 가지는지 확인하십시오:

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### 정규표현식 제약

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
#### 글로벌 제약

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

패턴을 한번 정의하고나면, 해당 파라미터 이름을 사용하는 모든 라우트들에 자동으로 적용됩니다:

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="parameters-encoded-forward-slashes"></a>
#### 인코딩된 슬래쉬 파라미터

라라벨의 라우팅 컴포넌트는 `/` 문자를 제외한 모든 문자를 사용가능합니다. `/` 를 사용하려면, `where` 정규식 조건을 통해서 지정해야합니다:

    Route::get('search/{search}', function ($search) {
        return $search;
    })->where('search', '.*');

> {note} 인코딩된 슬래쉬 파라미터의 경우에는 가장 마지막 세그먼트에 대해서만 지원됩니다.

<a name="named-routes"></a>
## 이름이 지정된 라우트

이름이 지정된 라우트는 URL 이나 지정된 라우트로의 리다이렉션을 손쉽게 생성하기 편리하게 해줍니다. 라우트 정의에 `name` 메소드를 체이닝 하여 라우트에 이름을 지정할 수 있습니다:

    Route::get('user/profile', function () {
        //
    })->name('profile');

컨트롤러의 액션에도 라우트 이름을 지정할 수 있습니다:

    Route::get('user/profile', 'UserProfileController@show')->name('profile');

#### 이름이 지정된 라우트들에 대한 URL 생성하기

주어진 라우트에 대한 이름이 할당되면, 전역 `route` 함수를 통해서 URL 또는 리다이렉션을 생성할 때 라우트 이름을 사용할 수 있습니다:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

이름이 지정된 라우트에 파라미터를 정의하였다면, `route` 메소드의 두번째 인자로 파라미터를 전달할 수 있습니다. 주어진 파라미터는 자동으로 올바른 위치에 있는 URL 에 삽입될 것입니다:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

#### 현재의 라우트 검사하기

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
## 라우트 그룹

라우트 그룹을 사용하면 미들웨어나, 네임스페이스와 같은 라우트 속성을 공유할 수 있어, 많은 수의 라우트를 등록할 때 각각의 개별 라우트에 매번 속성들을 정의하지 않아도 되게 해줍니다. 공유하려는 속성은 배열 형식으로 지정되어 `Route::group` 메소드의 첫번째 인자로 전달됩니다.

<a name="route-group-middleware"></a>
### 미들웨어

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
### 네임스페이스

라우트 그룹을 사용하는 또 다른 사용 예로는 `namespace` 메소드를 사용하는 컨트롤러들에 동일한 PHP 네임스페이스를 할당하는 경우 입니다:

    Route::namespace('Admin')->group(function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

주의할점은, 기본적으로 `RouteServiceProvider` 는 `App\Http\Controllers` 네임스페이스를 접두사로 굳이 지정하지 않아도 컨트롤러가 등록되도록, 네임스페이스 그룹 안에서 라우트 파일을 로드한다는 것입니다. 따라서 여러분들이 네임스페이스에서 필요한 부분은 `App\Http\Controllers` 네임스페이스 뒷부분만 지정하면 됩니다.

<a name="route-group-sub-domain-routing"></a>
### 서브 도메인 라우팅

라우트 그룹은 또한 카드형태의 서브 도메인을 처리하는데 사용할 수도 있습니다. 서브 도메인은 라우트 URI와 같이 서브 도메인의 일부를 추출하여, 라우트 파라미터로 할당할 수 있습니다. 서브 도메인은 그룹을 정의하기 전에 `domain` 메소드를 호출하여 지정할 수 있습니다:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### 라우트 Prefix

`prefix` 메소드는 그룹안의 라우트에 특정 URI을 접두어로 지정할 때 사용합니다. 그룹의 모든 라우트 URI 앞에 `admin` 을 붙이고 싶다면 다음과 같이 지정하면 됩니다:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>
### 라우트 이름 접두사

`name` 메소드는 주어진 문자열을 가진 그룹 내의 각각의 라우트 이름에 접두사를 붙이는데 사용됩니다. 예를 들어, 그룹화 된 라우트들 전부에 `admin` 접두사를 붙일 수 있습니다. 전달된 문자열 그대로 라우트 이름 앞에 붙기 때문에, 접두사는 뒤에 `.` 문자를 포함하여 전달해야하는 것을 잊지말아야 합니다.

    Route::name('admin.')->group(function () {
        Route::get('users', function () {
            // Route assigned name "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## 라우트 모델 바인딩

모델 ID 를 라우트나 컨트롤러 액션에 주입한 경우, 여러분은 곧잘 ID와 일치하는 모델을 조회하기 위해서 쿼리를 수행할 것입니다. 라라벨 라우트 모델 바인딩은 여러분의 라우트에 자동으로 모델 인스턴스를 직접 주입하는 편리한 방법을 제공합니다. 예를 들어 사용자의 ID를 주입하는 대신 주어진 ID와 매칭되는 전체 `User` 모델 인스턴스를 주입할 수 있습니다.

<a name="implicit-binding"></a>
### 묵시적 바인딩

라라벨은 자동으로 라우트나 컨트롤러 액션에서 정의되어 있는 라우트 세그먼트 이름과 일치하는 타입힌트된 변수에 해당하는 Eloquent 모델을 의존성 해결합니다. 예를 들어:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

`App\User` Eloquent 모델로 타입힌트된 `$user` 변수와 `{user}` 세그먼트가 일치하기 때문에, 라라벨은 자동으로 request URI 로 부터 일치하는 ID 값을 가진 모델 인스턴스를 주입할것입니다. 만약 데이터베이스에서 매칭되는 모델 인스턴스를 찾을 수 없으면, 자동으로 404 HTTP response 가 생성됩니다.

#### 키의 이름을 변경하기

주어진 모델의 클래스를 찾을 때 `id` 와는 다른 데이터베이스 컬럼을 사용하는 모델 바인딩을 하고자 한다면, Eloquent 모델의 `getRouteKeyName` 메소드를 재지정하면 됩니다:

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
### 명시적 바인딩

명시적 바인딩을 등록하기 위해서, 주어진 파라미터에 대한 클래스를 지정하려면 라우터의 `model` 메소드를 사용하십시오. 여러분은 `RouteServiceProvider` 클래스의 `boot` 메소드 안에서 명시적 모델 바인딩을 정의해야 합니다:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

다음으로, `{user}` 파라미터를 포함한 라우트를 정의합니다:

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

`{user}` 파라미터와 `App\User` 모델이 바인딩되어 있기 때문에 라우트에는 `User` 인스턴스가 주입 될것입니다. 예를 들어 `profile/1`으로 요청이 들어오면 데이터베이스로 부터 ID가 `1`인 `User`의 인스턴스가 주입됩니다.

만약 데이터베이스에서 일치하는 모델 인스턴스를 찾지 못하는 경우, 404 응답이 자동으로 생성될 것입니다.

#### 의존성 해결 로직 커스터마이징하기

만약 여러분의 고유한 의존성 해결 로직을 사용하려면 `Route::bind` 메소드를 사용할 수 있습니다. `bind` 메소드에 전달되는 `클로저`에는 URI 세그먼트에 해당하는 값이 전달되고 라우트에 주입되어야 하는 클래스의 인스턴스를 반환해야 합니다:

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first() ?? abort(404);
        });
    }

<a name="fallback-routes"></a>
## 대체 라우트

`Route::fallback` 메소드를 사용하면 들어오는 요청과 일치하는 라우트가 없을 때 실행 할 라우트를 정의 할 수 있습니다. 일반적으로 처리하지 못한 요청은 어플리케이션의 exception 핸들러를 통해 자동으로 "404" 페이지를 렌더링합니다. 그러나 `routes/web.php` 파일에서 `fallback` 라우트를 정의 할 경우 `web` 미들웨어 그룹의 모든 미들웨어가 라우트에 적용됩니다.  물론, 필요할 경우 얼마든지 이 라우트에 미들웨어를 추가 할 수 있습니다 :

    Route::fallback(function () {
        //
    });
    
> {note} 대체 라우트는 항상 어플리케이션에서 등록한 마지막 라우트 여야합니다.

<a name="rate-limiting"></a>
## Rate 제한

라라벨은 라우트 접속을 제한하는 [미들웨어](/docs/{{version}}/middleware) 를 포함하고 있습니다. 이를 시작하려면, `throttle` 미들웨어를 라우트나 라우트 그룹에 지정해야 합니다. `throttle` 미들웨어는 지정된 분 동안의 최대 리퀘스트 수를 정하는 2개의 파라미터를 받습니다. 예를 들어, 인증된 유저가 아래의 라우트 그룹에 1분 당 60번까지 접속을 제한할 수 있습니다.

    Route::middleware('auth:api', 'throttle:60,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

#### 동적인 Rate 제한

인증된 `User` 모델의 attribute 를 기반으로 동적인 Request-요청의 최대치를 정할 수 있습니다. 예를 들어, `User` 모델이 `rate_limit` 라는 attribute 를 가지고 있다고 할 때, 최대 request-요청 수를 계산하기 위해  attribute 의 이름을 `throttle` 미들웨어에 전달 할 수 있습니다.

    Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

<a name="form-method-spoofing"></a>
## Form-폼 메소드 Sppring-스푸핑

HTML form은 `PUT`, `PATCH` 와 `DELETE` 액션을 지원하지 않습니다. 따라서 `PUT`, `PATCH` 나 `DELETE` 로 지정된 라우트를 호출하는 HTML form을 정의한다면 `_method` 의 숨겨진 필드를 지정해야합니다. `_method` 필드로 보내진 값은 HTTP request 메소드를 판별하는데 사용됩니다:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

`_method` 입력을 생성하기 위해서 `@method` 블레이드 지시어를 사용할 수 있습니다:

    <form action="/foo/bar" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## 현재 라우트에 엑세스하기

`Route` 파사드의 `current`, `currentRouteName` 그리고 `currentRouteAction` 메소드를 사용하여, 유입된 request-요청을 처리하는 라우트에 대한 정보에 엑세스 할 수 있습니다:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

모든 메소드를 확인하고자 한다면 [Route 파사드 뒤에서 동작하는 클래스](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) 와 [Route 인스턴스](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) API 문서를 참고하십시오.
