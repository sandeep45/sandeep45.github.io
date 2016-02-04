---
layout: post
title: prototype-chain
categories: [javascript, es6]
tags: [prototype-chain, es6, javascript, classes, inheritence]
published: True
]
published: True

---

## Introduction

When you do

````
var a1 = new A();
````

A new object is created in memory.
`this` in A's constructor is defined to the new instance which will be returned,
`a1.[[Prototype]]` is set to `A.prototype`
When a property is accessed like `a1.name`, its first looked up on a1's owned properties, and if not found then its looked up in its [[Prototype]]. This means that all the stuff you define in prototype is effectively shared by all instances, and you can even later change parts of prototype and have the changes appear in all existing instances, if you wanted to

````
var A = function(){ }
A.prototype = {
  name: "yo yo honey singh"
}
var a1 = new A();
console.log(a1.name); // yo yo honey singh
var b1 = new A();
console.log(b1.name); // yo yo honey singh
var c1 = new A();
console.log(c1.name); // yo yo honey singh

A.prototype.name = "Pankaj Udas";
var a2 = new A();
console.log(a2.name); // Pankaj Udas
var b2 = new A();
console.log(b2.name); // Pankaj Udas
var c2 = new A();
console.log(c2.name); // Pankaj Udas
````

## __proto__ & prototype

An instance has a `__proto__` which points to its constructor's `prototype`, when there is a constuctor.

````
var A = function() { }
var a = new A();

console.log(a.__proto__ == A.prototype); // true
console.log(a.constructor == A); // true
console.log(a.constructor.prototype == A.prototype);
````

`__proto__` is now deprectated and as of ES6 can be accessed and set by `getProtoTypeOf()` & `setProtoTypeOf()`. Its stadard notation is `[[prototype]]`.

## getPrototypeOf

It returns the `prototype` of an object. This is the value its `__proto` points and its constructor holds.

````
var A = function() { }
var a = new A();

console.log(Object.getPrototypeOf(a) == A.prototype); // true
console.log(Object.getPrototypeOf(a) == a.__proto__); // true
````

I still like `__proto__`. I find it much easier to do `obj.__proto__.__proto__` over `Object.getPrototypeOf(Object.getPrototypeOf(obj))`. As you lookup higher and higer in the chain you can see why its easier to just use `__proto__`, but since the lookup on the prototype chain is autaomatically looked up recursively, you rarely not have to manually lookup on the higher chain.

## Access of properties

When we access a property on an object, first its lookuped on the object, if its not found, then its looked up on the objects prototype obbject, if not found their, it is then looked up on the prototype object of the prototpe object and so on and forth till it reaches null which has no prototpe object. Note this is possible because each object has a reference `__proto__` to a protype object.

````
var proto = {"name": "yo yo honey singh"}
var a = Object.create(proto);
a.age = 31;

console.log(a);
console.log(a.age) // 31
console.log(a.name); // yo yo honey singh
console.log(Object.getPrototypeOf(a) == proto); // true
````

## `own` property & `shadowed` property

When a object has the property on it, its called that the object `owns` the property. When the protype also has the same property by name, then the objects owned property is said to shadow the prototype objets property.

````
var proto = {"name": "yo yo honey singh"}
var a = Object.create(proto);
a.age = 31;

console.log(a.age); // 31
console.log(a.name); // yo yo honey singh

a.hasOwnProperty("age"); // true
a.hasOwnProperty("name"); // false

a.name = "Pankaj Udas";

console.log(a.name); // Pankaj Udas / Shadows
````

## `this` in inheritence via prototype chain

`this` in a function on the protype chain, once inherted and accessed from the inheriting object, now points to the inheriting object.

````
var proto = {
  age: 31,
  add: function(){
    this.age = this.age + 1;
  }
}
var a = Object.create(proto);
a.age = 5;
a.add();
console.log(a.age); // 6
````

