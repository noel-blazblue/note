## 一、评论功能



### 传送ID

1.父组件向子组件传值

```
<comment-box :id="this.id"></comment-box>
```

2.子组件接收

```
props: ["id"]
```

即可通过this.id 访问



### 接收评论

1.定义一个变量接收评论

```
  comments: [], // 所有的评论数据
```

2.在文本输入区域进行双向数据绑定

```
v-model="msg"
```



### 定义发送评论方法

1.校验是否为空

```
      if (this.msg.trim().length === 0) {
        return Toast("评论内容不能为空！");
      }
```

2.发送ajax请求

```
this.$http
        .post("api/postcomment/" + this.$route.params.id, {
          content: this.msg.trim()
        })
```

3.在回调中，把评论拼接成一个对象，保存到msg中

```
.then(function(result) {
          if (result.body.status === 0) {
            // 1. 拼接出一个评论对象
            var cmt = {
              user_name: "匿名用户",
              add_time: Date.now(),
              content: this.msg.trim()
            };
            this.comments.unshift(cmt);
            this.msg = "";
          }
        });
```

### 调用

在创建完成期间调用

```
  created() {
    this.getComments();
  },
```



二、