# fed-e-task-01-01

## 一

异步编程是程序执行过程中获取结果的通信机制。JS是单线程同步执行的，好处是不用担心线程安全问题，坏处是一旦调用栈中出现耗时的任务，就造成阻塞，导致程序假死，为了避免这种情况，对于耗时长的任务可以使用异步处理。

当程序执行一段异步事件时，把异步事件放入到异步事件容器中，如果时间有延迟，等待延迟时间后放入消息队列中，如果没有延迟，里面放入消息队列中，EventLoop在调用栈事件都结束后，把从消息队列的头部把异步事件放入调用栈中执行。

宏任务是调用栈任务清空后，在消息列别等待执行的任务，微任务是调用栈中某个任务执行完后会把附带的任务入栈

## 二

```javascript
function say(text, time) {
  return new ***\**Promise\**\***((resolve => {
    setTimeout(function () {
      resolve(text)
    }, time);
  }));
}

say("hello ", 10)
  .then(value => {
    return say(value + "lagou", 10)
  })
  .then(value => {
    return say(value + "...", 10);
  })
  .then(***\**console\**\***.log);
```

## 三

```javascript
// 练习1
const fl = fp.flowRight( fp.prop("in_stock"), fp.last);
let isLastInStock = fl(cars);

// 练习2
const ff = fp.flowRight(fp.prop("name"), fp.first);
let name = ff(cars);

// 练习3
let _average = function (xs) {
  return fp.reduce(fp.add, 0, xs) / xs.length;
};
let averageDollarValue = fp.flowRight(_average, fp.pluck("dollar_value"));

// 练习4
let _underscore = fp.replace(/\W+/g,"_");
let sanititizeNames = fp.flowRight(_underscore, fp.toLower, fp.split(" "));
let sName = sanititizeNames(["Hello World"]);
```

## 四

```javascript
// 练习1
let ex1 = value => {
  return fp.map(function (res) {
    return fp.add(res, value);
  });
};

// 练习2
let ex2 = () => {
  return fp.first;
};

// 练习3
let ex3 = () => {
  return safeProp("name")(user).map(fp.first);
};

// 练习4
let ex4 = n => {
  return Maybe.**of**(n).map(parseInt)._value;
}
```

## 五

```javascript
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";


/**
 * 解析promise对象
 * @param self
 * @param result
 * @param resolve
 * @param reject
 */
const resolvePromise = function (self, result, resolve, reject) {
    // 返回结果是否为自身
    if (self === result) {
        return reject(new TypeError("Chaining cycle detected for promise #<Promise>"));
    }

    // 返回对象为promise，继续执行then，否则执行resolve方法
    if (result instanceof MyPromise) {
        result.then(resolve, reject);
    } else {
        resolve(result);
    }
}

class MyPromise {
    // 执行executor，入参为this.resolve,this.reject
    constructor(executor) {
        try {
            executor(this.resolve, this.reject);
        } catch (e) {
            this.reject(e);
        }
    }

    // 默认状态PENDING
    status = PENDING;
    value = undefined;
    reason = undefined;
    successCallbacks = [];
    rejectedCallbacks = [];
    /**
     * 更改状态，并保存value，如果有延迟执行的函数，则执行
     * @param value
     */
    resolve = value => {
        // 状态只能从PENDING->FULFILLED || PENDING->REJECTED
        if (this.status !== PENDING) return;
        this.status = FULFILLED;
        this.value = value;

        // 成功回调函数延迟到resolve执行
        while (this.successCallbacks.length) this.successCallbacks.shift()();
    }
    /**
     * 更改状态，并保存reason，如果有延迟执行的函数，则执行
     * @param reason
     */
    reject = reason => {
        // 状态只能从PENDING->FULFILLED || PENDING->REJECTED
        if (this.status !== PENDING) return;
        this.status = REJECTED;
        this.reason = reason;

        // 失败回调函数延迟到reject执行
        while (this.rejectedCallbacks.length) this.rejectedCallbacks.shift()();
    }

    /**
     * 根据状态，执行不同逻辑
     * @param successCallback
     * @param rejectedCallback
     */
    then(successCallback, rejectedCallback) {
        successCallback = successCallback ? successCallback : value => value;
        rejectedCallback = rejectedCallback ? rejectedCallback : reason => {
            throw reason
        };
        let myPromise = new MyPromise((resolve, reject) => {
            // 1、成功
            if (this.status === FULFILLED) {
                setTimeout(() => {
                    // 链式调用,返回一个新的promise，接受下一个promise的返回值
                    let result = successCallback(this.value);
                    resolvePromise(myPromise, result, resolve, reject);
                }, 0);
            } else if (this.status === REJECTED) {
                // 2、失败
                setTimeout(() => {
                    // 链式调用,返回一个新的promise，接受下一个promise的返回值
                    let result = rejectedCallback(this.reason);
                    resolvePromise(myPromise, result, resolve, reject);
                }, 0);
            } else {
                // 3、等待
                this.successCallbacks.push(() => {
                    setTimeout(() => {
                        // 链式调用,返回一个新的promise，接受下一个promise的返回值
                        let result = successCallback(this.value);
                        resolvePromise(myPromise, result, resolve, reject);
                    }, 0);
                });
                this.rejectedCallbacks.push(() => {
                    setTimeout(() => {
                        // 链式调用,返回一个新的promise，接受下一个promise的返回值
                        let result = rejectedCallback(this.reason);
                        resolvePromise(myPromise, result, resolve, reject);
                    }, 0);
                });
            }
        });
        return myPromise;
    }

    /**
     * 要么都成功拿到结果，只要有一个失败都拿不到结果
     * 返回promise对象
     * @param array
     * @returns {MyPromise}
     */
    static all(array) {
        let results = [];
        let index = 0;
        return new MyPromise((resolve, reject) => {
            // 加入结果集
            let addData = function (key, value) {
                results[key] = value;
                index++;
                // 判断是否都执行完
                if (index === array.length) {
                    resolve(results);
                }
            }
            for (let i = 0; i < array.length; i++) {
                let cur = array[i];
                // 判断是否为promise对象
                if (cur instanceof MyPromise) {
                    cur.then(value => addData(i, value), reject);
                } else {
                    addData(i, cur);
                }
            }
        })
    }

    /**
     * 根据入参判断是否是promise类型，
     * 如果是，直接返回，
     * 如果不是，创建一个新的promise对象返回
     * @param value
     * @returns {MyPromise}
     */
    static resolve(value) {
        if (value instanceof MyPromise) return value;

        return new MyPromise(resolve=>resolve(value));
    }
}

module.exports = MyPromise;
```