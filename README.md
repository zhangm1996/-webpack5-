一、初始化
新建文件夹react-cli，初始化
npm init -y

安装webpack
yarn add webpack webpack-cli -D

新建个文件src/index.js
console.log('hello webpack')

新建 webpack.config.js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'app.js',
  },
}

package.json设置打包命令
"scripts": {
  "build": "npx webpack --config webpack.config.js"
},

因为我没有全局安装 webpack，这里使用 npx（npm5.2 开始自带 npx 命令）
执行打包命令
npm run build

就会看到生成了dist文件夹和打包生成的文件app.js
二、引入html模板
html-webpack-plugin简化了 HTML 文件的创建，该插件将为你生成一个 HTML5 文件， 其中包括使用 script 标签的 body 中的所有 webpack 包。
yarn add html-webpack-plugin -D

新建模板 HTML 文件public/index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="root">Hello</div>
  </body>
</html>

在webpack.config.js配置
module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
    }),
  ],
}

再执行npm run build，会看到生成了一个index.html文件，打开它会发现app.js也被引入到文件且能正常运行
三、编译js文件
ES6 或以上版本的语法有些浏览器还是不识别语法的，所以我们需要用 babel 对它进行编译，编译成 ES5 代码 在src/index.js写一个 ES6 的箭头函数
document.querySelector("button").onclick = () => {     
     console.log(100);     
     }; 

安装 babel 插件
yarn add babel-loader @babel/preset-env @babel/core core-js
 @babel/plugin-transform-runtime -D

babel-loader：使用 Babel 和 webpack 来转译 JavaScript 文件。
@babel/preset-env：将语法翻译成 es5 语法
@babel/core：babel 的核心模块
@babel/plugin-transform-runtime
core-js负责将新的api进行转化，例如promise、Generator、Set、Maps、Proxy等
在webpack.config.js中配置解析 js 的规则
  module: {
    rules: [
        {
            test: /\.m?js$/,
            exclude: /(node_modules|bower_components)/,
            use: 'babel-loader'
        },
    ],
  },

根目录新建.babelrc文件，配置balel
{
    "presets": [
        [
            "@babel/preset-env",
            {
              "useBuiltIns": "usage", // 按需引入，它能自动给每个文件添加其需要的polyfill
              "corejs": 3 // 根据你的版本来写
            }
          ],
    ],
    "plugins": [
      [
        "@babel/plugin-transform-runtime",
        {
            "corejs": 3,
            "helpers": true, // 开启内联的babel helpers(即babel或者环境本来的存在的垫片或者某些对象方法函数)
            "regenerator": true, // 开启generator函数转换成使用regenerator runtime来避免污染全局域
            "useESModules": false
          }
      ]
    ]
  }


执行npm run build，查看app.js已经把箭头函数编译成 ES5 语法了
document.querySelector("button").onclick=function(){console.log(100)};

后面运行时发生报错Module not found: Error: Can't resolve '@babel/runtime-corejs3...‘
安装@babel/runtime-corejs3后解决报错
yarn add @babel/runtime-corejs3

四、支持react
安装 @babel/preset-react
yarn add @babel/preset-react -D

更新.babelrc:
    "presets": [
        [
            "@babel/preset-env",
            {
              "useBuiltIns": "usage", // 按需引入，它能自动给每个文件添加其需要的polyfill
              "corejs": 3 // 根据你的版本来写
            }
          ],
      "@babel/preset-react"
    ],

五、编译css，less
安装less less-loader css-loader style-loader
yarn add less less-loader css-loader style-loader-D

六、热更新
安装devServer
yarn add webpack-dev-server -D

