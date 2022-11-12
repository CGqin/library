# for循环

- index.wxml
```xml
<view wx:for="{{array}}">
  {{index}}: {{item.message}}
</view>
```

- index.js
```js
Page({
  data: {
    array: [{
      message: 'Hello',
    }, {
      message: 'World'
    }]
  }
})
```