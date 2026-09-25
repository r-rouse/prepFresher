# JavaScript & React Interview Prep --- Session Notes

## 1. Closures and Separate Function State

A closure lets a function retain access to variables from the scope
where it was created, even after the outer function has finished.

``` javascript
function createCounter() {
  let count = 0;

  return function () {
    count++;
    console.log(count);
  };
}

const counterA = createCounter();
const counterB = createCounter();

counterA(); // 1
counterA(); // 2
counterB(); // 1
```

Each call to `createCounter()` creates a separate `count`. `counterA`
and `counterB` therefore maintain independent state.

### Shared outer variables

If the variable exists outside the function, multiple closures can
reference the same variable.

``` javascript
let count = 0;

function increment() {
  count++;

  return function () {
    count++;
    console.log(count);
  };
}
```

Here there is only one `count`.

**Remember:** Closures capture variables from their surrounding scope.
Whether state is shared depends on where that variable was created.

------------------------------------------------------------------------

## 2. Objects Are References

``` javascript
const user = {
  name: "Randall"
};

const user2 = user;

user2.name = "Billy";
```

Both variables reference the same object.

``` javascript
console.log(user.name);  // "Billy"
console.log(user2.name); // "Billy"
console.log(user === user2); // true
```

Two objects containing identical data are still different objects:

``` javascript
const a = { name: "Billy" };
const b = { name: "Billy" };

console.log(a === b); // false
```

For objects, `===` checks whether both variables reference the same
object, not whether their contents have the same shape.

------------------------------------------------------------------------

## 3. Spread Operator and Shallow Copies

The spread operator creates a new top-level object:

``` javascript
const user2 = {
  ...user
};
```

But it only makes a **shallow copy**.

``` javascript
const user = {
  name: "Randall",
  address: {
    city: "Portland"
  }
};

const user2 = {
  ...user
};
```

`user` and `user2` are different objects, but their `address` properties
still reference the same nested object.

``` javascript
user2.name = "Billy";
user2.address.city = "Seattle";

console.log(user.name);         // "Randall"
console.log(user.address.city); // "Seattle"
```

To copy the nested object too:

``` javascript
const user2 = {
  ...user,
  address: {
    ...user.address
  }
};
```

**Remember:** Spread every level that you intend to change.

------------------------------------------------------------------------

## 4. React State Must Not Be Mutated

Bad:

``` javascript
function changeCity() {
  user.address.city = "Seattle";
  setUser(user);
}
```

This mutates the existing state and then gives React the same object
reference.

Better:

``` javascript
function changeCity() {
  setUser({
    ...user,
    address: {
      ...user.address,
      city: "Seattle"
    }
  });
}
```

React state updates should create new references for the parts of state
being changed.

------------------------------------------------------------------------

## 5. React State Is a Snapshot

Consider:

``` javascript
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
}
```

If `count` is `0`, all three calls see the same render's value:

``` javascript
setCount(1);
setCount(1);
setCount(1);
```

The resulting count is `1`, not `3`.

To perform three updates based on the previous value:

``` javascript
setCount(prev => prev + 1);
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

Now:

``` text
0 → 1 → 2 → 3
```

**Rule:** When the next state depends on the previous state, prefer the
functional updater:

``` javascript
setState(prev => ...)
```

------------------------------------------------------------------------

## 6. State Doesn't Change Immediately Inside the Current Render

``` javascript
function handleClick() {
  setCount(prev => prev + 1);
  console.log(count);
}
```

If `count` was `0`, the console prints:

``` text
0
```

The UI will display `1` after React re-renders.

Think of state as a snapshot belonging to a particular render.

------------------------------------------------------------------------

## 7. `useEffect` and Cleanup

``` javascript
useEffect(() => {
  console.log("effect:", count);

  return () => {
    console.log("cleanup:", count);
  };
}, [count]);
```

On initial render:

``` text
effect: 0
```

After changing `count` to `1`:

``` text
cleanup: 0
effect: 1
```

After three separate increments:

``` text
effect: 0
cleanup: 0
effect: 1
cleanup: 1
effect: 2
cleanup: 2
effect: 3
```

When a dependency changes, React runs the cleanup for the previous
effect before running the new effect.

Cleanup also runs when the component unmounts.

------------------------------------------------------------------------

## 8. Arrow Functions and Implicit Returns

This does **not** return the doubled number:

``` javascript
const doubled = numbers.map(num => {
  num * 2;
});
```

Because `{}` creates a function body, an explicit `return` is required:

``` javascript
const doubled = numbers.map(num => {
  return num * 2;
});
```

Or use an implicit return:

``` javascript
const doubled = numbers.map(num => num * 2);
```

**Remember:**

``` text
Arrow + {} → explicit return needed
Arrow without {} → expression returned automatically
```

------------------------------------------------------------------------

## 9. `map()`

`map()` transforms each item and returns a new array.

``` javascript
const numbers = [1, 2, 3];

