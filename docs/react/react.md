> ### 为什么要做这个网站

* `学习到的知识点很快就忘没几天又要重新学习` 
* `百度搜索来的结果除了广告就是各种复制粘贴答案，真正可用资源很少` 
* `Google搜索 一般人又不会科学上网` 

私人谷歌的概念就是：获得一个知识点后把有用的信息整理出来，利用分类整理，变成自己的东西

建立自己的知识题库，从而查询分享，打造一个纯净的知识体系库

> ### React知识要点

* React默认：父组件更新子组件则无条件更新 如需性能优化
* 需要在shouldComponentUpdate函数中判断不需要更新的数值 如果没有变化则return false 则子组件不渲染
* shouldComponentUpdate不一定每次都要优化 需要的时候才去优化 如果影响不大 可以不用优化 仅仅是一个优化的钩子函数暴露 自行决定使用或者不使用

  
  

``` jsx
    shouldComponentUpdate(nextProps, nextState) {
        if(nextProps.text !== this.props.text) {
            return true; // 可以渲染
        }
        return false; // 不重复渲染
    }

    // 如果是判断引用类型 对象或者数组
    // const a = {a:1, b:2} const b = {a:1, b:2}
    // a === b  结果false 因为是指针指向不同所以false
    // 这时就用_.isEqual 做对象或者数组的深度比较
    // 可以参考 https://lodash.com/docs/#isEqual
    shouldComponentUpdate(nextProps, nextState) {
        // _.isEqual 做对象或者数组的深度比较(一次性递归到底 联想Vue的ObServer递归比较 不建议使用)
        // 所以在设计state的时候 数据结构不能嵌套的太深 否则深度比较太慢 性能很差
        // 那么有一个技术选择方案就是浅比较(尽量不要深比较) PureComponent(class组件) memo(函数组件)
        if(_.isEqual(nextProps.list, this.props.list)) {
            return true; // 可以渲染
        }
        return false; // 不重复渲染
    }
```

#### Event

**react中的合成event(SyntheticEvent 组合事件)，并非原生的mouseEvent， vue中的event是原生的并且触发时绑定在当前的元素; 判别原生的Event方式是event.nativeEvent. react中event.nativeEvent.currentTarget指向的是document 而 Vue中 event.nativeEvent.currentTarget 指向当前绑定的元素**

* SyntheticEvent 组合事件：模拟DOM元素event事件所有能力，是react封装的一套事件由冒泡机制冒泡到document然后通过SyntheticEvent来派发事件到绑定的target元素上面，而并非原生event事件，event.nativeEvent是原生事件对象

* 优势：更好的兼容性和跨平台（因为SyntheticEvent 组合事件），如果完全使用dom事件在移动端上可能需要重新写一遍事件绑定
* 优势：挂载到document，减少内存消耗，避免频繁解绑
* 优势：方便事件的统一管理（如事务机制）

  

#### setState中的不可变值

* `不可变值（函数式编程，纯函数）` 
* 纯函数的概念应该满足三个条件：1. 相同输入总是会返回相同的输出。2. 不产生副作用。 3. 不依赖于外部状态。

``` jsx
class Demo extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            value: null,
        };
    }

    render() {
        return ( 
           <p>{this.state.count}</p>
           <button onClick={this.increase}>+1</button>
        );
    }

    increase = () => {
        // 不可变值 - 赋值
`正确写法` 
        this.setState({
            count: this.state.count + 1; 
        });
        
`错误写法` 
        this.state.count++;
        this.setState({
            count: this.state.count
        });

`错误写法` 
        this.setState({
            count: this.state.count++; 
        });

        // ------ 我是分割线 ------
        // 不可变值 - 数组
        const list5Copy = this.state.list5.slice(); // 相当于list5被拷贝了一份
        list5Copy.splice(2, 0, 'a'); // 中间插入/删除
        this.setState({
            list1: this.state.list1.concat(100), // 追加
            list2: [...this.state.list2, 100], // 追加
            list3: this.state.list3.slice(0, 3), // 截取
            list4: this.state.list4.filter(item=>item > 100), // 筛选
            list5: list5Copy; // 其他操作
        });

        // 注意, 不能直接对 this.state.list进行 push pop splice等, 违反不可变值
        // 因为如果直接修改原数组 在shouldComponentUpdate做优化的时候 会出BUG state已经被改变则对比没有改变所以就返回false 组件不渲染
    
        // 不可变值 - 对象
        this.setState({
            obj1: Object.assign({}, this.state.obj1, {a: 100}),
            obj2: {...this.state.obj2, a: 100}
        });
        // 注意, 不能直接对 this.state.obj 进行属性设置，这样违反不可变值
    }
}
```

* `可能是异步更新` 

