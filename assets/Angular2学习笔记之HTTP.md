## Angular2学习笔记之HTTP

### service

一般HTTP请求在service中完成，因此需要在service中封装一个公共方法用来获取数据。

```service.ts```

```
import {Injectable} from '@angular/core';
import {Http,Response} from '@angular/http';
import 'rxjs/Rx';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/catch';

@Injectable()
export class dataService{
	data:Object[];//声明变量类型
	constructor(private http: Http) {}
	getData(sendData:Object){
		if (this.data) {
		   return Promise.resolve(this.data);
		 }
		return new Promise(resolve=>{
			this.http
			.post('requestUrl',sendData)
			.map((r:Response)=>r.json()) 
			.subscribe(data => {
		        this.data = JSON.parse(data.result);//此处对获取到的数据类型的处理特别重要，必须为Object,否则angular会报错Cannot find a differ supporting object
		        resolve(this.data);
	      });
		})
	}
}
```

### component	

在组件中引入对应的service，执行service中的相关函数，将获取到的数据渲染到组件中。

```
import { Component,OnInit  } from '@angular/core';
import {dataService} from '../../service/service';
@Component({
  selector: 'page-home',
  providers:[dataService],//在providers添加引入的service
  templateUrl: 'home.html'
})
export class HomePage implements OnInit {
	data:Object[];//声明变量类型，此处不能省略，否则会报错
  constructor(
  public navCtrl: NavController,
  private dataService:dataService
  ){
  }
  ngOnInit():void{
  	var sendData={
  		
  	};
  	this.dataService.getData(sendData)
    .then(data=>{
      this.data=data;
    })
  	}
}
```

### Promise与Observable

这两种都是对异步数据流的处理方法。

angular的http.get返回的是一个 *RxJS `Observable* ，需要使用一个toPromise方法转成一个Promise。

如果直接使用Observable的话就免去了这个转换的过程，按照我对文档的理解，如果数据不是一次请求结束后就万事大吉了，比如我们可能发起一个请求，中途可能要结束这次请求，在第一次请求结束前又要发起另一个请求。如果要实现这个需求，Promise会比较困难。

因此，Observable算是更高级一点的Promise,所以一般情况下还是用Observable比较好一点。

用法：(可以与上面的Promise进行对照)

```service.ts	```

```
getData(sendData:Object,method:string):Observable<any>{
		return this.http
		.post('requestUrl',sendData)
		.map((r:Response)=>r.json().result);
	});
	}
```

```component```

```
this.dataService.getData(sendData)
  	.subscribe(
  		homeData=>this.data=Json.parse(homeData),
  		err=>{
  			console.log(err);
  		});
  	}
```





