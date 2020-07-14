* [HOC和render-props是hooks出现前，react复用逻辑的两种方式。](https://zhuanlan.zhihu.com/p/62791765)

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