``` jsx

    // 可能是异步更新(有可能是同步更新)
    this.setState({
        count: this.state.count + 1
    }, () => {
        // 联想vue中的$nextTick
        console.log('callback: ', this.state.count);  // 回调函数中 可以拿到最新的值
    });
    console.log('count', this.state.count); // 异步渲染 拿不到最新的值

    // setTimeout 中 setState 是同步的
    setTimeout(()=>{
        this.setState({
            count: this.state.count + 1;
        });
        console.log('count in setTimeout: ', this.state.count);
    }, 0)

    // ------ 我是分割线 ------

    // 自定义DOM事件，setState是同步的
    bodyClickHandler = () => {
        this.setState({
            count: this.state.count + 1
        });
        console.log('count in body event: ', this.state.count);
    }

    componentDidMount() {
        document.body.addEventListener('click', this.bodyClickHandler);
    }

    // 联想 vue beforeDestroy 需要手动解绑自定义事件 自定义事件需要event.$off解绑
    componentWillUnmount() {
        document.body.removeEventListener('click', this.bodyClickHandler);
        // 如果有计时器 也需要clearTimeout清除
    }

```

* `可能会被合并` 

``` jsx
    // state异步更新的话，更新前会被合并
    // 传入对象，会被合并 (Object.assign)，执行结果只有一次 +1
    // 联想vue  viewmodel data在响应更新的时候 先压入栈 如果相同也就执行一次多余的被舍弃
    this.setState({
        count: this.state.count + 1
    });
    this.setState({
        count: this.state.count + 1
    });
    this.setState({
        count: this.state.count + 1
    });
    // 差不多这样 Object.assign({ count: 1},{ count: 1},{ count: 1}) => {count : 1}

    // 传入函数，不会被合并(因为函数是执行代码无法像对象一样合并成新值)， 执行结果是 +3
    this.setState((preState, props) => {
        return {
            count: preState.count + 1
        }
    });
    this.setState((preState, props) => {
        return {
            count: preState.count + 1
        }
    });
    this.setState((preState, props) => {
        return {
            count: preState.count + 1
        }
    });
```

* setState同步更新和异步更新的原理 通过捕获 isBatchingUpdates值 true是异步表明命中batchUpdate机制 false是同步

  

``` jsx
    // 自定义事件
    componentDidMount(){
        // 开始：处于batchupdate机制
        // react机制会默认执行 isBatchingUpdates = true，但是这条定义不存在于函数体中
        document.body.addeventListener('click', () => {
            // 此时 isBatchingUpdates 是 false
            this.setState({
                count: this.state.count + 1
            });
            console.log('count: ', this.state.count);
        });

        // 结束
        // 默认执行 isBatchingUpdates = false
    }

    increase = () => {
        // 开始：处于batchupdate机制
        // isBatchingUpdates = true，但是这条定义不存在于函数体中
        this.setState({
            count: this.state.count + 1
        });

        // 结束
        // 默认执行 isBatchingUpdates = false
    }

    increase = () => {
        // 开始：处于batchupdate机制
        // isBatchingUpdates = true，但是这条定义不存在于函数体中
        setTimeout(()=>{
            // 此时，isBatchingUpdates = false
            this.setState({
                count: this.state.count + 1
            });
        }, 0)

        // 结束
        // 默认执行 isBatchingUpdates = false
    }

```

#### 什么是纯函数

* 纯函数 仅输入props 输出jsx被称之为纯函数

#### 非受控组件 

* 定义 : state设默认值 后面取值和state无关
* refs : React.createRef() 创建的一个对象, 并通过 ref 属性附加到 React 元素
* defaultValue defaultChecked
* 手动操作DOM元素

``` jsx
    this.nameInputRef = React.createRef(); // 创建ref
    this.fileInputRef = React.createRef(); // 创建ref

    render() {
        return (
            <div>
                {/* 使用defaultValue而不是value，使用 ref */}
                <input defaultValue={this.state.name} ref={this.nameInputRef}>
                {/* state 并不会随着改变 */}
                <span>state.name: {this.state.name}</span>
                <br>
                <button onClick={this.alertName}>alert name</button>
            </div>
        )
    }

    alertName = () => {
        const elem = this.nameInputRef.current; // 通过ref 获取 DOM节点
        alert(elem.value);
    }
    alertFile = () => {
        const elem = this.fileInputRef.current; // 通过ref 获取 DOM节点
        alert(elem.files[0].name);
    }
```

#### createPortal

* 将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案
* 第一个参数（child）是任何可渲染的 React 子元素，例如一个元素，字符串或 fragment。第二个参数（container）是一个 DOM 元素。
* 通常为了解决position:fixed的时候的兼容问题
* 使用场景：fixed需要放在body第一层 UC浏览器兼容问题
* 使用场景：父组件 z-index 值太小
* 使用场景：overflow:hidden 逃离父组件的隐藏

  

``` jsx
    reander(
        ReactDOM.createPortal(<div className="model">{this.props.children}</div>, document.body);
    )
```

#### createContext

* 从组件树中离自身最近的那个匹配的 Provider 中读取到当前的 context 值

  

``` jsx
    const MyContext = React.createContext(defaultValue);

    // 提供者
    <MyContext.Provider value={this.state['自定义的state的值']}>
        <Demo />
    </MyContext.Provider>

    // ------ 分割线 消费者 ------

    // 如果是函数组件
    function Demo(props) {
        <MyContext.Consumer>
            {value => <p>{value}</p>}
        </MyContext.Consumer>
    }

    // 如果是class组件
    class Demo extends React.Component{
`static contextType = MyContext 也可以 在class外定义Demo.contextType = MyContext;` 
        render() {
            const value = this.context; // react 会往上查询最近匹配的Provider
            return (
                <div>{value}</div>
            )
        }
    }
`特别强调 也可以这样写 static contextType = MyContext` 
    Demo.contextType = MyContext;
```

