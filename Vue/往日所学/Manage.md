## 一、网络请求

### 1.请求封装：

```
一、request：
 *    1. 说明：封装对后台的请求，可以选择自动处理一些异常。
 *    2. 参数：
 *        - url：            后台地址，必填，String，如："/user/add"
 *        - params：         请求参数，必填，Object，如：{"name":"xxx"}
 *        - config：         axios参数，选填，Object，默认值：{}
 *        - auto_error_res： 是否自动处理响应错误，选填，Boolean，默认值：true
 *        - auto_error_data：是否自动处理后台错误，选填，Boolean，默认值：true
 *    3. 返回：
 *        - 成功：Promise.resolve(请求成功后的结果：response.data.result)
 *        - 失败：
 *            - 请求异常：Promise.reject(http响应错误)
 *            - 请求失败：Promise.reject(请求失败后的结果：response.data.error)
 *    4. 约定后台返回数据格式：
 *        response.data = {
 *          "success": true/false,         //请求成功或失败
 *          "result": {},                  //请求成功后的结果
 *          "error":{
 *            "code": 100001,              //请求失败错误码
 *            "message": "用户名字重复"     //请求失败描述
 *          }
 *        }
 *
 * 二、sessionRequest：
 *    1. 说明：利用sessionStorage缓存请求，可以选择out_time，其他同request。
 *    2. 参数：
 *        - out_time：距离上次请求多少秒后需要重新请求，选填，Integer，小于0表示不重新请求，默认值：-1
 *
 * 三、localRequest：
 *    1. 说明：利用localStorage缓存请求，可以选择out_time，其他同request。
 *    2. 参数：
 *        - out_time：距离上次请求多少秒后需要重新请求，选填，Integer，小于0表示不重新请求，默认值：604800（一周）

```



#### 封装axios

- 封装request ，为axios请求，传入参数
  - 把参数中有关axios参数的，集合为一个对象，传入axios，并发送请求
    - 成功
      - 服务端请求失败，返回服务端错误信息
      - 服务端请求成功，返回数据
    - 失败
      - 打印网络错误信息，返回Promise.reject(error) 



#### 封装sessionStorage缓存

- 封装sessionRequest 为sessionStorage缓存的请求 ，传入同上的参数
  - 解析参数为JSON格式
  - 获取sessionStorage 参数，获取当前时间
    - 如果sessionStorage 存在数据，且时间不低于outtime，返回数据
    - 不存在数据，重新发一个request 请求，并把当前时间和返回的数据添加到sessionStorage



#### 封装localStorage 缓存

代码同上



### 2.后台接口

返回：调用request 请求，传入URL。

#### 登陆接口

```
  return request('/api/user/login', params).then(data => {
    localStorage.setItem('user-token', JSON.stringify(data.token))
    return data
  })
```

请求成功后，把token保存到本地localstorage

#### 注册接口

```
export const requestRegister = params => {
  return request('/api/user/register', params)
}
```

#### 退出登陆接口

```
export const requestLogout = params => {
  return request('/api/user/logout', params)
}
```

​	退出登陆，返回一个token 

#### 修改密码接口

```
export const requestChangePassword = params => {
  return request('/api/user/changePassword', params).then(data => {
    localStorage.setItem('user-token', JSON.stringify(data.token))
    return data
  })
}
```

​	请求成功后，把token保存到本地localstorage

#### 用户信息接口

```
export const requestUserInfo = params => {
  return request('/api/user/info', params).then((data) => {
    sessionStorage.setItem('user-info', JSON.stringify(data))
    return data
  })
}
```

把用户信息保存到sessionStorage 

#### 查询用户接口

```
export const requestUserQuery = params => {
  return request('/api/user/query', params)
}
```

返回30~60条用户数据

#### 权限表查询接口

```
export const requestPermissionsQuery = params => {
  return request('/api/user/permissions', params)
}
```

返回用户、角色、页面、指令

### 3.模拟数据

用Mock拦截接口的URL，返回约定格式的模拟数据

