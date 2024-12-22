```javascript
function deepClone(obj) {
  // 如果是对象的值为 null，直接返回 null
  if (obj === null) return null;

  // 判断类型，如果是基本数据类型，直接返回
  if (typeof obj !== 'object') return obj;

  // 处理日期对象
  if (obj instanceof Date) return new Date(obj);

  // 处理正则表达式
  if (obj instanceof RegExp) return new RegExp(obj);

  // 处理数组
  if (Array.isArray(obj)) {
    const newArr = [];
    for (let item of obj) {
      newArr.push(deepClone(item)); // 递归复制每个元素
    }
    return newArr;
  }

  // 处理对象
  const newObj = {};
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      newObj[key] = deepClone(obj[key]); // 递归复制每个属性
    }
  }

  return newObj;
}

// 测试示例
const original = {
  name: 'John',
  age: 30,
  hobbies: ['running', 'reading'],
  details: {
    address: {
      city: 'New York',
      zip: 10001
    },
    dateOfBirth: new Date(1990, 1, 1) // 注意月份从0开始
  },
  regex: /[A-Z]/g
};

const copy = deepClone(original);

// 修改原对象，验证深拷贝是否成功
original.name = 'Jane';
original.hobbies.push('coding');
original.details.address.city = 'Los Angeles';

console.log('Original:', original);
console.log('Copy:', copy);

```

