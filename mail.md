# 메일

- [소개하기](#introduction)
    - [드라이버 사전 준비사항](#driver-prerequisites)
- [Mailables 클래스 생성하기](#generating-mailables)
- [Mailables 클래스 작성하기](#writing-mailables)
    - [발송자 설정하기](#configuring-the-sender)
    - [View-뷰 설정하기](#configuring-the-view)
    - [뷰 데이터](#view-data)
    - [첨부 파일](#attachments)
    - [인라인 첨부](#inline-attachments)
- [메일 발송](#sending-mail)
    - [큐를 통한 메일 처리](#queueing-mail)
- [메일 & 로컬 개발환경](#mail-and-local-development)
- [이벤트](#events)

<a name="introduction"></a>
## 소개하기

라라벨은 인기있는 [SwiftMailer](http://swiftmailer.org)를 통해서 깔끔하고 단순한 API 를 제공하며, 로컬과 클라우드 기반의 메일 서비스를 통해서 어렵지 않게 메일을 사용할 수 있도록 SMTP, Mailgun, SparkPost, 아마존 SES, PHP 내장 `mail` 함수 그리고 `sendmail` 드라이버를 제공합니다.

<a name="driver-prerequisites"></a>
### 드라이버 사전준비사항

API를 기반으로한 Mailgun 과 SparkPost 드라이버의 경우 대게 SMTP 서버 보다 빠르고 간편합니다. 가능하다면, 이 드라이버들 중 하나를 사용하길 바랍니다. 모든 API 드라이버는 컴포저 패키지 매니저를 통해서 설치할 수 있는 Guzzle HTTP 라이브러리를 필요로 합니다:

    composer require guzzlehttp/guzzle

#### Mailgun 드라이버

Mailgun 드라이버를 사용하려면 먼저 Guzzle 을 설치하고, `config/mail.php` 설정파일에 `driver` 옵션을 `mailgun`으로 설정하면 됩니다. 다음으로 `config/services.php` 설정 파일이 다음 내용을 포함하고 있는지 확인하십시오:

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### SparkPost 드라이버

SparkPost 드라이버를 사용하려면 먼저 Guzzle 을 설치하고, `config/mail.php` 설정파일에 `driver` 옵션을 `sparkpost`으로 설정하면 됩니다. 다음으로 `config/services.php` 설정 파일이 다음 내용을 포함하고 있는지 확인하십시오:

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

#### AWS SES 드라이버

아마존 SES 드라이버를 사용하려면 먼저 반드시 아마존 AWS SDK for PHP 를 설치해야 합니다. `composer.json` 파일의 `require` 부분에 다음 라인을 추가하고 `composer update` 명령어를 실행하여 설치할 수 있습니다.

    "aws/aws-sdk-php": "~3.0"

다음으로 `config/mail.php` 설정 파일의 `driver` 옵션을 `ses` 로 설정하고 `config/services.php` 설정 파일이 다음과 같은 옵션을 포함하고 있는지 확인하십시오:

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="generating-mailables"></a>
## Mailables 클래스 생성하기

라라벨에서, 어플리케이션에 의해서 발송되는 이메일의 각 타입들은 "mailable" 클래스로 표현할 수 있습니다. 이 클래스들은 `app/Mail` 디렉토리안에 들어 있습니다. 어플리케이션에서 이 디렉토리가 보이지 않더라도 걱정하지 마십시오. `make:mail` 명령어를 사용하여 첫번째 mailable 클래스를 생성하면 자동으로 생성됩니다:

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## Mailables 클래스 작성하기

모든 mailable 클래스의 설정은 `build` 메소드에 들어 있습니다. 이 메소드 안에서 여러분은 `from`, `subject`, `view` 그리고 `attach` 와 같은, 이메일의 형태와 배송에 대해서 설정할 수 있는 다양한 메소드르르 사용할 수 있습니다.

<a name="configuring-the-sender"></a>
### 발송자 설정하기

#### `from` 메소드 사용하기

먼저 이메일의 발송자 설정을 살펴보겠습니다. 또는 다른 말로 누구로 부터 이메일이 전달되는지에 대해서 말입니다. 발송자를 설정하는 방법에는 두가지가 있습니다. 먼저 mailable 클래스의 `build` 메소드 안에서 `from` 메소드를 사용하는 것입니다:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

#### 글로벌 `from` 메일 주소 사용하기

하지만 어플리케이션이 이메일에서 사용하는 모든 "발송자" 주소가 동일하다면, 생성한 각각의 mailable 클래스 에서 매번 `from` 메소드를 호출하는 것은 번거로운 일입니다. 대신에, 글로벌 "발송자" 주소를 `config/mail.php` 설정 파일에서 지정할 수 있습니다. mailable 클래스 안에서 다른 "from" 주소를 지정하지 않는다면 이 글로벌 주소가 사용될 것입니다:

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### View-뷰 파일 설정하기

mailable 클래스의 `build` 메소드 안에서 이메일 컨텐츠를 렌더링 할때 사용해야 하는 템플릿을 지정하기 위해서 `view` 메소드를 사용할 수 있습니다. 각각의 이메일은 컨텐츠를 렌더링 하기 위해서 일반적으로 [블레이드 템플릿](/docs/{{version}}/blade)을 사용하기 때문에, 이메일의 HTML을 구성하는데 블레이드 템플릿 엔진의 강력하고 편리한 기능을 사용할 수 있습니다:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} You may wish to create a `resources/views/emails` directory to house all of your email templates; however, you are free to place them wherever you wish within your `resources/views` directory.

> {tip} 모든 이메일 템플릿을 모아놓기 위한 `resources/views/emails` 디렉토리를 만들기를 원할 수도 있습니다; 하지만, 실제로는 `resources/views` 디렉토리 안에 어디에 구성하더라도 상관없습니다.

#### 텍스트 전용 이메일

이메일을 순수 텍스트 버전으로 정의하고자 한다면, `text` 메소드를 사용하면 됩니다. `view` 메소드와 같이 `text` 메소드는 이메일의 컨텐츠를 렌더링하는데 사용하게될 템플릿의 이름을 인자로 전달받습니다. 이메일 메세지를 HTML 과 순수 텍스트 버전 어느것으로도 정의할 수 있습니다:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>
### 뷰 데이터

#### public 속성을 사용하여

일반적으로 이메일의 HTML을 렌더링 할 때, 뷰에서 구성할 수 있는 데이터를 전달하기를 원할 것입니다. 여기에는 뷰에 데이터를 전달하는 두 가지 방법이 있습니다. 먼저, mailable 클래스에 정의되어 있는 public 속성은 자동으로 뷰에서 사용할 수 있습니다. 따라서, 예를들어 데이터를 mailable 클래스의 생성자에 전달하고 이 데이터를 클래스에 정의되어 있는 public 속성에 지정할 수 있습니다:

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        public $order;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

데이터가 public 속성에 지정되고 나면, 뷰에서 자동적으로 이를 사용할 수 있습니다. 따라서 블레이트 템플릿안에서 다른 데이터에 엑세스하는 것처럼 데이터에 접근할 수 있습니다:

    <div>
        Price: {{ $order->price }}
    </div>

#### `with` 메소드를 사용하여:

만약 여러분이 이메일 데이터의 유형이 템플릿에 전달되기 전에 수정을 가하고 싶다면, `with` 메소드를 사용하여 수동으로 데이터를 뷰에 전달할 수 있습니다. 일반적으로, 이경우에도 여전히 데이터를 mailable 클래스의 생성자에 전달 될것입니다; 하지만 템플릿에서 자동으로 사용가능하지 않도록, 이 데이터를 `protected` 나 `private` 속성에 지정해야 합니다. 이제 템플릿에서 사용하고자 하는 데이터의 배열을 인자로 `with` 메소드를 호출 하십시오.

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        protected $order;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }

`with` 메소드에 데이터가 전달되면, 이는 자동적으로 뷰에서 사용이 가능합니다. 따라서 블레이트 템플릿안에서 다른 데이터에 엑세스하는 것처럼 데이터에 접근할 수 있습니다:

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### 첨부 파일

이메일에 파일을 첨부하려면, mailable 클래스의 `build` 메소드 안에서 `attach` 메소드를 사용하면 됩니다. `attach` 메소드는 파일의 전체 패스(full path)를 첫번째 인자로 전달 받습니다:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file');
        }

이메일에 파일이 첨부 될 때, `attach` 메소드의 두번째 인자로 첨부 파일의 표시되는 이름과 MIME 타입을 지정할 수 있는 `배열`을 지정할 수도 있습니다:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file', [
                            'as' => 'name.pdf',
                            'mime' => 'application/pdf',
                        ]);
        }

#### Raw 데이터 첨부하기

`attachData` 메소드는 raw string 의 바이트를 첨부하는데 사용됩니다. 예를 들어 메모리에서 PDF 파일을 생성하고 이 파일을 디스크에 저장하지 않고 바로 메일에 첨부하는 경우에 사용할 수 있습니다. `attachData` 메소드는 첫번째 인자로 raw 데이터 바이트를, 두번째 인자로 파일의 이름을, 그리고 세번째 인자로 옵션배열을 전달 받습니다:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attachData($this->pdf, 'name.pdf', [
                            'mime' => 'application/pdf',
                        ]);
        }

<a name="inline-attachments"></a>
### 인라인 첨부

이메일에 인라인 이미지를 포함시키는 것은 번거로운 일입니다. 하지만 라라벨에서는 이메일에 이미지를 첨부하고 최적의 CID를 얻을 수 있는 편리한 방법을 제공합니다. 인라인 이미지를 포함시키기 위해서는 이메일 뷰 안에서 `$message` 메소드에 `embed` 메소드를 사용하면 됩니다. 라라벨은 자동적으로 모든 이메일 뷰 안에서 `$message` 변수를 사용할 수 있도록 만들기 때문에, 이를 수동으로 전달하는 것을 걱정할 필요가 없습니다:

    <body>
        Here is an image:

        <img src="{{ $message->embed($pathToFile) }}">
    </body>

#### Raw 데이터를 첨부하는 방법

이미 이메일 템플릿에 포함시키고자 하는 raw 데이터 문자열을 가지고 있다면, `$message` 변수에서 `embedData` 메소드를 사용하면 됩니다:

    <body>
        Here is an image from raw data:

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="sending-mail"></a>
## 메일 발송하기

메일을 발송하기 위해서는 `Mail` [파사드](/docs/{{version}}/facades)에서 `to` 메소드를 사용하면 됩니다. `to` 메소드는 이메일 주소, 하나의 사용자 인스턴스, 그리고 사용자들의 컬렉션을 전달 받습니다. 하나의 객체나 객체들의 컬렉션이 전달된다면 메일러는 자동으로 이것들의 `email` 과 `name` 속성을 이메일 수신자로 설정할 것입니다. 따라서 객체에서 이 속성들이 사용가능한지 확인해야 합니다. 수신자들을 지정하고 나면, mailable 클래스의 인스턴스를 `send` 메소드에 전달합니다:

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Mail\OrderShipped;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);

            // Ship order...

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

물론 메일을 보낼 때 "to"에서 수신자를 지정하는데 제한이 있지는 않습니다. "to", "cc", "bcc" 를 사용한 수신자 설정을 하나의 체이닝된 호출로 사용할 수도 있습니다:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### 메일을 큐를 통해서 보내기

#### 이메일을 큐로 보내기

메일을 송신하는 것은 어플리케이션의 응답 시간을 크게 저하시키기 때문에 많은 개발자들은 이메일 메세지를 백그라운드 에서 보낼 수 있도록 큐-대기행열에 넣어 처리하도록 합니다. 라라벨에서는 내장되어 있는 [일관된 큐 API](/docs/{{version}}/queues)를 통해서 이러한 작업을 손쉽게 수행할 수 있게 합니다. 이메일 메세지를 대기 큐에 넣기 위해서는 메세지의 수신자를 지정한 다음에, `Mail` 파사드의 `queue` 메소드를 호출하도록 하면 됩니다.

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

이 메소드는 자동으로 큐에 작업을 추가하여 메세지가 백그라운드에서 보내지도록 할것입니다. 이 기능을 사용하기 위해서는 [큐 설정하기](/docs/{{version}}/queues)를 확인하셔야 합니다.

#### 큐에서 메일을 지연시켜서 보내기

큐를 통해서 이메일을 보낼 때 시간을 지연시켜서 보내고자 한다면, `later` 메소드를 사용하면 됩니다. `later` 메소드의 첫번째 인자로 몇초동안 지연시킬 것인지 나타내는 `DateTime` 인스턴스를 전달 받습니다:

    $when = Carbon\Carbon::now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### 지정된 큐로 작업 보내기

`make:mail` 명령어를 사용하여 생성된 모든 mailable 클래스들은 `Illuminate\Bus\Queueable` 트레이트를 사용하기 때문에 어떤 mailable 클래스 인스턴스라도 `onQueue` 와 `onConnection` 메소드를 호출할 수 있습니다. 이렇게 하면 메세지를 보내는 커넥션과 큐 이름을 지정할 수 있습니다:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### 기본을 큐로 발송하도록 설정하기

mailable 클래스가 항상 큐를 통해서 처리되도록 하려면, 클래스에 `ShouldQueue` implement 를 추가하면 됩니다. 그러면 `send` 메소드가 호출되어 메일이 발송될 때 contract에 의해서 큐를 메일이 통해서 발송됩니다:   

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="mail-and-local-development"></a>
## 메일 & 로컬 개발환경

이메일을 송신하는 어플리케이션을 개발 중이라면 실제로 이메일이 보내지기를 원하지는 않으실겁니다. 라라벨은 로컬 개발환경에서 이메일을 보내는 것은 "비활성화" 시킬 수 있는 몇가지 방법을 제공합니다.

#### 로그 드라이버

이메일을 발송하는 대신, `log` 메일 드라이버는 모든 이메일 메세지를 확인하기 위해서 로그파일에 기록합니다. 보다 구동 환경별 어플리케이션을 설정하는 보다 자세한 정보는 [설정 매뉴얼](/docs/{{version}}/configuration#environment-configuration)을 확인하십시오.

#### 모든 메일의 수신자 고정하기

두번째 해결책은 프레임워크에서 보내는 모든 이메일의 수신자를 고징시켜버리는 방법입니다. 이렇게 하면 어플리케이션에서 보내는 이메일들이 각자의 수신자에게 보내지는 대신에, 모두 하나의 지정된 주소로만 보내 질 것입니다. 이렇게 하려면 `config/mail.php` 설정 파일에서 `to` 옵션을 지정하면 됩니다:

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

마지막으로 [Mailtrap](https://mailtrap.io)과 같은 서비스를 사용하여 `smtp` 드라이버에서 실제 이메일 클라이언트에서 내용을 확인할 수 있는 "더미"의 수신함으로 메일을 보내는 방법입니다. 이 방법의 장점은 최종적으로 보내지는 메일을 실제로 확인할 수 있다는 점입니다:

<a name="events"></a>
## 이벤트

라라벨은 이메일을 보내기 전에 이벤트 하나를 발생시킵니다. 주의할 점은 이벤트는 이메일이 큐를 통하지 않고 바로 *보내질* 때 발생한다는 것입니다. 여러분은 `EventServiceProvider` 에서 이 이벤트를 위한 이벤트 리스너를 등록할 수 있습니다:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSentMessage',
        ],
    ];