统一数据：

- success：true/false
- result：data
- error：错误码，可略



#### 登陆数据

```
    Mock.mock('/api/user/login', {
      'success': true,
      'result': {
        'token': 'fdsjfhjkdshfkldsajfjasdfbjsdkfhsdajfj'
      }
    })
```

#### 注册数据

```
Mock.mock('/api/user/register', {
      'success': true,
      'result': {}
    })
```

#### 退出登陆数据

```
export const requestLogout = params => {
  return request('/api/user/logout', params)
}
```

返回一个token

#### 修改密码数据

```
    Mock.mock('/api/user/changePassword', {
      'success': true,
      'result': {
        'token': 'fdsjfhjkdshfkldsajfjasdfbjsdkfhsdajfj'
      }
    })
```

#### 用户信息数据

| **属性名**  |             |         |       |             |
| ----------- | ----------- | ------- | ----- | ----------- |
| **success** | true /false |         |       |             |
| **result**  | id          | name    | roles | permissions |
| **error**   | code        | message |       |             |

- permissions 权限表
  - path  页面路径
    - permission  数据指令

permissions 里包含了页面路径和数据指令



#### 查询用户数据

- `data|30-60 `30~60条数据
  - 'name|1': 姓名
  - 'date|1' :日期
  - 'address|1': 地址

返回30~60条用户数据，每一条都有用户的姓名、日期、地址三项



#### **权限表数据**

| 属性               |      |      |                 |                      |
| ------------------ | ---- | ---- | --------------- | -------------------- |
| **users用户**      | id   | name | role_ids 角色id |                      |
| **roles角色**      | id   | name | page_ids 页面id | directive_ids 指令id |
| **pages 页面**     | id   | name | path 页面路径   |                      |
| **directive 指令** | id   | name | page_id 页面id  |                      |

一个用户可以有多个角色，角色里包含了页面和指令的权限



### **4.使用**

**导入对应接口**

```
import {requestLogin} from '../api/user'
```

**即可使用接口中的函数请求**



## **二、登陆与注册**

### **校验规则**

**用EL-form校验的方法**



在data外定义两个自定义校验规则，把它作为变量传入rules的字段中



**在data中定义一个`rules`属性作为规则，在其中可以设置账号或密码的校验，再将`rules`绑定到最外层form**

然后将Form-Item 的 `prop` 属性设置为需校验的`rules`中的字段名即可 

```
      rules: {
        oldPass: [
          {required: true, message: this.$t('login.inputOldPass'), trigger: 'blur'}
        ],
        newPass: [
          {validator: validatePass, trigger: 'blur'}
        ],
        checkPass: [
          {validator: validatePass2, trigger: 'blur'}
        ]
      }
```



### **登陆**

- **用El-Form搭建页面**
- **给账号和密码一个默认值**
- **输入账号密码时，双向数据绑定到数据中**
- **点击登陆按钮触发登陆事件**



#### **handleSubmit** 

- **用EL-form的validate 方法校验表单**
  - **判断返回值valid为true**
    - **调用登陆接口**
    - **获取账号密码参数，传入到登陆请求，然后导航到下个页面**
    - **登陆失败则打印错误信息**
  - **判断返回值valid为false**
    - **打印表单错误信息**



### **注册**



### 修改密码

- 用validate 表单验证
  - 校验成功
    - 调用el确认弹框`$confirm `，传入弹框的参数
    - 在回调函数中 ，调用修改密码接口，传入账号密码参数
    - 成功则`$message `消息提醒，失败则提醒失败
  - 校验失败





## **三、权限控制**

**权限策略：前端记录页面表，后台记录权限表，用户登录时后台返回用户权限，前端根据权限表显示页面。**



### **接口权限控制**

**使用jwt去进行验证**

登录时后台返回一个token，然后把它保存在localstorage中，以后前端每次调用接口都带上此token，服务器获得请求后比较token从而进行权限控制。 

