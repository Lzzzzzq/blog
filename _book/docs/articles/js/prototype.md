# __proto__ 和 prototype 的关系

经常有看到对于 proto 和 prototype 的总结，现整理一下

```javascript

function Foo () {}

let foo = new Foo()

// Foo 为 foo 的构造函数
foo.__proto__ === Foo.prototype
foo.constructor === Foo

// Function 为 Foo 的构造函数
Foo.__proto__ === Function.prototype
Foo.constructor === Function

// Function 由自己创建
Function.__proto__ === Function.prototype
Function.constructor === Function

// 所有构造函数的 prototype.__proto__ 都指向 Object.prototype
Function.prototype.__proto__ === Object.prototype
Foo.prototype.__proto__ === Object.prototype
Function.prototype.__proto__.constructor === Object

// Object.prototype 为所有函数的源头，它的 __proto__ 为 null
Object.prototype.__proto__ === null


let obj = {}

obj.__proto__ === Object.prototype
obj.__proto__.__proto__ === null
obj.__proto__.constructor === Object
obj.__proto__.constructor.__proto__ === Object.__proto__
obj.__proto__.constructor.__proto__.__proto__ === Object.prototype

```

