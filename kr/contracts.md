# Contracts

- [Introduction](#introduction)
- [Why Contracts?](#why-contracts)
- [Contract Reference](#contract-reference)
- [How To Use Contracts](#how-to-use-contracts)

<a name="introduction"></a>
## Introduction

Laravel's Contracts are a set of interfaces that define the core services provided by the framework. 라라벨의 콘트렉트는 프레임워크에서 제공하는 코어 서비스들을 정의한 인터페이스들의 모음입니다. For example, a `Queue` contract defines the methods needed for queueing jobs, while the `Mailer` contract defines the methods needed for sending e-mail. 예를 들어, `Queue` 콘트렉트에는 어떤 작업들을 큐에서 다룰때 필요한 메소드들이 정의되어 있고, `Mailer` 콘트렉트에는 이메일을 보내기 위해 필요한 메소드들을 정의되어 있습니다.

Each contract has a corresponding implementation provided by the framework. 라라벨 프레임워크에는 각각의 콘트렉트에 상응하는 구현체(구현 클래스)가 있습니다 . For example, Laravel provides a `Queue` implementation with a variety of drivers, and a `Mailer` implementation that is powered by [SwiftMailer](http://swiftmailer.org/). 예를 들어, 라라벨은 다양한 드라이버로 구현된 `Queue`의 구현체를 가지고 있고, `Mailer`의 구현체는 [SwiftMailer](http://swiftmailer.org/)를 통해 가지고 있습니다.

All of the Laravel contracts live in [their own GitHub repository](https://github.com/illuminate/contracts). 라라벨의 모든 콘트렉트는 [각각의 Github 저장소](https://github.com/illuminate/contracts)를 가지고 있습니다. This provides a quick reference point for all available contracts, as well as a single, decoupled package that may be utilized by other package developers. 이 것은 사용이 가능한  모든 콘트렉트들과 다른 패키지 개발자들에게 활용될지도 모르는 라라벨의 단일 패키지나 분리된 패키지에 대한 빠른 참조 포인트를 제공합니다.

<a name="why-contracts"></a>
## Why Contracts? 왜 콘트렉트인가?

You may have several questions regarding contracts. Why use interfaces at all? Isn't using interfaces more complicated? 아마 여러분은 콘트렉트와 관련해 몇가지 질문을 가지고 있을 것입니다. 왜 모든 것에 인터페이스를 사용하는 것일까? 인터페이스를 사용하면 더 복잡해지지 않나?

Let's distill the reasons for using interfaces to the following headings: loose coupling and simplicity. 다음의 주제로 인터페이스를 사용하는 이유에 대하여 살펴봅시다: 느슨한 결합과 단순성

### Loose Coupling 느슨한 결합

First, let's review some code that is tightly coupled to a cache implementation. Consider the following: 우선, 한 캐시 구현체에 밀접하게 결합돼 있는 코드를 살펴봅시다.

	<?php namespace App\Orders;

	class Repository {

		/**
		 * The cache.
		 */
		protected $cache;

		/**
		 * Create a new repository instance.
		 *
		 * @param  \SomePackage\Cache\Memcached  $cache
		 * @return void
		 */
		public function __construct(\SomePackage\Cache\Memcached $cache)
		{
			$this->cache = $cache;
		}

		/**
		 * Retrieve an Order by ID.
		 *
		 * @param  int  $id
		 * @return Order
		 */
		public function find($id)
		{
			if ($this->cache->has($id))
			{
				//
			}
		}

	}

In this class, the code is tightly coupled to a given cache implementation. 이 클래스의 코드는 주어진 캐시 구현체와 밀접하게 결합돼 있습니다. It is tightly coupled because we are depending on a concrete Cache class from a package vendor.  우리는 특정 패키지 벤더로부터 제공받은 캐시의 구상클래스에 의존하기 때문에 밀접하게 결합돼 있는 것입니다. If the API of that package changes our code must change as well. 만약 그 패키지의 API가 변경되면 우리의 코드 또한 변경되어야 합니다. 

Likewise, if we want to replace our underlying cache technology (Memcached) with another technology (Redis), we again will have to modify our repository. 또한, 우리가 기반을 두는 캐시 기술(Memcached)을 다른 기술(Redia)로 변경하고 싶다면, 우리는 저장소 클래스를 다시 수정해야만 할 것입니다.
Our repository should not have so much knowledge regarding who is providing them data or how they are providing it. 우리의 저장소클래스는 누가 어떻게 데이터를 제공하는지에 대한 너무 많은 정보를 가지고 있어서는 안 됩니다.
**Instead of this approach, we can improve our code by depending on a simple, vendor agnostic interface:** **이렇게 접근하는 대신, 특정 벤더에 구속되지 않고 단순한 인터페이스에 의존하도록 하여 코드를 개선할 수 있습니다:**

	<?php namespace App\Orders;

	use Illuminate\Contracts\Cache\Repository as Cache;

	class Repository {

		/**
		 * Create a new repository instance.
		 *
		 * @param  Cache  $cache
		 * @return void
		 */
		public function __construct(Cache $cache)
		{
			$this->cache = $cache;
		}

	}

Now the code is not coupled to any specific vendor, or even Laravel. 이제 코드는 어떤 특정 벤더, 심지어 라라벨과도 결합되지 않습니다. Since the contracts package contains no implementation and no dependencies, you may easily write an alternative implementation of any given contract, allowing you to replace your cache implementation without modifying any of your cache consuming code.
콘트랙트 패키지는 구현체를 가지지 않고 의존성도 없기 때문에, 우리는 캐시를 사용하는 코드를 수정하지 않고 캐시 구현체를 대체하는 것을 허용하는 콘트렉트의 다른 구현체를 작성할 수 있습니다.

### Simplicity 단순성

When all of Laravel's services are neatly defined within simple interfaces, it is very easy to determine the functionality offered by a given service. 모든 라라벨의 서비스들이 단순한 인터페이스로 깔끔하게 정의돼 있을 때, 그 서비스들에 의해 제공되는 기능을 알아내는 것은 매우 쉽습니다. **The contracts serve as succinct documentation to the framework's features.** **콘트렉트들은 프레임워크의 기능들에 대한 간결한 도큐먼트의 역할을 합니다.**
In addition, when you depend on simple interfaces, your code is easier to understand and maintain. 또한, 여러분이 단순한 인터페이스에 의존할 때, 여러분의 코드는 이해하거나 유지보수하기가 더 쉽워집니다.  Rather than tracking down which methods are available to you within a large, complicated class, you can refer to a simple, clean interface. 크고 복잡한 클래스에서 사용할 수 있는 메소드들을 훑어보는 대신, 단순하고 깨끗한 인터페이스를 참고할 수 있습니다.

<a name="contract-reference"></a>
## Contract Reference 콘트렉트 레퍼런스

This is a reference to most Laravel Contracts, as well as their Laravel "facade" counterparts: 아래는 대부분의 라라벨 콘트랙트와 그에 대응되는 파사드들의 레퍼런스입니다.

Contract  |  Laravel 4.x Facade
------------- | -------------
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="how-to-use-contracts"></a>
## How To Use Contracts 콘트랙트 사용법

So, how do you get an implementation of a contract? 그럼 어떻게 콘트랙트의 구현체를 얻을 수 있을까요? It's actually quite simple. 사실 매우 간단합니다. Many types of classes in Laravel are resolved through the [service container](/docs/5.0/container), including controllers, event listeners, filters, queue jobs, and even route Closures. 라라벨에 있는 여러 종류의 클래스들은 컨트롤러, 이벤트리스너, 필터, 큐 작업, 라우트 클로저들을 포함하는 [서비스 컨테이너](/docs/5.0/container)를 통해 마련됩니다. So, to get an implementation of a contract, you can just "type-hint" the interface in the constructor of the class being resolved. For example, take a look at this event handler: 그래서 어떤 콘트랙트의 구현체를 얻으려면 단지 "type-hint"를 

	<?php namespace App\Handlers\Events;

	use App\User;
	use App\Events\NewUserRegistered;
	use Illuminate\Contracts\Redis\Database;

	class CacheUserInformation {

		/**
		 * The Redis database implementation.
		 */
		protected $redis;

		/**
		 * Create a new event handler instance.
		 *
		 * @param  Database  $redis
		 * @return void
		 */
		public function __construct(Database $redis)
		{
			$this->redis = $redis;
		}

		/**
		 * Handle the event.
		 *
		 * @param  NewUserRegistered  $event
		 * @return void
		 */
		public function handle(NewUserRegistered $event)
		{
			//
		}

	}

When the event listener is resolved, the service container will read the type-hints on the constructor of the class, and inject the appropriate value. To learn more about registering things in the service container, check out [the documentation](/docs/5.0/container).
