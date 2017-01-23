###  按索引取值监测

因为数组属于对象类型，所以对于`arr[0]`形式取值与普通对象无异，都可以通过Object.defineProperty来对其进行处理，从而达到响应式的目的。

如下所示：
```js
var person = [{}]
            var p = []
            Object.defineProperty(person, '0', {
                get: function () {
                    return p[0];
                },
                set: function (val) {
                    p[0] = val;
                                        //directive.update();
                    console.log('0', person[0]);
                }
            })
            person[0] = {};
            person[0] = {a: 1}
            person[0].a = 2
            console.log('1', person[0])
```
实现了person数组第一个元素的响应式处理。

### 数组实例方法监测

数组的操作除了数组元素的取值赋值外，还需要考虑数组本身的更改。数组的下面这八个实例方法会改变数组本身，所以需要监测其更改操作。

```js
        'pop',
        'push',
        'reverse',
        'shift',
        'unshift',
        'splice',
        'sort'
```
我们来看如下代码（为vue初期源码）：
```js
var proto = Array.prototype,
    slice = proto.slice,
    mutatorMethods = [
        'pop',
        'push',
        'reverse',
        'shift',
        'unshift',
        'splice',
        'sort'
    ]

module.exports = function (arr, callback) {
    mutatorMethods.forEach(function (method) {
        arr[method] = function () {
            proto[method].apply(this, arguments)
            callback({
                event: method,
                args: slice.call(arguments),
                array: arr
            })
        }
    })
}
```

可以看到，这里是对需要监测得数组的这八个实例方法进行了重写，重写之后的方法除了进行原有的数组操作之外会执行一个回调函数，即只要发生了`pop,posh`等方法调用，回调函数就会调用，这就达到了检测的目的，将该回调函数和对应的指令更新操作绑定在一起即实现了数组的响应式。

### length属性不支持响应式

为什么说length属性不支持响应式，因为这个属性不能够重定义，即不能通过Object.defineProptery进行重写，所以无法实现对length属性的监测。

```js
                         var length = 0;
                         Object.defineProperty(person, 'length', {
                get: function () {
                    return length;
                },
                set: function (val) {
                    length = val;
                }
            })
```
这段代码会报类型错误：`Uncaught TypeError: Cannot redefine property: length`.