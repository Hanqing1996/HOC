* [HOC和render-props是hooks出现前，react复用逻辑的两种方式。](https://zhuanlan.zhihu.com/p/62791765)
> 常在各种文章里看到的 WrappedComponent 意思是被包裹的组件，即UI组件
![](https://user-gold-cdn.xitu.io/2019/4/10/16a04a93f9a729dc?imageslim)

---
```
<容器组件>
  <UI组件>
  </UI组件>
</容器组件>
```
* [示例](https://codesandbox.io/s/cool-khayyam-7n7gl?file=/src/index.js)
```
import React from "react";
import reactDom from "react-dom";

const Demo = props => {
  return <div>这是一个测试:{props.data}</div>;
};

// WarpperComponent 是容器组件;UIComponent 是UI组件
const WarpperComponent = UIComponent => {
  return () => {
    return <UIComponent data={"测试"} />;
  };
};

const App = WarpperComponent(Demo);
reactDom.render(<App />, document.getElementById("root"));
```
*
```
import React from "react";
import reactDom from "react-dom";

const Demo = props => {
  return (
    <>
      <div> data is :{props.data}</div>
      <div> name is :{props.name}</div>
    </>
  );
};

// WarpperComponent 是容器组件;UIComponent 是UI组件
const WarpperComponent = name => UIComponent => {
  return () => {
    return <UIComponent name={name} data={"测试"} />;
  };
};

const App = WarpperComponent('libai')(Demo);
reactDom.render(<App />, document.getElementById("root"));
```
*
```
import React from "react";
import reactDom from "react-dom";

const Demo = props => {
  return (
    <>
      <div> data is :{props.data}</div>
      <div> name is :{props.name}</div>
    </>
  );
};

// WarpperComponent 是容器组件;UIComponent 是UI组件
const WarpperComponent = name => UIComponent => {
  return props => {
    console.log(props.age);
    return <UIComponent name={name} data={"测试"} />;
  };
};

const App = WarpperComponent("libai")(Demo);
reactDom.render(<App age="13" />, document.getElementById("root"));
```
---
#### [https://juejin.im/post/5e169204e51d454112714580#heading-18](具体实践)
* [页面复用](https://juejin.im/post/5e169204e51d454112714580#heading-19)
```
// HOC
import React from 'react';
const withFetchingHOC = (WrappedComponent, fetchingMethod, defaultProps) => {
  return class extends React.Component {
    async componentDidMount() {
      const data = await fetchingMethod();
      this.setState({
        data,
      });
    }
    
    render() {
    // this.props 是传递给容器组件的 props
      return (
        <WrappedComponent 
          data={this.state.data} 
          {...defaultProps} 
          {...this.props} 
        />
      );
    }
  }
}

// 使用：
// views/PageA.js
import React from 'react';
import withFetchingHOC from '../hoc/withFetchingHOC';
import fetchMovieListByType from '../lib/utils';
import MovieList from '../components/MovieList';
const defaultProps = {emptyTips: '暂无喜剧'}

export default withFetchingHOC(MovieList, fetchMovieListByType('comedy'), defaultProps);

// views/PageB.js
import React from 'react';
import withFetchingHOC from '../hoc/withFetchingHOC';
import fetchMovieListByType from '../lib/utils';
import MovieList from '../components/MovieList';
const defaultProps = {emptyTips: '暂无动作片'}

export default withFetchingHOC(MovieList, fetchMovieListByType('action'), defaultProps);;

// views/PageOthers.js
import React from 'react';
import withFetchingHOC from '../hoc/withFetchingHOC';
import fetchMovieListByType from '../lib/utils';
import MovieList from '../components/MovieList';
const defaultProps = {...}

export default withFetchingHOC(MovieList, fetchMovieListByType('some-other-type'), defaultProps);
```
* [权限控制](https://juejin.im/post/5e169204e51d454112714580#heading-20)
> 请求鉴权接口，按照不同结果返回不同内容
```
import React from 'react';
import { whiteListAuth } from '../lib/utils'; // 鉴权方法

function AuthWrapper(WrappedComponent) {
  return class AuthWrappedComponent extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        permissionDenied: -1,
      };
    }
    
    async componentDidMount() {
      try {
        await whiteListAuth(); // 请求鉴权接口
        this.setState({
          permissionDenied: 0,
        });
      } catch (err) {
        this.setState({
          permissionDenied: 1,
        });
      }
    }
    
    render() {
      if (this.state.permissionDenied === -1) {
        return null; // 鉴权接口请求未完成
      }
      if (this.state.permissionDenied) {
        return <div>功能即将上线，敬请期待~</div>;
      }
      // 这里的this.props是传递给容器组件的props，容器组件又将其传递给UI组件
      return <WrappedComponent {...this.props} />;
    }
  }
}

export default AuthWrapper;
```
```
// PageContent 为UI组件
const App=AuthWrapper(PageContent)

reactDom.render(<App background='black' type='home'/> , document.getElementById("root"));
```
---
#### 容器组件：状态和业务逻辑；UI组件：只负责根据 props 渲染UI
* 官方是这么说的
> 您可能已经注意到 HOC 与容器组件模式之间有相似之处。容器组件担任分离将高层和低层关注的责任，由容器管理订阅和状态，并将 prop 传递给处理渲染 UI。HOC 使用容器作为其实现的一部分，你可以将 HOC 视为参数化容器组件。
