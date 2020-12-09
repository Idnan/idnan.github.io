---
layout: post
title: Primitive vs. Reference Values
comments: true
---

In javascript, a variable may store two types of values: primitive and reference. So before we discuss them let's first discuss two important concepts stack and heap.

## Stack
In layman's terms, a stack is a pile of objects. In computing, architecture stacks are basically regions of memory where data is added or removed in a last in first out manner (LIFO).

<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20201209/stack.png" style="max-width: 490px; height: auto; margin-left: auto; margin-right: auto; display: block;" alt="stack"/>
</figure>

## Heap
In layman's terms, a heap is an untidy collection of things piled up haphazardly. In computing, architecture heap is an area of dynamically-allocated memory that is managed automatically by the operating system

<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20201209/heap.jpg" style="max-width: 490px; height: auto; margin-left: auto; margin-right: auto; display: block;" alt="heap"/>
</figure>

Now let's get back to our main topic.

## Primitive Types
Javascript has six primitive types null, undefined, boolean, string, number, symbol.
The size of the primitive values is fixed. That's why javascript stores the primitive values on the stack. When you assign a variable of this type to another variable, the new variable copies the value. Let's take an example.

```javascript
let x = 1;
let y = x;
console.log(x, y); // 1 1

x = 2;
console.log(x, y); // 2 1

y = 3;
console.log(x, y); // 2 3
``` 

At line 5 we modified the value of `x`. But it didn't modify the value of `y`. Because `y` copied the value of `x`.

## Reference Types
In javascript, there are two reference types objects and array. Array are also objects so technically one type. The size of reference types is dynamic so javascript stores the reference values on the heap. When we create an object and assign it some value. The value is not directly stored in the variable instead a reference to the value is stored in that variable. Let's take an example

```javascript
let student1 = {name: "john", age: 20};
let student2 = student1;

console.log(student1); // {name: "john", age: 20}
console.log(student2); // {name: "john", age: 20}

student2.name = "doe";
console.log(student1); // {name: "doe", age: 20}
console.log(student2); // {name: "doe", age: 20}
``` 

In the example, we modified the value of `student2` at line 7 and it also modified the value of `student1`. It's because both `student1` and `student2` reference to the same value. So modifying `student1` will affect `student2` and vice versa. 

Now the question is how to overcome this problem. The solution is to create a new reference for the new object. This way the new object will point to its own object instead of overlapping each other. We can create a new reference using the spread operator. Let's take an example.
```javascript
let student1 = {name: "john", age: 20};
let student2 = {...student1};

console.log(student1); // {name: "john", age: 20}
console.log(student2); // {name: "john", age: 20}

student2.name = "doe";
console.log(student1); // {name: "john", age: 20}
console.log(student2); // {name: "doe", age: 20}
```

This way we can solve the overlapping problem.

I hope that you liked the article. Feel free to leave your questions and comments below.
