#### 本项目采用vue-cli 4 搭建  使用全局样式库 全局api 全局工具库
#### 请花费十分钟阅读说明 提高你的开发效率 提高项目的可维护性
***

### 1. 目录说明
```
  ├─assets            # 静态资源 
  │  └─img 
  ├─components        # 组件库
  │  └─global         # 全局组件 <放入后不需要注册也不需要引入>
  ├─decorator         # 装饰器
  ├─filter            # 过滤器
  ├─http              # 网络请求
  ├─layouts           # layout文件
  ├─plugins           # 插件库 如需要引入第三方组件等 <推荐使用cdn>
  ├─router            # 路由
  ├─store             # vuex
  │  └─modules        # vuex 模块
  ├─style             # 样式
  │  ├─css            # css 文件
  │  └─sass           # sass 工具库样式
  ├─utils             # 工具库目录
  └─views             # 页面文件
```
### 2. 项目配置
  + 使用技术栈  vue 全家桶
  + 关闭vue esm 错误提示 (请注意你的代码标准哦!)
  + 代码格式化 eslint standard标准  [![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)
  + 网络请求使用 axios 
  + 本地存储使用 vuex 和 session (钉钉环境)
  + 高阶函数库使用 lodash
  + 时间处理器 moment
  + 数字精确计算 decimal.js
  + SDK  dingtalk-jsapi 2.8.33
  + UI 组件库 ant-design-vue: 1.4

### 3. 项目约定
  + 凡是以 $ 开头的都是挂载到vue 原型链的
  + 装饰器使用 @demo()
  + window.IsDingBrowser 用来判断 是否是钉钉浏览器环境
  + 能用 ui 库实现的样式 尽量使用 ui 库
  + lodash 引入

### 4. 项目工具
  + 项目语法使用 ES6+
  + 项目样式统一管理 style 文件夹下 **使用方法见样式库工具**
  + 正则表达式 在 **validate.js**中
  + 网络请求 由 axios 封装
  + api 统一管理在 **src/api/index.js** 添加之前 请确认 url 没有被添加过
  + 项目支持实验语法**[可选链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/%E5%8F%AF%E9%80%89%E9%93%BE)**
  + 项目支持**装饰器** 强烈推荐使用
  
### 5. 全局变量与原型链
  + window.IsDingBrowser
  + window.$SWT 全局vue 可在控制台调试使用哦
  + Vue.prototype.$apis **全局api** 已经挂载

      ``` typescript
        await this.$api.demo(params)
      ```
  + axios 拦截器 所有网络请求 理论上都走过滤 interceptor.js

### 6. 网络请求 axios拦截器配置说明
  + 提供的方法如下 请求都会经过 **JSON.parse(JSON.stringify(obj))**
  + 网络请求的数据 都会调用 **调用 toJSON 原因请看** 项目插件介绍
  + 网络拦截器可调用的方法 请在 api.js 中使用 不要在外部使用
     
    ```text
      get
      post
      binary post 上传文件 (二进制文件)
      form  post 表单
      put 上传文件
      download 下载文件
      temp 临时post 拼接URL **调试后请联系后端删除**
    ```

  + 请求格式如下
  
      ```json
        {
          "success":true,
          "result":null
        }
      ```
      
  + 拦截器配置 (http/request.js)
  
    ```
    
    // 拦截器 请求头添加token
    http.interceptors.request.use(
      config => {
        const token = session.getSession('token')
        if (token) config.headers.token = token.replace(/"/g, '')
        return config
      },
      error => {
        throw new Error(JSON.stringify(error))
      }
    )
    
    // 拦截器 相应头自动处理各类错误code，未枚举code直接toast报错提示
    http.interceptors.response.use(
      response => {
        const res = response.data
        if (!res.success) { // 逻辑错误 自定义错误码
          if (res.errorMsg) {
            ErrorMessage(res.errorMsg)
          }
          globalError('接口调用失败')
          switch (res.errorCode) {
            case 401:
            case 403:
            case 405:
            case '-3':
              // 登录失效的code，自动执行重新登录操作
              reLogin()
              break
            default:
              break
          }
          throw new Error(JSON.stringify(res))
        }
        return res.result
      },
      error => {
        // http error
        globalError(new Date(), 'err' + error) // for debug reject
        ErrorMessage(ERROR_MSG[error?.response?.status] || `连接出错(${error.response.status})!`) // 消抖
        throw new Error(JSON.stringify(error))
      }
    )
    
    // 跳转登录 并记忆当前路由参数  登录后可跳回原页面
    function reLogin () {
      session.destroy('token')
      const { hash, name, path, params, query } = window.vue.$route
      session.setSession('backRoute', { hash, name, path, params, query })
      router.push({ name: 'login' })
    }
    ```
  
  
  + 只有 success 为true 你才能获取到 result (也只有result) 不需要判断接口正确与否 success 为true 或http状态码不对则**报错终止执行**
    
    demo

      ```typescript
        @loading('someLoading')
        @confirm('确定要执行么?')
        async demoFunction (){
          this.res = await this.$api.demoGet(params)
          // xxxxx 剩余逻辑 如果接口出错 都不会执行
        }
      ``` 
    上面的@loading @confirm 是装饰器

### 7. 项目插件介绍
  + 引入了 antdv 所有的组件 
    + 通过 cdn 引入  cdn 的管理在 vue.config.js 中
    + 通过 plugins 文件夹内 引入组件
    + main.js 引入plugins内的文件
    + main.js 重写moment 对象

    ``` javascript
      // 通过在 main.js 中重写了 moment 
      // 所以日期选择插件可以不处理 直接上传给后端 如果需要修改格式的 则自己修改属性的toJSON
      moment.locale('zh-cn')
      moment.fn.toJSON = function () { return this.format('YYYY-MM-DD') }
    ```
  + antdv 原型的注入 但是不推荐使用原型的方法调用

    ```javascript
      Vue.prototype.$message = antd.message
      Vue.prototype.$notification = antd.notification
      Vue.prototype.$confirm = antd.Modal.confirm
    ```
  + 关于 $message 在utils/antdvUtils.js 中有全局的防抖
    ```javascript
      // 成功的提示
      export const SuccessMessage = debounce((msg) => { antd.message.success(msg) }, 1500, {
        leading: true,
        trailing: false
      })

      // 警告提示
      export const WarningMessage = debounce((msg) => { antd.message.warning(msg) }, 1500, {
        leading: true,
        trailing: false
      })

      // 失败提示
      export const ErrorMessage = debounce((msg) => { antd.message.error(msg) }, 1500, {
        leading: true,
        trailing: false
      })
    ```
  + 关于 $confirm 则推荐使用装饰器

### 8. 关于工具类库的介绍

  ```text
     此工具库应与业务尽量解耦
     编写文件时 应维护声明文件 (.d.ts)
  ```

  + utils/index.js 
    + 此文件应该为纯函数组件
    + 此文件导出的函数应该在任意环境下通用
    + 此文件导出的函数 尽量遵循数据不可变原则 不改变source data 而是返回一个新的 data
  + validate.js
    + 此文件是正则表达式类库
    + 此文件内正则应可维护 (业务正则可直接写入代码 不具备可重用性)
    + 此文件内除导出正则外也应导出一份校验函数 
  + ddApi.js
    + 此文件为你添加鉴权参数仅此而已
  + antdvUtils.js
    + 此文件是ant design for vue 二次封装函数库
    + 修改此函数库 应保证不影响现有代码
    + 具体说明请看antdvUtils.d.ts


### 9. webpack优化配置介绍

  + 目录
    + 配置全局cdn，包含js、css
    + 开启Gzip压缩，包含文件js、css
    + 去除生产环境console
    + 本地代理
    + 配置别名
    + 配置环境变量开发模式、测试模式、生产模式
    + 开启分析打包日志
  
  + 配置全局cdn，包含js、css 配置
  
    + vue.config.js
    
    ```
      // cdn预加载使用
      const externals = {
          vue: 'Vue',
          moment: 'moment',
          'vue-router': 'VueRouter',
          vuex: 'Vuex',
          vant: 'vant',
          axios: 'axios',
          'lodash-es': '_',
          'dingtalk-jsapi': 'dd'
      }
      // 基础注入
      const cdn = { // 将会注入index.html js 顺序不可乱 注意版本
        css: [
        ],
        js: [
          'https://g.alicdn.com/dingding/dingtalk-jsapi/2.8.33/dingtalk.open.js',
          'https://xfw-bscnym-test.oss-cn-hangzhou.aliyuncs.com/static/js/vue.min.js',
          'https://xfw-bscnym-test.oss-cn-hangzhou.aliyuncs.com/static/js/vue-router.min.js',
          'https://xfw-bscnym-test.oss-cn-hangzhou.aliyuncs.com/static/js/vuex.min.js',
          'https://xfw-bscnym-test.oss-cn-hangzhou.aliyuncs.com/static/js/axios.min.js',
          'https://xfw-bscnym-test.oss-cn-hangzhou.aliyuncs.com/static/js/moment.min.js',
          'https://xfw-bscnym-test.oss-cn-hangzhou.aliyuncs.com/static/js/moment-zh-cn.js',
          'https://xfw-bscnym-test.oss-cn-hangzhou.aliyuncs.com/static/js/lodash.min.js',
          'https://cdn.jsdelivr.net/npm/vant@2.6/lib/vant.min.js'
        ]
      }
      // 预加载 依赖变量
      config.merge({
        externals: externals
      })
      
    ```
    + html模板配置cdn
    
    ```
      // DNS预解析 提升加载效率
      <link rel="dns-prefetch" href="xfw-bscnym-test.oss-cn-hangzhou.aliyuncs.com" />
      // 动态加载 cdn 静态资源
      <% if (htmlWebpackPlugin.options.buildInfo.env === 'production') { %>
      <% for (let link of htmlWebpackPlugin.options.cdn.css) { %>
      <link rel="stylesheet" href="<%= link %>"><% } %>
      <% for (let src of htmlWebpackPlugin.options.cdn.js) { %><script src="<%= src %>"></script><% } %>
      <% } %>        
    
    ```

    + 开启Gzip压缩，包含文件js、css
    
    ``` 
        // 默认开启gzip
        config
        .plugin('CompressionPlugin')
        .use(CompressionPlugin)
        .end()
    ```
    
    + 去除生产环境console
        
    ``` 
      config.optimization.minimizer('terser').tap((args) => {
        args[0].terserOptions.compress.drop_console = true // 移除 console.log
        return args
      })
    ```
    
    + 本地代理
    ```
    devServer: {
        open: false, // 自动启动浏览器
        host: '0.0.0.0', // localhost
        port: 8080, // 端口号
        https: false,
        hotOnly: false, // 热更新
        proxy: {
            '^/sso': {
                target: process.env.VUE_APP_SSO, // 重写路径
                ws: false, //开启WebSocket
                secure: false, // 如果是https接口，需要配置这个参数
                changeOrigin: true
            }
        }
    }
    ```
    
    + 配置别名
    ```
    config.resolve.alias
    .set('@', resolve('src'))
    ```
    
    + 配置环境变量开发模式、测试模式、生产模式
    
      在根目录新建
    
      + env.dev
      ```
      NODE_ENV=development
      ```
      
      + env.prod
      ```
      NODE_ENV=production
      ```
        
      + env.test
      ```
      NODE_ENV=production  // 如果我们在.env.test文件中把NODE_ENV设置为test的话，那么打包出来的目录结构是有差异的
      ```
   
    + 开启分析打包日志
    
      安装cnpm i webpack-bundle-analyzer -D
      ```
        chainWebpack: config => {
            config
                .plugin('webpack-bundle-analyzer')
                .use(require('webpack-bundle-analyzer').BundleAnalyzerPlugin)
        }
      ```