const doubled = numbers.map(num => num * 2);

// [2, 4, 6]
```

A common React pattern:

``` javascript
const updatedUsers = users.map(user => {
  if (user.id === 2) {
    return {
      ...user,
      name: "William"
    };
  }

  return user;
});
```

------------------------------------------------------------------------

## 10. `filter()`

`filter()` returns a new array containing every item that passes a
condition.

``` javascript
const numbers = [1, 2, 3, 4, 5];

const result = numbers.filter(num => num > 2);

// [3, 4, 5]
```

It does not mutate the original array.

A common React deletion pattern:

``` javascript
setUsers(prev =>
  prev.filter(user => user.id !== 2)
);
```

------------------------------------------------------------------------

## 11. `find()`

`find()` returns the **first matching item**, not an array of every
match.

``` javascript
const numbers = [5, 10, 15, 20];

numbers.find(num => num > 10);
// 15
```

Compare:

``` javascript
numbers.filter(num => num > 10);
// [15, 20]
```

If `find()` finds nothing, it returns `undefined`.

**Remember:**

``` text
find()   → first matching item
filter() → array of all matching items
```

------------------------------------------------------------------------

## 12. Chaining Array Methods

``` javascript
const numbers = [1, 2, 3, 4, 5];

const result = numbers
  .filter(num => num > 2)
  .map(num => num * 2);

// [6, 8, 10]
```

Each method operates on the result returned by the previous method.

------------------------------------------------------------------------

## 13. `reduce()`

`reduce()` can reduce an array to a single accumulated value.

``` javascript
const numbers = [1, 2, 3, 4];

const result = numbers.reduce((total, num) => {
  return total + num;
}, 0);

// 10
```

Terminology:

-   `total` --- accumulator
-   `num` --- current array item
-   `0` --- initial accumulator value

Execution:

``` text
0 + 1 = 1
1 + 2 = 3
3 + 3 = 6
6 + 4 = 10
```

Practical example:

``` javascript
const cart = [
  { name: "Shirt", price: 20 },
  { name: "Shoes", price: 50 },
  { name: "Hat", price: 15 }
];

const total = cart.reduce((sum, item) => {
  return sum + item.price;
}, 0);

// 85
```

------------------------------------------------------------------------

## 14. Combining `filter()` and `reduce()`

``` javascript
const total = cart
  .filter(item => item.onSale)
  .reduce((sum, item) => {
    return sum + item.price;
  }, 0);
```

First `filter()` creates an array containing only sale items. Then
`reduce()` adds their prices.

------------------------------------------------------------------------

## 15. Immutable Array Updates in React

This mutates state:

``` javascript
users.push({ id: 3, name: "Sarah" });
setUsers(users);
```

Instead:

``` javascript
setUsers(prev => [
  ...prev,
  { id: 3, name: "Sarah" }
]);
```

The new array gives React a new reference.

------------------------------------------------------------------------

## 16. New Array Does Not Mean New Objects

`map()` creates a new array, but the objects inside it remain the same
objects unless you explicitly copy them.

Bad:

``` javascript
setUsers(prev =>
  prev.map(user => {
    if (user.id === 2) {
      user.name = "William";
    }

    return user;
  })
);
```

Better:

``` javascript
setUsers(prev =>
  prev.map(user => {
    if (user.id === 2) {
      return {
        ...user,
        name: "William"
      };
    }

    return user;
  })
);
```

------------------------------------------------------------------------

## 17. Updating Nested React State

Given:

``` javascript
{
  id: 1,
  name: "Randall",
  stats: {
    score: 10,
    level: 2
  }
}
```

To increase `score` without mutation:

``` javascript
return {
  ...user,
  stats: {
    ...user.stats,
    score: user.stats.score + 1
  }
};
```

Pattern:

``` javascript
{
  ...outerObject,
  nestedProperty: {
    ...outerObject.nestedProperty,
    thingToChange: newValue
  }
}
```

------------------------------------------------------------------------

## 18. Object Destructuring

``` javascript
const user = {
  name: "Randall",
  location: {
    city: "Portland",
    state: "Oregon"
  }
};

