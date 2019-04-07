## 一、购物车

### 本地保存购物车数据

#### 1.定义数组

- 把localstorage中的数据添加到数组car中

  ```
  var car = JSON.parse(localStorage.getItem('car') || '[]')
  ```

- 定义数据

  ```
   state: { // this.$store.state.***
      car: car // 将 购物车中的商品的数据，用一个数组存储起来，在 car 数组中，存储一些商品的对象， 咱们可以暂时将这个商品对象，设计成这个样子   
      // { id:商品的id, count: 要购买的数量, price: 商品的单价，selected: false  }
    },
  ```

  

#### 2.保存到数组

​	用flag判断购物车中有无对应商品

- 如果购物车中已有商品，则增加数量即可

  ```
      addToCar(state, goodsinfo) {
        var flag = false
        state.car.some(item => {
          if (item.id == goodsinfo.id) {
            item.count += parseInt(goodsinfo.count)
            flag = true
            return true
          }
        })
  ```

- 如果没有，则push到购物车中

  ```
  if (!flag) {
          state.car.push(goodsinfo)
  }
  ```

- 保存到本地localstorage

  ```
  localStorage.setItem('car', JSON.stringify(state.car))
  ```

  

#### 3.调用

- 把商品信息保存为一个对象

  ```
        var goodsinfo = {
          id: this.id,
          count: this.selectedCount,
          price: this.goodsinfo.sell_price,
          selected: true
        };
  ```

- 调用store 的方法传送数据

  ```
  this.$store.commit("addToCar", goodsinfo);
  ```

  

### 自动更新购物车数量

1.在getters中定义获取数量的方法

```
 getAllCount(state) {
      var c = 0;
      state.car.forEach(item => {
        c += item.count
      })
      return c
    }
```

2.调用

```
{{ $store.getters.getAllCount }}
```



### 同步数量值

#### 1.初始化数量

- 父向子传值

  ```
  :initcount="$store.getters.getGoodsCount[item.id]"
  ```

- 接收

  ```
  props: ["initcount"]
  ```

- 赋值

  ```
  :value="initcount"
  ```

  

#### 2.同步

定义方法

```
    updateGoodsInfo(state, goodsinfo) {
      // 修改购物车中商品的数量值
      // 分析： 
      state.car.some(item => {
        if (item.id == goodsinfo.id) {
          item.count = parseInt(goodsinfo.count)
          return true
        }
      })
```

传送id

`:goodsid="item.id`

调用

  props: ["initcount", "goodsid"] 

```
    countChanged() {
      // 数量改变了
      // console.log(this.$refs.numbox.value);
      // 每当数量值改变，则立即把最新的数量同步到 购物车的  store 中，覆盖之前的数量值
      this.$store.commit("updateGoodsInfo", {
        id: this.goodsid,
        count: this.$refs.numbox.value
      });
    }
```



### 删除商品

1.store定义方法

```
    removeFormCar(state, id) {
      state.car.some((item, i) => {
        if (item.id == id) {
          state.car.splice(i, 1)
          return true;
        }
      })
    }
```

- 保存到本地

  ```
  localStorage.setItem('car', JSON.stringify(state.car))
  ```

  

2.调用

```
 <a href="#" @click.prevent="remove(item.id, i)">删除</a>
```

```
 remove(id, index) {
      // 点击删除，把商品从 store 中根据 传递的 Id 删除，同时，把 当前组件中的 goodslist 中，对应要删除的那个商品，使用 index 来删除
      this.goodslist.splice(index, 1);
      this.$store.commit("removeFormCar", id);
    },
```



## 二、返回按钮

```
    goBack() {
      // 点击后退
      this.$router.go(-1);
    }
```

