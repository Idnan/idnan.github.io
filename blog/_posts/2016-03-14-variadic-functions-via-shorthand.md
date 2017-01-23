---
layout: post
title: PHP 5.6 Variadic Functions via "..." & ** Shorthand
comments: true
---

Earlier we were using `func_get_args()` to get all the arguments available in a function call, but with PHP 5.6, this can be removed as we can easily get that facility with the `...` operator.

```php
function mySkills($name, ...$skills) {
    echo "Name: " . $name;
    echo "My Skills Count: " . count(skills);
}
 
mySkills("Adnan", "PHP");
// Output:
// Name: Adnan
// My Skills Count: 1
 
mySkills("Adnan", "Codeigniter", "Laravel");
// Output:
// Name: Adnan
// My Skills Count: 2
```

## ** Shorthand

The `**` operator has been added for exponentiation. We have support for the shorthand operator as easily.

```php
echo 2 ** 3;    // Output: 8
$a = 2;
$a **= 3;
echo $a;        // Output: 8
```

Note that the order of operations comes into play using this operator. Please take a look at the following example for a clear understanding:

```php
echo 2 ** 2 ** 4;   // Output: 65536
```

You might expect it to return 256 as the grouping would be like `(2 ** 2) ** 4`, but that is not the case here. The real result would be `65536` as the grouping would be from right to left and it will parse as `2 ** (2 ** 4)`.