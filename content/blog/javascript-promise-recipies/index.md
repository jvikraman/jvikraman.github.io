---
title: "🍲 Javascript Promise Recipies"
date: "2020-07-30T13:00:00.000Z"
description: "A quick 101 on javascript promises and real-world uses cases."
---

A quick _overview_ on _Javascript Promises_ and how to apply them to _real-world_ uses cases.

### Promise States

Promises can be in one of three different states - `Pending`, `fulfilled` and `rejected`. If the promise is either in a _fulfilled_ (or) _rejected_ state, it can also said to be in a `settled` state and remains _forever_ in that state.

### Promise.prototype.then()

Use this method to _construct a promise chain_. Below is an example of how to use one with the native `js÷fetch` api to _retrieve_ a list of _pokemon_ characters from an online _pokemon_ REST API:

```js {4,5}
const API_URL = `https://pokeapi.co/api/v2/`;

fetch(API_URL + "pokemon")
  .then(response => response.json())
  .then(data => console.log("pokemon data", data));
```

### Promise.prototype.catch()

Use this method to _handle_ any _errors_ in your promise chain. Below is an example:

```js {1,6}
const API_URL = `https://poke-api.co/api/v2/`; // Incorrect URL!

fetch(API_URL + "pokemon")
  .then(response => response.json())
  .then(data => console.log("pokemon data", data))
  .catch(error => console.log(error));
```
In the example above, the API points to an _incorrect_ URL and by using the `js÷.catch()` handler you can gracefully _handle_ the `Type Error: Failed to Fetch` error.

### A quick look in to _handlers_

Inside of a `js÷.then()` _handler_ within a promise chain, you have _two handlers_ - also known as `fulfillment` and `rejection` handlers. These handlers are called with the `response` in case of _fulfillment_ and the `reason` (or) `error` in case of _rejection_. 

**Note:** If you provide both  fulfillment and rejection handlers in your promise chain, **ONLY one of them** will be called and **NOT both**!

Below is an example:

```js {1,8-12}
const API_URL = `https://poke-api.co/api/v2/`; // Incorrect URL!

fetch(API_URL + "pokemon").then(
  // onFulfilled
  response => {
    console.log(response)
  },
  // onRejected
  error => {
    // Do something with the error.
    console.warn(error)
  }
)
```

Below is a more proper way to _implement_ the _onFulfilled_ and _onRejected_ handlers to _handle_ various points of failure:

```js {2,5,11-13,17-23,26}{numberLines:true}
// Incorrect URL!
const API_URL = `https://poke-api.co/api/v2/`;

// Incorrect endpoint!
fetch(API_URL + "something").then(
  // onFulfilled
  response => {
    // With the `fetch` api, for incorrect endpoints, the
    // request will still be fulfilled and you must handle 
    // the status and data checks on your own!
    if(!response.ok) {
      thrown Error("Unsuccessful response!");
    }

    // You must return the `response` or else you 'll
    // end up with a dangling promise chain!
    return response.json()
      .then(data => {
        const pokemonNames = data.results
          .map(pokemon => pokemon.name)
          .join("\n");
        console.log(pokemonNames);
      });
  // The below handler doesn't work as expected if you
  // don't return your inner promise from above on line #15.
  }).then(undefined, error => {
      // do something with the error. for e.g.
      // log to a service or custom logic etc.
      console.warn(error);
  });
```

**Key takeaways:**

- Always _remember to return_ your promises within the promise chain. Otherwise, you will end up with a _dangling promise chain_ and you _cannot get hold_ of them later in your chain - in this scenario, the final `js÷.then()` handler _will not work_ as expected
- The final `js÷.then()` handler can be _simplified_ to the modern-day `js÷.catch()` handler that handles all points of failure outlined in the example. Even if the `js÷response.json()` operation on _line #15 returns a rejected promise_
- **Note:** The `js÷fetch()` api returns a _fulfilled_ promise with _empty results for incorrect endpoints_ - so, remember to handle those _empty scenarios_ in your code by checking the response's _status/ok_ properties and performing actions accordingly

### Promise.prototype.finally()

Use this method to _perform clean-up logic_ (or) _release resources_ in your promise chain. Below is an example of using one to _hide_ the _loading indicator_ (in our case a _spinner element_) in both _fulfilled_ and _rejected_ scenarios:

```html {10-12,15}{numberLines:true}
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    ...
  </head>
  <body>
    <h1>Pokemon Characters</h1>
    <div>
      <div id="spinner">
        <img src="/spinner.gif" alt="Spinner" height="20" />
      </div>
      <div id="root"></div>
    </div>
    <script src="index.js"></script>
  </body>
</html>
```

```js {3,19-21,29-35}{numberLines:true}
// index.js
const output = document.getElementById("root");
const spinner = document.getElementById("spinner");
const API_URL = `https://pokeapi.co/api/v2/`;

