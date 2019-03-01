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

### 基本用法

- let [a,b,c] = [1,2,3]
- 模式匹配就能赋值let [a, [b,c]] = [1,[2,3]]
- let [a, ,c] = [1,2,3]
- let [a, ...b] = [1,2,3] // b = [2,3]
- 如果右边是不可遍历的（具有interator接口），就会报错
- generator函数原生具有interator接口，可以复制

### 默认值

- let [a, b = 2] = [1]
- 当值严格等于===undefined时，才会使用默认值
- let [a = 1] = [null] // a === 
- 默认值是惰性求值，需要用到的时候才会调用
- 可以用结构变量的其他值做默认值
  - let [a = 1, b = a] = [1]
  - let [a = b, b = 1] = [] 会报错，因为赋值a的时候，b还没声明；
  - 但是let [a = b, b = 1] = [1]不会报错，因为惰性求值，a不会使用b的值

### 对象的解构赋值

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

### 圆括号问题

- 变量申明不能使用圆括号
- 函数参数不能使用圆括号
- 赋值语句的模式不能使用圆括号
- 只有赋值语句的非模式部分可以使用圆括号 // [(b)] = [4]   ({p: (d)}) = {p: 3}

### 用途

- 变量交换  // var x = 1, y = 2; [x, y] = [y, x]
- 从函数返回多个值 let {foo, bar} = example()
- 函数参数的定义，函数参数的默认值
- 提取对象的值
- 遍历map结构
- 输入模块的指定方法  // import {cos, tan} from MyMoudle

## 字符串的扩展

- javascript允许采用\uxxxx表示一个字符; xxxx表示字符的unicode
  - let a = 1   //   \u0061 === 1
  - let b = '\u0061' // b === 'a'
- 使用 \u0000 - \uFFFF表示法表示，超出这个范围的要使用双字节
  - 比如 '\uD83D\uDE80' === '\u{1F680}'     "\uD842\uDFB7" === "𠮷"
  - 比如'\u00619'会打印'a9',只能识别上面范围内，后面的9直接加在后面
  - es6做了大括号处理 '\u{00619}' 或者 '\u{619}' 能够读取这个字符
- js有六种表示一个字符的方法(待研究)
  - 'z'   '\z'   '\172'   '\x7A'   '\u007A'   '\u{7A}'
- js使用utf-16的格式存储，就是固定两个字节；但是对于超出\u0000 - \uFFFF的字符，会被认为是两个字符
  - '𠮷'.length ==== 2
  - 's'.charAt(0) === 's',但是'𠮷'.charAt(0) 返回空; 使用charCodeAt(0)会返回 55362(\uD842)的十进制表示)
  - 同时s.charCodeAt(0)会返回NaN；charCodeAt()返回前两个字节跟后两个字节的值；对于单字节不适用
  - 新方法codePointAt(index) 会自动识别，然后返回字符的码点
  - 's𠮷'.codePointAt(0) // 115
  - 's𠮷'.codePointAt(1) // 55362
  - (115).toString(16)  返回16进制表示