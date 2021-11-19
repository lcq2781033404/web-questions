### 1. 介绍
拍平数组即把一个多维数组转为一维数组，比如：[1, 2, [3, 4], [5, [6]]] => [1, 2, 3, 4, 5, 6]。这个功能可以直接用Array.prototype.flat方法来实现，当然这个功能也能手写。这里讲讲手写的实现方式。
### 2. 实现
这个功能的实现都是利用了递归，这里主要介绍两个实现方式，推荐第二种方法

#### 2.1 命令式写法
```javascript
function flat(array) {
  const res = [];
  for (let i = 0; i < array.length; i++) {
    // 数组空位处理
    if (!array[i]) {
      continue;
    }
    if (!Array.isArray(array[i])) {
      res.push(array[i]);
      continue;
    }
    flat(array[i]).map((item) => res.push(item));
  }
  return res;
}
```

#### 2.2 函数式编程写法
```javascript
function flat(array) {
  return array.reduce((total, cur) => {
    return total.concat(Array.isArray(cur) ? flat(cur) : cur);
  }, []);
}
```
