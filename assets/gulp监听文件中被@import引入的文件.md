## gulp监听文件中被@import引入的文件

### 背景
由于公司静态文件目录的css文件众多，没办法监听所有的.scss文件然后编译成一个css文件，所以需要监听具体哪个.scss文件变化然后编译为对应css文件。随之而来带来的问题是，如果a.scss中通过@import引入了一个b.scss文件，如果b.scss文件变化，也需要编译一下a.scss。实际上这个时候a.scss是监听不到变化的，所以这个a.scss也不会去编译。

### 寻找解决方案

网上一通搜索，最后通过google **watch scss import file** 这些关键字找到了 [Watch for changes in imported files as well](https://github.com/sass/node-sass/issues/700)，然后按图索骥找到了 [Use sass-graph for accurate sass watching](https://github.com/sass/node-sass/pull/629) 。最后就是用**sass-graph** 这个插件获取到目录内文件的依赖关系。如下：

```
{ index: {,
    '/path/to/test/fixtures/a.scss': {
        imports: ['b.scss'],
        importedBy: [],
    },
    '/path/to/test/fixtures/b.scss': {
        imports: ['_c.scss'],
        importedBy: ['a.scss'],
    },
    '/path/to/test/fixtures/_c.scss': {
        imports: [],
        importedBy: ['b/scss'],
    },
}}
```

这时候监听所有被引入的文件变化，然后拿到被引入的文件list就迎刃而解了。

### 项目代码实现

* config.json

  在这个文件里配置需要监听的被引入的文件目录

  ```
   "importPaths":["sass/m5/common/**.scss"]
  ```

* Gulpfile.js

  ```
   var grapher = require('sass-graph')；
   var importPaths=require('./config.json').importPaths;
   var importFolders=grapher.parseDir('./sass',['scss']);
   gulp.task('importWatch',function(){
      gulp.watch(importPaths,function (e) {
          var importedFiles=importFolders.index[e.path].importedBy;
          importedFiles.forEach(function (el) {
              var relativePath=getPath(el);//获取相对目录
              sassTask(relativePath);//执行sass任务编译为css
          });
      });
  });
  ```

  ​

  ​



