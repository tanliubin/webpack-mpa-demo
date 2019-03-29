# webpack4_mpa_demo
webpack4搭建传统多页面多环境

### 动态配置入口
配置过webpack的同学都知道,我们每增加页面，就必须新加一个入口，这样页面少还好说，但是如果页面多，那么我们配置起来就比较麻烦，所以
我们需要用到动态配置入口,函数如下：
```angular2html
// 动态配置入口
function getEntry() {
    const entry = {};
    //读取src目录所有page入口，这里js名字与page页面名字相同
    glob.sync('./src/js/*.js')
        .forEach(function (name) {
            const start = name.indexOf('src/') + 4,
                end = name.length - 3;
            const eArr = [];
            let n = name.slice(start, end);
            n = n.split('/')[1];
            eArr.push(name);
            entry[n] = eArr;
        });
    return entry;
}
```
同样，我们也需要动态生成html
```angular2html
// 获取html-webpack-plugin参数的方法
const getHtmlConfig = function (name, chunks) {
    return {
        template: `./src/pages/${name}.html`,
        filename: `${name}.html`,
        // favicon: './favicon.ico',
        // title: title,
        inject: true,
        hash: false, //开启hash  ?[hash]
        chunks: chunks,
        minify: process.env.NODE_ENV !== "production" ? false : {
            removeComments: true, //移除HTML中的注释
            collapseWhitespace: true, //折叠空白区域 也就是压缩代码
            removeAttributeQuotes: true, //去除属性引用
        },
    };
};
const htmlArray = [];
Object.keys(entryObj).forEach(element => {
    htmlArray.push({
        _html: element,
        title: '',
        chunks: ['vendor', 'common', element]
    })
});

//自动生成html模板
htmlArray.forEach((element) => {
    module.exports.plugins.push(new htmlWebpackPlugin(getHtmlConfig(element._html, element.chunks)));
});
```

### 配置loader
配置loader,这里就不多说了，需要注意的就是css，需要在对应的js里面用require或import引入，如果不这样使用，打包的时候是不会讲对应的css打包的
```angular2html
{
        test: /\.(css|scss|sass)$/,
        // 区别开发环境和生成环境
        use: process.env.NODE_ENV === "development" ? ["style-loader", "css-loader", "sass-loader", "postcss-loader"] : extractTextPlugin.extract({
            fallback: "style-loader",
            use: ["css-loader", "sass-loader", "postcss-loader"],
            publicPath: "../"
        })
    },
```
这里用到了postcss-loader,需要配置一个postcss.config.js
```angular2html
module.exports = {
	plugins: [
		//自动添加css前缀
        require('autoprefixer')
	]
};
```
要在package.json里面写上这个才会管用
```angular2html
"browserslist": [
    "defaults",
    "not ie < 11",
    "last 2 versions",
    "> 1%",
    "iOS 7",
    "last 3 iOS versions"
  ]
```
### 配置插件plugins
```angular2html
plugins: [
        // 全局暴露统一入口
        new webpack.ProvidePlugin({

        }),
        //静态资源输出
        new copyWebpackPlugin([{
            from: path.resolve(__dirname, "../src/assets"),
            to: './assets',
            ignore: ['.*']
        }]),
        // 消除冗余的css代码
        new purifyCssWebpack({
            paths: glob.sync(path.join(__dirname, "../src/pages/*/*.html"))
        })
    ]
```
### 配置服务器
```angular2html
devServer: {
        contentBase: path.join(__dirname, "../src"),
        publicPath: '/',
        historyApiFallback: true,
        host: "127.0.0.1",
        port: "8080",
        overlay: true, // 浏览器页面上显示错误
        open: true, // 开启浏览器
        stats: "errors-only", //stats: "errors-only"表示只打印错误：
        hot: true, // 开启热更新
        //服务器代理配置项
        proxy: {
            '/testing/*': {
                target: 'https://www.baidu.com',
                secure: true,
                changeOrigin: true
            }
        }
    },
```
### package.json中的scripts
"scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server --config build/webpack.dev.conf.js ",
    "build": "cross-env NODE_ENV=production webpack --config build/webpack.prod.conf.js"
  },
