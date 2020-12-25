---
layout: post
title: Freeze and Seal Objects in JavaScript
comments: true
---

Variables we declare in javascript with the help of `const`, are not purely constant. Let's say that if we have a variable called config with a bunch of properties and if we print it to the console you will see it has a name and a database object.

```javascript
const config = {
    name: "module-account",
    database: {
        host: "127.0.0.1",
        port: "2020",
        username: "admin",
        password: "r@@t",
    },
};

console.log(config);    // {"name":"module-account","database":{"host":"127.0.0.1","port":"2020","username":"admin","password":"r@@t"}}
```

But if we update the value of let's say `name` to be `xyz`, you will see you can do that. Although it's a constant. 

```javascript
config.name = "xyz";

console.log(config.name);   // xyz 
```

To prevent this javascript comes with a bunch of methods, such as `Object.freeze`, `Object.seal` and `Object.preventExtensions`. Which we can use to make them immutable. Let's look at the examples to understand how they work and how we can use them in our codebase.

## Object.freeze

If we freeze an object, let's say `Object.freeze(config)` and print the `name` you will see that we are still able to read the value from the config.

```javascript
Object.freeze(config);

console.log(config.name);       // xyz
```

But if we try to update any of the existing values, let's say `config.name` is `abc`, we will get the error that we cannot assign the value to a read-only property.

```javascript
config.name = "abc";    // error 
```

Similarly, if we try to delete a property, let's say delete `config.name`, we will not be able to do that, and also if we try to add a new property, let's say `config.timeout` is `3`, we will still get the error because the object is not extensible.

```javascript
delete config.name;     // error
config.timeout = 3;     // error
```

The only thing we can do is reading the properties from the existing object. One important thing to remember about the freeze is that it works only at the top level. So now, in this case, we have a database object, which is nested inside the config object.

If we try to update the value for, let's say `config.database.host` is `10.10.10.20` and if we print the config, you will see that the database host has been updated.

```javascript
config.database.host = "10.10.10.20";

console.log(config.database.host);      // 10.10.10.20
```

If we want the object to be fully frozen, with all the objects inside, we have to recursively freeze all the objects. So in this case now if we want the database to be frozen also, we will have to do 

```javascript
Object.freeze(config.database);
```

And now we'll get the error while updating the `host` that the database host cannot be updated because `config.database` is frozen

```javascript
config.database.host = "10.10.10.20";     // error
```

## Object.seal

Next, we have `Object.seal` which is similar to `Object.freeze` in a way that you cannot add or remove properties from an object but you can update the values of the existing properties. Let's say that we seal our config object so `Object.seal(config)`.

And now, if we do `config.name` to be `XYZ`, you will see that the `name` has been updated.

```javascript
config.name = "XYZ";

console.log(config.name);
```

But if we try to delete the property from the config object. Let's say delete `config.database`, we will not be able to do that because, with seal, you cannot delete the properties from the object. And also, if we try to add a new property, let's say `config.timeout` is `3` we will get the error, that you cannot add a new property to the object.

```javascript
delete config.database;     // error
config.timeout = 3;     // error
```

And similar to `object.freeze`, `object.seal` also works on the top-level only. So, the seal will not be applied to the database object here and if we try to delete a property from the database object, let's say delete `config.database.host`, we will see that the database host has been deleted from here.

```javascript
delete config.database.host;        // success
```

So, if we want to prevent this also, we will have to seal the nested objects.

```javascript
Object.seal(config.database);
```

And now we will get the error that we cannot delete a property from a sealed object.

## Object.preventExtensions

The last one we have is the `Object.preventExtensions`, which allows us to update the values and delete the properties from an existing object but it does not allow us to add new properties to the object.

So now, if we call `Object.preventExtensions` on our `config` object, and try to update the value for one of the properties, let's say `name`, you will see that the name has been updated, and also if we try to delete one of the properties, let's say delete `config.database`, we can also delete the properties but if we try to extend our object or let's say add new properties, for example, `config.timeout` is `3` we can't do that because our object is not extensible.

```javascript
config.name = "XYZ";        // success
delete config.database;     // success

config.timeout = 3;         // error
```

One more thing to know about the `preventExtensions` is that if you delete a property from an object, you cannot add the same property back again and the reason for that is because adding a new property is considered extension. So if I do `config.database` again with something, it will give me the error that you cannot add a new property to the object.

```javascript
config.database = {host: "10.20.20.10"};        // error
```

Similar to `freeze` and `seal`, `preventExtensions` also applies only to the top-level properties.

There are three more methods that can be used to check if the objects are `frozen`, `sealed`, or `extensible`.

## Helper Methods
So `Object.freeze` is to freeze the objects and `Object.isFrozen` can be used to check if the object is frozen or not.

```javascript
const user1 = {firstName: "John"};
const user2 = {firstName: "Doe"};

Object.freeze(user1);

console.log(Object.isFrozen(user1));        // true
console.log(Object.isFrozen(user2));        // false
```

`Object.seal` is to seal and `Object.isSealed` is to check if the object is sealed or not. And for the `Object.preventExtensions`, we have `Object.isExtensible` which can be used to check if the new properties can be added to the object or not.

## Conclusion
We can conclude this topic using a CRUD table.

|                            | CREATE  | READ    | UPDATE  | DELETE  |
|----------------------------|---------|---------|---------|---------|
| `Object.freeze`            | ✗ | ✓ | ✗ | ✗ |
| `Object.seal`              | ✗ | ✓ | ✓ | ✗ |
| `Object.preventExtensions` | ✗ | ✓ | ✓ | ✓ |

And that wraps it up for this article. Feel free to leave your feedback or questions in the comments section.
