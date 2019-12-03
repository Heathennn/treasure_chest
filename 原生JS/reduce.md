#### 数组reduce的使用
```

let arr = [1, 2, 3];
// 伪代码
let 最终计算完的result = arr.reduce( (会被重复利用的result, 数组中的当前值item) => { return 每次计算后的result }, result的初始值)

```