const {
  name,
  location: { city, state }
} = user;
```

Creates:

``` javascript
name  // "Randall"
city  // "Portland"
state // "Oregon"
```

It does **not** create a variable named `location`.

------------------------------------------------------------------------

## 19. Destructuring Default Values

``` javascript
const user = {
  name: "Randall"
};

const {
  name,
  age = 30
} = user;
```

Because `age` is `undefined`, it becomes `30`.

Defaults only apply to `undefined`.

``` text
undefined → default
null      → null
0         → 0
false     → false
```

------------------------------------------------------------------------

## 20. `||` vs `??`

``` javascript
const age = 0;

age || 30; // 30
age ?? 30; // 0
```

`||` falls back for any falsy value:

``` text
false
0
""
null
undefined
NaN
```

`??` falls back only for:

``` text
null
undefined
```

------------------------------------------------------------------------

## 21. Optional Chaining `?.`

``` javascript
const user = {
  settings: null
};

user.settings?.theme;
```

Instead of throwing an error when `settings` is `null`, optional
chaining returns `undefined`.

It combines nicely with `??`:

``` javascript
user.settings?.theme ?? "dark";
```

Result:

``` text
"dark"
```

------------------------------------------------------------------------

## 22. Scope and Variable Shadowing

``` javascript
let x = 10;

function test() {
  let x = 20;

  if (true) {
    let x = 30;
    console.log(x);
  }

  console.log(x);
}

test();
console.log(x);
```

Output:

``` text
30
20
10
```

Each inner `x` shadows the outer `x`.

------------------------------------------------------------------------

## 23. `var` vs `let` Scope

`var` is function-scoped.

`let` and `const` are block-scoped.

``` javascript
function test() {
  if (true) {
    var x = 10;
    let y = 20;
  }

  console.log(x); // 10
  console.log(y); // ReferenceError
}
```

------------------------------------------------------------------------

## 24. Hoisting with `var`

``` javascript
function test() {
  console.log(x);
  var x = 10;
  console.log(x);
}
```

Output:

``` text
undefined
10
```

A useful mental model:

``` javascript
function test() {
  var x;

  console.log(x);
  x = 10;
  console.log(x);
}
```

The declaration is hoisted, but the assignment is not.

------------------------------------------------------------------------

## 25. `let`, `const`, and the Temporal Dead Zone

``` javascript
function test() {
  console.log(x);
  let x = 10;
}
```

This throws:

``` text
ReferenceError: Cannot access 'x' before initialization
```

`let` and `const` declarations are inaccessible before their declaration
is reached. This period is called the **Temporal Dead Zone (TDZ)**.

------------------------------------------------------------------------

## 26. Function Declaration Hoisting

This works:

``` javascript
sayHello();

function sayHello() {
  console.log("Hello");
}
```

Function declarations are hoisted with their function bodies.

But:

``` javascript
sayHello();

const sayHello = () => {
  console.log("Hello");
};
```

throws a `ReferenceError` because `sayHello` is a `const` variable and
is in the TDZ before initialization.

------------------------------------------------------------------------

## 27. Closures Revisited

``` javascript
function outer() {
  let count = 0;

  function inner() {
    count++;
    return count;
  }

  return inner;
}

const counter = outer();

