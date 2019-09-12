## MV*设计模式的起源

起初**计算机科学家（现在的我们是小菜鸡）在设计GUI（图形用户界面）应用程序的时候，代码是杂乱无章的，通常难以管理和维护。GUI的设计结构一般包括视图**（View）、**模型**（Model）、**逻辑**（Application Logic、Business Logic以及Sync Logic），例如：

- 用户在**视图**（View）上的键盘、鼠标等行为执行**应用逻辑**（Application Logic），**应用逻辑**会触发**业务逻辑**（Business Logic），从而变更**模型**（Model）
- **模型**（Model）变更后需要**同步逻辑**（Sync Logic）将变化反馈到**视图**（View）上供用户感知

可以发现在GUI中**视图**和**模型**是天然可以进行分层的，杂乱无章的部分主要是**逻辑**。于是我们的程序员们不断的绞尽脑汁在想办法优化GUI设计的**逻辑**，然后就出现了MVC、MVP以及MVVM等设计模式。



## MVC（Model-View-Controller）

![MVC](https://user-gold-cdn.xitu.io/2019/5/13/16aae4f327f8e68c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- **View**：检测用户的键盘、鼠标等行为，传递调用**Controller**执行应用逻辑。**View**更新需要重新获取**Model**的数据。
- **Controller**：**View**和**Model**之间协作的应用逻辑或业务逻辑处理。
- **Model**：**Model**变更后，通过观察者模式通知**View**更新视图。

> **Model**的更新通过观察者模式，可以实现多视图共享同一个**Model**。



优点：

- 职责分离：模块化程度高、**Controller**可替换、可复用性、可扩展性强。
- 多视图更新：使用观察者模式可以做到单**Model**通知多视图实现数据更新。

缺点：

- 测试困难：**View**需要UI环境，因此依赖**View**的**Controller**测试相对比较困难（现在Web前端的很多测试框架都已经解决了该问题）。
- 依赖强烈：**View**强依赖**Model**(特定业务场景)，因此**View**无法组件化设计。



## MVP（Model-View-Presenter)

![MVP](https://user-gold-cdn.xitu.io/2019/5/13/16aae4f32c5e6933?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

MVP是MVC的模式的一种改良，打破了**View**对于**Model**的依赖，其余的依赖关系和MVC保持不变。

- **Passive View**：**View**不再处理同步逻辑，对**Presenter**提供接口调用。由于不再依赖**Model**，可以让**View**从特定的业务场景中抽离，完全可以做到组件化。
- **Presenter**（**Supervising Controller**）：和经典MVC的**Controller**相比，任务更加繁重，不仅要处理应用业务逻辑，还要处理同步逻辑(高层次复杂的UI操作)。
- **Model**：**Model**变更后，通过观察者模式通知**Presenter**，如果有视图更新，**Presenter**又可能调用**View**的接口更新视图。



MVP模式可能产生的优缺点如下：

- **Presenter**便于测试、**View**可组件化设计
- **Presenter**厚、维护困难



## MVVM（Model-View-ViewModel)

![MVVM](https://user-gold-cdn.xitu.io/2019/5/13/16aae4f32c4c587a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

MVVM模式是在MVP模式的基础上进行了改良，将**Presenter**改良成**ViewModel**（抽象视图）：

- **ViewModel**：内部集成了**Binder**(Data-binding Engine，数据绑定引擎)，在MVP中派发器**View**或**Model**的更新都需要通过**Presenter**手动设置，而**Binder**则会实现**View**和**Model**的双向绑定，从而实现**View**或**Model**的自动更新。
- **View**：可组件化，例如目前各种流行的UI组件框架，**View**的变化会通过**Binder**自动更新相应的**Model**。
- **Model**：**Model**的变化会被**Binder**监听(仍然是通过观察者模式)，一旦监听到变化，**Binder**就会自动实现视图的更新。



可以发现，MVVM在MVP的基础上带来了大量的好处，例如：

- 提升了可维护性，解决了MVP大量的手动同步的问题，提供双向绑定机制。
- 简化了测试，同步逻辑是交由**Binder**处理，**View**跟着**Model**同时变更，所以只需要保证**Model**的正确性，**View**就正确。

当然也带来了一些额外的问题：

- 产生性能问题，对于简单的应用会造成额外的性能消耗。
- 对于复杂的应用，视图状态较多，视图状态的维护成本增加，**ViewModel**构建和维护成本高。



对前端开发而言MVVM是非常好的一种设计模式。在浏览器中，路由层可以将控制权交由适当的**ViewModel**，后者又可以更新并响应持续的View，并且通过一些小修改MVVM模式可以很好的运行在服务器端，其中的原因就在于**Model**与**View**已经完全没有了依赖关系（通过View与Model的去耦合，可以允许短暂**View**与持续**View**的并存），这允许**View**经由给定的**ViewModel**进行渲染。


