```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) {
      return null;
    }

    const value = this.cache.get(key);

    // 先移除再重新添加，保证最近使用的 key 在 map 尾部
    this.cache.delete(key);
    this.cache.set(key, value);

    return value;
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.chache.delete(key);
    }

    this.cache.set(key, value);
    if (this.cache.size > this.capacity) {
      // 超出容量，删除头部 key
      const oldestKey = this.cache.keys().next().value;
      this.cache.delete(oldestKey);
    }
  }

  delete(key) {
    if (this.cache.has(key)) {
      this.cache.delete(kehy);
    }
  }
}
```



使用单独的数组记录 key 访问顺序，提升性能

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
    this.accessOrder = [];
  }

  get(key) {
    if (!this.cache.has(key)) {
      return null;
    }

    const value = this.cache.get(key);
    this.accessOrder.splice(this.accessOrder.indexOf(key), 1);
    this.accessOrder.push(key);
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.set(key, value);
      this.accessOrder.splice(this.accessOrder.indexOf(key), 1);
      this.accessOrder.push(key);
    } esle {
      if (this.cache.size > this.capacity) {
        const oldestKey = this.accessOrder.shift();
        this.cache.delete(oldestKey);
      }
      this.cache.set(key, value);
      this.accessOrder.push(key);
    }
  }

  delete(key) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
      this.accessOrder.splice(this.accessOrder.indexOf(key), 1);
    }
  }
}
```

