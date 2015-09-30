# Typeof-in ( + instanceof )
allow you to compare the type (or instance) of your value with several types (or constructor), and finally return a **Boolean**.

## Why use typeof-in ? 
#### JS is, somehow, broken

> **typeof** null **===** 'object'
>
> null **instanceof** Object === **false**

null shouldn't be an object, and even if it is in JS, it is not an instance of Object.

> **typeof** /regularexpression/ **===** 'object'

every type I see this line, I would expect *'regexp'*

**new** Number(42) and 42 have the same constructor name, but are completely different. (String and Boolean have the same "problem")

> **typeof** NaN **===** 'number'

Quite famous, but still incoherent.

> **typeof** new Number(NaN) **===** 'object'

and this one doesn't help more the case of NaN...

and some more...

#### typeof-in supports:
- Regex
- Multi choices
- both: new String('test') and 'test' return the same type
- same thing with *Numbers* and *Booleans*
- NaN, Undefined, Null values have their own types 
- use instanceof when necessary: typeOf(instance).in(constructor)
- and more!

In some ways, it is the fusion of **typeof** and **instanceof**

#### You can check the following types:
> - Boolean / String / Number / Symbol 
>
> - NaN / Undefined / Null
>
> - Function / GeneratorFunction / Iterator / Promise
>
> - Error / TypeError / ...
>
> - HTMLDocument / ...
>
> - RegExp
>
> - Array / ArrayBuffer / UInt32Array / (Weak)Map / (Weak)Set / ...
>
> - built-in Objects like JSON and Math
>
>   and many more... (in fact, you can check almost everything)

[JavaScript reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects)

## Use cases:
```js
    const typeOf = require('typeof-in'); 
```
#### Through an object
###### basic:
You can retrieve the type or compare it with a string representing an expecting type.
```js
typeOf('test').getType(); //return 'String' (see 'typeOf only' below);

//return a Boolean, in these cases: true.
typeOf('lolipop').in('String');
typeOf(null).in('Null');
typeOf(undefined).in('Undefined');
typeOf(NaN).in('NaN');
typeOf(new Number(NaN)).in('NaN');
```

###### multi:

You might also want to compare your value with a set of different types.

```js
//using an Array (better performance)
typeOf('lolipop').in(['Number', 'String', 'Object','Array']);
//or multiple arguments
typeOf('lolipop').in('Number', 'String', 'Object', 'Array');
```

###### regex:
Furthermore, typeof-in also supports Regex against your value, 

which is quite useful with < ****Error> types for example. [about Error types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error)
```js
typeOf(new TypeError()).in(/.+Error$/); //is an error other than 'Error' type
```

###### ES6 and others objects:
This library can check all kind of objects. Allowing you to compare *ES6* features like Promises, Generators, fat arrows... which is pretty neat.
```js
typeOf(new Promise(function(){}).in('Promise')
typeOf(function*(){}).in('GeneratorFunction')
typeOf(()=>{}).in('Function')
```

###### calling several times:
This is the main advantage of using typeof-in through an object, it allows you to deal with the same value in different ways
```js
var myType = typeOf('test');
//call the "in" method several times
if(myType.in('String')){
    //do something with the value as a string
}else if(myType.in(['Null','Undefined'])){
    //you need to init your value!
}else if(myType.in(/.+Error$/)){
    //something bad happened! but not a global error
}else if(myType.in(Error)){
    //global error handler... :(
}
```

###### some other tricks:
*it might impact performance*

If the value passed inside typeOf is not an Object, its **in()** method will never call instanceof when a constructor is passed as parameter, however, it will retrieve its constructor name to check if it match the type of your value. (**important:** some constructors are not supported by all browsers)

Therefore, you can use typeOf-in like this:
```js
//with contructors
typeOf(1).in([null, undefined,NaN,Array,Object,Number,String,GeneratorFunction,Function])

//with random values, however: strings and arrays must absolutely be empty! ('' & [])
typeOf(1).in([null,undefined,NaN,[], {}, 42,'',function*(){}, function hi(){console.log('hellow world')}]); 
//is equal to
typeOf(1).in(['Null','Undefined','NaN','Array','Object','Number','String','GeneratorFunction','Function'])
```

###### dealing with instanceof:
The following examples show different cases when typeof-in will use instanceof to compare the value with the constructor of a prototype.

However, the library will not return an empty string('') but a "#Anonymous" value in the case of an instance of an anonymous prototype. 
```js
    //with primitive value, if the type passed in "in()" is a constructor, then typeof-in will retrieve its constructor name
    typeOf(42).in(Number)  // is equal to typeOf(42).in('Number')
    typeOf(new Number(42)).in(Number) //will use instanceof

    typeOf(new String('test')).in(String)
    typeOf({}).in(Object)
    typeOf([]).in(Object)   //return true! an Array is an instance of Object. however
    typeOf([]).in('Object') //return false
    
    //OR

    function Human(){}; // ES6: class Human {}
    var Person = Human;
    var Person2 = function Human(){};

    var person = new Person();
    var _person = new Person();
    var __person = new Human();
    var person2 = new Person2();  
    
    //#instance against instance
    typeOf(person).in(_person); //true
    typeOf(person).in(__person); //true    
    typeOf(person).in(person2); //false
    
    //#instance against prototype
    typeOf(person).in(Person); //true   
    typeOf(person).in(Human); //true
    typeOf(person2).in(Human); //false
    typeOf(person2).in('Human'); //true
    typeOf(person2).in(Object); //true
    
    typeOf(person).getType(); // return 'Human'
    typeOf(person2).getType(); //return 'Human'
    typeOf(new(function $(){})).getType(); //return '$'
    typeOf(new(function _(){})).getType(); //return '_'
    
    //#special cases: instance of Anonymous (it behaves as the above examples)
    var myAnonymous = function(){};
    typeOf(new(function (){})).getType(); //return '#Anonymous'
    typeOf(new (myAnonymous)).in('#Anonymous') //true
    typeOf(new (myAnonymous)).in(myAnonymous) //true
    typeOf(new (myAnonymous)).in(Object) //true
    typeOf(new (myAnonymous)).in(new(function(){})) //false
    
```


#### Through a function
*improve performance (unique call), decrease readability*

The recent version of typeof-in (>= 3.0.0) allows you to directly call the function in charge of the comparison, and by extension, not create an object every time you use typeOf() in your code. 

This feature works exactly like the previous examples. however, you cannot retrieve the type with **getType()**.

```js
typeOf('lolipop','String'); 
typeOf('lolipop',Number, [], 'String'); 
typeOf('lolipop',[Number, [], 'String']);
```

#### TypeOf only
In the case you only need the function used to retrieve the type (as a String) of a specific value, you might be interested in the [typeof-- library](https://www.npmjs.com/package/typeof--)

## NPM commands
> npm install typeof-in -S
> npm test

#### words of advice
this library use some *ES6* features => use babel or --harmony (with node.js < v4.0.0) if necessary

you might need to polyfill Object.getfPrototypeOf() for cross-browser compatibility too (cf: IE < 9).
    
Finally, I'm open to any suggestions