In the example above `this` points to the inheriting object. This is because of `scope-by-flow` which i talked about in my [earlier post](http://sandeep45.github.io/javascript/this/es6/fat-arrow/2016/02/03/this-keyword-in-javascript-part2.html) on the `this` keyword. When we call the function lile this: `a` `.` `add()`, the object before the `.` (dot) becomes the value of `this` in the function under `add`. The exception to this is when the method is bounded to a particular object or we are using fat arrow syntax.


## Obect creation & its prototype

1\. Object Literal

````
Object.prototype.boss = "me"
var a = {}
console.log(Object.getPrototypeOf(a) == Object.prototype);
console.log(a.boss); // me
console.log(Object.getPrototypeOf(Object.prototype) == null); // true
````

The ineritence chain seen above is the instance -> Object -> null

2\. An Array

An array, is also an object. Its an instance of type `Array`. The instance gets methods like `indexOf`, `length` etc. from the protoype object on the Array.

````
var a  = [];
console.log(Object.getPrototypeOf(a) == Array.prototype); // true
console.log(a.indexOf == Array.prototype.indexOf); // true
console.log(a.hasOwnProperty("indexOf")); // false
console.log(Object.getPrototypeOf(Array.prototype) == Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype) == null); // true
````

The ineritence chain seen above is the instance -> Array -> Object -> null

3\. A String

A string, just like anything else is also an instance of type `String`.

````
var a = "yo yo honey singh";
console.log(Object.getPrototypeOf(a) == String.prototype);
console.log(Object.getPrototypeOf(a).length == String.prototype.length);
console.log(a.hasOwnProperty("indexOf")); // false
console.log(Object.getPrototypeOf(String.prototype) == Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype) == null); // true
````

The ineritence chain seen above is the instance -> String -> Object -> null

4\. Via Function Constructor

A constructor is a function which is called via the `new` keyworrd. Its a practice to name these functions with a capital letter.

````
var Car = function(n, y){
  this.name = n;
  this.year = y;
}

Car.prototype.toString = function() { return "I am a CAR"; }

var ford = new Car("ford", "2002");

console.log(ford); // I am a CAR
console.log(ford.__proto__ == Car.prototype); // true
console.log(Car.prototype.__proto__ == Object.prototype); //true
console.log(Object.prototype.__proto__ == null); // true
````

Here the instance of `Car` called `ford` has access to the `Car.prototype`, and Car's prototype object has access to `Object.prototype` and Object's prototype has access to null.

Also, in the example above i reverted to use `__proto__` to access the prototype object rather than using `Object.prototype`.

5\. `Object.create`

This method creates an object just like the object literal would do. The difference here is that the passed in argument becomes the prototype object which the new created object will have pointer to.

````
var proto = {"name": "yo yo honey singh"}
var a = Object.create(proto);
console.log(a.__proto__ == proto); // true
console.log(proto.__proto__ == Object.prototype); // true
````

6\. ES6 Keyword: `class`

Javascript remains prototype based inheritence language and also has new keywords like `class`, `static`, `constructor`, `extends` & `super`. These make it look like other languages which have class based inheritence.

````
class Person {

  constructor(initialName){
    this.name = initialName;
  };

  static yell(){
    console.log("bla bla bla BLA BLA BLA!!!!!");
  };

  get name(){
    return this._n;
  };

  set name(newName){
    this._n = newName;
  };

  introduce(){
    console.log(`Hi i am ${this.name}`);
  };

}

var p = new Person("sandeep");
console.log(p.__proto__ == Person.prototype); // true
Person.yell(); // bla bla bla BLA BLA BLA!!!!!
console.log(Person.hasOwnProperty("yell")); // true

p.introduce();
p.hasOwnProperty("introduce") // false
Person.prototype.hasOwnProperty("introduce") // true

Object.getOwnPropertyNames(p); // [_n]
Object.getOwnPropertyNames(p.__proto__); // [name, introduce, constructor]
````

This creates a Person function. `_n` is put on the instance which is returned from the constructor. The constructor still returns a new object which is `this`. The `__proto__` of `p` is the Person's prototype.

`yell` function which is made `static` is put on the Person Object.

