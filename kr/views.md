# Views
# 뷰-Views

- [Creating Views](#creating-views)
- [뷰 생성하기](#creating-views)
- [Passing Data To Views](#passing-data-to-views)
- [뷰에 데이터 전달하기](#passing-data-to-views)
    - [Sharing Data With All Views](#sharing-data-with-all-views)
    - [모든 뷰에서 데이터 공유하기](#sharing-data-with-all-views)
- [View Composers](#view-composers)
- [뷰 컴포저](#view-composers)

<a name="creating-views"></a>
## Creating Views
## 뷰 생성하기

Views contain the HTML served by your application and separate your controller / application logic from your presentation logic. Views are stored in the `resources/views` directory. A simple view might look something like this:

뷰는 애플리케이션에서 제공하는 HTML 로 구성되어 있으며, 컨트롤러 / 애플리케이션 로직을 프리젠테이션 로직에서 분리하는 역할을 수행합니다. 뷰파일들은 `resources/views` 디렉토리에 위치합니다. 간단한 뷰의 경우 다음처럼 보일 것입니다:

    <!-- View stored in resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>

Since this view is stored at `resources/views/greeting.blade.php`, we may return it using the global `view` helper like so:

이 뷰파일이 `resources/views/greeting.blade.php`로 저장되어 있다면, 여러분은 `view` 헬퍼를 사용하여 반환할 수 있습니다.

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

As you can see, the first argument passed to the `view` helper corresponds to the name of the view file in the `resources/views` directory. The second argument is an array of data that should be made available to the view. In this case, we are passing the `name` variable, which is displayed in the view using [Blade syntax](/docs/{{version}}/blade).

보이는 바와 같이 `view` 헬퍼 함수에 전달하는 첫번째 인자는 `resources/views` 디렉토리에 있는 파일의 이름이 됩니다. 두번째 인자는 뷰에서 사용될 데이터의 배열입니다. 이 예제에서는 뷰에서 [블레이드 문법](/docs/{{version}}/blade)을 통해서 보여지게 될 `name` 변수를 전달하고 있습니다.

Of course, views may also be nested within sub-directories of the `resources/views` directory. "Dot" notation may be used to reference nested views. For example, if your view is stored at `resources/views/admin/profile.blade.php`, you may reference it like so:

당연하게도 뷰는 `resources/views` 디렉토리의 중첩된 서브 디렉토리를 구성할 수 있습니다. 중첩된 뷰 파일을 참조하려면 "점"으로 구성된 표기법을 사용할 수 있습니다. 예를 들어 뷰파일이 `resources/views/admin/profile.blade.php` 처럼 저장되었다면 다음처럼 호출해야 합니다.

    return view('admin.profile', $data);

#### Determining If A View Exists
#### 뷰파일이 존재하는지 확인하기

If you need to determine if a view exists, you may use the `View` facade. The `exists` method will return `true` if the view exists:

뷰파일이 존재하는지 확인해야 되는 경우, `View` 파사드를 사용할 수 있습니다. `exist` 메소드는 뷰 파일이 존재한다면 `true` 를 반환할 것입니다.

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

<a name="passing-data-to-views"></a>
## Passing Data To Views
## 뷰에 데이터 전달하기

As you saw in the previous examples, you may pass an array of data to views:

이전 예제에서 확인하였듯이, 뷰에 데이터의 배열을 전달할 수 있습니다.

    return view('greetings', ['name' => 'Victoria']);

When passing information in this manner, `$data` should be an array with key/value pairs. Inside your view, you can then access each value using its corresponding key, such as `<?php echo $key; ?>`. As an alternative to passing a complete array of data to the `view` helper function, you may use the `with` method to add individual pieces of data to the view:

이러한 방식으로 정보를 전달할 때,`$data`는 키/값으로 구성된 배열이어야 합니다. 뷰 안에서 여러분은 `<?php echo $key; ?>` 와 같이 각각의 키에 해당하는 값에 엑세스 할 수 있습니다. 데이터를 `view` 헬퍼 함수에 전달하는 대신 `with` 메소드를 사용하여 뷰에 전달할 데이터를 개별적으로 추가 할 수도 있습니다.

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### Sharing Data With All Views
#### 모든 뷰파일에서 데이터 공유하기

Occasionally, you may need to share a piece of data with all views that are rendered by your application. You may do so using the view facade's `share` method. Typically, you should place calls to `share` within a service provider's `boot` method. You are free to add them to the `AppServiceProvider` or generate a separate service provider to house them:

때때로 애플리케이션에서 표시하는 모든 뷰에서 데이터를 공유할 필요가 있을 수도 있습니다. 여러분은 뷰 파사드의 `share` 메소드를 사용하면 됩니다. 일반적으로 이 코드는 서비스 프로바이더의 `boot` 메소드에 구성해 놓아야 합니다. 여러분은 `AppServiceProvider`에 이 코드를 구성해 놓거나, 다른 분리된 서비스 프로바이더를 생성할 수도 있습니다.

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="view-composers"></a>
## View Composers
## 뷰 컴포저

View composers are callbacks or class methods that are called when a view is rendered. If you have data that you want to be bound to a view each time that view is rendered, a view composer can help you organize that logic into a single location.

뷰 컴포저는 뷰가 렌더링 될 때 호출되는 콜백 또는 클래스 메소드입니다. 만약 뷰가 렌더링 될 때마다 뷰에 전달해야할 데이터를 가지고 있다면 뷰 컴포저는 해당 로직을 한곳에서 구성할수 있게 도와줄 수 있습니다.

For this example, let's register the view composers within a [service provider](/docs/{{version}}/providers). We'll use the `View` facade to access the underlying `Illuminate\Contracts\View\Factory` contract implementation. Remember, Laravel does not include a default directory for view composers. You are free to organize them however you wish. For example, you could create an `App\Http\ViewComposers` directory:

예를 들어, 뷰 컴포저를 [서비스 프로바이더](/docs/{{version}}/providers)를 통해서 구성해 봅시다. `Illuminate\Contracts\View\Factory` contract 구현체에 엑세스 하기 위해서 `View` 파사드를 사용할 것입니다. 기억할 것은 라라벨은 뷰 컴포저를 위한 어떠한 기본적인 디렉토리도 포함하고 있지 않다는 것입니다. 여러분은 자유롭게 뷰 컴포저를 구성할 수 있습니다. 예를 들어 `App\Http\ViewComposers` 디렉토리를 새롭게 생성할 수 있습니다.

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note} Remember, if you create a new service provider to contain your view composer registrations, you will need to add the service provider to the `providers` array in the `config/app.php` configuration file.

> {note} 기억할 것은 여러분이 뷰 컴포저를 등록하기 위한 새로운 서비스 프로바이더를 생성했다면, `config/app.php` 설정 파일의 `providers` 배열에 이 서비스 프로바이더를 추가해야 한다는 것입니다.

Now that we have registered the composer, the `ProfileComposer@compose` method will be executed each time the `profile` view is being rendered. So, let's define the composer class:

이제 뷰 컴포저를 등록했다면 `profile` 뷰가 렌더링 될 때마다 `ProfileComposer@compose` 메소드가 실행될 것입니다. 이제 컴포저 클래스를 정의해봅시다.

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\View\View;
    use App\Repositories\UserRepository;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

Just before the view is rendered, the composer's `compose` method is called with the `Illuminate\View\View` instance. You may use the `with` method to bind data to the view.

뷰가 렌더링되기 전에 뷰컴포저의 `composer` 메소드가 `Illuminate\View\View` 인스턴스와 함께 호출됩니다. 데이터를 전달하기 위해서 `with` 메소드를 사용할 수 있습니다.

> {tip} All view composers are resolved via the [service container](/docs/{{version}}/container), so you may type-hint any dependencies you need within a composer's constructor.

> {tip} 모든 뷰 컴포저의 의존성 주입은 [service container](/docs/{{version}}/container)를 통해서 이루어 집니다. 그렇기 때문에 필요한 객체의 경우 뷰 컴포저의 생성자에서 타입힌트를 지정한 형태로 지정하면 됩니다.

#### Attaching A Composer To Multiple Views
#### 컴포저를 다수의 뷰에 적용하기

You may attach a view composer to multiple views at once by passing an array of views as the first argument to the `composer` method:

`composer` 메소드의 첫번째 인자로 뷰 파일들의 배열을 전달하여, 뷰 컴포저가 적용될 다수의 뷰 파일들을 지정할 수 있습니다.

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

The `composer` method also accepts the `*` character as a wildcard, allowing you to attach a composer to all views:

`composer` 메소드는 또한 `*` 와일드 캐릭터로 인자를 받을 수 있는데, 이렇게 하면 모든 뷰에 뷰컴포저를 지정하게 됩니다.

    View::composer('*', function ($view) {
        //
    });

#### View Creators
#### 뷰 크리에이터

View **creators** are very similar to view composers; however, they are executed immediately after the view is instantiated instead of waiting until the view is about to render. To register a view creator, use the `creator` method:

뷰 **크리에이터**는 뷰 컴포저와 거의 비슷하게 동작합니다. 하지만 뷰 크리에이터는 뷰가 렌더링 되기를 기다리는 대신 인스턴스화 된 다음에 바로 실행됩니다. 뷰 크리에이터를 등록하기 위해서는 `creator` 메소드를 사용하면 됩니다.

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
