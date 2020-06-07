## instanceof 使用方法

```
function Car(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
}
const auto = new Car('Honda', 'Accord', 1998);

console.log(auto instanceof Car);
// expected output: true

console.log(auto instanceof Object);
// expected output: true

```

instanceof 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上。      

## instanceof 实现原理

### prototype

### 原型

### 原型链