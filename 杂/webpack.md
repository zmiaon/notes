# webpack -- 模块打包工具
使用步骤：
1. npm init 创建一个初始项目
2. ```npm install webpack-cli --save-dev``` 安装webpack脚手架(安装后可以在命令行里使用webpack这个命令)
3. npx webpack index.js (网页不能识别import等，需要webpack翻译) 用webpack翻译index.js文件
4. dist目录下的main文件就是翻译后的文件

>---==save(-S)== --- 安装的依赖包将添加到package.json文件中的```dependencies```字段中。这些依赖包在开发或生产环境中都需要（当克隆或下载项目后，可以通过npm install命令自动安装这样保存的所有依赖包）
>---==save-dev(-D)== --- 安装的依赖包将添加到package.json文件的```devDependencies```字段中,这些依赖包仅在开发环境中需要，生产环境不需要

## 引入模块
1.ESModule 模块引入方式 ---- 导入：import moduleName from './' 导出：export default moduleName
2.CommonJS 导入：var module = require('./') 导出：module.exports = moduleName
3.CMD
4.ADM

## webpack.config.js  //自定义配置webpack (没有此文件则会使用默认配置)
可以指定文件为配置文件  `npx webpack --config webpackconfig.js`
module.exports = {
    mode:  设置环境为开发环境development还是生产环境production;决定编译后的bundle.js文件是否被压缩
    entry：入口文件,
    devtool:source-map -- 它是一个映射关系，当页面报错时可以根据报错追踪到源代码中的哪一行出错，而不会只显示出打包后的代码。development环境默认打开    eval为值是打包速度最快的，但是如果项目比较复杂，提示信息会太简单。module第三方模块报错的时候也指向提示信息，cheap映射代码放在一个打包文件里
    module:{        //如何处理项目中的不同类型的模块
        reules:[      匹配请求的规则数组（对木块引用loader或者修改解析器parser
            {
                test:/\.jpg$/,
                use:{
                    loader:'file-loader',
                    options:{       //额外的参数
                        name:'[name].[ext]'  //配置打包后的文件名字 name为当前名字，ext为后缀
                        outputPath:'images/',  //指定打包后所在的文件夹
                    },
                   loader:'url-loader',    //与file-loader作用相似
                       但是会把图片打包成base64的js代码,而不会生成单独的图片文件
                        options:{
                           limit:2048 //以此大小为界，图片大小超过则生成文件，不超过则打包成代码
                        }
                }
            }
        ]
    },
     externals:{             //防止将某些import的包打包到bundle中，而是在运行时再去外部获取这些扩展依赖(库代码)
        lodash:{
            commonjs:'lodash'  //commonjs方式引入的lodash固定只能为lodash名
        }
    },
    resolve:{
        extensions:['jsx','js'],        //配置文件后缀名，引入文件时会自动带入后缀
    },
    plugins:[           //插件
        new HtmlWebpackPlugin({     htmlWebpackPlugin 会在打包结束后自动生成一个html文件，并把打包生成的js自动引入到这个htmlwenjain 
            template:'../'          //可以设置模板文件（需要挂载的根节点不会自动生成
        })
    ],
    output:{  编译导出
         publicPath:'http://cdn.com.cn',  //按需加载或加载外部资源的时候需要用到
        clean:  webpack5已内置cleanWebpackPlugin不用引入使用，直接clean:true可以达到效果
        filename:  配置翻译文件的名称
        path:   配置翻译文件所在位置
        library:'library',  //会在全局变量里增加library变量，可以通过script引入库
        libraryTarget:'',   //值为'umd':commonJs ,ESModule引入方式都可以引入这个库;值为'this'，则library变量会挂载到this上
    }
}

### 热更新
1.package.json页面script配置"webpack --watch"即可监听页面代码的变化此时只需要刷新就可以获取新页面，不用重新构建
2.webpack.config.js 配置devServer  //不需要刷新会自动获取新的内容
3.自己手动写一个服务器 server.js