#### React. Suspense 和 React.lazy

* 性能优化

``` jsx
    const ContextDemo = React.lazy( () => {import('./ContextDemo')} )
    render() {
        return (
            <div>
                <React.Suspense fallback={<div>loading...</div>}>
                    <ContextDemo />
                </React.Suspense>
            </div>
        )
    }
```

#### PureComponent 和 memo

* PureComponent中, shouldComponentUpdate中实现了浅比较
* memo, 函数组件中的PureCom
* _.isEqual 做对象或者数组的深度比较(一次性递归到底 联想Vue的ObServer递归比较 不建议使用)
* 所以在设计state的时候 数据结构不能嵌套的太深 否则深度比较太慢 性能很差
* 那么有一个技术选择方案就是浅比较(尽量不要深比较) PureComponent(class组件) memo(函数组件)

  

``` jsx
    // PureComponent 用法
    Class Demo extends React.PureComponent{

        render() {
             return(
                // jsx
            )
        }
       shouldComponentUpdate() {
           // 浅比较
       }
    }

    // React.memo 用法
    function Demo(props) {
        // 使用props渲染
    }

    function areEqual(prevProps, nextProps) {
        // 如果把 nextProps传入 render 方法的返回结果
        // 与
        // 将prevProps传入 render方法的返回结果一致
        // 则返回true否则返回false
    }

    export default React.memo(Demo, areEqual);
```

* 此方法仅作为性能优化存在。不要依靠它来“阻止”渲染，因为这可能会导致错误。

#### immutable.js 彻底拥抱 “不可变值”

* 彻底拥抱 “不可变值”
* 基于共享数据（不是深拷贝），速度快
* 有一定学习和迁移成本，按需使用

#### 公共逻辑抽离

* mixin, 已被React弃用
* 高阶组件 HOC
* Render Props

``` jsx
    // 高阶组件不是一种功能 而是一种模式
    const HOCFactory = (Component) => {
        Class HOC extends React.Component {
            // 再次定义多个组件的公共逻辑
            render(){
                return <Component {...this.props} /> // 返回拼装的结果
            }
        }
        return HOC
    }
    const EnhancedComponent1 = HOCFactory(WrappedComponent1);
    const EnhancedComponent2 = HOCFactory(WrappedComponent2);
```

#### batchUpdate机制

* 本质上setState不存在同步或者异步，实则是捕获isBatchingUpdates的布尔值来判定是否触发batchUpdate机制
* 生命周期（和它调用的函数）可以命中🎯batchUpdate机制
* React中注册的事件（和它调用的函数）可以命中🎯batchUpdate机制
* 综合来看就是 React可以“管理”的入口（即在react里注册的） 可以命中🎯batchUpdate机制

* setTimeout setInterval等（和它调用的函数）不可以命中batchUpdate机制
* 自定义的DOM事件（和它调用的函数）不可以命中batchUpdate机制
* React“管不到”的入口 不可以命中batchUpdate机制

#### transaction 事务机制

``` jsx
    increase = () => {
        // 开始：处于batchUpdate
        // isBatchingUpdates = true

        // 其他操作

        //结束
        // isBatchingUpdates = false
    }

    // ----------------------------下面这幅图是react中的流程源码注释----------------------------

    -                       wrappers (injected at creation time)
    -                                      +        +
    -                                      |        |
    -                    +-----------------|--------|--------------+
    -                    |                 v        |              |
    -                    |      +---------------+   |              |
    -                    |   +--|    wrapper1   |---|----+         |
    -                    |   |  +---------------+   v    |         |
    -                    |   |          +-------------+  |         |
    -                    |   |     +----|   wrapper2  |--------+   |
    -                    |   |     |    +-------------+  |     |   |
    -                    |   |     |                     |     |   |
    -                    |   v     v                     v     v   | wrapper
    -                    | +---+ +---+   +---------+   +---+ +---+ | invariants
    - perform(anyMethod) | |   | |   |   |         |   |   | |   | | maintained
    - +----------------->|-|---|-|---|-->|anyMethod|---|---|-|---|-|-------->
    -                    | |   | |   |   |         |   |   | |   | |
    -                    | |   | |   |   |         |   |   | |   | |
    -                    | |   | |   |   |         |   |   | |   | |
    -                    | +---+ +---+   +---------+   +---+ +---+ |
    -                    |  initialize                    close    |
    -                    +-----------------------------------------+

    * 

    - 这部分的流程图可以这样去理解 在注入和创建的时候会默认穿插initialize和close 然后中间会执行自定义函数
    - `（可参考https://github.com/zz1211/Doraemon/issues/9）` 
    - initialize -> 然后perform -> 然后close

```

> ### Vue相关知识要点

#### hash模式 Vue-Router的原理