fetch(API_URL + "pokemon")
  .then(
    // onFulfilled
    response => {
      if (!response.ok) {
        throw Error("Unsuccessful response!");
      }

      return response.json().then(data => {
        const pokemonCharacters = data.results
          .map(pokemon => pokemon.name)
          .join("\n");
        output.innerText = pokemonCharacters;
        // If you need to pass down the pokemon character
        // data, you can return it like so.
        return pokemonCharacters;
      });
    }
  )
  .catch(error => {
    output.innerText = "Unable to retrieve pokemon data!";
    console.warn(error);
  })
  .finally(() => {
    spinner.remove();
    // You can also get hold of the data returned within
    // the first fulfillment handler on line #21 by attaching
    // a .then() handler like below.
  })
  .then(result => console.log(result));
```

### Promise.prototype.reject()

Use this method to _reject a promise_ in your promise chain. For e.g. instead of throwing an error on `line #11` in our previous example, we can _replace_ it with a _rejected promise_ like so:

```js {numberLines:true}{7-9}
// ...
fetch(API_URL + "pokemon")
  .then(
    // onFulfilled
    response => {
      if (!response.ok) {
        return Promise.reject(
          new Error("Unsuccessful response!")
        );
      }

      return response.json().then(data => {
        // ...
      });
    }
);
```
**Note**: Return a new `Error()` _object_ within the _rejected_ promise instead of _plain strings_. for e.g. `js÷return Promise.reject("Unsuccessful response!");` _Error objects_ will provide a _helpful stack trace for debugging_ in large applications vs the latter.

### Promise.prototype.resolve()

Use this method to _resolve a promise_ in your promise chain. Below are some _caveats_ with the `resolve()` method:

```js {4}
const fulfilledPromise = Promise.resolve(100);

// returns true - reference identies are equal.
Promise.resolve(fulfilledPromise) === fulfilledPromise;
```

```js {3,8}
// The below line will trigger an `Uncaught promise error`
// remember to handle those errors with `rejection` handlers.
const rejectedPromise = Promise.reject(new Error("!"));

// returns true - Promise.resolve will NOT return a `fulfilled`
// promise all the time. If you pass a `rejected` promise to it,
// it will return a rejected promise.
Promise.resolve(rejectedPromise) === rejectedPromise;
```

```js {3}
// The below line will return a `fulfilled` promise with the
// `Error` object as the return value.
Promise.resolve(new Error("!"));
```

One of the biggest _benefits_ of using the `resolve()` method is to _convert thenables_ (or) _promise-like_ objects to _native_ promises. Below is an example of how to convert such a thenable `jQuery` method to a native promise.

```js {1}
Promise.resolve($.getJSON(API_URL + "pokemon"))
  .then(data => {
    console.log(data);
  }).catch(error => {    
    console.warn(error);
  });  
```

### The promise constructor

Use the promise constructor to _create_ a new `Promise` object. The constructor receives a _single executor function_ with _two parameters_ named `resolve` and `reject` that are both _functions_. Within the executor function you basically kick-off some operation and call either `resolve(value)` or `reject(reason)` to either _fulfill_ or _reject_ the promise.

Here's an example:

```js
// index.js file

// Create a new promise that will resolve after 1sec.
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve();
  }, 1000);
});

// Using the promise.
promise.then(
  // Fulfillment handler.
  () => console.log("Fulfilled."),
  // Rejection handler.
  () => console.log("Rejected!")
);

// Output: The string `Fulfilled` after 1sec.
```

Below is another example demonstrating `sleep` functionality using _Promises_:

```js
// index.js file.

// Custom fn that resolves after user-specified `n` ms.
const sleep = (ms) => {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}

// Using the sleep function.
sleep(1000)
  .then(() => console.log("After 1s."))
  .then(() => sleep(1000))
  .then(() => console.log("After 2s."));
```

### 💡 Converting a callback-based function to Promise-based

Below is an example of how you can _convert_ the _node-based_ `fs.readFile()` api to _promise-based_:

```js
// index.js file.

// Example of a node callback-based function.

// Import the node file system module.
const fs = require("fs");

// Attempt to read current file asynchronously 
// using special identifier `__filename`
fs.readFile(__filename, "utf8", (error, contents) => {
  if (error) {
    console.log(error);
  } else {
    console.log(contents);
  }
});
```

