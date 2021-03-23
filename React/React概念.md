

## React 为什么是 Immutale


## React hooks 的优点

### 粒度更细

- 从class的一个总的state状态，拆分为单独的一个个更细粒度的state，并使得“状态”与“变更状态的逻辑”两两配对，这样就能更加灵活的去控制。

### 从命令式变成声明式

对于某个功能，我们通常需要在挂载后，状态变更后，组件卸载前进行调用。如果用class的声明周期，那就分别需要在componentDidMount,componentDidUpdate,ComponentWillUnmount，这三处地方去写逻辑。相比起来，使用hook的话只需要在useEffect这一个地方做处理。会在效率上和可维护性上远高于class

### 数据和视图解耦

react class，其实只是UI框架，状态和视图耦合在一块，一切都是为视图而服务，没有独立的一层model。

很多本该由 service层，model层去做的事情，都放在了view层去做。导致view层在处理前端交互的同时，还要去处理业务逻辑，与接口通信。代码耦合性严重，不利于复用。

## Hooks 与 Class 的区别
hooks把对象这个概念瓦解了。

在class中，有构造函数一些数据初始化，有method方法和生命周期方法作为一个行为，数据和行为之间也能进行关联。

但实际上，数据——行为——关联是三个维度，他们可以不被捆绑在一个对象中。

在hooks里，数据可以单独声明（useState），行为也可以单独声明（useEffect），数据和行为可以进行可选的关联（custom-hooks）。并且数据可以单独组合，行为可以单独组合，组合的数据和组合的行为可以进行再度组合。组合的维度得到了横向和纵向的自由度扩展。

这个就是hooks与class的区别。