#  自定义tabbar解决方案

<a name="k10lhw"></a>
### [](#k10lhw)背景
用于微信小程序自定义tabbar开发，以解决原生tabbar样式无法高度定制化的情况

<a name="agk4xa"></a>
### [](#agk4xa)需求

- 使用 wx.switchTab 方式进行跳转

- 解决网上常见的解决方案中闪屏的情况（使用wx.navigateTo进行跳转）

- 可自定义样式


<a name="6hs7ts"></a>
### [](#6hs7ts)实现方案及思路

<a name="na6eqx"></a>
#### [](#na6eqx)方案A
实现方式：使用单页面的方式，通过底部的tabbar点击来控制各个视图隐藏与显示的方式来实现<br />实施时遇到的问题：A页面需要小程序原生的下拉刷新功能，B页面不需要下拉刷新功能，由于小程序单独页面的下拉刷新功能需写死在 page 的 json 文件里，无法通过设置动态开关，导致该需求无法很好的实现，后想到了一个弥补措施，即开启页面的下拉刷新功能，在可以下拉刷新的A页面，将原生加载区域的三个点的样式动态设置为与背景颜色相反的颜色，下拉后去请求数据，在不需要下拉刷新功能的B页面，动态修改三个点的样式为与背景相同的颜色，下拉后直接执行下拉的 stop 方法，这样从视觉上看，是没有做下拉刷新的操作了，从体验上来说，还是和下拉刷新时一样，会有一下停顿，对用户体验来说不是很友好

```javascript
wx.setBackgroundTextStyle({
    textStyle: 'light' // light or dark
})
```

<a name="2s2evg"></a>
#### [](#2s2evg)方案B（目前最终解决方案）
实现方式：使用多页面方式，结合小程序原生的 tabbar 及自定义 tabbar 实现<br />实现方式：在小程序的 app.json 中设置 tabbar 的样式，保证背景颜色与自定义tabbar的背景颜色相同，如我的项目中，自定义tabbar的背景颜色为 #fff，则 app.json 中配置如下

```json
"tabBar": {
"color": "#fff",
"backgroundColor": "#fff",
"borderStyle": "white",
"list": [
  {
    "pagePath": "pages/news/index",
    "text": ""
  },

    "pagePath": "pages/mine/index",
    "text": 
  }
]
```

注意 list 中的设置，不设置 icon 及 text，这样在页面上看，就是小程序的地步，有一个白色的区域，这里我们先把他放在这不管，然后开始写自定义的 tabbar，我这边是以组件的方式实现的，样式这里就不贴出来了，关键在于，默认让这个自定义的 tabbar 隐藏，然后在组件中加如下代码

```javascript
attached: function () {
    wx.hideTabBar({
      complete: (e) => {
        if (!this.data.showNav) this.setData({showNav: true})
      }
    })
}
```

这里的 showNav 即控制自定义 tabbar 是否显示的属性，这样的操作如下：

- tabbar 组件实例进入页面节点后

- 执行隐藏原生 tabbar 的方法

- 隐藏成功后，显示自定义 tabbar


该方式为目前尝试过的所有方式中，用户体验相对来说最好的一个方案，但该方案仍存在缺点：

- 小程序刚进入时，会显示底部白色区域（原生 tabbar），然后该区域会变为自定义的 tabbar，视觉上会看到一点变化

- 在安卓下，由于自定义 tabbar 是通过 position: fixed 固定在页面底部的，当页面有小程序原生的刷新功能时，下拉页面会导致底部 tabbar 被推出视野范围，对用户体验有一定影响，这种情况可考虑不适用原生下拉刷新功能，改用自行实现，即可解决


