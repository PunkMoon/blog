## 屏幕尺寸

垂直方向上，iPhone X的显示宽度与iPhone 6，iPhone 7 和 iPhone 8 的 4.7 英寸一样，但是比4.7英寸的显示屏高145pt。

![](http://ww1.sinaimg.cn/large/6d691ee6ly1foy9044tnej21740pswg4.jpg)

## 安全区域

安全区域指的是一个可视窗口范围，处于安全区域的内容不受圆角（corners）、齐刘海（sensor housing）、小黑条（Home Indicator）影响

![](http://ww1.sinaimg.cn/large/6d691ee6ly1foy98uakmaj219e0lumyi.jpg)

## viewport-fit

通过对meta标签viewport的扩展，可以调整页面的展现区域。viewport-fit有三个可选值：

- contain： 使页面展示在安全区域内。
- cover： 使页面占满屏幕。
- auto： 和 contain 选项表现一样

## env() 和 constant()

iOS11 新增特性，Webkit 的一个 CSS 函数，用于设定安全区域与边界的距离，有四个预定义的变量：

* safe-area-inset-left：安全区域距离左边边界距离
* safe-area-inset-right：安全区域距离右边边界距离
* safe-area-inset-top：安全区域距离顶部边界距离
* safe-area-inset-bottom：安全区域距离底部边界距离

![](https://webkit.org/wp-content/uploads/safe-areas-1.png)



因为之前使用的constant已经被弃用，所以需要我们向后兼容：

```
padding-bottom: constant(safe-area-inset-bottom); /* 兼容 iOS < 11.2 */
padding-bottom: env(safe-area-inset-bottom); /* 兼容 iOS >= 11.2 */
```

## 适配 

1. ### 设置网页在可视窗口的布局方式

   使页面完全覆盖整个窗口

   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover"/>
   ```
   只有设置了 viewport-fit=cover，才能使用 env()。

2. ### 页面主体内容限定在安全区域内

   ```
   body {
     padding-bottom: constant(safe-area-inset-bottom);
     padding-bottom: env(safe-area-inset-bottom);
   }
   ```

3. ### fixed 元素的适配

   如果元素是fixed定位且bottom=0，那么除了设置body的padding-bottom还不够，还需要给它自身添加padding，因为它是相对于屏幕最底部定位的。

   ```
   {
     padding-bottom: constant(safe-area-inset-bottom);
     padding-bottom: env(safe-area-inset-bottom);
   }
   ```


   或者通过计算函数 calc 覆盖原来高度：

   ```
   {
        height: calc(60px(假设值) + constant(safe-area-inset-bottom));
        height: calc(60px(假设值) + env(safe-area-inset-bottom));
   }
   ```

   *注意，这个方案需要吸底条必须是有背景色的，因为扩展的部分背景是跟随外容器的，否则出现镂空情况。*

   如果元素是fixed定位且bottom不等于0，则只调整位置就可以了，增加margin-bottom或者改变bottom。

## 参考

* [网页适配 iPhoneX，就是这么简单](https://aotu.io/notes/2017/11/27/iphonex/index.html)

* [Designing Websites for iPhone X](https://webkit.org/blog/7929/designing-websites-for-iphone-x/)]

* [Human Interface Guidelines](https://developer.apple.com/ios/human-interface-guidelines/overview/iphone-x/)
