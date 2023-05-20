# ECMAScript `Array.prototype.ap` method

## Status

**Stage:** 0

**Champions:**
* Brandon McConnell ([@brandonmcconnell](https://github.com/brandonmcconnell))
* _looking for at least one more_

For more information see the [TC39 proposal process.](https://tc39.es/process-document/)

## Authors

* Brandon McConnell ([@brandonmcconnell](https://github.com/brandonmcconnell))

## Introduction

This proposal introduces the `Array.prototype.ap` method to ECMAScript, based on the concept of [applicative functors](https://github.com/fantasyland/fantasy-land#apply) found in functional programming. Functionally, this method works inversely to the existing `Array.prototype.map` method, allowing for simpler application of multiple functions to a single argument.

The `Array.prototype.ap` method applies an array of functions to a single argument. Given an array `fns` of functions, the expression `fns.ap(x)` is analogous to `fns.map(fn => fn(x))`. A new array is formed, each new array element the result of applying its associated function (by index) from the array to the argument provided.

## Example

```js
// utils.js

const buildSlug = ({ title }) => title.toLowerCase().replaceAll(' ', '-');
const getWordCount = ({ title, content }) => title.split(' ').length + content.split(' ').length;
const getDaysSince = ({ date }) => (Date.now() - new Date(date)) / 864e5;

// usage.js

const apiResponse = {
  title: 'A single day in Paris',
  subtitle: 'Let\'s talk Eiffel&hellip;',
  content: 'Lorem ipsum yada yada',
  category: 'travel',
  date: '7/8/2022 19:05:46 GMT-0400',
};

const [slug, words, days] = [buildSlug, getWordCount, getDaysSince].ap(apiResponse);
```

As mentioned, that final line is analogous to this:

```js
const [slug, words, days] = [buildSlug, getWordCount, getDaysSince].map(fn => fn(apiResponse));
```

## Use Cases

Functional programming often involves applying a collection of functions to a single value or set of values. This pattern can appear in various contexts, such as:

* **Data Transformation:** Multiple attributes might be derived from a single data entity. For instance, from an `article` object, one might wish to derive a slug, word count, and the number of days since the article was released.
* **Batch Processing:** Similar operations (like validations or transformations) might be required to be performed on a single input. These operations can be encapsulated as functions and bundled in an array used with `Array.prototype.ap` vs. looping over the functions or declaring them all separately.

Like other array methods, this proposed `Array.prototype.ap` method simplifies use cases like these by removing the need for boilerplate code.

## Advantages and Benefits

* **Code Brevity:** Reduce verbosity by eliminating the need to manually loop over an array of functions or opt for the `fns.map(fn => fn(value))` alternative pattern.
* **Functional Programming:** Enrich ECMAScript's functional programming capabilities by providing a more native way to apply functional concepts.
* **Code Readability:** By streamlining the application of multiple functions to a single argument, the `Array.prototype.ap` method can enhance code readability, making it easier to understand and maintain.

## Backward Compatibility

The `Array.prototype.ap` method does not interfere with existing functionalities and provides a new method without altering the behavior of any existing ones. As such, it preserves backward compatibility.

## Polyfill/shim

<details><summary>TypeScript</summary><br />

```ts
declare global {
  interface Array < T > {
    ap: < U > (this: ((x: U) => T)[], value: U) => T[];
  }
}

if (!Array.prototype.ap) {
  Array.prototype.ap = function < T, U > (this: ((x: U) => T)[], value: U): T[] {
    if (!Array.isArray(this)) {
      throw new TypeError('The Array.prototype.ap method can only be used on arrays');
    }

    if (value === undefined) {
      throw new TypeError('Value must be defined');
    }

    let O = Object(this);
    let len = O.length >>> 0;
    let A = new Array(len);
    let k = 0;

    while (k < len) {
      if (k in O) {
        let kValue = O[k];
        if (typeof kValue !== 'function') {
          throw new TypeError('All array elements must be functions');
        }
        let mappedValue = kValue(value);
        A[k] = mappedValue;
      }
      k++;
    }

    return A;
  }
}
```

</details>

<details><summary>Traditional JS</summary><br />

```js
if (!Array.prototype.ap) {
  Array.prototype.ap = function(value) {
    if (!Array.isArray(this)) {
      throw new TypeError('The Array.prototype.ap method can only be used on arrays');
    }
    if (value === undefined) {
      throw new TypeError('Value must be defined');
    }
    let O = Object(this);
    let len = O.length >>> 0;
    let A = new Array(len);
    let k = 0;
    while (k < len) {
      if (k in O) {
        let kValue = O[k];
        if (typeof kValue !== 'function') {
          throw new TypeError('All array elements must be functions');
        }
        let mappedValue = kValue(value);
        A[k] = mappedValue;
      }
      k++;
    }
    return A;
  };
}
```

</details>

This polyfill/shim is ready for any tinkering and experiment now. Please open an issue on this repo if you experience any bugs while working with it or if you have any ideas/recommendations.

---

### Credit
  
I have to give some credit to Bergi ([@bergus](https://github.com/bergus)), who helped me refine my idea of functional destructuring ([source](https://es.discourse.group/t/functional-destructuring-destructure-via-function-calls-using-new-syntax/1394/5)) down to this simpler and more versatile spec.
