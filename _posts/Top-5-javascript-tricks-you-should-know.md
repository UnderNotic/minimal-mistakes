---
title: "Top 10 javascript tricks that can save You on job interview"
excerpt: "Make your javascript life easier"
header:
  teaser: "assets/images/markup-syntax-highlighting-teaser.jpg"
tags: 
  - javascript  
  - tricks
  - utility   
---


## \#1 Array.range
```javascript
[...Array(10).keys()]
```


## \#3 Array distinct

```javascript
new Set([ ...arr])
```

## \#4 Difference of two arrays
```javascript
arr1.filter(i => !arr.includes(i))
```

## \#5 Array Group by
```javascript
arr1.reduce((res, obj) => {
  res[obj.propertyForGroupBy] = res[obj.propertyForGroupBy] || [];
  res[obj.propertyForGroupBy].push(obj);
}, {})
```


## \#6 Object into keyvalue pair
```javascript
Object.entries(obj); => [["key1", "value1"], ["key2", "value2"]]
```

## \#6 Max \\ Min by
```javascript
arr.reduce((res, obj) => res.prop > obj.prop ? res : obj) //Max
arr.reduce((res, obj) => res.prop < obj.prop ? res : obj) //Min
```

## \#6 Intersection of two arrays
```javascript
array1.filter(value => -1 !== array2.indexOf(value));
```

## \#6 Merge objects
```
var obj1 = { };
var obj2 = {};
var merged1 = { ...obj1, ...obj2};
//or
var merged2 = Object.assign();
```

## \#6 Array functions on string
```javascript
 "hello".split("").map(_ => "x").join("")
```



## \#2 Iterate over object properties in declarative way
```javascript
Object.keys({}).map(key => Object[key] = "ss")
```

Keep in mind if your looking for top notch performance all above and much more is covered with optimization in lodash library.
Of course during interview process you won't be using that.
