---
title: "Object Looping"
date: 2024-11-08
categories: [Objects, Functions, Methods]
keyterms: ["Objects", "Loops", "keys", "dot notation", "properties", "for in"]
---

# JavaScript Object Manipulation Guide

## 1. Object Declaration (const vs let)

When working with objects that will be used with MongoDB, it's generally recommended to use `const` unless you specifically need to reassign the variable to a completely different object. Here's why:

```javascript
// Using const
const person = { name: "John" };
person.name = "Jane"; // This is allowed
// person = { name: "Bob" }; // This would throw an error

// Using let
let user = { name: "John" };
user.name = "Jane"; // This is allowed
user = { name: "Bob" }; // This is also allowed
```

**Best Practice**: Use `const` by default because:

- It prevents accidental reassignment of the entire object
- The object's properties can still be modified
- It signals to other developers that this reference won't change
- MongoDB operations typically modify properties rather than replacing entire documents

## 2. Object.assign()

`Object.assign()` merges properties from one or more source objects into a target object.

```javascript
// Method 1: Modifying existing object
const car = { brand: "Toyota" };
Object.assign(car, { model: "Camry", year: 2024 });
// Result: car = { brand: "Toyota", model: "Camry", year: 2024 }

// Method 2: Creating new object
const newCar = Object.assign({}, car, { color: "red" });
// Result: newCar = { brand: "Toyota", model: "Camry", year: 2024, color: "red" }
```

**Use Case**: When updating MongoDB documents, you might use this to merge new fields with existing data:

```javascript
const updates = Object.assign({}, existingDoc, newData);
```

## 3. Spread Operator (...)

The spread operator provides a more concise way to clone and merge objects:

```javascript
const baseProduct = { name: "Widget", price: 9.99 };
const customProduct = {
  ...baseProduct,
  color: "blue",
  size: "large",
};
// Result: { name: "Widget", price: 9.99, color: "blue", size: "large" }
```

**Note**: Spread operator vs Object.assign()

- Spread is more readable and concise
- Both perform shallow copies
- Spread is commonly used in React and modern JavaScript

## 4. Object.defineProperty()

This method allows fine-grained control over property attributes:

```javascript
const product = {};

Object.defineProperty(product, "id", {
  value: "123",
  writable: false, // Can't be changed
  enumerable: true, // Shows up in loops
  configurable: false, // Can't be deleted
});

// Getter example
Object.defineProperty(product, "formattedPrice", {
  get() {
    return `$${this.price.toFixed(2)}`;
  },
});
```

`defineProperty` creates non-enumerable properties by default. To make them visible:

```javascript
Object.defineProperty(animal, "type", {
  value: "Dog",
  writable: true,
  enumerable: true, // Add this to make it visible
});
```

## 5. Looping Over Objects

There are several ways to iterate over objects:

- property name
- property value
- property and value

```javascript
const user = {
  name: "Alice",
  age: 30,
  role: "Admin",
};

// 1. for...in (iterates over enumerable properties)
for (let prop in user) {
  console.log(`${prop}: ${user[prop]}`);
}

// 2. Object.keys() with for...of
for (let key of Object.keys(user)) {
  console.log(`${key}: ${user[key]}`);
}

// 3. Object.entries() with destructuring
for (let [key, value] of Object.entries(user)) {
  console.log(`${key}: ${value}`);
}
```

**Key Differences**:

- `for...in` iterates over all enumerable properties (including inherited ones)
- `for...of` is used with iterables (arrays, strings, etc.)
- `forEach` is an array method that provides index and array reference

## 6. Object.entries(), Object.keys(), and Object.values()

These methods are crucial for MongoDB operations:

```javascript
const product = {
  name: "Laptop",
  price: 999,
  inStock: true,
};

// Keys
console.log(Object.keys(product));
// ["name", "price", "inStock"]

// Values
console.log(Object.values(product));
// ["Laptop", 999, true]

// Entries
console.log(Object.entries(product));
// [["name", "Laptop"], ["price", 999], ["inStock", true]]
```