`introduce` is a method added on `Person.prototype` which `p` and all other instances of `Person` have access to. The same happens with `name` and its getters and setters. They are also on the Persons's prototype.

````
class Male extends Person{
  constructor(initialName){
    super(initialName)
  }

  get sexType(){
    return "MALE"
  }

  introduce(){
    console.log(`I am a ${this.sexType} named: ${this.name}`)
  }
}

var s = new Male("SA");
s.introduce()
console.log(s.__proto__ == Male.prototype); // true
console.log(Male.prototype.__proto__ == Person.prototype); // true

console.log(Object.getOwnPropertyNames(Male.prototype)); // [sexType, constructor]
console.log(Object.getPrototypeOf(Male.prototype)); // Perseon.prototype
console.log(Object.getOwnPropertyNames(Male.prototype.__proto__)); // [name, introduce, constructor]

````

So this is how the inhertience is setup here:

At the bottom we have: `s`. It is an instance of Male. Therefore `s.__proto__ == Male.prototype`.

Then above it `Male.prototype.__proto__ == Person.prototype`.

Then above that we have `Person.prototype.__proto__ == Object.prototype`

Then above that we have `Object.prototype.__proto__ == null`

Read the sentence below by replacing `->` with `[which] has access to`:

s -> Male.prototype -> Person.protype -> Object.prototype -> null

The `extends` keyword was a nice way of setting the prototype of Male to of Person's. This could have also be done by saying: `Object.setPrototypeOf(Male.prototype, Person.prototype);` One caveat would be that when we take away the `extends` keyword, then `super` in the constructor wont work, so will have to tweak it and use the `super.prop` style to call the parents constructor. Here is an example:

````
class Person {

  constructor(initialName){
    this.name = initialName;
  };

  get name(){
    return this._n;
  };

  set name(newName){
    this._n = newName;
  };

  introduce(){
    console.log(`Hi i am ${this.name}`);
  };

}

class Male{
  constructor(initialName){
    super.constructor(initialName)
  }

  get sexType(){
    return "MALE"
  }

}

Object.setPrototypeOf(Male.prototype, Person.prototype);

var s = new Male("Yo Yo Honey Singh!")
s.introduce(); // Hi i am Yo Yo Honey Singh!
console.log(s.sexType); // MALE
````

## The `new` constructor

When we call a function with thew `new` constructor:
1. the function runs, with `this` set to a new object being returned
2. the new objects `__proto__` holds a reference to the `prototype` of the function.
3. the new object created is returned.

````
var A = function(){
  this.name = "hehe"
}
var o = new A();

console.log("we have it: ", o); // {name: hehe}
console.log(o.name); // hehe
console.log(o.__proto__ == A.prototype); // true
````

Well the above can be done manually as well, without the `new` keyword.

````
var A = function(){
  this.name = "hehe"
}

var o = Object.create({});
Object.setPrototypeOf(o, A.prototype)
A.call(o);

console.log("we have it: ", o); // {name: hehe}
console.log(o.name); // hehe
console.log(o.__proto__ == A.prototype); // true
````

## Lookup

1\. Whenever a property is looked up, the entire prototype chain could be checked looking for the property. This could be expensive.
2\. When iterating over the enumreable properties, all properties in the prototype chain are iterated over.
3\. Use `hasOwnProperty` to figure if the property is actually `owned` and not off the `prototype` chain.
4\. Use `getOwnPropertyNames` to fetch names of only `owned` properties

## Word of Caution

Don't monkey patch existing Object Type's Prototype chain as it breaks encapsulation unless its for backwards compatability. Note: Encapsulation is an Object Oriented Programming concept that binds together the data and functions that manipulate the data, and that keeps both safe from outside interference and misuse. Data encapsulation led to the important OOP concept of data hiding.

## Conclusion

Each object has a reference by `__proto__`. Properties are looked up on the object and when not found automatically and recursively looked up on the object to which `__proto__` is pointing. It should be noted that the object to which `__proto__` points, itself has a `__proto__` reference to another object. This goes all they way to the top to `Object.prototype`, whose `__proto__` points to null.

Reference:

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain
