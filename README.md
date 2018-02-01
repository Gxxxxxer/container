1.promise串行执行
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
