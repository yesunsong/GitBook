# 对number类型的数据进行保留小数操作


 ts数据的类型是一个any类型，它有一个number类型的属性，当我们对这个字段进行toFixed()操作时会有这样一个错误：

core.js:1427 ERROR TypeError: this.actualViscosity.value.toFixed is not a function

 解决办法很简单，借助 Number 类型就可以解决这个问题就像这样：

```ts
this.inkTemperature.value = Number(this.inkTemperature.value).toFixed(2);
```

