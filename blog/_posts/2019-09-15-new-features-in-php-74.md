---
layout: post
title: New Features in PHP 7.4
comments: true
---

Since the last year php community is working alot on bringing new features in PHP. So this year in PHP 7.4 is coming with alot of interesting features. Here's a list of some features coming this year.

## Arrow Functions
Arrow functions were first introduced in [2016](https://wiki.php.net/rfc/arrow_functions). But due to some reasons the rfc was withdrawn. And now after two years it's finally coming in [PHP 7.4](https://wiki.php.net/rfc/arrow_functions_v2). They are also called as "short clousers" or one-liner functions.

Here's the syntax
```php
$numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
array_filter($numbers, fn(int $number): int => $number % 2 == 0);
```

Somethings to note about the syntax
* They start with `fn`
* There's no `return` statement
* There's only one expression that is the default return
* `$this` is available like a normal clousre
* They can always access the parent scope, no need for the `use` keyword
* Arguments and return types can be type hinted

Check the [rfc](https://wiki.php.net/rfc/arrow_functions_v2) for more detail.

## Typed Properties
Finally after a long time this feature is accepted. Now we don't have to use annotations anymore.

```php
class Test { 
	public  string $x; 
	public  int	   $y;
	public  ?Foo   $foo; 
	protected  static  string $static = 'default';
}
```

For more detail check the [rfc](https://wiki.php.net/rfc/typed_properties_v2)

## Array spread operator
Array prefixed with `...` will expand it's values. Here's an example

```
$parts = ['apple', 'pear'];
$fruits = ['banana', 'orange', ...$parts, 'watermelon'];
// ['banana', 'orange', 'apple', 'pear', 'watermelon'];
```

Here's the link to the [rfc](https://wiki.php.net/rfc/spread_operator_for_array)

## Preloading
A new feature that can improve the performance of your code. Preload is implemented as a part of the opcache. Which will generate opcode cache. So all the scripts will be executed once than will be loaded into the memory and later on all the requests will be served from memory. So furture changes that you will do the scripts will not take effect until the server is restarted. 

Here's the link to the [rfc](https://wiki.php.net/rfc/preload)