counter(); // 1
counter(); // 2
counter(); // 3
```

`inner` retains access to the `count` variable created when `outer()`
ran.

------------------------------------------------------------------------

## 28. `var`, Closures, and `setTimeout`

``` javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}
```

Output:

``` text
3
3
3
```

All callbacks reference the same function-scoped `i`. By the time they
execute, the loop has finished and `i` is `3`.

------------------------------------------------------------------------

## 29. `let` in a `for` Loop

``` javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}
```

Output:

``` text
0
1
2
```

With `let`, each loop iteration receives its own binding for `i`. Each
callback closes over its iteration's value.

The loop does **not** restart when the timeout fires.

------------------------------------------------------------------------

## 30. The Event Loop and `setTimeout`

``` javascript
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");
```

Output:

``` text
A
C
B
```

`setTimeout(..., 0)` does not mean "run immediately." The callback waits
until the current synchronous work is finished and the event loop can
process it.

------------------------------------------------------------------------

## 31. Promises vs `setTimeout`

``` javascript
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve().then(() => {
  console.log("C");
});

console.log("D");
```

Output:

``` text
A
D
C
B
```

Useful simplified ordering:

``` text
1. Synchronous code
2. Microtasks (Promise callbacks)
3. Tasks/macrotasks (such as setTimeout callbacks)
```

------------------------------------------------------------------------

## 32. Chained Promises

``` javascript
Promise.resolve()
  .then(() => {
    console.log("C");
  })
  .then(() => {
    console.log("D");
  });
```

`C` occurs before `D` because the second `.then()` waits for the first
`.then()` to complete.

For:

``` javascript
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("C");
  })
  .then(() => {
    console.log("D");
  });

console.log("E");
```

the output is:

``` text
A
E
C
D
B
```

------------------------------------------------------------------------

## 33. `async` / `await` and the Event Loop

``` javascript
console.log("A");

async function test() {
  console.log("B");

  await Promise.resolve();

  console.log("C");
}

test();

console.log("D");
```

Output:

``` text
A
B
D
C
```

An async function begins executing synchronously. When it reaches an
`await`, execution of the remainder of that function is deferred.

A useful mental model:

> An async function runs synchronously until it reaches an `await`. The
> continuation after the `await` is scheduled asynchronously as a
> microtask.

------------------------------------------------------------------------

# Quick Interview Cheat Sheet

  -----------------------------------------------------------------------
  Concept                             Remember
  ----------------------------------- -----------------------------------
  Object assignment                   Copies the reference, not the
                                      object

  `{ ...obj }`                        Shallow copy

  Nested update                       Spread every level being changed

  React state                         Never mutate existing state

  Functional updater                  Use `prev => ...` when next state
                                      depends on previous state

  `map()`                             Transform items; returns new array

  `filter()`                          All matching items; returns array

  `find()`                            First matching item

  `reduce()`                          Accumulates into a value

  `?.`                                Safely access possibly
                                      null/undefined values

  `??`                                Fallback only for null/undefined

  `||`                                Fallback for any falsy value

  `var`                               Function-scoped

  `let` / `const`                     Block-scoped

  `var` before assignment             `undefined`

  `let`/`const` before initialization ReferenceError / TDZ

  Function declaration                Hoisted with function body

  Closure                             Function retains access to
                                      surrounding scope

  `setTimeout(..., 0)`                Runs after current synchronous work

  Promise `.then()`                   Microtask; runs before timer
                                      callbacks

  `await`                             Pauses that async function;
                                      continuation becomes a microtask
  -----------------------------------------------------------------------

## Biggest Themes From This Session

The concepts that connect many of these questions are:

1.  **References vs values** --- especially objects, arrays, shallow
    copies, and React state.
2.  **Scope** --- understanding which variable a function is actually
    accessing.
3.  **Immutability in React** --- new arrays are not necessarily new
    nested objects.
4.  **State as a render snapshot** --- `setState` schedules a future
    render rather than changing the current render's variable.
5.  **Execution order** --- synchronous code first, then Promise/`await`
    microtasks, then timer callbacks.
6.  **Closures** --- functions can preserve access to variables long
    after the scope that created them has finished executing.
