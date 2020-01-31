<img src="https://github.com/damcipolat-recargapay/js_developer_guide/blob/master/img/js_logo.png?raw=true" width="125px" align="right" />

# Javascript coding best practices.
This document is a summary of good programming practices in js in general.

Part of the document is based in Airbnb guideline, and other in profesional experience.

https://github.com/airbnb/javascript

<a name="table-of-contents"></a>
## Content list

  - [Paradigm](#fp)
  - [Naming conventions](#naming)
  - [Semicolons](#semicolons)
  - [Comments](#comments)
  - [Error handling](#error)
  - [Promise](#promise)
  - [Comparision](#comparision)
  - [Iterations](#iterations)
  - [Functions](#functions)
  - [String](#string)
  - [Destructuring](#destructuring)
  - [Arrays](#arrays)
  - [Objects](#objects)
  - [Properties](#properties)
  - [Modules](#modules)
  - [Primitives](#primitives)
  - [Variables](#variables)
  - [TL;DR](#tldr)

<a name="fp"></a>
## Paradigm - FP
> Javascript is a multiparadigm programming language. While we could make a mix of paradigms in our code is better, focus on just one. The trend today is to focus on functional programming, both to develop a frontend or a backend.

These are some functional programming principles that are useful to know.

1. Thinks in functions
2. Lambda
3. Curry
4. Stateless
5. Composing functions
6. pure functions: 
7. Side effects
8. Functor
9. High order functions
10. First class
11. Mutations

👉 To continue reading about FP, go to this [document](FP.md).


**[⮬ back to top](#table-of-contents)**

<a name="naming"></a>
## Naming conventions
How to name objects in js.
> Based in the Airbnb guide points: `23.1`, `23.2`, `23.3`, `23.6`

-   Avoid single letter names. Be descriptive with your naming.
    ```javascript
    // bad
    function q() {
    }

    // good
    function query() {
    }
    ```
    
-  Use **camelCase** when naming objects, functions, and instances.

    ```javascript
    // bad
    const OBJEcttsssss = {};
    const this_is_my_object = {};
    function c() {}

    // good
    const thisIsMyObject = {};
    function thisIsMyFunction() {}
    ```
   
- Use **PascalCase** only when naming constructors or classes.
    ```javascript
    // bad
    function user(options) {
      this.name = options.name;
    }

    const bad = new user({
      name: 'nope',
    });

    // good
    class User {
      constructor(options) {
        this.name = options.name;
      }
    }

    const good = new User({
      name: 'yup',
    });
    ```
    
- Use **uppercase** only in constants.

    ```javascript
    // allowed but does not supply semantic value
    export const apiKey = 'SOMEKEY';

    // better in most cases
    export const API_KEY = 'SOMEKEY';
    ``` 
    
**[⮬ back to top](#table-of-contents)**

<a name="semicolons"></a>
## Semicolons
> Based in the Airbnb guide points: `21.1`. 

Why? When JavaScript encounters a line break without a semicolon, it uses a set of rules called Automatic Semicolon Insertion to determine whether or not it should regard that line break as the end of a statement, and (as the name implies) place a semicolon into your code before the line break if it thinks so. ASI contains a few eccentric behaviors, though, and your code will break if JavaScript misinterprets your line break. These rules will become more complicated as new features become a part of JavaScript. Explicitly terminating your statements and configuring your linter to catch missing semicolons will help prevent you from encountering issues

    ```javascript
    // bad
    function foo() {
      return
        'search your feelings, you know it to be foo'
    }

    // good
    const luke = {};
    const leia = {};
    [luke, leia].forEach((jedi) => {
      jedi.father = 'vader';
    });
    ```

**[⮬ back to top](#table-of-contents)**

<a name="comments"></a>
## Comments
Standarize js comments in your projects. Visualstudio code recognize this format.
> Use JSDOC https://jsdoc.app/about-getting-started.html format.

-  Use block comment.
    ```javascript
   /** This is a description of the foo function. */
   function foo() {
   }
    ```
    
-  Use JSDOC tag to describe a function.
    ```javascript
   /**
    * Represents a book.
    * @constructor
    * @param {string} title - The title of the book.
    * @param {string} author - The author of the book.
    */
   function Book(title, author) {
   }
    ```    
    
**[⮬ back to top](#table-of-contents)**

<a name="promise"></a>
## Promises.
Change the way to handle callbacks.

- If you work with callback styled function, wrap it in a promise:

  ```javascript
  function doAsync(function(err, data) { 
    if (err) {
      // error
    } else {
      // success
    }
  });
  ```

- With Promises:

```javascript
const doAsyncPomise= () =>{

    return new Promise((resolve,reject)=>{

        if (err)
            reject(err);
        else
            resolve(..);

    });

}
```

**[⮬ back to top](#table-of-contents)**

<a name="error"></a>
## Error handling
Different ways of handle errors.

- Using sync functions:

  ```javascript
  try{
      makeSomething();
  } catch(err){
      rollBack();
  }
  ```

- Using a function that return promise:

  ```javascript
  makeSomething()
      .then(data=>{
          //....
      })
      .catch(err=>{
          rollback(...)
      });
  ```
  
- Using into a async/await function:

  ```javascript
  const run = async ()=>{

      try{
          const result = await makeSomething();
      } catch(err){
          rollBack(..)
      }

  };
  ```
  
- Avoid to return "error structures" to comunicate an error, is better to launch an exception.

  ```javascript
    //bad
    const run = (param)=>{

      const result = makeSomething(param);

      if (result){
          return result;
      } else {
          return {
              error:'processing error'
          };
      }
  }
  
  //good
  const run = (param)=>{
  
    if (!param)
      throw new Error('Bad parameters');
  
    const result = makeSomething(param);
    
    if (!result)
      throw new Error('Processing error');
      
    return result;
    
  }
  ```
  
**[⮬ back to top](#table-of-contents)**

<a name="comparision"></a>
## Comparision
Improve your comparision methods.
> Based in the Airbnb guide points: `15.1`, `15.2`, `15.3`, `15.5`, `15.6`, `15.7`

-  Use === and !== over == and !=.
-  Conditional statements such as the if statement evaluate their expression using coercion with the ToBoolean.
   https://github.com/airbnb/javascript/blob/master/README.md#comparison--if
    ```javascript
    if ([0] && []) {
      // true
      // an array (even an empty one) is an object, objects will evaluate to true
    }
    ```
-  User shorcuts for booleans.
    ```javascript
    // bad
    if (isValid === true) {
      // ...
    }

    // good
    if (isValid) {
      // ...
    }
    ```
-  Ternaries should not be nested and generally be single line expressions.
    ```javascript
    // bad
    const foo = maybe1 > maybe2
      ? "bar"
      : value1 > value2 ? "baz" : null;

    // split into 2 separated ternary expressions
    const maybeNull = value1 > value2 ? 'baz' : null;

    // better
    const foo = maybe1 > maybe2
      ? 'bar'
      : maybeNull;

    // best
    const foo = maybe1 > maybe2 ? 'bar' : maybeNull;
    ```
-  Ternaries should not be nested and generally be single line expressions.
    ```javascript
    // bad
    const foo = a ? a : b;
    const bar = c ? true : false;
    const baz = c ? false : true;

    // good
    const foo = a || b;
    const bar = !!c;
    const baz = !c;
    ```
**[⮬ back to top](#table-of-contents)**

<a name="iterations"></a>
## Iterations
Handle the lopps in a functional style.
> Based in the Airbnb guide points: `11.1`

-  Don’t use iterators, prefer js higher-order functions instead of for / for..in
    ```javascript
    // bad
    const increasedByOne = [];
    for (let i = 0; i < numbers.length; i++) {
      increasedByOne.push(numbers[i] + 1);
    }

    // good
    const increasedByOne = [];
    numbers.forEach((num) => {
      increasedByOne.push(num + 1);
    });
    ```
**[⮬ back to top](#table-of-contents)**

<a name="functions"></a>
## Functions
How to handle functions in a modern way.
> Based in the Airbnb guide points: `7.1`, `7.5`, `7.6`, `7.7`, `7.10`, `7.12`, `7.13`

- Use named arrow function expressions instead of function declarations.
    ```javascript
    // bad
    function foo() {
      // ...
    }

    // bad
    const foo = () => {
      // ...
    };
    ```
- Never name a parameter arguments.. 
    ```javascript
        // bad
        function foo(name, options, arguments) {
          // ...
        }

        // good
        function foo(name, options, args) {
          // ...
        }
    ``
- Never use arguments, opt to use rest syntax ... instead. 
    ```javascript
    // bad
    function concatenateAll() {
      const args = Array.prototype.slice.call(arguments);
      return args.join('');
    }

    // good
    function concatenateAll(...args) {
      return args.join('');
    }
    ``
    
- Avoid side effects with default parameters.. 
    ```javascript
    const b = 1;
    // bad
    function count(a = b++) {
      console.log(a);
    }
    count();  // 1
    count();  // 2
    count(3); // 3
    count();  // 3
    ``    
    
 - Never mutate parameters. 
    ```javascript
    // bad
    function f1(obj) {
      obj.key = 1;
    }

    // good
    function f2(obj) {
      const key = Object.prototype.hasOwnProperty.call(obj, 'key') ? obj.key : 1;
    ``    
    
**[⮬ back to top](#table-of-contents)**

<a name="string"></a>
## String
Best way to handle strings.
> Based in the Airbnb guide points: `6.1`, `6.2`, `6.3`, `6.4`

- Use single quotes '' for strings, don't mix with "".
    ```javascript
    // bad
    const name = "Bart";

    // bad - template literals should contain interpolation or newlines
    const name = `Marge`;

    // good
    const name = 'Homer';
    ```
- Use template string instead of concatenate string with values. 
    ```javascript
    const name    = 'Bart';
    const surname = 'Simpson';
    
     // bad
    const txt = 'hello mr. '+name+',  '+surname';
    
    // good
    const txt = `hello mr. ${name},  ${surname}`;
    ``
**[⮬ back to top](#table-of-contents)**

<a name="destructuring"></a>
## Destructuring
Destructuring simply implies breaking down a complex structure into simpler parts.
> Based in the Airbnb guide points: `5.1`, `5.2`, `5.3`

- Use destructuring when accessing and using multiple properties of an object.
    ```javascript
    // bad
    function getFullName(user) {
      const firstName = user.firstName;
      const lastName = user.lastName;

      return `${firstName} ${lastName}`;
    }

    // good
    function getFullName(user) {
      const { firstName, lastName } = user;
      return `${firstName} ${lastName}`;
    }
    ```
- Use array destructuring. 
    ```javascript
    // bad
    const first = arr[0];
    const second = arr[1];

    // good
    const [first, second] = arr;
    ``
**[⮬ back to top](#table-of-contents)**

<a name="arrays"></a>
## Arrays
Array manipulation practices.
> Based in the Airbnb guide points: `4.1`, `4.3`, `4.4`, `4.7`

- Is important to know this array prototypes: `map`, `reduce`, `forEach`, `filter`, `find`,  `push`, `pop`, `slice`. https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/prototype
- Use the literal syntax for array creation.

    ```javascript
    // bad
    const items = new Array();

    // good
    const items = [];
    ```
    
 - Use array spreads `...` to copy arrays:
    ```javascript
    // bad
    const len = items.length;
    const itemsCopy = [];
    let i;

    for (i = 0; i < len; i += 1) {
      itemsCopy[i] = items[i];
    }

    // good
    const itemsCopy = [...items];
    ```   
    
 - To convert an iterable object to an array, use spreads `...` instead of `Array.from`. 
    ```javascript
    // bad
    const items = new Array();

    // good
    const items = [];
    ```
    
 - Use array spreads `...` to copy arrays:
    ```javascript
    // bad
    const len = items.length;
    const itemsCopy = [];
    let i;

    for (i = 0; i < len; i += 1) {
      itemsCopy[i] = items[i];
    }

    // good
    const itemsCopy = [...items];
    ```
    
 - Use return statements in array method callbacks:
    ```javascript
    // bad
    inbox.filter((msg) => {
      const { subject, author } = msg;
      if (subject === 'Mockingbird') {
        return author === 'Harper Lee';
      } else {
        return false;
      }
    });

    // good
    inbox.filter((msg) => {
      const { subject, author } = msg;
      if (subject === 'Mockingbird') {
        return author === 'Harper Lee';
      }

      return false;
    });
    ```  
    
**[⮬ back to top](#table-of-contents)**

<a name="objects"></a>
## Objects
Some tips of how to improve the object manipulation.
> Based in the Airbnb guide points: `3.1`,`3.3`,`3.4`,`3.6`,`3.8`

- Use the literal syntax for object creation.
    ```javascript
    // bad
    const item = new Object();

    // good
    const item = {};
    ```
- Use object method shorthand.
    ```javascript
    // bad
    const atom = {
      value: 1,

      addValue: function (value) {
        return atom.value + value;
      },
    };

    // good
    const atom = {
      value: 1,

      addValue(value) {
        return atom.value + value;
      },
    };
    ```
- Use property value shorthand.
    ```javascript
    const bart = 'Bart Simpson';

    // bad
    const obj = {
      bart: bart,
    };

    // good
    const obj = {
      bart,
    };
    ```
- Only quote properties that are invalid **identifiers** in the example is 'bla-bla'.
    ```javascript
    // bad
    const bad = {
      'foo': 3,
      'bar': 4,
      'data-blah': 5,
    };

    // good
    const good = {
      foo: 3,
      bar: 4,
      'bla-bla': 5,
    };
    ```
- If you need to access dinamycali to one object atributte:
    ```javascript
    const person = {
        name:'Damian',
        age:32
    };
    
    const key = 'age';
    console.log(person[key]);
    ```
- Prefer the object spread operator over Object.assign to **shallow-copy** objects:
    ```javascript
    // bad
    const original = { a: 1, b: 2 };
    const copy = Object.assign({}, original, { c: 3 }); // copy => { a: 1, b: 2, c: 3 }

    // good
    const original = { a: 1, b: 2 };
    const copy = { ...original, c: 3 }; // copy => { a: 1, b: 2, c: 3 }
    ```
    
**[⮬ back to top](#table-of-contents)**

<a name="properties"></a>
## Properties
> Based in the Airbnb guide points: `12.1`, `12.2`

- Use dot notation when accessing properties.

    ```javascript
    const luke = {
      jedi: true,
      age: 28,
    };

    // bad
    const isJedi = luke['jedi'];

    // good
    const isJedi = luke.jedi;
    ```
    
 - Use bracket notation [] when accessing properties with a variable:
    ```javascript
    const person = {
        name:'Damian',
        age:32
    };
    
    const key = 'age';
    console.log(person[key]);
    ```    
    
**[⮬ back to top](#table-of-contents)**

<a name="primitives"></a>
## Primitives
The basic type data provided in js.
> Based in the Airbnb guide points: `1.1`

When you access a primitive type you work directly on its value.

- string
- number
- boolean
- null
- undefined
- symbol

**[⮬ back to top](#table-of-contents)**

<a name="variables"></a>
## Variables
Some points of how to handle and declare variables in javascript.
> Based in the Airbnb guide points: `2.1`, `2.2`, `13.1`, `13.2`, `13.3`, `13.4`, `13.5`, `13.8`

- Avoid to use global variable in the projects.
- Avoid use `var` in variable declaration, use `const`.
- If you must reassign references, use `let` instead of `const`.
- Group all your `const` and then group all your `let`.
- Remove unused variables.

    ```javascript
    // bad
    var a = 1;
    var b = 2;

    // good
    const a = 1;
    const b = 2;

    // bad
    var count = 1;
    if (true) {
      count += 1;
    }

    // good, use the let.
    let count = 1;
    if (true) {
      count += 1;
    }
    
    // bad
    superPower = new SuperPower();

    // good
    const superPower = new SuperPower();
    ```
    
**[⮬ back to top](#table-of-contents)**

<a name="tldr"></a>
## TL;DR;

### Don't use:
- No global vars.
- Declare variables using "var".
- Declare functions using "function" keyword.
- Avoid use "for" in loops.
- Array push, inmutability.
- Class.
- Use delete to remove a object atribute.
- Avoid nested if.
- else if.
- Heavy nesting https://www.w3.org/wiki/JavaScript_best_practices#Avoid_heavy_nesting.
- Avoid to add prototype to functions that could be used in a module.

### Use:
- Common code in functions, follow D.R.Y principle.
- Shorcut notation.
- Spread operator over Object.assign (airbnb 3.8).
- Pascal case naming.
- Modularize your code in modules.
- const and let!.
- Literal syntax for object creation (airbnb 3.1).
- Computed property names when creating objects (airbnb 3.2).
- Property value shorthand (airbnb 3.4).
- Group your shorthand properties at the beginning of your objec (airbnb 3.5).
- use the literal syntax for array creation (airnbnb 4.1).
- Use array spreads ... to copy arrays. (airbnb 4.3).
- use spreads ... instead of, Array.from. (airbnb 4.4).
- Use return statements in array method callbacks (airbnb 4.7).

**[⮬ back to top](#table-of-contents)**

Have fun!  🛸🐧🐲👽👆👻👺
