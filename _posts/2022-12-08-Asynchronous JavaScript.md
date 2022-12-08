---
title: Asynchronous JavaScript
date: 2022-12-08
categories: []
tags: []
---
### **Callbacks**
> "I will call back later!"
> 
> A callback is a function passed as an argument to another function. This technique allows a function to call another function. A callback function can run after another function has finished

#### **Function Sequence**

JavaScript functions are executed in the sequence they are called. Not in the sequence they are defined.

This example will end up displaying "Goodbye":
```js
function myFirst() {
  myDisplayer("Hello");
}

function mySecond() {
  myDisplayer("Goodbye");
}

myFirst();
mySecond();
```
This example will end up displaying "Hello":
```js
function myFirst() {
  myDisplayer("Hello");
}

function mySecond() {
  myDisplayer("Goodbye");
}

mySecond();
myFirst();
```
#### **Sequence Control**
Sometimes you would like to have better control over when to execute a function.

Suppose you want to do a calculation, and then display the result.

You could call a calculator function (myCalculator), save the result, and then call another function (myDisplayer) to display the result:
```js
function myDisplayer(some) {
  document.getElementById("demo").innerHTML = some;
}

function myCalculator(num1, num2) {
  let sum = num1 + num2;
  return sum;
}

let result = myCalculator(5, 5);
myDisplayer(result);
```
Or, you could call a calculator function (myCalculator), and let the calculator function call the display function (myDisplayer):
```js
function myDisplayer(some) {
  document.getElementById("demo").innerHTML = some;
}

function myCalculator(num1, num2) {
  let sum = num1 + num2;
  myDisplayer(sum);
}

myCalculator(5, 5);
```
The problem with the first example above, is that you have to call two functions to display the result.

The problem with the second example, is that you cannot prevent the calculator function from displaying the result.

Now it is time to bring in a callback.
#### **JavaScript Callbacks**
> A callback is a function passed as an argument to another function.

Using a callback, you could call the calculator function (myCalculator) with a callback (myCallback), and let the calculator function run the callback after the calculation is finished:
```js
function myDisplayer(some) {
  document.getElementById("demo").innerHTML = some;
}

function myCalculator(num1, num2, myCallback) {
  let sum = num1 + num2;
  myCallback(sum);
}

myCalculator(5, 5, myDisplayer);
```
In the example above, myDisplayer is a called a callback function.

It is passed to myCalculator() as an argument.

#### **When to Use a Callback?**
The examples above are not very exciting.

They are simplified to teach you the callback syntax.

Where callbacks really shine are in asynchronous functions, where one function has to wait for another function (like waiting for a file to load).

### **Asynchronous JavaScript**
> "I will finish later!"
>
> Functions running in parallel with other functions are called asynchronous. A good example is JavaScript setTimeout()

### **Asynchronous JavaScript**
The examples used in the previous chapter, was very simplified.

The purpose of the examples was to demonstrate the syntax of callback functions:
```js
function myDisplayer(something) {
  document.getElementById("demo").innerHTML = something;
}

function myCalculator(num1, num2, myCallback) {
  let sum = num1 + num2;
  myCallback(sum);
}

myCalculator(5, 5, myDisplayer);
```
In the example above, myDisplayer is the name of a function.

It is passed to myCalculator() as an argument.
> In the real world, callbacks are most often used with asynchronous functions. A typical example is JavaScript setTimeout().

#### **Waiting for a Timeout**
When using the JavaScript function setTimeout(), you can specify a callback function to be executed on time-out:
```js
setTimeout(myFunction, 3000);

function myFunction() {
  document.getElementById("demo").innerHTML = "I love You !!";
}
```
In the example above, myFunction is used as a callback.

myFunction is passed to setTimeout() as an argument.

3000 is the number of milliseconds before time-out, so myFunction() will be called after 3 seconds.

#### **Callback Alternatives**
With asynchronous programming, JavaScript programs can start long-running tasks, and continue running other tasks in parallell.

But, asynchronus programmes are difficult to write and difficult to debug.

Because of this, most modern asynchronous JavaScript methods don't use callbacks. Instead, in JavaScript, asynchronous programming is solved using **Promises** instead.

### **JavaScript Promises**
> "I Promise a Result!"
> 
> "Producing code" is code that can take some time. "Consuming code" is code that must wait for the result. A Promise is a JavaScript object that links producing code and consuming code.

#### **JavaScript Promise Object**
A JavaScript Promise object contains both the producing code and calls to the consuming code:
```js
let myPromise = new Promise(function(myResolve, myReject) {
// "Producing Code" (May take some time)

  myResolve(); // when successful
  myReject();  // when error
});

// "Consuming Code" (Must wait for a fulfilled Promise)
myPromise.then(
  function(value) { /* code if successful */ },
  function(error) { /* code if some error */ }
);
```
When the producing code obtains the result, it should call one of the two callbacks:
|Result	|Call|
|  ----  | ----  |
|Success|	myResolve(result value)|
|Error|	myReject(error object)|

#### **Promise Object Properties**
A JavaScript Promise object can be:
- Pending
- Fulfilled
- Rejected

The Promise object supports two properties: state and result.

While a Promise object is "pending" (working), the result is undefined.

When a Promise object is "fulfilled", the result is a value.

When a Promise object is "rejected", the result is an error object.
> You cannot access the Promise properties state and result.
> 
>You must use a Promise method to handle promises.

### **JavaScript Async/Await**
> "async and await make promises easier to write"
>
> async makes a function return a Promise. await makes a function wait for a Promise
#### **Async Syntax**
The keyword async before a function makes the function return a promise:
```js
async function myFunction() {
  return "Hello";
}
// is the same as:
function myFunction() {
  return Promise.resolve("Hello");
}
```
Here is how to use the Promise:
```js
myFunction().then(
  function(value) { /* code if successful */ },
  function(error) { /* code if some error */ }
);
// Or simpler, since you expect a normal value (a normal response, not an error):
myFunction().then(
  function(value) {function(value);}
);
```

#### **Await Syntax**
The await keyword can only be used inside an async function.

The await keyword makes the function pause the execution and wait for a resolved promise before it continues:
```js
let value = await promise;
```
Let's go slowly and learn how to use it by examples.
```js
// Basic Syntax
async function myDisplay() {
  let myPromise = new Promise(function(resolve, reject) {
    resolve("I love You !!");
  });
  document.getElementById("demo").innerHTML = await myPromise;
}
myDisplay();

// The two arguments (resolve and reject) are pre-defined by JavaScript. We will not create them, but call one of them when the executor function is ready. Very often we will not need a reject function.
async function myDisplay() {
  let myPromise = new Promise(function(resolve) {
    resolve("I love You !!");
  });
  document.getElementById("demo").innerHTML = await myPromise;
}
myDisplay();

// waiting for timeout
async function myDisplay() {
  let myPromise = new Promise(function(resolve) {
    setTimeout(function() {resolve("I love You !!");}, 3000);
  });
  document.getElementById("demo").innerHTML = await myPromise;
}
myDisplay();

// waiting for a File
async function getFile() {
  let myPromise = new Promise(function(resolve) {
    let req = new XMLHttpRequest();
    req.open('GET', "mycar.html");
    req.onload = function() {
      if (req.status == 200) {
        resolve(req.response);
      } else {
        resolve("File not Found");
      }
    };
    req.send();
  });
  document.getElementById("demo").innerHTML = await myPromise;
}
getFile();
```