### babeljs
导入：```npm install --save-dev babel-loader @babel/core```
>    presets:      -----如果当前代码是在项目中，可以使用这个
>      ```npm install @babel/preset-env --save-dev```
>      ```npm install --save @babel/polyfill```  import引入需要用到的js文件
> 部分浏览器无法识别es6语法，使用babel可以将es6转义为es5代码 
>    babel-loader ---- 只是babel与webpack之间的桥梁
>    preset-env ---- 转义es6语法为es5

>或者：plugins --- 当前为库代码 --- 闭包的方式注入，不会像polyfill一样污染全局环境
>导入：   ```npm install --save-dev @babel/plugin-transform-runtime```
>        ```npm install --save @babel/runtime```
>        ```npm install --save @babel/runtime-corejs2```
>
>        也可以单独放到一个文件中：.babelrc

### Tree Shaking        
当引入模块的时候，不引入这个模块所有的代码
在development环境下默认没有Tree Shaking,要在
webpack.config.js 配置：
optimization:{
    usedExports:true
}
>另外只是单纯引入整个模块的，例如：import '@babel/polyfill'
>需要另外在package.json文件下加:"sideEffects":['@babel/polyfill'],否则tree Shaking会将它排除出去

### 开发环境线上环境配置代码分离
可以在package.json文件配置npx webpack --config webpack.dev.js的相应指令 指定dev或者prod环境的文件，分别进行配置，相同的代码放倒公共文件webpack.common.js
使用webpack.merge合并配置信息：
```npm install webpack-merge -D```

### 拆分代码
Code Splitting
1.当main.js直接导入第三方库代码，这第三方代码一般不会改变，但是访问页面时每次都会重新加载main.js
2.将页面导入放到另一个js文件，也即拆分main.js，当页面业务逻辑变化时，只要加载main.js即可
3.使用webpack的配置：
                    同步代码：optimization.splitChunks.chunks
                    异步代码：无须做任何配置，会自动进行代码分割

### webpack项目打包性能分析
webpack --profile --json > stats.json

### 预获取 预加载模块
生命import时使用的内置指令：
- 预获取（prefetch） 等核心代码加载完之后页面空闲的时候再加载
- 预加载（preload）     和核心代码同时加载

### css代码分割
```npm install --save-dev mini-css-extract-plugin```
css代码压缩
~~npm install optimize-css-assets-webpack-plugin -D~~
```npm install css-minimizer-webpack-plugin -D```

### 浏览器缓存
导出文件名加contenthash,文件内容发生变化，hash值才会改变，从而让浏览器缓存识别出变化，重新加载

- 当在一个模块中需要使用另一个第三方模块，又不想重复引入，可以配置：webpack.ProvidePlugin

- 需要将所有js文件中的this默认指向window:
```npm install imports-loader --save-dev```

### library
库项目代码完成后希望其他项目可以引入它时：
- 在库项目package.json中main改成打包后的文件位置
- 在npm 官网注册账号，然后在控制台npm add user,填入用户名，密码
- 运行npm publish命令，发布到npm 仓库上
- 其他项目可以通过npm install 下载

### 用http服务器运行打包后的文件
安装：```npm install http-server--save-dev```
在package.json配置："http-start":"http-server dist"（每次修改代码后需要重新打包dist再执行命令）

### PWA(service worker)
安装：```npm install workbox-webpack-plugin --save-dev```
相当于缓存页面，当服务器停止后，页面会获取已缓存的代码执行
通过plugins形式引入（因为只在线上环境才会用到，在prod配置文件配置，打包后会生成service-worker.js文件，可以使用navigator.serviceWorker对象的功能（存在这个对象的话

### 使用typeScript
引入：```npm install ts-loader typescript --save-dev```
需要增加tsconfig.json文件做配置
当需要对引入的其他模块的用法有提示信息，需要再安装npm install @types/lodash --save-dev  组件对应的类型文件（这里是lodash
>在github上搜索找到Definitely/Typed/DefinitelyTyped对应的页面可以有链接'TypeSearch'点击搜索

还有一种方式是：安装typings（将废弃

### react框架使用
安装：npm install react react-dom --save
npm install --save-dev @babel/preset-react

### eslint
安装：npm isntall eslint --save-dev
命令生成文件：npx eslint --init