## 问题

在ios11系统中模态框如果使用fixed定位，input在输入的时候会出现光标错位的问题。

![](http://ww1.sinaimg.cn/large/6d691ee6ly1foxfr1krdog20af0fvjyy.gif)

之所以会有这个 bug 是因为，当输入框获取焦点弹出输入法时，浏览器向下移动页面，导致光标不跟随焦点。

有人已经给苹果提了提了[bug](https://bugs.webkit.org/show_bug.cgi?id=176896) ，不过截止到11.2.1中仍然没有解决。

## DEMO

### html

```
<section class="modal-layer">
        <div class="address">
          ...
        </div>
</section>
```

### css 

```
.modal-layer{
    display: none;
    position:fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    width: 100%;
    background: rgba(0, 0, 0, 0.77);
    z-index: 99;
  }
  .address{
      	position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
  }
```

### 页面

![](http://ww1.sinaimg.cn/large/6d691ee6ly1fox3kyoqfuj20l00ygacu.jpg)

## 解决方案

如果将fixed定位改成relative定位是可行的，但是需要考虑这一层蒙版的对于body的定位，在触发focus事件时，记录scrollTop值，同时给蒙版.modal-layer加上一个类:

```
.ios-bugfix{
        position: absolute;
        height: 100vh;
    }
```

同时监听touchmove事件，禁止默认事件，禁止body滚动。

失去焦点时，去掉这个类即可，body滚动到之前记录的位置。