为什么要单独将v-if/v-for指令提出来单独说明，因为与v-text/v-show等指令不同，他们会改变DOM结构，改变DOM结构就会影响到vue的编译过程，同时会涉及到重新编译的过程。

以v-for为例，根据条件的不同，v-for指令会删除或是新增该DOM节点，如果是删除，那么会导致父节点的子元素的个数减少一个，那么久会影响父节点后续节点的编译，新增的过程会更麻烦，因为新增后的节点需要重新编译，为了不影响整个组件，所以需要单独进行编译处理。

### DOM结构更改问题

因为指令的预处理会更改节点的结构，所以会对其余节点的编译，所以需要自己了解哪些节点已编译，哪些节点是未编译的，不会少编译也不会重复编译。

如下为compilerNode函数的部分代码：
```js
function  compilerNode (el) {
          for (; el.index < el.childNodes.length; el.index++) {
                /**
                 * el.index 表示当前第几个子元素,在compileNode函数中可能会更改el的子元素结构，
                 * 所以需要el.index来标识编译的节点索引
                 */
                let child = el.childNodes[el.index];
                compileNode(child);
            }
}
```

如上为一个基本的DOM递归遍历，如果递归的过程中DOM结构没有发生变化，那么是不会有什么问题的，但是当遇到v-for这样的指令时，会对v-for节点进行编译处理，结果就是可能会当前节点元素增量的增加了，那么上述的遍历就会有问题，所以通过el.index来控制编译的流程，比如v-for指令增加了5个节点，那么el.index会加5，从而跳过新增的编译处理后的节点。

### 局部编译问题

为了提高编译的效率，当v-for相关的节点需要重新编译时不需要整体编译而进行局部编译，所以v-for得到的每一个子元素都会实例化一个viewModel对象，这样它们就可以进行单独编译而不影响整个组件。

这里有一个需要注意的点的，子元素实例化的vm对象应该与整个组件公用上下文（父子组件则是分别具有独立的上下文），即childVm可以访问parentVm的数据。

如下为v-for子实例的简单创建：

```js
let vm, node = this.el.cloneNode(true);

        this.parent.insertBefore(node, this.endRef);
        this.parent.index++;

        /**
         * array item data process
         */
        let data = {
            $index: index
        };
        data[this.subKey] = item;
        vm = new Vm({
            el: node,
            data: data
        }, this.vm);
        //父子实例数据共享
        vm.__proto__ = this.$vm;

        vm.appendTo(this.vm);
        this.childElements[index] = node;
        this.childVms[index] = vm;
```

这里巧用了原型链的功能，达到了父子实例公用上下文的目的。