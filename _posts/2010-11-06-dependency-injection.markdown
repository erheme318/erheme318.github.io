---
layout: post
title:  "Dependency Injection гэж юу вэ?"
date:   2010-11-06 22:42:33 +0800
categories: di
---

## Dependency Injection гэж юу вэ?

Dependency injection бол хамгийн энгийн загварчлалын аргуудын нэг. Гэсэн ч тайлбарлахад нилээд төвөгтэй аргачлалуудын нэг. Ихэнх тайлбар дээр Dependency Injection-ны талаар ойлгомжгүй, төвөгтэйгөөр тайлбарласан байдаг. Би PHP –тэй илүү тохиромжтойгоор тайлбарлахыг оролдоно. PHP голчлон вэб хөгжүүлэхэд зориулагдсан хэл учираас жишээ болгон вэб-ийн нэг энгийн жишээг авч үзье.

HTTP протокол-н дагуу төлөвгүй (stateless) байлгах шаардлагыг хангахын тулд вэб-д хэрэглэгчийн мэдээллийг вэбийн хүсэлтүүдэд хадгалж өгөх хэрэгтэй болдог. Ингэхийн тулд тун энгийнээр cookie эсвэл PHP-ын session гэсэн механизмуудыг ашиглаж болно.

```
$_SESSION['language'] = 'fr';
```

Дээрх код нь хэрэглэгчийн хэлийг session-ын language гэсэн хувьсагчид хадгална. Ингээд хэрэглэгчийн дараа дараагийн хүсэлтүүдэд language гэсэн утгыг глобал `$_SESSION` array-с авах боломжтой болно.

```
$user_language = $_SESSION['language'];
```

Dependency Injection нь зөвхөн Объект-Хандалтад програмчлалд яригддаг болохоор, бид PHP session механизмыг бүхэлд нь хамарсан SessionStorage гэсэн класс байгаа гэж үзэцгээе.

```php
class SessionStorage
{
  function __construct($cookieName = 'PHP_SESS_ID')
  {
    session_name($cookieName);
    session_start();
  }
 
  function set($key, $value)
  {
    $_SESSION[$key] = $value;
  }

  function get($key)
  {
    return $_SESSION[$key];
  }

  // ...
}
```

Мөн User класс interface-тэй:

```
class User
{
  protected $storage;

  function __construct()
  {
    $this->storage = new SessionStorage();
  }

  function setLanguage($language)
  {
    $this->storage->set('language', $language);
  }

  function getLanguage()
  {
    return $this->storage->get('language');
  }

  // ...
}
```

Эдгээр класууд нь маш энгийн бөгөөд User классыг ашиглахад ч мөн их хялбархан:

```
$user = new User();
$user->setLanguage('fr');
$user_language = $user->getLanguage();
```

Бид илүү уян хатан зүйлийг хүсэх хүртэл бүгд ямар ч асуудалгүй байна. Тухайлбал хэрэв чи cookie-ынхээ нэрийг өөрчилмөөр байвал яах уу? Зарим боломжууд бас байнаа.

 - User классын SessionStorage constructor -д нэрийг шууд оноож өгнө

```
class User
{
  function __construct()
  {
    $this->storage = new SessionStorage('SESSION_ID');
  }

  // ...
}
```


 - User классын гадна `constant` зарлана

```
class User
{
  function __construct()
  {
    $this->storage = new SessionStorage(STORAGE_SESSION_NAME);
  }

  // ...
}

define('STORAGE_SESSION_NAME', 'SESSION_ID');
```

 - Session-ны нэрийг User-ын `constructor` -н аргументараар нэмнэ

```
class User
{
  function __construct($sessionName)
  {
    $this->storage = new SessionStorage($sessionName);
  }

  // ...
}

$user = new User('SESSION_ID');
```

 - Storage класс-д боломжит утгуудыг array-р нэмнэ.

```
class User
{
  function __construct($storageOptions)
  {
    $this->storage = new SessionStorage($storageOptions['session_name']);
  }

  // ...
}

$user = new User(array('session_name' => 'SESSION_ID'));
```

