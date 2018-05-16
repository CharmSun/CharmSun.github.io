---
title: gulp使用小结
tags:
  - gulp
categories: 前端
img: /2016/07/08/gulp-summary/gulp.png
date: 2016-07-08 15:18:51
---


前段时间做了一个项目，使用了gulp来实现广告样式模板文件的自动化构建和热更新开发，对gulp的使用体验感受是，插件丰富，构建流程易于控制，使用简单，构建效率也不错，相比于同样插件体系丰富的grunt和webpack构建机制而言，由于gulp其实是整合众多的gulp插件来编写Node.js可执行任务脚本，因此构建过程中构建流程对开发人员更清晰，也更易于编码进行控制，自定义化更强。当然，gulp与webpack并不冲突，webpack的模块化是gulp没有的，两者可结合使用。  

本文不对gulp做过多介绍，介绍可以看官网，就只总结一些常用gulp插件的用途和用法，希望起到参考作用。

## 插件介绍
Gulp现在大约有2000多左右的插件，可以到[http://gulpjs.com/plugins/](http://gulpjs.com/plugins/)或者[http://npmsearch.com/?q=keywords:gulpplugin](http://npmsearch.com/?q=keywords:gulpplugin)查找。
这里列出一些出场率较高或者我了解的插件，并可能在将来的开发中更新。

### gulp-jshint
gulp的jshint插件，用来检测js代码中语法和代码风格问题。  
其中jshint有两个参数：
- lookup: 默认是true, 为false时不查找.jshintrc 文件
- linter: 可以指定其他的linter， 例如 “jsxhint”

jshint.reporter 可以指定错误报告的形式

> gulp-jshint用法：

```
var jshint = require('gulp-jshint');
// 检查脚本
gulp.task('lint', function() {
    return gulp.src(config.scripts)
        .pipe(jshint())
        .pipe(jshint.reporter('default'));  // 输出检查结果
});
```


```
gulp.task('lint', function() {
    return gulp.src(['src/*.js', 'src/ui/*.js'])
      .pipe(jshint())
      .pipe(jshint.reporter('jshint-stylish')) //输出结果会带颜色
      .pipe(jshint.reporter('fail')); //检查有错则任务失败
});
```

### gulp-concat
用来把多个文件合并为一个文件,我们可以用它来合并js或css文件等，这样就能减少页面的http请求数了 
> gulp-concat用法：

```
var concat = require('gulp-concat');
gulp.task('scripts', function() {
  gulp.src('./lib/*.js')
    .pipe(concat('all.js'))
    .pipe(gulp.dest('./dist/'))
});
```


### gulp-uglify  
用来压缩js文件，使用的是uglify引擎

> gulp-uglify 用法：

```
var uglify = require('gulp-uglify');
gulp.task('compress', function() {
  gulp.src('lib/*.js')
    .pipe(uglify())
    .pipe(gulp.dest('dist'))
});
```
### gulp-rename  
改变管道中的文件名。
> gulp-rename 用法：

```
var concat = require('gulp-concat');
// rename via hash
gulp.src("./src/main/text/hello.txt", { base: process.cwd() })
    .pipe(rename({
        dirname: "main/text/ciao",
        basename: "aloha",
        prefix: "bonjour-",
        suffix: "-hola",
        extname: ".md"
    }))
    .pipe(gulp.dest("./dist")); // ./dist/main/text/ciao/bonjour-aloha-hola.md
```

### gulp-clean-css
用来压缩css文件，减小文件大小，之前gulp-minify-css已经被废弃，现在都使用gulp-clean-css，用法一致，最终是调用clean-css，使用参数[请看这里](https://github.com/jakubpawlowicz/clean-css#how-to-use-clean-css-api)。

### gulp-if
有条件的执行任务

### gulp-ftp
将文件上传至ftp服务器

> gulp-clean-css、gulp-if、gulp-ftp 用法：

```
var cleanCSS = require('gulp-clean-css');
var concat = require('gulp-concat');
var ftp = require('gulp-ftp');
var gulpif = require('gulp-if');

var CDN_CONFIG = {
    host: 'ftp.cdn.qcloud.com',
    user: '1251479438',
    pass: '*******',
    remotePath: '/0/brandad'
};
// 合并、压缩css
gulp.task('css', function() {
    return gulp.src(config.styles)
        .pipe(concat(config.id + '.css'))
        .pipe(cleanCSS())
        .pipe(gulp.dest('./' + config.id + '/dist'))
        .pipe( gulpif(config.cssCDN, ftp(CDN_CONFIG)) );  //根据配置判断是否使用ftp上传至cdn服务器
});
```
### gulp-less 和 gulp-sass
分别用来编译less和sass
> 用法：

```
var gulp = require('gulp'),
    less = require("gulp-less");
 
gulp.task('compile-less', function () {
    gulp.src('less/*.less')
    .pipe(less())
    .pipe(gulp.dest('dist/css'));
});
```
```
var gulp = require('gulp'),
    sass = require("gulp-sass");
 
gulp.task('compile-sass', function () {
    gulp.src('sass/*.sass')
    .pipe(sass())
    .pipe(gulp.dest('dist/css'));
});
```

### gulp-css-base64
用来将图片转成base64
> 用法：

```
var cssBase64 = require('gulp-css-base64');
gulp.task('img2base64', function() {
    return gulp.src('node_modules/jstree/dist/themes/default/style.css')
        .pipe(cssBase64())
        .pipe(rename('style.base64.css'))
        .pipe(gulp.dest('node_modules/jstree/dist/themes/default'));
});
```

### gulp-sequence 和 run-sequence
gulp 的 task 都是并行(异步)执行，如果遇见需要串行的场景，那么这个插件就可以解决问题。不过好像gulp4.0发布后，不需要RS也可以搞定串行任务了。
> 用法：

```
var gulp = require('gulp')
var gulpSequence = require('gulp-sequence')
 
gulp.task('test', gulpSequence(['a', 'b'], 'c', ['d', 'e'], 'f'))
```

### gulp-inject-string
用来在管道中对文件拼接字符串，更多用法可[参见这里](https://www.npmjs.com/package/gulp-inject-string)
> 用法：

```
var gulp = require('gulp'),
    rename = require('gulp-rename'),
    inject = require('gulp-inject-string');
 
gulp.task('inject:append', function(){
    gulp.src('src/example.html')
        .pipe(inject.append('\n<!-- Created: ' + Date() + ' -->'))
        .pipe(rename('append.html'))
        .pipe(gulp.dest('build'));
});
 
gulp.task('inject:prepend', function(){
    gulp.src('src/example.html')
        .pipe(inject.prepend('<!-- Created: ' + Date() + ' -->\n'))
        .pipe(rename('prepend.html'))
        .pipe(gulp.dest('build'));
});
```

---
还有一些插件简略说明，可自行搜索：  
- gulp-rev ：把静态文件名改成hash的形式。
- gulp-sourcemaps ： 处理 JavaScript 时生成 SourceMap。
- gulp-markdown ：写手的福音，可以把 Markdown 转成 HTML。
- gulp-autoprefixer：给 CSS增加前缀。解决某些CSS属性不是标准属性，有各种浏览器前缀的情况。


## 启动服务和实时构建
在实际项目中，可以利用browser-sync来启动服务，并进行事件监听实时构建，热更新。一旦修改，即刻构建并渲染数据刷新页面，在开发过程中非常方便。

```
gulp.task('default', ['render-ad'], function() {
    browserSync.init({
        server: {
            baseDir: './'
        }
    });
    gulp.watch(config.styles, ['render-ad']);
    gulp.watch(config.scripts, ['render-ad']);
    gulp.watch(config.tpl, ['render-ad']);
    gulp.watch(config.data, ['render-ad']);
});
```
