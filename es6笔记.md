# es6笔记、

## let和const命令

- 在块级作用域中申明函数，相当于let，只能在块级作用域中访问；为了兼容老版本，会报错
  - 建议使用let 或者 var申明函数
- const也有作用域，存在按时性死区；不可重复申明
  - 只是地址不可更改，对于引用型数据还是可以修改里面的属性的
  - 可以用object.freeze(obj)冻结对象，不能修改；但是如果属性是引用类型，也还是可以修改的；建议属性如果是对象也一起冻结
- 申明方式一共六种：var、function、let、const、import、class
- new Function('return this')()总是返回全局对象
- 浏览器里面，this返回window顶层对象；web worker里面this跟self都会返回window顶层对象；node里面global返回顶层对象
- require是commonJS的写法；import是es6模块的写法

## 变量的结构赋值

- 基本用法
  - let [a,b,c] = [1,2,3]
  - 模式匹配就能赋值let [a, [b,c]] = [1,[2,3]]
  - let [a, ,c] = [1,2,3]
  - let [a, ...b] = [1,2,3] // b = [2,3]
  - 如果右边是不可遍历的（具有interator接口），就会报错
  - generator函数原生具有interator接口，可以复制
- 默认值
  - let [a, b = 2] = [1]
  - 当值严格等于===undefined时，才会使用默认值
  - let [a = 1] = [null] // a === 
  - 默认值是惰性求值，需要用到的时候才会调用
  - 可以用结构变量的其他值做默认值
    - let [a = 1, b = a] = [1]
    - let [a = b, b = 1] = [] 会报错，因为赋值a的时候，b还没声明；
    - 但是let [a = b, b = 1] = [1]不会报错，因为惰性求值，a不会使用b的值
- 对象的解构赋值
  - let {a, b} = {a: 1, b: 2}  跟属性名对象，不需要按顺序
  - 鉴于对象的缩写，等价于 let {a: a, b: b} = {a: 1, b: 2}
  - 属性名不一样的 let {a: c} = {a: 1, b: 2}  // c === 1  （a是匹配模式，c是匹配变量）
  - 结构嵌套,主要是明白匹配模式跟匹配变量
  - 可以有 let {a, a: c} = {a: 1}
  - 同理有 let {p, p:{a, a: c}} = {p: {a: 1}} // p === {a: 1}    a === 1  c === 1
  - let {a: obj.attr, b: arr[0]} = {a: 1, b: 2}也能匹配
  - let {a = 3, b: y = 4} = {a: 1} 对象结构赋值的默认值,也是要严格等于undefined
  - let x; {x} = {x: 1}会报错，因为会把{x}理解为代码块；可以使用小括号包裹起来({x} = {x: 1})
  - 左边可以不放置变量名，可执行不报错 ({} = [])；要使用小括号
  - function move({x = 0, y = 0} = {}) {return [x, y];}
    - 参考function fun(x = 3) {return x} 可知，只有在x为undefined的时候，才使用默认值x = 3
    - 只有move传参undefined的时候，才是用默认值{};
    - 所move({x: 3}) => [3,0]   move({}) => [0, 0]
    - 在传递undefined时候，使用默认值{};这时候因为{}是空对象，所以使用x/y的默认值0，0
  - function move({x, y} = { x: 0, y: 0 }) {return [x, y];}
    - 由上可知，只有传递的参数为undefined的时候，才是用默认值；所以传递{x: 3}的时候，不使用默认值；同时又因为y没有默认值，所以y的值是undefined
    - 所以move({x: 3}) => [3, undefined]
    - move({}) => [undefined, unndefined]
    - move() => [0, 0]