Эдгээр бүх оролдлогууд ер нь тааруухан хувиалбарууд. Session-ны нэрийг User класс-д кодчилж өгөх нь үнэхээр асуудлыг шийдэж чадахгүй. Учир нь User классыг өөрчлөхгүйгээр дараа нь чи санаагаа хялбархан өөрчлөх боломжгүй. constant ашиглах нь мөн муу учир нь User класс нь constant тодорхойлогдохоос хамаарах учираас. Session-ны нэрийг аргументаар дамжуулах эсвэл боломжит утгуудыг array-р ашиглах нь хамгийн тохиромжтой боловч, бас ч тиймч сайн санаа биш. Энэ нь User-ын constructor аргументуудыг объекттой хамааралгүй өөр зүйлүүдтэй хольж замбраагүй болгоно.

Дээрхээс гадна хялбар шийдэх боломжгүй дахиад нэг асуудал байнаа. SessionStorage классыг яаж өөрчлөх вэ? Тухайлбал хялбархан тестлэхийн тулд `Mock` объектоор солих эсвэл session-оо баазад юм уу санах ойдоо хадгалахаар бол яах вэ? Одоогийн байгаа implementation-р (User класс-г өөрчлөхгүйгээр) энэ боломжүй.

Dependency Injection руу орцгооё. User классын дотор SessionStorage-ыг үүсгэхийн оронд User-ын объектыг SessionStorage-ын объектоор inject хийе: Энэ яах вэ гэвэл User классын constructor функцын аргументаар SessionStorage –ын объектыг дамжуулна гэсэн үг.

```
class User
{
  function __construct($storage)
  {
    $this->storage = $storage;
  }

  // ...
}
```

Энэ бол Dependency Injection. Өөр зүйл байхгүй. Одоо User классыг ашиглахад таныг эхлээд SessionStorage классын объект үүсгэх хэрэгтэй болно:

```
$storage = new SessionStorage('SESSION_ID');
$user = new User($storage);
```

Одоо session-ны хадгалах объектыг тохируулах нь тун энгийн бөгөөд session-г хадгалах классыг өөрчлөхөд мөн хялбархан болсон. Одоо User классыг өөрчлөхгүйгээр бүх зүйл боломжтой болсон (separation of concern).

[Pico Container website](http://www.picocontainer.org/injection.html) дээр Dependency Injection-г дараах маягаар тайлбарласан байна.

“Dependency Injection бол хаана компонент өөрсдөөсөө хамааралтайгаар `constructor` функцаар, функцуудаар (method), эсвэл шууд талбар руу оноох юм.”

Dependency Injection зөвхөн constructor функцаар хязгаарлагддаггүй.

 - Constructor Injection:

```
class User
{
  function __construct($storage)
  {
    $this->storage = $storage;
  }

  // ...
}
```

 - Setter Injection

```
class User
{
  function setSessionStorage($storage)
  {
    $this->storage = $storage;
  }

  // ...
}
```

 - Setter Injection:

```
class User
{
  public $sessionStorage;
}

$user->sessionStorage = $storage;
```

required dependencies буюу заавал шаардлагатай хамаарлуудыг constructor injection -р тодорхойлох, харин optional буюу заавал шаардлагатай биш хамаарлуудыг (жишээ нь cache объект) setter функцуудаар дамжуулж тодорхойлох нь илүү оновчтой байдаг.

Одоо ихэнх PHP framework-ууд Dependency Injection-г салангад (decoupled) боловч хоорондоо хамааралтай (cohesive) объектуудыг холбохын тулд ихээр ашигладаг:

```
// symfony: A constructor injection example
$dispatcher = new sfEventDispatcher();
$storage = new sfMySQLSessionStorage(array('database' => 'session', 'db_table' => 'session'));
$user = new sfUser($dispatcher, $storage, array('default_culture' => 'en'));

// Zend Framework: A setter injection example
$transport = new Zend_Mail_Transport_Smtp('smtp.gmail.com', array(
  'auth'     => 'login',
  'username' => 'foo',
  'password' => 'bar',
  'ssl'      => 'ssl',
  'port'     => 465,
));

$mailer = new Zend_Mail();
$mailer->setDefaultTransport($transport);
```

Хэрэв та Dependency Injection –ны талаар илүү мэдэхийг хүсвэл, [Martin Fowler introduction](http://www.martinfowler.com/articles/injection.html) эсвэл [Jeff More presentation](http://www.procata.com/talks/phptek-may2007-dependency.pdf) –аас харах боломжтой.

Өнөөдөртөө ингээд боллоо. Одоо та Dependency Injection-ны талаар илүү ойлголттой болсон байх гэж найдаж байна.

Эх сурвалж: [http://bit.ly/3ggju](http://bit.ly/3ggju)