- **拦截登陆请求，返回一个token**

  ```
      Mock.mock('/api/user/login', {
        'success': true,
        'result': {
          'token': 'fdsjfhjkdshfkldsajfjasdfbjsdkfhsdajfj'
        }
      })
  ```

- **在登陆请求的回调函数中，把token保存在localstorage**

  ```
  export const requestLogin = params => {
    return request('/api/user/login', params).then(data => {
      // 把token保存到localstorage
      localStorage.setItem('user-token', JSON.stringify(data.token))
      return data
    })
  }
  ```

- **设置axios请求拦截器**

- **每次请求的时候，取出localstorage中的token**

- **添加到请求头的Authorization 中**

  ```
  axios.interceptors.request.use(function (config) {
    // 取出localstorage中的token，添加在请求头的Authorization项
    config.headers.Authorization = localStorage.getItem('user-token')
    return config
  })
  ```



**在axios请求中判断token失效，则返回登陆页面**

```
 if (res.data.error.code === 100000) {
        Message({
          message: '登录失效，请重新登录',
          type: 'error'
        })
        window.location.href = '/#/login'
        return Promise.reject(res.data.error)
      }
```



### **页面权限控制**

**登录成功后，前端继续请求获取权限表并存储（考虑到现在后端常常使用微服务，因此把token的获取和权限表的获取拆开，而不是登录后一起返回）。**

**当用户要访问某页面时，判断一下该页面是否在权限表中，若不在，则跳转至401（用户无权限）页面。**

**备注：记录权限表时，还要将数据级的权限数据放置到路由表中，以便页面能够根据此权限数据进行渲染。**



- **设置一个flag为false**
- **遍历权限表**
- **判断访问路径和权限路径是否一致，是则flag为ture**
- **判断flag**
  - **为true，跳转到下个钩子**
  - **为false，跳转到错误页面401**



### **数据权限控制**

通过读取sessionStorage中用户的权限列表，利用`v-if`只显示有权限的菜单 

**router.match 可以根据请求的 path 和 method 筛选出匹配的 route**



- **遍历权限表**
- **通过router.match路径的对应的route**
- **判断数据表**
  - **存在，则保存到route的meta中**
  - **不存在，则保存一个空数组**



**获取用户信息的请求**

```
export const requestUserInfo = params => {
  return request('/api/user/info', params).then((data) => {
    sessionStorage.setItem('user-info', JSON.stringify(data))
    return data
  })
}
```



### **通过全局前置路由实现权限控制**

**设置一个全局前置守卫：**

- **如果是错误页面路由，则不需要判断权限，直接跳转**
- **如果访问路径是登陆 /注册，则删除localstorage和sessionstorage中的token和用户信息**
- **如果是白名单中的路径，直接跳转**
- **获取sessionStorage中的用户信息**
  - **不存在**
    - **发送获取用户信息的请求**
    - **从返回的数据中获取权限表**
    - **保存数据表到route-meta**
    - **判断用户是否有权限访问当前页面**
      - **失败，则打印错误信息**
  - **存在**
    - **获取权限表**
    - **判断用户是否有权限访问当前页面**



![img](https://linjinze999.github.io/assets/img/vue-llplatform/permission-process.png) 



## **四、页面布局**

### **layout 整体布局**

**两栏el-col ，上部分头顶，下部分主体**

**给三栏布局传入open-nav （true,false)属性，控制是否折叠**

**给header 传入方法，控制open-nav的值**



- **上部分**
  - **header 顶栏**
- **下部分**
  - **sidebar  左边侧边栏**
  - **section  右边主体**



### **header 顶部栏**

分为左边Logo，中间主体，右边用户。 

