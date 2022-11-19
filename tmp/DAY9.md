# from

## 功能描述

表单。将组件内的用户输入的switch input checkbox slider radio picker提交。
当点击 form 表单中 form-type 为 submit 的 button 组件时，会将表单组件中的 value 值进行提交，需要在表单组件中加上 name 来作为 key。

## 示例

- index.wxml
```xml
<form bindsubmit="submitFn">
  用户名: <input type="text" name="username" style="margin-left: 70px;margin-top: -20px;"/>
  密码: <input type="text" password name="password" style="margin-left: 70px;margin-top: -20px;"/>
  <button type="primary" form-type="submit">submit</button>
</form>```

- index.js
```js
Page({
  submitFn(e){
      var name = e.detail.value.username
      var psw = e.detail.value.password

      if (name=='admin' && psw == '123'){
        wx.showToast({
          title: '登录成功',
          icon: 'success'
        })
      }else{
        wx.showToast({
          title: '用户名或密码错误',
          icon: 'error'
        })
	   }
  }
})
```

![[Pasted image 20221119224520.png]]

![[Pasted image 20221119224552.png]]