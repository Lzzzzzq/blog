# Koa2 中的 Middleware

本文主要讲述 Koa2 中 Middleware ( 中间件 ) 的实现原理。

Middleware 是 Koa 中的而一个比较重要的核心概念，中间件就是类似于一个过滤器一样的东西，是在客户端和应用程序之间处理请求和响应的方法，中间件的执行顺序就像一个洋葱，也可以理解为 V 型，下图应该很常见。

![洋葱圈](http://upload-images.jianshu.io/upload_images/4262751-5b215ec2cb8fcb5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面通过一些代码展示一下这个洋葱圈的过程。

```javascript
function a(next) {
  console.log(1)
  next()
  console.log(2)
}
function b(next) {
  console.log(3)
  next()
  console.log(4)
}
function c(next) {
  console.log(5)
  next()
  console.log(6)
}
```

如上三个函数，这里我们把他们当成是中间件，先不看其中的 next 的话，依次调用这三个中间件，最后控制台输出的结果应该是：

```javascript
// 方法 a 中 第一段
1
// 方法 a 中 第二段
2
// 方法 b 中 第一段
3
// 方法 b 中 第二段
4
// 方法 c 中 第一段
5
// 方法 c 中 第二段
6
```

我们反过来将 next 加进去，再按照 Middleware 的方法来执行，最后的结果就会变成：

```javascript
// 方法 a 中 第一段
1
// 方法 b 中 第一段
3
// 方法 c 中 第一段
5
// 方法 c 中 第二段
6
// 方法 b 中 第二段
4
// 方法 a 中 第二段
2
```

就好像函数执行到 next 的时候就停住了，然后跳到下一个方法执行了，最后都执行完以后，再返回来执行了每个方法的后半段，我们来看一下源码。

```javascript
function compose(middleware) {
  return function(context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch(i) {
      if (i <= index)
        return Promise.reject(new Error('next() called multiple times'))
      index = i
      const fn = middleware[i] || next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(
          fn(context, function next() {
            return dispatch(i + 1)
          })
        )
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

我们假设所有的中间件都不存在异步操作，于是把 promise 都去掉，转换成同步的方式。

```js
function compose(middleware) {
  dispatch(0)
  function dispatch(i) {
    var fn = middleware[i]
    if (!fn) return
    fn(function() {
      return dispatch(i + 1)
    })
  }
}
compose([a, b, c])
```

这样就清爽多了，结合这段代码，我们再去看前面的 a，b，c 三个中间件，通过这个 compose，我们可以理解为，当程序执行到 next() 的时候，检查是否有下一段需要执行的代码，这样我们可以理解为，将这三个函数合成了一个函数

```js
function abc() {
  // 函数 a 的第一段
  console.log(1)

    // next();
    // 这里遇到了 next() ，执行下一个函数

    // 函数 b 的第一段
    console.log(3)

      // next();
      // 这里遇到了 next() ，执行下一个函数

      // 函数 c 的第一段
      console.log(5)

        // next();
        // 这里遇到了 next() ，但后面没有要执行的函数了

      // 函数 c 的第二段
      console.log(6)

    // 函数 b 的第二段
    console.log(4)

  // 函数 a 的第二段
  console.log(2)
}
```
