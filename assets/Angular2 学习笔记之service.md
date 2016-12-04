# Angular2 servive组件学习笔记
## 命名方式
官方推荐的命名方式感觉有点繁琐，我比较习惯于在app目录下新建一个service的文件夹存放可复用的service组件
##　Injectable
在export我们的service之前，需要引入Injectable，并增加@Injectable装饰器。
```
import { Injectable } from '@angular/core';

@Injectable()
export class myService {
}
```

##  调用service

* 定义一个私有变量```constructor(private myservice: myService) { }```
* 在调用service的@Component添加```providers: [myService]```

##  **ngOnInit** 生命周期钩子

每个组件都有自己从创建到销毁的生命周期，angular提供了能够捕获到这个生命周重要节点(key life moments)的钩子。ngOnInit是其中的一个。

在调用service中的方法时，使用ngOninit

```
 ngOnInit(): void {
    this.method();
  }
```

## promise

在调取后台数据时，使用promise异步调取数据，等获取到数据时，通过回调函数返回到组件中.

```
getHeroes(): Promise<Hero[]> {
  return Promise.resolve(HEROES);
}
```

调用service方法时：

```
getHeroes(): void {
  this.heroService.getHeroes().then(heroes => this.heroes = heroes);
}
```

