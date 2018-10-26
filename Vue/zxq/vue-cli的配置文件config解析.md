# vue-cli config 的配置文件的解析

###首先来看看 config 的文件结构

<table width=100%>
    <tr>
        <td  bgcolor=#ccc>
       　<font color=#333 size=2 >|-config</font><br>
        　　<font color=#333 size=2 >|---dev.env.js</font><br>
        　　<font color=#333 size=2 >|---index.js</font><br>
        　　<font color=#333 size=2 >|---prod.env.js</font><br>
        </td>
        </td>
    </tr>
</table>
###dev.env.js文件内容，在严格模式下运行，首先引入webpack-merge，用来合成两个配置文件
然后引入./prod.env，第三步将两个配置文件合并。
<table width=100%>
    <tr>
        <td  bgcolor=#ccc>
       　<font color=#333 size=2 >"use strict";</font><br>
        　　<font color=#333 size=2 >const merge = require("webpack-merge");</font><br>
        　　<font color=#333 size=2 >const prodEnv = require("./prod.env");<br>
        　　<font color=#333 size=2 >module.exports = merge(prodEnv, {</font><br>
          　　　  <font color=#333 size=2 >NODE_ENV: '"development"'</font><br>
           　　 <font color=#333 size=2 >});</font><br>           
        </td>
        </td>
    </tr>
</table>
###prod.env.js文件内容，同样在严格模式下运行，声明是执行环境是生产环境
<table width=100%>
    <tr>
        <td  bgcolor=#ccc>
       　<font color=#333 size=2 >"use strict";</font><br>
        　　<font color=#333 size=2 >module.exports = {</font><br>
          　　　  <font color=#333 size=2 > NODE_ENV: '"production"'</font><br>
           　　 <font color=#333 size=2 >};</font><br>           
        </td>
        </td>
    </tr>
</table>
###index.js文件内容
<table width=100%>
<tr>
<td >
    ```
    "use strict";
// Template version: 1.3.1
// see http://vuejs-templates.github.io/webpack for documentation.

const path = require("path");

module.exports = {<br>
dev: {<br>
// Paths
assetsSubDirectory: "static", //静态文件目录
assetsPublicPath: "/", //
proxyTable: {}, //e 配置代理 api

    // Various Dev Server settings
    host: "localhost", // can be overwritten by process.env.HOST主机地址
    port: 8080, // can be overwritten by process.env.PORT, if port is in use, a free one will be determined端口号
    autoOpenBrowser: false, //是否自动打开浏览器
    errorOverlay: true, //查询错误
    notifyOnErrors: true, //错误通知
    poll: false, // https://webpack.js.org/configuration/dev-server/#devserver-watchoptions-

    /**
     * Source Maps
     */
    // https://webpack.js.org/configuration/devtool/#development
    devtool: "cheap-module-eval-source-map", //webpack提供的用来方便调试的配置，它有四种模式，可以查看webpack文档了解更多
    // If you have problems debugging vue-files in devtools,
    // set this to false - it *may* help
    // https://vue-loader.vuejs.org/en/options.html#cachebusting
    cacheBusting: true, //一个配合devtool的配置，当给文件名插入新的hash导致清楚缓存时是否生成souce maps，默认在开发环境下为true},
    cssSourceMap: false //是否开启cssSourceMap(css资源文件遍历)

build: {<br>
// Template for index.html 编译后 html 的文件路径<br>
index: path.resolve(\_\_dirname, "../dist/index.html"),<br>

    // Paths
    // 打包后的文件根路径
    assetsRoot: path.resolve(__dirname, "../dist"),
    assetsSubDirectory: "static", //打包后的静态文件
    assetsPublicPath: "/",

    /**
     * Source Maps
     */
    productionSourceMap: true,
    // https://webpack.js.org/configuration/devtool/#production
    devtool: "#source-map",
    // Gzip off by default as many popular static hosts such as压缩程序默认关闭许多静态主机
    // Surge or Netlify already gzip all static assets for you. Surge 或者 Netlify已经压缩了静态资源
    // Before setting to `true`, make sure to: 如果想将设置为true那么请确保安装了plugin
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false, //是否压缩
    productionGzipExtensions: ["js", "css"], //需要压缩的文件的相应扩展名

    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report //是否开启打包后的分析报告}

};

</td>
</tr>

</table>
