####1.promise串行执行
```text
//创建方法是，不能传入Promise作为参数，因为在创建Promise的一刻它就开始pending等待处理，因此应该传入一系列的resolve
//等上一个 Promise resolved 之后再创建新的 Promise

function promiseQueue (executors) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(executors)) { executors = Array.from(executors) }
    if (executors.length <= 0) { return resolve([]) }

    var res = []
    executors = executors.map((x, i) => () => {
      var p = typeof x === 'function' ? new Promise(x) : Promise.resolve(x)
      p.then(response => {
        res[i] = response
        if (i === executors.length - 1) {
          resolve(res)
        } else {
          executors[i + 1]()
        }
      }, reject)
    })
    executors[0]()
  })
}

function alarm (msg, time) {
  return resolve => {
    setTimeout(() => {
      console.log(msg)
      resolve(msg)
    }, time)
  }
}

promiseQueue([alarm('1', 4000), alarm('2', 1000), 12, 'hellp', alarm('3', 3000)]).then(x => console.log(x))

```
####2.Promise实例的异步方法和then()中返回promise的区别
```text
// p1异步方法中返回p2
let p1 = new Promise ( (resolve, reject) => {
    resolve(p2)
} )
let p2 = new Promise ( ... )

// then()中返回promise
let p3 = new Promise ( (resolve, reject) => {
    resolve()
} )
let p4 = new Promise ( ... )
p3.then(
    () => return p4
)

```
p1的状态取决于p2，如果p2为pending，p1将等待p2状态的改变，p2的状态一旦改变，p1将会立即执行自己对应的回调，即then()中的方法针对的依然是p1

由于then()本身就会返回一个新的promise，所以后一个then()针对的永远是一个新的promise，但是像上面代码中我们自己手动返回p4，那么我们就可以在返回的promise中再次通过 resolve() 和 reject() 来改变状态
