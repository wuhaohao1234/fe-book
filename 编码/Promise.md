## 最简单实现
```javascript
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class MyPromise {
  constructor(executor) {
    this.status = PENDING;
    this.value = undefined;

    const resolve = (value) => {
      if (!this.status === PENDING) {
        return;
      }
      this.status = FULFILLED;
      this.value = value;
    };

    const reject = (reason) => {
      if (!this.status === PENDING) {
        return;
      }
      this.status = REJECTED;
      this.value = reason;
    }

    try {
      executor(resolve, reject);
    } catch (ex) {
      reject(ex);
    }
  }

  then(onFulfilled, onRejected) {
    if (this.status === FULFILLED) {
      onFulfilled(this.value);
    }

    if (this.status === REJECTED) {
      onRejected(this.value);
    }
  }
}
```

## 支持异步
```javascript
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class MyPromise {
  constructor(executor) {
    this.state = PENDING;
    this.value = undefined;

    this.onFulfilledCallbacks = [];
    this.onRejectedCallbacks = [];

    const resolve = value => {
      if (this.status !== PENDING) {
        return;
      }
      this.status = FULFILLED;
      this.value = value;

      this.onFulfilledCallbacks.forEach(fn => fn());
    };

    const reject = reason => {
      if (this.status !== PENDING) {
        return;
      }
      this.status = REJECTED;
      this.value = reason;

      this.onRejectedCallbacks.forEach(fn => fn());
    };

    try {
      executor(reject, resolve);
    } catch (ex) {
      reject(ex);
    }
  }

  then(onFulfilled, onRejected) {
    if (this.status === FULFILLED) {
      onFulfilled(this.value);
    } else if (this.status === REJECTED) {
      onRejected(this.value);
    } else (this.status === PENDING) {
      this.onFulFilledCallbacks.push(() => onFulfilled(this.value));
      this.onRejectedCallbacks.push(() => onRejected(this.value));
    }
  }
}
```

## 支持链式调用
:::info
.then 应该是微任务，为了实现简单实用 setTimeout 宏任务

:::

```javascript
class MyPromise {
  constructor(executor) {
    // ...
  }

  then(onFulfilled, onRejected) {
    // 支持透传
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason; };

    return new MyPromise((resolve, reject) => {
      const handle = (callback, value) => {
        setTimeout(() => {
          try {
            const result = callback(value);
            if (result instanceof MyPromise) {
              result.then(resolve, reject);
            } else {
              resolve(result);
            }
          } catch (ex) {
            reject(ex);
          }
        });
      };

      if (this.status === FULFILLED) {
        setTimeout(() => {
          handle(onFulfilled, this.value);
        });
      } else if (this.status === REJECTED) {
        setTimeout(() => {
          handle(onRejected, this.value);
        });
      } else if (this.status === PENDING) {
        this.onFulfilledCallbacks.push(value => handle(onFulfilled, value));
        this.onRejectedCallbacks.push(reason => handle(onRejected, reason));
      }
    });
  }

  catch(onRejected) {
    return this.then(null, onRejected);
  }
}
```

## 支持静态函数
```javascript
class MyPromise {
  constructor(executor) {
    // ...
  }

  then(onFulfilled, onRejected) {
    // ...
  }

  catch(onRejected) {
    return this.then(null, onRejected);
  }

  static resolve(value) {
    return new MyPromise(resolve => resolve(value));
  }

  static reject(reason) {
    return new MyPromise((_, reject) => reject(reason));
  }

  static all(promises) {
    return new MyPromise((resolve, reject) => {
      let results = [];
      let completed = 0;

      for(let i = 0; i < promises.length; i++) {
        promises[i].then(value => {
          result[i] = value;
          completed++;
          if (completed === promises.length) {
            resolve(results)
          }
        }, reject);
      }
    });
  }

  static race(promises) {
    return new MyPromises((resolve, reject) => {
      for (let i = 0; i < promises.length; i++) {
        promises[i].then(resolve, reject);
      }
    });
  }
}
```