```js
// Converting the callback-based fn to promise-based.
const fs = require("fs");

// Promise-based wrapper fn wrapping the callback-based fn.
const readFile = (path, encoding) => {
  return new Promise((resolve, reject) => {
    fs.readFile(path, encoding, (error, contents) => {
      if (error) {
        reject(error);
      } else {
        resolve(contents);
      }
    });
  });
}

// Usage:
// output: fulfilled promise with contents of current file.
readFile(__filename, "utf8")
  .then(contents => console.log(contents))
  .catch(error => console.log(error));

// output: rejected promise with the error indicating unable
// to locate/read file!
readFile("somefile.txt", "utf8")
  .then(contents => console.log(contents))
  .catch(error => console.log(error));
```

In _node_ environment, you can use node's `js÷util.promisify()` api to _simplify_ the code and _avoid creating_ the custom promisify function by hand. Below is an example:

```js {6}
const fs = require("fs");
const util = require("util");

// Call the `util.promisfy()` api and pass in the
// callback-based function you'd like to promisify.
const readFile = util.promisify(fs.readFile);

// Usage:
// output: fulfilled promise with contents of current file.
readFile(__filename, "utf8")
  .then(contents => console.log(contents))
  .catch(error => console.log(error));
```

### Promise.prototype.race()
Use this method to _handle multiple promises_ at any point in time and _wait for the fastest promise to settle_. It accepts an _array of promises_ and returns the result of whichever promise _settles first_. Below is an example:

```js {15}
// Custom fn that will resolve after `n` ms.
const resolveAfter = (ms, value) => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(value);
    }, ms);
  });
}

// Construct two different promises that'll resolve
// after 1 and 2 secs. and wait for the fastest to settle.
const promiseA = resolveAfter(1000);
const promiseB = resolveAfter(2000);

const fastestPromise = Promise.race([promiseA, promiseB]);

// Usage:
// Output: "A" after 1sec.
fastestPromise
  .then(result => console.log(result));
```

**Key things to note:**

- The _result_ of `Promise.race()` is a _promise itself_ and depends on the _outcome of input promises_ - for e.g. final outcome is _fulfilled_ if the fastest promise is _fulfilled_ and vice versa.
- If you pass an _empty array_ as the input, for e.g. `js÷Promise.race([]);` the outcome will _forever_ be `pending`

### Promise.prototype.all()

This method accepts an _array of input promises_ and returns a final promise whose outcome is either _fulfilled_ if _all of the_ input promises are _fulfilled_ and _rejected_ if _any of the_ input promises are _rejected_. The return values are _array-based_ and in the _same order_ as the input promises.

Below is an example of how to _convert_ a regular, _sequential_ promise chain to fetch resources in _parallel_ using the `js÷.all()` method:

```js {18-21}
// The sequential version!

const API_URL = "https://swapi.dev/api/";
const output = document.getElementById("root");
const spinner = document.getElementById("spinner");

// Custom fn to fetch resources from Starwars API.
const queryAPI = endpoint => {
  return fetch(API_URL + endpoint).then(response => {
    return response.ok
      ? response.json()
      : Promise.reject(new Error("Unsuccessful response!"));
  });
};

// This fn fetches resources sequentially resulting
// in a waterfall timeline in devtools.
queryAPI("films")
  .then(films => {
    return queryAPI("planets").then(planets => {
      return queryAPI("species").then(species => {
        output.innerText = `
        ${films.length} films
        ${planets.length} planets
        ${species.length} species
        `;
      });
    });
  })
  .catch(error => console.warn(error))
  .finally(() => {
    spinner.remove();
  });
```

```js {6-10}
// Now, the parallel version. Only the specific
// changes are shown.

// ...

Promise.all([
  queryAPI("films"),
  queryAPI("planets"),
  queryAPI("species")
]).then(({ films, planets, species }) => {
    output.innerText = `
        ${films.length} films
        ${planets.length} planets
        ${species.length} species
      `;
}).catch(error => {
    console.warn(error);
    output.innerText = error.message;
}).finally(spinner.remove());
```

**Note:** In the above example all three endpoints are _fetched simultaneously_ instead of a _sequential waterfall_.

### Promise.prototype.allSettled() [ES2020]

**Added in ES2020**, this method accepts a _list of input promises_ and returns a _final Promise object_ that is _fulfilled_ with the results of _settled input promises_ (i.e. both fulfilled and rejected)

The **ONLY** time the final outcome will be _rejected_ is when an _error_ occurs while iterating over the input promises (or) if a _non-iterable input_ is passed in. 

**NOTE:** The main _difference_ between this method and the `js÷.all()` method is that for the latter _all_ the input promises needs to be _fulfilled_ in order for the final outcome to be _fulfilled_. Similarly, the final outcome will be _rejected_ if any of the input promises are _rejected_ (_more like either all or nothing_)

Below is an example of how to use this method with our previous example:

