Welcome to the Rubico tour. This tour covers high level concepts and provides runnable and editable code examples. All code areas have the Rubico core methods imported globally.

```javascript
const {
  pipe, tap,
  switchCase, tryCatch,
  fork, assign, get, set, pick, omit,
  map, filter, reduce, transform, flatMap,
  and, or, not, any, all,
  eq, gt, lt, gte, lte,
  thunkify, always,
  curry, __,
} = rubico // available globally
```

# Table of Contents

 1. [[a]synchrony](#a-synchrony)
 2. [Composition](#composition)
 3. [Polymorphism](#polymorphism)
 4. [Control Flow](#control-flow)
 5. [Error Handling](#error-handling)
 6. [Transducers](#transducers)

# [a]synchrony
**Stop worrying about async**. Pass synchronous or asynchronous functions to any Rubico operator - all promises are resolved for their promised value before continuing. Run things in parallel without having to call `Promise.all` on `someArray.map(...)`. For more on this behavior, see this [blog post](https://dev.to/richytong/rubico-a-synchrnous-functional-syntax-motivation-20hf).

```javascript [playground]
const getTodo = id => fetch(`https://jsonplaceholder.typicode.com/todos/${id}`)

map(pipe([
  getTodo,
  res => res.json(),
  console.log,
]))([1, 2, 3, 4, 5])
```

Press the `run` button to make five requests using `fetch`, parse five request bodies, and log five todos out to the console - all in parallel.

# Composition
**Reduce coupling and complexity**. Use Rubico's operators to create compositions of small, reusable functions. Add functionality to your program by composing the desired function - Rubico gives you the tools to make this as simple and seamless as possible.

```javascript [playground]
const identity = value => value

const square = number => number ** 2

const doMathsWithLogs = pipe([
  tap(number => console.log('input:', number)),
  fork({
    number: identity,
    numberSquared: pipe([
      square,
      tap(curry.arity(2, console.log, 'result of square:', __)),
    ]),
  }),
])

console.log(doMathsWithLogs(3))
// input: 3
// result of square: 9
// { number: 3, numberSquared: 9 }
```

The `run` button above executes the pipeline `doMathsWithLogs` that logs a number out to the console, then parallelizes its identity operation and another pipeline into an object `{ number, numberSquared }`. The above example also introduces Rubico's `curry` and placeholder `__`; use these to compose any function by creating a partially applied variant suited for the task at hand.

# Polymorphism
**Reuse code across arrays and asynchronous streams**. Use `map`, `filter`, `reduce`, `transform`,  or `flatMap` to transform async iterables, strings, sets, maps, binary arrays, and object values. This concept applies generally to any function of Rubico; if a transformation + data structure pairing makes sense by math, it should be supported.

```javascript [playground]
const square = number => number ** 2

const iterables = [
  [1, 2, 3, 4, 5],
  '12345',
  new Set([1, 2, 3, 4, 5]),
  new Uint8Array([1, 2, 3, 4, 5]),
  { a: 1, b: 2, c: 3, d: 4, e: 5 },
  new Map([['a', 1], ['b', 2], ['c', 3], ['d', 4], ['e', 5]]),
]

iterables.forEach(pipe([map(square), console.log]))
// [1, 2, 3, 4, 5]
// '1491625'
// Set { 1, 4, 9, 16, 25 }
// Uint8Array [1, 4, 9, 16, 25]
// { a: 1, b: 4, c: 9, d: 16, e: 25 }
// Map { 'a' => 1, 'b' => 4, 'c' => 9, 'd' => 16, 'e' => 25 }
```

# Control Flow
**Create declarative, SQL-esque logical expressions** by composing predicates with Rubico's logical operators. Below is a map of vanilla JavaScript operators to their functional analogs in Rubico.

* Conditional (Ternary), `a ? b : c` → `switchCase([f, g, h])`
* Logical And, `a && b` → `and([f, g])`
* Logical Or, `a || b` → `or([f, g])`
* Logical Not, `!a` → `not(f)`
* Identity, `a === b` → `eq(f, g)`
* Greater Than, `a > b` → `gt(f, g)`
* Less Than, `a < b` → `lt(f, g)`
* Greater Than or Equal, `a >= b` → `gte(f, g)`
* Less Than or Equal, `a <= b` → `lte(f, g)`

```javascript [playground]
const hasFlag = flag => array => array.includes(flag)

const log = message => () => console.log(message)

const cli = switchCase([
  or([
    hasFlag('-h'),
    hasFlag('--help'),
  ]), log('USAGE: ...'),
  or([
    hasFlag('-v'),
    hasFlag('--version'),
  ]), log('v0.0.0'),
  log('USAGE: ...'),
])

cli(['-h']) // USAGE: ...
cli(['--version']) // v0.0.0
cli(['???']) // USAGE: ...
```

# Error Handling
**Handle errors with functions** - a `tryer` and a `catcher`. The `tryer` is tried, while the `catcher` catches any errors thrown or Promises rejected. You can use a single error handler with `tryCatch` to catch all errors thrown by sync or async functions including promise rejections in a `pipe`.

```javascript [playground]
console.log(
  tryCatch(
    JSON.parse,
    error => ({ error, timestamp: Date.now() })
  )('hello')
)
// {
//   error: SyntaxError: Unexpected token h in JSON at position 0,
//   timestamp: number,
// }
```

# Transducers
**Wrangle large or infinite streams of data** with Rubico's transducers. Create transducers via the transformation functions `map`, `filter`, or `flatMap`, then consume them with `reduce` or `transform`. Additionally, transducers created with Rubico can act on asynchronous data streams or operate asynchronously. More on this behavior [here](https://rubico.land/blog/2020/10/02/transducers-crash-course).

```javascript [playground]
// length number => string => Generator<string>
const generateSegmentsOfLength = length => function* (string) {
  for (let i = 0; i < string.length; i += length) {
    yield string.slice(i, i + length)
  }
}

// bigint -> string
const toBinaryString = value => value.toString(2)

// string -> number
const toBinaryInt = value => parseInt(value, 2)

// decimal string -> notes string
const decimalStringToNotes = pipe([
  BigInt,
  toBinaryString,
  generateSegmentsOfLength(7),
  transform(map(pipe([
    toBinaryInt,
    String.fromCharCode,
  ])), ''), // map(pipe([...])) is a transducer consumed by transform
])

console.log(
  decimalStringToNotes('16791573288892525934609440079317541905554393653557736896280802239551592289061061348368963')
) // CCGGAAGFFEEDDCGGFFEEDGGFFEEDCCGGAAGFFEEDDC
```

You've arrived at the end of the tour. From here, you could get started with Rubico in a project ([installation](/#installation)) or read more at the [docs](/docs).
