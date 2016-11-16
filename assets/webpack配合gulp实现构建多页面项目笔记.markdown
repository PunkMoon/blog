md之前在网上找前端模块化以及自动化构建的解决方案，大多主要是针对单页面的，而且介绍的也很浅显，后来找到这篇博客[gulp + webpack 构建多页面前端项目](https://github.com/fwon/blog/issues/17)，然后在他的基础上针对自己项目做了修改和优化，并加上适当的注释，方便理解，在此对原作者表示感谢。
## 项目结构
入口文件放在根目录```src```：
```
├─app
│  │  index.html
│  │
│  └─user
│          index.html
│          login.html
│          register.html
│
├─css
│      reset.css
│      sprite.css
│      style.scss
│
├─images
│      logo.png
│      user_bg.jpg
│
└─js
    │  index.js
    │  login.js
    │  reg.js
    │
    └─lib
            touch.js
            utils.js
```
出口文件在根目录```dist```下.
## 页面配置
* css合并为一个文件，在head内通过link标签引入
## webpack配置
* 在body底部引入公共js文件以及页面对应的出口文件
```
var path=require('path');
var webpack=require('webpack');
var fs=require('fs');
var uglifyJsPlugin = webpack.optimize.UglifyJsPlugin;
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
var srcDir = path.resolve(process.cwd(), 'src');
//获取入口文件
function getEntry() {
    var jsPath = path.resolve(srcDir, 'js');
    var dirs = fs.readdirSync(jsPath);
    var matchs = [], files = {};
    var chunks=[];
    dirs.forEach(function (item) {
        matchs = item.match(/(.+)\.js$/);
        if (matchs) {
          chunks.push(matchs[1]);
            files[matchs[1]] = path.resolve(srcDir, 'js', item);
        }
    });
    return {
      files:files,
      chunks:chunks
    };
}
module.exports={
  devtool: 'eval-source-map',//配置生成Source Maps,方便出错时调试
  entry:getEntry().files,
  output:{
    path:path.join(__dirname,'dist/js/'),
    filename:'[name].js',
    chunkFilename: "[chunkhash].js",
    publicPath:'/js/'//设置devServer时需要
  },
  //生成静态服务器
  devServer: {
    contentBase: __dirname+'/dist',//本地服务器所加载的页面所在的目录
    colors: true,//终端中输出结果为彩色
    historyApiFallback: true,//不跳转
    inline: true,//实时刷新
    port:3000
  },
  module:{
    loaders:[
      {
        test:/\.scss$/,
        loader:'css!sass'
      },
      {
        test:/\.css$/,
       loader: 'css'
      },
      {
        test: /\.(png|jpg|gif)$/,
        loader: 'url',
        query:{
          limit:8192,//小于80k的图片自动转成base64
          name:'../images/'+'[name].[ext]'
        }
       }
    ]
  },
  resolve:{
    alias:{
      //定义别名，方便引用
      zepto:__dirname+'/node_modules/webpack-zepto/index.js',//不使用webpack-zepto会报错
      touch:srcDir+'/js/lib/touch.js',
      utils:srcDir+'/js/lib/utils.js',
      scss:srcDir+'/css/style.scss',
      reset:srcDir+'/css/reset.css'
    }
  },
  plugins:[
    //公用模块，自动加载为全局变量
    new webpack.ProvidePlugin({
      $:'zepto',
      Zepto:'zepto'
    }),
    //将入口文件里的公用js提取出来并合并成common.js
    new CommonsChunkPlugin('common.js',getEntry().chunks)
     new uglifyJsPlugin({
       compress:{
          warnings:false
       }
     })
  ]
};
```
## gulp配置
gulp这一部分主要任务是实现css压缩合并、修改文件后自动刷新、自动生成精灵图等功能。
```
var gulp = require('gulp'),
    sass=require('gulp-sass'),
    gutil = require('gulp-util'),
    fileinclude = require('gulp-file-include'),
    webpack = require('webpack'),
    webpackConfig = require('./webpack.config.js'),
    spritesmith = require('gulp.spritesmith'),
    clean=require('gulp-clean'),
    browserSync = require('browser-sync').create(),
    reload = browserSync.reload,
    minifyCss = require('gulp-minify-css'),
    buffer = require('vinyl-buffer'),
    merge = require('merge2'),
    concat = require('gulp-concat');
//实现自动刷新
gulp.task('browser-sync',function() {
    browserSync.init({
      //这里自动生成http://loaclhost:3000的静态服务器
        server: {
            baseDir: "./dist"
        }
    });
    gulp.watch('src/css/*',['sass']);//监听css改动，无需刷新就可以看到改动效果
    gulp.watch('src/app/**/*.html',['fileinclude'])
    .on('change', reload);//监听html文件改动，自动刷新
});
//拷贝图片
gulp.task('copy:images', function (done) {
    gulp.src(['src/images/**/*']).pipe(gulp.dest('dist/images')).on('end', done);
});
//用于在html文件中直接include文件
gulp.task('fileinclude', function (done) {
    gulp.src(['src/app/**/*.html'])
        .pipe(fileinclude({
          prefix: '@@',
          basepath: '@file'
        }))
        .pipe(gulp.dest('./dist'))
        .on('end', done);
});
gulp.task('sass', ['sprite'],function() {
   return  gulp.src(['src/css/*.scss', 'src/css/*.css'])
    .pipe(sass())
    .on('error', sass.logError)
    .pipe(concat('style.min.css'))
    .pipe(minifyCss({
      keepSpecialComments: 0
    }))
    .pipe(gulp.dest('./dist/css/'))
    .pipe(browserSync.stream());//此处重要
});
//自动生成精灵图并生成公共样式
gulp.task('sprite', function () {
  var spriteData = gulp.src('src/images/icons/*.png').pipe(spritesmith({
    imgName: 'sprite.png',
    cssName: 'sprite.css',
    cssFormat: 'css'
  }));
  //生成的精灵图及样式分别存放在不同的目录下
  var imgStream=spriteData.img
  .pipe(buffer())
  .pipe(gulp.dest('./dist/images/'));
  var cssStream=spriteData.css
  .pipe(gulp.dest('./src/css'));
  return merge(imgStream, cssStream);
});
gulp.task('watch', function (done) {
    gulp.watch('src/js/**/*.js',['build-js'])
        .on("change", reload)
        .on('end', done);
    gulp.watch('src/images/**/*',['copy:images'])
        .on("change", reload)
        .on('end', done);
    gulp.watch('src/images/icons/*',['sass'])
        .on("change", reload)
        .on('end', done);
});
var myDevConfig = Object.create(webpackConfig);
var devCompiler = webpack(myDevConfig);
//引用webpack对js进行操作
gulp.task("build-js", ['fileinclude'], function(callback) {
    devCompiler.run(function(err, stats) {
        if(err) throw new gutil.PluginError("webpack:build-js", err);
        gutil.log("[webpack:build-js]", stats.toString({
            colors: true,
            progress:true
        }));
        callback();
    });
});
//清除目录
gulp.task('clean',function(done){
  gulp.src('dist')
  .pipe(clean())
  .on('end',done);
});
//发布
gulp.task('dev', [ 'build-js','browser-sync','copy:images','sass','watch']);

```

```package.json```文件：

```
{
  "name": "nsyc",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "webpack --progress --colors",
    "build": "NODE_ENV=production webpack --config ./webpack.production.config.js --progress"
  },
  "keywords": [],
  "author": "vanboo",
  "license": "ISC",
  "devDependencies": {
    "browser-sync": "^2.17.6",
    "css-loader": "^0.25.0",
    "file-loader": "^0.9.0",
    "gulp": "^3.8.10",
    "gulp-clean": "^0.3.2",
    "gulp-concat": "^2.6.0",
    "gulp-file-include": "0.13.7",
    "gulp-sass": "latest",
    "gulp-util": "~2.2.9",
    "gulp-watch": "4.1.0",
    "gulp.spritesmith": "^6.2.1",
    "merge2": "^1.0.2",
    "sass-loader": "^4.0.2",
    "style-loader": "^0.13.1",
    "url-loader": "^0.5.7",
    "vinyl-buffer": "^1.0.0",
    "webpack": "^1.13.3",
    "webpack-dev-server": "^1.16.2",
    "webpack-zepto": "0.0.1"
  }
}
```

## 不足
原本想webpack任务和gulp任务单独运行的，webpack监听js，gulp监听HTML和css文件，但是devServer与gulp的browser-sync在引用js时有冲突，研究了一段事件也没找到好的解决方案，所以每次js变化都会重新打包一次。希望后面能继续优化吧。