在``package.json`中设置启动命令
"dev": "webpack server --mode=development",

webpack.config.js配置
  devServer: {
    // 当使用 [HTML5 History API] 时，任意的 `404` 响应被替代为 `index.html`
     historyApiFallback: true,
     open: true, // 自动打开浏览器
     // 默认为true
     hot: true,
     // 是否开启代码压缩
     compress: true,
     // 启动的端口
     port: 9001,
   },


七、react-router-dom
安装react-router-dom:
yarn add react-router-dom

路由配置
import React from "react";
import ReactDOM from "react-dom";
import './common/index.css'
import './common/index.less'
import Login from './pages/login'
import Home from './pages/home'
import { BrowserRouter, Route, Routes, Navigate  } from 'react-router-dom'
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/login" element={<Login/>} />
        <Route path="/home" element={<Home/>} />
        <Route exact path="/" element={<Home/>} />
        <Route path="*" element={<Navigate to="/"/>} />
      </Routes>
    </BrowserRouter>
  );
}


注意：react-router-dom为v6版本，跟之前的版本有了不少的改变
八、Redux
安装redux，react-redux，react-thunk
yarn add redux react-redux react-thunk

Redux
状态管理工具，与React没有任何关系，其他UI框架也可以使用Redux
react-redux
React插件，作用：方便在React项目中使用Redux
react-thunk
中间件，作用：支持异步action
目录结构
|--src
    |-- store           	  Redux目录
        |-- actions.js
        |-- index.js
        |-- reducers.js
        |-- state.js
    |-- page     页面组件目录
        |-- home.jsx
    |-- App.js               项目入口


准备工作
第1步：提供默认值，既然用Redux来管理数据，那么数据就一定要有默认值，所以我们将state的默认值统一放置在state.js文件
// state.js

// 声明默认值
// 这里我们列举两个示例
// 同步数据：pageTitle
// 异步数据：infoList（将来用异步接口获取）
export default {
    pageTitle: '首页',
    infoList: []
}



第2步：创建reducer，它就是将来真正要用到的数据，我们将其统一放置在reducers.js文件
// reducers.js

// 工具函数，用于组织多个reducer，并返回reducer集合
import { combineReducers } from 'redux'
// 默认值
import defaultState from './state.js'

// 一个reducer就是一个函数
function pageTitle (state = defaultState.pageTitle, action) {
  // 不同的action有不同的处理逻辑
  switch (action.type) {
    case 'SET_PAGE_TITLE':
      return action.data
    default:
      return state
  }
}

function infoList (state = defaultState.infoList, action) {
  switch (action.type) {
    case 'SET_INFO_LIST':
      return action.data
    default:
      return state
  }
}

// 导出所有reducer
export default combineReducers({
    pageTitle,
    infoList
})





第3步：创建action，现在我们已经创建了reducer，但是还没有对应的action来操作它们，所以接下来就来编写action
// actions.js

// action也是函数
export function setPageTitle (data) {
    return (dispatch, getState) => {
      dispatch({ type: 'SET_PAGE_TITLE', data: data })
    }
  }
  
  export function setInfoList (data) {
    return (dispatch, getState) => {
      // 使用fetch实现异步请求
      window.fetch('/api/getInfoList', {
          method: 'GET',
          headers: {
              'Content-Type': 'application/json'
          }
      }).then(res => {
          return res.json()
      }).then(res => {
          let { code, data } = res
          if (code === 0) {
              dispatch({ type: 'SET_INFO_LIST', data: data })
          }
      })
    }
  }
  


最后一步：创建store实例
// index.js

// applyMiddleware: redux通过该函数来使用中间件
// createStore: 用于创建store实例
import { applyMiddleware, createStore } from 'redux'

// 中间件，作用：如果不使用该中间件，当我们dispatch一个action时，需要给dispatch函数传入action对象；但如果我们使用了这个中间件，那么就可以传入一个函数，这个函数接收两个参数:dispatch和getState。这个dispatch可以在将来的异步请求完成后使用，对于异步action很有用
import thunk from 'redux-thunk'

// 引入reducer
import reducers from './reducers.js'

// 创建store实例
let store = createStore(
  reducers,
  applyMiddleware(thunk)
)

export default store


开始使用
首先，我们来编写应用的入口文件APP.js
import React from "react";
import ReactDOM from "react-dom";
import './common/index.css'
import './common/index.less'
import Login from './pages/login'
import Home from './pages/home'
import { BrowserRouter, Route, Routes, Navigate  } from 'react-router-dom'
import { Provider } from 'react-redux'
// 引入创建好的store实例
import store from './store/index.js'
function App() {
  return (
    <Provider store = {store}>
    <BrowserRouter>
      <Routes>
        <Route path="/login" element={<Login/>} />
        <Route path="/home" element={<Home/>} />
        <Route exact path="/" element={<Home/>} />
        <Route path="*" element={<Navigate to="/"/>} />
      </Routes>
    </BrowserRouter>
    </Provider>
  );
}
ReactDOM.render(<App />, document.getElementById("root"));


最后是我们的组件 /home/index.js
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { setPageTitle, setInfoList } from '../../store/actions.js'
class Home extends Component {
    componentDidMount () {
        let { setPageTitle, setInfoList } = this.props
        
        // 触发setPageTitle action
        setPageTitle('新的标题')
        
        // 触发setInfoList action
        setInfoList()
      }
    
    render() {
         // 从props中解构store
    let { pageTitle, infoList } = this.props
        return (
            <div>
                <h1>{pageTitle}</h1>
                {
            infoList.length > 0 ? (
                <ul>
                    {
                        infoList.map((item, index) => {
                            <li>{item.data}</li>
                        })
                    }
                </ul>
            ):null
        }
            </div>
        )
    }
}

// mapStateToProps：将state映射到组件的props中
const mapStateToProps = (state) => {
  return {
    pageTitle: state.pageTitle,
    infoList: state.infoList
  }
}

// mapDispatchToProps：将dispatch映射到组件的props中
const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    setPageTitle (data) {
        // 如果不懂这里的逻辑可查看前面对redux-thunk的介绍
        dispatch(setPageTitle(data))
        // 执行setPageTitle会返回一个函数
        // 这正是redux-thunk的所用之处:异步action
        // 上行代码相当于
        /*dispatch((dispatch, getState) => {
            dispatch({ type: 'SET_PAGE_TITLE', data: data })
        )*/
    },
    setInfoList (data) {
        dispatch(setInfoList(data))
    }
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(Home)