主体提供触发菜单栏展开收起的按钮，收起菜单的同时，左边Logo的文字也会隐藏起来，令Logo宽度和左侧折叠的菜单宽度一致。右边用户，当鼠标悬浮时可以操作退出登录（利用[Element 下拉菜单组件](http://element-cn.eleme.io/#/zh-CN/component/dropdown)）  

**$emit 传递给父组件，控制折叠**

从sessionStorage 中获取用户名 



### **sidebar  菜单栏**

通过`$router.options`就可以访问到当前路由数据 



- **最外层el-menu容器（ul）**
  - **遍历`$router.options.routes `路由表，v-if判断menu 属性为true则显示**
    - **单级菜单       //子路由长度等于1**
      - **取出一级菜单的图标和名字**
    - **多级菜单       //子路由长度大于1**
      - **取出一级菜单图标和名字**
      - **遍历二级路由**
        - **二级菜单**
          - **取出二级菜单名字     //子路由长度不存在则显示**
        - **多级菜单**
          - **取出二级菜单名字**
          - **遍历二级菜单子路由**
            - **取出三级菜单名字**



#### **路由key等级：**

| **等级**     | **key**                          |
| ------------ | -------------------------------- |
| **一级菜单** | **index1**                       |
| **二级菜单** | **index1+'-'+index2**            |
| **三级菜单** | **index1+'-'+index2+'-'+index3** |

#### **路由绑定路 径：**

| **等级**     | **index（path）**                    |
| ------------ | ------------------------------------ |
| **一级菜单** | **:index="level1.children[0].path"** |
| **二级菜单** | **:index="level2.path"**             |
| **三级菜单** | **:index="level3.path"**             |

#### **编程式导航**

```
menuSelect (index) {
   this.$router.push(index)
}
```



### **main 主体**

**设置最小高度，防止footer上来**

```
  const win_height = window.innerHeight - 100 + 'px'
    return {
      mainStyle: {
        minHeight: win_height
      }
```



## **五、功能页面**

### **图表**

#### **折线图**



chartData 

**||**

columns 

**['日期', '访问用户', '下单用户', '下单率'],**

第一个值为维度，其他的值为指标 

rows: 

 {'日期': '1/1', '访问用户': 1393, '下单用户': 1093, '下单率': 0.32},

 {'日期': '1/2', '访问用户': 3530, '下单用户': 3230, '下单率': 0.26},

**为具体的数值**



chartSettings 

**||**

 axisSite: {right: ['下单率']}, 

**// y轴类型，KMB：格式，percent：百分比**

 yAxisType: ['KMB', 'percent'],

 // 坐标轴名称

**yAxisName: ['数值', '比率']**



**把chartData 和chartSettings 转入图表组件中**

```
<ve-line :data="chartData" :settings="chartSettings"></ve-line>
```



#### **柱状图**

#### **仪表盘**



### **表单**

#### **form	表单**

每一个表单域由一个 Form-Item 组件构成，表单域中可以放置各种类型的表单控件 



#### **Upload 	上传**

**用JSONPlaceholder  模拟假数据**



#### **Transfer   穿梭框**



### **富文本编辑框**

**使用  vue-quill-editor**



### **表格**

`el-table`元素中注入`data`对象数组后，在`el-table-column`中用`prop`属性来对应对象中的键名即可填入数据，用`label`属性来定义表格的列名。可以使用`width`属性来定义列宽。 

`page-sizes`接受一个整型数组，数组元素为展示的选择每页显示个数的选项，`[100, 200, 300, 400]`表示四个选项，每页显示 100 个，200 个，300 个或者 400 个。 



### **拖拽**





## **六、权限配置**





### 初始化



```
 requestPermissionsQuery().then(data => {
      this.dbData = data.permissions
      this.dbDataToWebData()
    })
```

调用权限表接口，保存到“数据库”

```
     dbData: {
        users: [],
        roles: [],
        pages: [],
        directive: []
      }
```





#### 保存数据

**主数据**

| 名称tableAll |         |           |            |            |
| ------------ | ------- | --------- | ---------- | ---------- |
| 用户user     | user.id | user.name | roles      |            |
| 角色role     | role.id | role.name | permission |            |
| 页面page     | page.id | page.name | page.path  | directives |

| 数据名称 总体 |            |               |                   |                |
| ------------- | ---------- | ------------- | ----------------- | -------------- |
| user 用户     | query 查询 | dialog 对话框 | tableAll 所有数据 | table 列表数据 |
| role 角色     | query 查询 | dialog 对话框 | tableAll 所有数据 | table 列表数据 |
| selectOptions |            |               |                   |                |



- **selectOptions**  
  - roleOption  角色选择
    - role.name 
  - pageOption  页面选择
    - page.name 
  - pathOption  路径选择
    - page.path 
  - dirOption  指令选择
    - directive.name 
  - pageDirective  页面指令选择
    - page.name : [directive.name ...]

- **pages**
  - tableAll 
    - id: page.id 
    - name: page.name 
    - path: page.path 
    - directive: directives 
  - table 

- **roles** 
  - tableAll 
    - id: role.id 
    - name: role.name 
    - permission 
      - id:   page.id 
      - name:  page.name 
      - path:  page.path
      - directive: 
        - id: page.id 
        - name: page.name 
        - path: page.path 
        - directive: 
          - id: directive.id 
          - name: directive.name 
  - pageTotal 

- **users** 
  - tableAll 
    - id: user.id 
    - name: user.name 
    - roles: 
      - id: role.id 
      - name: role.name, 
      - pages: 
        - id: page.id 
        - name: page.name 
        - path: page.path 
        - directive: 
          - id: directive.id 
          - name: directive.name 
      - directive : 
        - id: directive.id 
        - name: directive.name 

**中间变量：**

- **rolesJson** 
  - role.id 
    - id: role.id 
    - name: role.name, 
    - pages: 
      - id: page.id 
      - name: page.name 
      - path: page.path 
      - directive: 
        - id: directive.id 
        - name: directive.name 
    - directive : 
      - id: directive.id 
      - name: directive.name 
- **pagesJson** 
  - page.id 
    - id: page.id 
    - name: page.name 
    - path: page.path 
    - directive: 
      - id: directive.id 
      - name: directive.name 
- **directiveJson** 
  - page_id 
    - id: directive.id 
    - name: directive.name 

directives = [directive.name ...]



#### 调用filter查询



### dialog对话框



#### 双向数据绑定

对应user的数据

```
        dialogShow: false,
        dialogType: 'add',
        dialogForm: {
          id: '',
          name: '',
          roles: []
        },
        dialogRules: {
          name: [{required: true, message: '请输入用户名字', trigger: 'blur'}]
        }
```



#### 添加数据

- 用validate 校验
  - 成功
    - 是add，就提示添加成功
    - 是其他，在提示修改成功
  - 失败
    - return false



### **1.用户模块**

#### **头部查询**

查询query,数据tableall

遍历tableall，判断query有没有包含前者的数据，有则添加到table

1.id

2.name

3.role





#### **用户列表添加**

 

1.清空各dialog的表单数据，并把dialog 的显示为true



#### **用户列表编辑**框

传入$indnex,和row



1.渲染旧数据到新的对话框

​	1.1根据传入的$index，选择table 中对应的数据

2.赋值给dialog 对话框对应的数据

​	2.1  dialogShow = true 



#### **用户列表删除**

**假消息**

**调用el消息弹窗**

```
        this.$message({
          message: '恭喜你，删除成功',
          type: 'success'
        });
```



#### **表格渲染**

user.table  中的数据

分割一页的数据，然后把数据传入表格的data中，通过solt-scoped 去插入每条数据



### **2.角色模块**

#### 角色**查询**

设置三个flag，初始为false，permission包含数据，则为true

- id
- name
- permission
  - page.name 
  - page.path 
  - directive 
    - page.name

遍历permission时，判断上面三个有没有查询的数据，有则flag为true，三个flag为true，就添加到table



#### **编辑**

**当前索引**

roleHandleEdit(scope.$index, scope.row) 



**赋值给对话框**



#### **对话框**

**校验，弹出假消息**



#### **表格渲染**

**权限项**

**遍历**

页面名称: 表格. 地址URI: /tables. 指令权限: modify; delete; 



### **3.页面模块**

#### 页面查询

