#### 在现代javaScript应用程序中，通过webpack，我们可以缩减代码大小，处理各种不同类型文件的资源，对输入文件进行拆分，对输出文件进行优化，在这里，我简单介绍一下webpack是怎样去实现这些的。

##### 1.entry：接受一个字符串或者对象作为参数。
```
单页入口
module.exports = {
   app:'index.js',
   vendors:'other.js'
}
```
构建两个完全独立的，彼此分离的依赖图。通过此操作配合CommonChunkPlugin，可以从应用bundle中提取对vendor的引用，并把vendor的引用部分替换成_webpack_require_(),如果应用bundle中没有vendor，那么我们可以对vendor实现长效缓存

```
多页应用
module.exports = {
   pageA:'indexA.js',
   pageB:'indexB.js',
   PageC:'indexC.js'
}
```
构建三个独立的webpack依赖图，当页面跳转时，服务器会获取一个新的HTML文档，浏览器会刷新页面重新获取资源。通过CommonChunkPlugin，可以在多个页面间提取公用bundle，实现代码复用。

##### 2.output:只有一个输出bundle，可以在此配置输出bundle名称，以及输出位置
##### 3.loader:loader实现在import或加载的时候预处理文件，对模块的源代码进行处理，loader的使用，通过配置，内联，以及在命令行使用，loader的加载遵从于模块解析的方式
##### 4.plugins：webpack的插件是一个具有apply属性的javaScript对象，所以在使用它的时候可以通过new一个实例配置使用。
##### 5.module，webpack支持url，import，require，@import以及(require，define)这样的方式，模块化将应用程序的功能进行切分，便于定位，调试，测试。模块通过resolve实现模块解析
1. 相对路径
2. 绝对路径，结合当前的上下文解析模块路径
3. 模块是文件名时，直接解析文件名，没有文件扩展名根据resolve.extension['js','json']进行解析，如果是文件，通过package.json进行定位