**MongoDB Use Case**:

```javascript
// Converting MongoDB document fields to an update operation
const updates = Object.entries(changes).reduce((acc, [key, value]) => {
  acc[`$set.${key}`] = value;
  return acc;
}, {});
```

## 7. Object.freeze()

Prevents any modifications to an object:

```javascript
const config = {
  apiKey: "abc123",
  endpoint: "api.example.com",
};

Object.freeze(config);
config.apiKey = "xyz"; // This will fail silently or throw error in strict mode
delete config.endpoint; // This will also fail
```

**Note**: `Object.freeze()` is shallow - nested objects can still be modified unless also frozen.

The reason we can't loop over objects directly is that objects are not iterable by default in JavaScript. They need to be converted to an iterable format first (using Object.keys(), Object.values(), or Object.entries()).

Let's clarify the difference between `for...in` and `for...of` with array indices.

When using `for...in` with arrays, it iterates over the enumerable properties, which include the array indices as strings. This can lead to unexpected behavior:

```javascript
const arr = ["a", "b", "c"];

// Using for...in with array
for (let index in arr) {
  console.log(index, typeof index);
  // Output:
  // "0" "string"
  // "1" "string"
  // "2" "string"
}

// Using for...of with array
for (let value of arr) {
  console.log(value);
  // Output:
  // "a"
  // "b"
  // "c"
}
```

This is one of the key reasons why `for...in` is not recommended for arrays. Here's a breakdown of when to use each:

```javascript
// 1. For Arrays:
const numbers = [10, 20, 30];

// ✅ GOOD: for...of gets values
for (let num of numbers) {
  console.log(num); // 10, 20, 30
}

// ✅ GOOD: forEach gets value, index, and array
numbers.forEach((value, index, array) => {
  console.log(value, index); // 10 0, 20 1, 30 2
});

// ❌ BAD: for...in gets string indices
for (let i in numbers) {
  console.log(i); // "0", "1", "2"
}

// 2. For Objects:
const person = {
  name: "John",
  age: 30,
};

// ✅ GOOD: for...in gets property names
for (let prop in person) {
  console.log(prop, person[prop]); // "name" "John", "age" 30
}

// ✅ GOOD: Object.entries() with for...of
for (let [key, value] of Object.entries(person)) {
  console.log(key, value); // "name" "John", "age" 30
}

// ❌ ERROR: for...of doesn't work directly with objects
// for (let item of person) { // TypeError: person is not iterable
```

The key differences:

1. `for...in`:

   - Works with objects to iterate over enumerable property names
   - With arrays, gives string indices ("0", "1", "2")
   - Can include inherited properties
   - Generally slower than alternatives

2. `for...of`:

   - Works with iterables (arrays, strings, Maps, Sets)
   - Gives actual values
   - Cannot be used directly with objects
   - Generally faster and more predictable

3. `forEach`:
   - Array method only
   - Provides value, index, and array reference
   - Cannot break out of the loop early
   - More functional programming style

Best practices:

```javascript
// For arrays:
const arr = ["a", "b", "c"];
for (let value of arr) {
  /* use value */
}
// or
arr.forEach((value) => {
  /* use value */
});

// For objects:
const obj = { a: 1, b: 2 };
for (let key in obj) {
  if (obj.hasOwnProperty(key)) {
    // Safety check
    /* use obj[key] */
  }
}
// or (modern approach)
for (let [key, value] of Object.entries(obj)) {
  /* use key and value */
}
```

This is particularly relevant when working with MongoDB, as you often need to iterate over document fields:

```javascript
const doc = await collection.findOne({ _id: someId });

// To process all fields in a document:
for (let [field, value] of Object.entries(doc)) {
  // Skip the _id field
  if (field !== "_id") {
    // Process each field
    console.log(`Field ${field} has value ${value}`);
  }
}
```
