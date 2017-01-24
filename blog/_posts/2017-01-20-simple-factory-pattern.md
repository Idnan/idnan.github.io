---
layout: post
title: Simple Factory Pattern
comments: true
---

What is a factory? Let's imagine you order a new car; the dealer sends your order off to the factory and the factory builds your car. Your car is sent to you in its assembled form and you don't need to care about how it was made.

Similarly, a software factory produces objects for you. The factory takes your request, assembles the object using the constructor and gives them back to you to use. One of these types of Factory pattern is known as the Simple Factory. Let me show you how it works.

Firstly, we define an abstract class, which we want to extend with other classes:

```php
<?php

abstract class Notifier
{
    protected $to;
  
    public function __construct(string $to)
    {
        $this->to = $to;
    }
  
    abstract public function validateTo(): bool;
  
    abstract public function sendNotification(): string;
}
```

This class serves to allow us to have common methods and define whatever common functionality we want all the classes we build in our factory to have in common. We could also use interfaces instead of abstract classes for the implementation without defining any functionality whatsoever.

Using this interface, we can build two notifiers, `SMS` and `Email`.

The `SMS` notifier is as follows in the `SMS.php` file:

```php
<?php

class SMS extends Notifier
{
    public function validateTo(): bool
    {
        $pattern = '/^(\+44\s?7\d{3}|\(?07\d{3}\)?)\s?\d{3}\s?\d{3}$/';
        $isPhone = preg_match($pattern, $this->to);
  
        return $isPhone ? true : false;

    }
  
    public function sendNotification(): string
    {
        if ($this->validateTo() === false) {
            throw new Exception("Invalid phone number.");
        }
  
        $notificationType = get_class($this);
  
        return "This is a " . $notificationType . " to " . $this->to . ".";
    }
}
```

Similarly, let's put out `Email` notifier in the `Email.php` file:

```php
<?php

class Email extends Notifier
{
    private $from;
  
    public function __construct($to, $from)
    {
        parent::__construct($to);
  
        if (isset($from)) {
            $this->from = $from;
        } else {
            $this->from = "Anonymous";
        }
    }
  
    public function validateTo(): bool
    {
        $isEmail = filter_var($this->to, FILTER_VALIDATE_EMAIL);
  
        return $isEmail ? true : false;
    }
  
    public function sendNotification(): string
    {
        if ($this->validateTo() === false) {
            throw new Exception("Invalid email address.");
        }
  
        $notificationType = get_class($this);
  
        return "This is a " . $notificationType . " to " . $this->to . " from " . $this->from . ".";
    }
}
```

So now we can build our factory as follows:

```php
<?php

class NotifierFactory
{
    public static function getNotifier($notifier, $to)
    {

        if (empty($notifier)) {
            throw new Exception("No notifier passed.");
        }
  
        switch ($notifier) {
            case 'SMS':
                return new SMS($to);
                break;
            case 'Email':
                return new Email($to, 'Junade');
                break;
            default:
                throw new Exception("Notifier invalid.");
                break;
        }
    }
}
```

Lets put this all together in `index.php` file

```php
<?php

require_once('Notifier.php');
require_once('NotifierFactory.php');
  
require_once('SMS.php');
$mobile = NotifierFactory::getNotifier("SMS", "07111111111");
echo $mobile->sendNotification();   // Output: 07111111111
  
require_once('Email.php');
$email = NotifierFactory::getNotifier("Email", "test@example.com");
echo $email->sendNotification();    // Output: test@example.com
```

---
layout: post
title: Simple Factory Pattern
comments: true
---

What is a factory? Let's imagine you order a new car; the dealer sends your order off to the factory and the factory builds your car. Your car is sent to you in its assembled form and you don't need to care about how it was made.

Similarly, a software factory produces objects for you. The factory takes your request, assembles the object using the constructor and gives them back to you to use. One of these types of Factory pattern is known as the Simple Factory. Let me show you how it works.

Firstly, we define an abstract class, which we want to extend with other classes:

```php
<?php

abstract class Notifier
{
    protected $to;
  
    public function __construct(string $to)
    {
        $this->to = $to;
    }
  
    abstract public function validateTo(): bool;
  
    abstract public function sendNotification(): string;
}
```

This class serves to allow us to have common methods and define whatever common functionality we want all the classes we build in our factory to have in common. We could also use interfaces instead of abstract classes for the implementation without defining any functionality whatsoever.

Using this interface, we can build two notifiers, `SMS` and `Email`.

The `SMS` notifier is as follows in the `SMS.php` file:

```php
<?php

class SMS extends Notifier
{
    public function validateTo(): bool
    {
        $pattern = '/^(\+44\s?7\d{3}|\(?07\d{3}\)?)\s?\d{3}\s?\d{3}$/';
        $isPhone = preg_match($pattern, $this->to);
  
        return $isPhone ? true : false;

    }
  
    public function sendNotification(): string
    {
        if ($this->validateTo() === false) {
            throw new Exception("Invalid phone number.");
        }
  
        $notificationType = get_class($this);
  
        return "This is a " . $notificationType . " to " . $this->to . ".";
    }
}
```

Similarly, let's put out `Email` notifier in the `Email.php` file:

```php
<?php

class Email extends Notifier
{
    private $from;
  
    public function __construct($to, $from)
    {
        parent::__construct($to);
  
        if (isset($from)) {
            $this->from = $from;
        } else {
            $this->from = "Anonymous";
        }
    }
  
    public function validateTo(): bool
    {
        $isEmail = filter_var($this->to, FILTER_VALIDATE_EMAIL);
  
        return $isEmail ? true : false;
    }
  
    public function sendNotification(): string
    {
        if ($this->validateTo() === false) {
            throw new Exception("Invalid email address.");
        }
  
        $notificationType = get_class($this);
  
        return "This is a " . $notificationType . " to " . $this->to . " from " . $this->from . ".";
    }
}
```

So now we can build our factory as follows:

```php
<?php

class NotifierFactory
{
    public static function getNotifier($notifier, $to)
    {

        if (empty($notifier)) {
            throw new Exception("No notifier passed.");
        }
  
        switch ($notifier) {
            case 'SMS':
                return new SMS($to);
                break;
            case 'Email':
                return new Email($to, 'Junade');
                break;
            default:
                throw new Exception("Notifier invalid.");
                break;
        }
    }
}
```

Lets put this all together in `index.php` file

```php
<?php

require_once('Notifier.php');
require_once('NotifierFactory.php');
  
require_once('SMS.php');
$mobile = NotifierFactory::getNotifier("SMS", "07111111111");
echo $mobile->sendNotification();   // Output: 07111111111
  
require_once('Email.php');
$email = NotifierFactory::getNotifier("Email", "test@example.com");
echo $email->sendNotification();    // Output: test@example.com
```

That's how the factory pattern works. But there's one problem with this pattern. Lets say in future you implement another notifier `Voice Call`, then to adopt this new change we have to modify our `NotifierFactory` class. And it will be a violation of <a href="https://8thlight.com/blog/uncle-bob/2014/05/12/TheOpenClosedPrinciple.html" target="_blank">Open-Closed Principle</a>. To avoid that we can use `Factory Method Pattern` which will be discussed in a later post.
