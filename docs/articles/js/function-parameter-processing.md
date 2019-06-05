# 函数参数是值传递还是引用传递

比较常见的一道面试题

### 入参为引用类型

先来看代码

```js
var a = { b: 2 }

function change (obj) {
  obj.b = 3
  return obj
}

var aa = change(a)

console.log(a) // { b: 3 }
console.log(aa) // { b: 3 }

aa = { b: 4 }

console.log(a) // { b: 3 }
console.log(aa) // { b: 4 }
```

下面来一波分析

首先 js 内存中，分为**栈内存**和**堆内存**，基本类型的数据将会存在栈内存中，而引用类型的数据会存在堆内存中，并在栈内存中存入一个指向堆内存中的地址。

```js
var a = { b: 2 }
```

执行后，内存中变化如下

![](https://i.loli.net/2019/06/05/5cf777adb4c5f75987.jpg)

继续

```js
function change (obj) {
  obj.b = 3
  return obj
}

var aa = change(a)
```

首先调用 change 的时候，内存中变化如下

![](https://i.loli.net/2019/06/05/5cf778193324c54414.jpg)

这里因为传入的参数是对象，所以这里其实是引用传递，函数内对 obj.b 进行了修改时，其实就是修改了 obj 所指向的堆内存中的值，并创建了 aa 变量，同样指向了该值，内存中变化为下

![](https://i.loli.net/2019/06/05/5cf7794702a2c78628.jpg)

所以此时，a 所指向的值其实也就被更改了，变量 a 和变量 aa 的值都为 { b: 3 }

```js
aa = { b: 4 }
```

这步操作时，是给变量 aa 进行了重新赋值，此时会在堆内存中，重新开辟一部分空间去存储新的值

![](https://i.loli.net/2019/06/05/5cf779f84b2f952022.jpg)

所以此时，变量 a 的值依然是 { b: 3 }，而变量 aa 的值已经变成了 { b: 4 }

### 入参为基本类型

当入参为基本类型时，其实操作的就是变量对应的值，此时的函数内部，就变为了值传递，这里比较简单，就不详细解释了

### 扩展

代码中的 change 中，改动一行

```js
  var a = { b: 2 }

  function change (obj) {
  obj = 4
  return obj
  }

  var aa = change(a)

  console.log(a) // { b: 2 }
  console.log(aa) // 4

  aa = { b: 4 }

  console.log(a) // { b: 2 }
  console.log(aa) // { b: 4 }
```
结果就发生了这样的变化，此时在执行了 change 函数时，其实内存是这样变化的

![](https://i.loli.net/2019/06/05/5cf77d207579923892.jpg)

下面代码执行后，obj 将不再是堆内存中值的引用了，而是被赋了一个新的值，所以这里返回给 aa 变量的值，也就是这个新的值，所以 aa 变量此时的值是 4，而变量 a 的值是没有发生变化的

```js
obj = 4
```