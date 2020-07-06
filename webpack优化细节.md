### webpack优化细节

#### 使用webpack进行优化主要体现在两个方面
##### 1.构建过程花费时间太多
##### 2.打包过程的结果体积太大

##### 一，构建过程进行提速
##### 1.我们使用相应的loader对相应的文件进行编译，比如babel-loader，通常，我们可以使用exclude来对不必要的文件进行转译

```
 module:{
        rules:[
            {
                test:/\.js$/,
                loader:'babel-loader?cacheDirectory=ture',
                exclude:[
                    path.join(__dirname,'./node_modules')
                ]
            },
        ]
    }
```
##### 通过限定文件范围来进行优化带来的效果是有限的，对于构建成功的应用，通过开启缓存，可以将转译后的结果存储到文件系统,这样，使得通过babel-loader构建的速度提高两倍。


```
    loader:'babel-loader?cacheDirectory=ture'

```

##### 2.第三方库打包

##### 通过使用DllPlugin和DllReferencePlugin，将第三方库抽离出来，在热更新的时候不需要对第三方库再进行打包

```

module.exports = {
    entry:{
      vendor:[
          'react',
          'react-router-dom',
          'react-dom',
          'redux',
      ]
    },
    plugins:[
        new webpack.DllPlugin({
            // DllPlugin的name属性需要和libary保持一致
            name: '[name]_[hash]',
            path: path.join(__dirname, 'dist', '[name]-manifest.json'),
            // context需要和webpack.config.js保持一致
            context: __dirname,
          })
    ],
  }
```
##### 构建成功之后，生成的vendor-manifest.json文件包含打包成功的库的引用路径和id


```
{
    "name": "vendor_057a5d66871150ce1658",
    "content": {
        "./node_modules/@babel/runtime/helpers/esm/extends.js": {
            "id": "./node_modules/@babel/runtime/helpers/esm/extends.js",
            "buildMeta": {
                "exportsType": "namespace",
                "providedExports": [
                    "default"
                ]
            }
        },
        "./node_modules/@babel/runtime/helpers/esm/inheritsLoose.js": {
            "id": "./node_modules/@babel/runtime/helpers/esm/inheritsLoose.js",
            "buildMeta": {
                "exportsType": "namespace",
                "providedExports": [
                    "default"
                ]
            }
        }
    }
}
```

##### 之后，再通过DllReferencePlugin将打包后的库进行引用

```
 plugins: [
    new webpack.DllReferencePlugin({
      context: __dirname,
      // manifest就是我们第一步中打包出来的json文件
      manifest: require('./dist/vendor-manifest.json'),
    })
  ]
```

##### 3.对JS文件进行压缩
##### webpack最新版本，通过对optimization配置项进行设置，压缩JS是通过terser-webpack-plugin插件实现的。


```
 optimization: {
        minimize: true,
        minimizer: [new TerserPlugin({
            parallel: 4,            //启用多进程进行运行
            extractComments:'all'  //提取all或some（使用/^\**!|@preserve|@license|@cc_on/iRegExp）注释。
        })],
      },
```

##### 4.tree-shaking
##### 简而言之，就是用于移除JS上下文中未引用代码，由于JS是通过export和import方式进行导出和引入，在此过程中对于未引用的模块，我们需要在打包过程中将其移除

##### 而通过 package.json 的"sideEffects"属性作为标记，向 compiler 提供提示，表明项目中的哪些文件是 "pure(纯的 ES2015 模块)"，由此可以安全地删除文件中未使用的部分。

##### 找出这些未引用的代码，并在bundle中删除，然后引入一个能够删除未引用代码的压缩工具，在webpack4中，通过在optimization选项中进行minimize设定，即可启用压缩


##### 5.按需加载
##### 即不需要一次性将所有内容全部加载出来，而是用到的时候才去加载。


```
output: {
    path: path.join(__dirname, '/../dist'),
    filename: 'app.js',
    // 指定 chunkFilename
    chunkFilename: '[name].[chunkhash:5].chunk.js',
}
```


```
const getComponent => (location, cb) {
  require.ensure([], (require) => {
    cb(null, require('../pages/DetailComponent').default)
  }, 'detail')
},
...
<Route path="/detail" Component={getComponent}>
```

##### 核心方法主要是：require.ensure(dependencies, callback, chunkName)


##### 6.代码拆分
##### 将代码拆分成多个捆绑包，然后将重复代码进行删除，使得代码可以按需或并行运行。

##### 拆分方法
###### 1.入口点：使用entry配置手动拆分代码。
###### 2.防止重复：使用SplitChunksPlugin重复数据删除和拆分块。
###### 3.动态导入：通过模块内的内联函数调用拆分代码。

##### 其中，动态导入是将已经生成对bundle在应用对时候才进行引入


```
async function getComponent() {
   return import(/* webpackChunkName: "lodash" */ 'lodash').then(({ default: _ }) => {
     const element = document.createElement('div');

     element.innerHTML = _.join(['Hello', 'webpack'], ' ');

     return element;

   }).catch(error => 'An error occurred while loading the component');
   const element = document.createElement('div');
   const { default: _ } = await import(/* webpackChunkName: "lodash" */ 'lodash');

   element.innerHTML = _.join(['Hello', 'webpack'], ' ');

   return element;
  }

  getComponent().then(component => {
    document.body.appendChild(component);
  });
```