```js {7-11}
// Only the specific parts are shown below.

// ...

// Query the endpoints via `.allSettled()` method and filter 
// only the `fulfilled` ones.
Promise.allSettled([
  queryAPI("films").then(films => `${films.length} films`),
  queryAPI("planets").then(planets => `${planets.length} planets`),
  queryAPI("species").then(species => `${species.length} species`)
]).then(results => {
    const stats = results
      .filter(result => result.status === "fulfilled")
      .map(result => result.value)
      .join("\n");

    output.innerText =
     stats.length === 0
      ? "Failed to load stats!"
      : stats;
}).catch(error => {
    console.warn(error);
    output.innerText = error.message;
}).finally(spinner.remove());
```

### Promise.prototype.any()

This method accepts a _list of input promises_ and returns a _Promise_ object that is _fulfilled_ if _any of the_ input promises are _fulfilled_ and _rejected_ if _all of the_ input promises are _rejected_. It is useful in scenarios where you need _only one_ of the input promises to succeed _vs all_ the promises like in the `js÷Promise.all()` method.

**Key things to note:**

- If _more than one_ input promise is _fulfilled_, then _only the first fulfilled_ promise value is returned and the rest is _ignored_
- If _all_ the input promises are _rejected_, the final outcome is a _rejected_ promise with an _aggregate error_ indicating the _reason_ for rejection. You can _access_ the individual errors in your `js÷.catch()` (or) `rejection handler` using the `.errors` property/field which is an _array of errors_

Below is an example:

```js {26-36}
const API_URL_1 = "https://swapi.dev/api/";
const API_URL_2 = "https://starwars.egghead.training/";

// Getting hold of our output <div /> and the spinner
// element from index.html.
const output = document.getElementById("root");
const spinner = document.getElementById("spinner");

// Helper fn to query the API.
const query = (rootURL, endpoint) => {
  return fetch(rootURL + endpoint).then(response => {
    return response.ok
      ? response.json()
      : Promise.reject(new Error("Unsuccessful response!"));
  });
};

// Query both the APIs and wait for the fastest
// promise to resolve/reject. 

// Note: As of Aug 2020, the `Promise.any` method is
// only available on FF 79+, Chrome 85+, Safari 14+
// So, remember to use a polyfill (or) handle
// unavailability scenarios like below.
const queryAPI = endpoint => {
  return new Promise((resolve, reject) => {
    try {
       Promise.any([
        query(API_URL_1, endpoint),
        query(API_URL_2, endpoint)
      ]).then(result => resolve(result))
        .catch(error => reject(error))
    } catch (error) {
        reject(error);
    }   
  });  
};

// Query the `films` endpoint and render the film titles in UI.
queryAPI("_films")
  .then(result => {
    const filmTitles = result
      .sort((a, b) => a.episode_id - b.episode_id)
      .map(film => `${film.episode_id}. ${film.title}`)
      .join("\n");
    output.innerText = filmTitles;
  })
  .catch(error => {
    console.warn(error);
    output.innerText = error;
  })
  .finally(() => spinner.remove());
```

### Async/Await with Promises

Use _Async/Await_ with promises to write _asynchronous_ code that looks like _synchronous_ code with the normal _control flows_ you'd expect in a synchronous program. With `async` functions, the `await` operator is used to _pause execution_ of the function until the promise is _settled_.

Below is an example:

```js {9,10,19,21}
const API_URL_1 = "https://swapi.dev/api/";

// Getting hold of our output <div /> and the spinner
// element from index.html.
const output = document.getElementById("root");
const spinner = document.getElementById("spinner");

// Async helper fn to query the API and return the results.
const query = async (rootURL, endpoint) => {
  const response = await fetch(rootURL + endpoint);
  if (response.ok) {
    return response.json();
  }
  throw Error("Unsuccessful response!");
};

// Async fn to handle the query results and render 
// some info to the UI.
const main = async () => {
  try {
    const [films, planets, species] = await Promise.all([
      query(API_URL_1, "films"),
      query(API_URL_1, "planets"),
      query(API_URL_1, "species")
    ]);

    output.innerText = 
      `${films.length} films
       ${planets.length} planets
       ${species.length} species`;  

  } catch (error) {
    output.innerText = error;
  } finally {
    spinner.remove();
  }
};

// Call the async function.
main();
```

**Key things to note**: 

- If you _throw_ an _error_ within your async function, the function _implicitly_ returns a _rejected promise_
- The `await` keyword returns the _direct result instead of a response_. So, there's no need to further call `js÷.then()` on the result.
- The plain-old `js÷try {} catch{}` statements _works well_ with async functions for _better readability and error handling_

🎉 And that's a _wrap_ on our _quick 101 on Javascript Promises_. These notes are an _inspiration_ from [Marius Schulz's](https://mariusschulz.com/) amazing [egghead course on this topic](https://egghead.io/courses/javascript-promises-in-depth). His courses are a _great resource_ for anyone learning Javascript. 