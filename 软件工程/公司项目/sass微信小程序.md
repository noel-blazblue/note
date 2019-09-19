



login

| 文件名      | 说明                            | 子组件                |
| ----------- | ------------------------------- | --------------------- |
| first_login | 首次登陆跳转的页面              |                       |
| login       | 登陆页面，密码登陆/动态密码登陆 | phone-login，mk-login |
| phone-login | 动态密码登陆                    | ask登陆请求           |
| mk-login    | 密码登录                        | ask登陆请求           |



utils

| 文件名      | 说明                         | 子组件                    |
| ----------- | ---------------------------- | ------------------------- |
| utils       | 获取日期                     |                           |
| index       | 请求服务器登录地址，请求方式 | requestLogin，setLoginUrl |
| global_data | 设置数据，获取数据           | set，get                  |
| utilss      | 密码登录                     | ask登陆请求               |



service

| 文件名 | 说明         | 子组件 |
| ------ | ------------ | ------ |
| api    | j接口地址    |        |
| ask    | 统一请求格式 | api    |
|        |              |        |
|        |              |        |



## 一、本地数据缓存

**指定一个key**

```javascript
let constants = require('./constants')
let SESSION_KEY = 'weapp_session_' + constants.WX_SESSION_MAGIC_ID;
```

**获取数据/设置数据/清除数据**

```javascript
let Session = {
    get: function () {
        return wx.getStorageSync(SESSION_KEY) || null;
    },

    set: function (session) {
        wx.setStorageSync(SESSION_KEY, session);
    },

    clear: function () {
        wx.removeStorageSync(SESSION_KEY);
    },
};
```



## 二、接口与请求

1.接口地址

```javascript
const host = 'https://sprog.makepolo.net'

export const LOGIN = `${host}/saas/weapp/login.php`
export const MK_LOGIN = `${host}/saas/login.php`
export const GET_CODE = `${host}/saas/get_code.php`

export const REGISTER = `${host}/saas/reg.php`
export const FORGET_PWD = `${host}/saas/forgot_passwd.php`
export const SET_NEW_PWD = `${host}/saas/set_new_passwd.php`
//服务超市
export const CPC_BUY = `${host}/saas/servuce_mall/cpc.php`
export const POINT_BUY = `${host}/saas/servuce_mall/purchase_score.php`
export const RE_BUY = `${host}/saas/servuce_mall/repay.php`
export const ORDER_LIST = `${host}/saas/servuce_mall/order_list.php`
```



2.请求封装

发送请求的时候实现登录控制，检查是否登录，否则跳转到第一次登录页面

```javascript
export default {
	api:function (url,data) {
		if (Session.get()) {
			return Taro.request({
				url:url,
				method:'POST',
				header:{'x-wx-skey':Session.get().skey},
				data:data,
				success:function (res) {
					if (res.data.state == '30001') {
						Taro.navigateTo({url:'/pages/first-login/first-login'})
					}
				}
			})
		} else {
			Taro.navigateTo({url:'/pages/first-login/first-login'})
		}
	}
}
```

然后通过 .then 获取返回的数据



## 三、文章列表

根据 page 获取文章列表，然后和旧列表合并，更新到state中

```javascript
	getArticalList () {
		let data = {page: this.state.page}
		let that = this
		let list = this.state.articalList
		api.api(ARTICAL_LIST,data).then(res => {
			if (res.data.state == 0 && res.data.data.result && res.data.data.result.length !== 0) {
				Taro.hideLoading()
				that.setState({ articalList: list.concat(res.data.data.result) })
			} else {
				Taro.showToast({ title:'没有更多了',icon:'none' })
			}
		})
	}
```



当页面滚动到底部的时候，显示加载中，然后 page + 1 ，调用 getArticalList 函数，实现更新列表

```javascript
	onReachBottom () {
		Taro.showLoading({title:'加载中...'})
		setTimeout(() => {
			this.setState({
				page: this.state.page + 1
			}, () => {
				this.getArticalList()
			})
		},500)
	}
```



