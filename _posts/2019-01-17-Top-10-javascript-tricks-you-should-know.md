---
title: "Top 10 javascript tricks that can save You on job interview"
excerpt: "Good to know javascript snippets"
header:
  teaser: "assets/images/markup-syntax-highlighting-teaser.jpg"
tags: 
  - javascript  
  - interview
  - tricks
  - utility   
---

## \#1 Array.range
```javascript
[...Array(10).keys()];
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

## \#2 Array distinct
```javascript
let arr = [1, 2, 2, 3];
new Set([ ...arr]);
// [1, 2, 3]
```

## \#3 Difference of two arrays
```javascript
let arr1 = [1, 2, 3];
let arr2 = [2, 3, 4];
arr1.filter(i => !arr2.includes(i))
// [1]
```

## \#4 Intersection of two arrays
```javascript
let arr1 = [1, 2, 3];
let arr2 = [2, 3, 4];
array1.filter(value => array2.indexOf(value) !== -1);
// [2, 3]
```

## \#5 Array Group by
```javascript
let arr = [
  { name: "Peter", city: "New York" },
  { name: "Jane", city: "Berlin" },
  { name: "John", city: "Berlin" }
];

arr.reduce((res, obj) => {
  res[obj.city] = res[obj.city] || [];
  res[obj.city].push(obj);
  return res;
}, {})

// {
//   "New York": [{ name: "Peter", city: "New York" }],
//   "Berlin":   [{ name: "Jane", city: "Berlin" }, { name: "John", city: "Berlin" }]
// }
```

## \#6 Object into keyvalue pair
```javascript
let obj = {
  key1: "values1",
  key2: "values2"
};
Object.entries(obj);
//[["key1", "value1"], ["key2", "value2"]]
```

## \#7 Max \\ Min by
```javascript
let arr = [{ prop: 1}, { prop: 3}, { prop: 2}]

arr.reduce((res, obj) => res.prop > obj.prop ? res : obj)
// 3
arr.reduce((res, obj) => res.prop < obj.prop ? res : obj)
// 1
```

## \#8 Merge objects
```javascript
var obj1 = { prop1: "prop1" };
var obj2 = { prop2: "prop2" };
var merged1 = { ...obj1, ...obj2};
// or
var merged2 = Object.assign(obj1, obj2);

// { prop1: "prop1", prop2: "prop2" }
```

## \#9 Array functions on string
```javascript
 "hello".split("").map(_ => "x").join("");
 // "xxxxx"
```

## \#10 Iterate over object properties in declarative way
```javascript
let obj = { prop: "prop" };
Object.keys(obj).map((key, i) => obj[key] + i);
```

## ...
Be aware that if you'r looking for top notch performance all above and much more is covered with optimization in mind in lodash library.
