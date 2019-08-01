## React.createElement

![img](https://user-gold-cdn.xitu.io/2019/4/23/16a48bfd967b7492?imageslim)

- 是否存在config 
  - 是：对其中的ref，key进行验证，把内建的几个属性剔除后丢到 props 中

- 处理children，判断长度

  - 等于1：是对象，赋值给props
  - 大于1：把他转换为数组，赋值给props

- 判断是否有给组件设置defaultProps，有的话判断是否有给props赋值，没有则设置为默认值

- 把所有数据整合为对象，返回ReactElement

  ```JavaScript
    return ReactElement(
      type,
      key,
      ref,
      self,
      source,
      ReactCurrentOwner.current,
      props,
    );
  ```

**核心**就是通过 `$$typeof` 来帮助我们识别这是一个 `ReactElement`