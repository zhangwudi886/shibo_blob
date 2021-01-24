# storybook使用和避坑指南

### storybook快速安装

> 这里告诉我们一个道理，少倒腾人家内部的代码。



[大牛自动去看官网内容](https://storybook.js.org/docs/guides/guide-react/)


storybook给我们提供了各种自动化安装和人工配置
![image.png](https://cdn.nlark.com/yuque/0/2020/png/278371/1596968808977-76ea2408-ae20-49b3-8fad-899bb8deb670.png#align=left&display=inline&height=306&margin=%5Bobject%20Object%5D&name=image.png&originHeight=722&originWidth=763&size=94046&status=done&style=none&width=323)

- 是什么能让一个开发眼前一亮，头发都能激动的竖起来的玩意。就是现成，可用，no bug。
  - 解释一下上面的话
    - automatic setup，自动配置 (下面自动无视，拜拜了您嘞)
    - manual setup，手动配置
    - -
    - - 

```javascript
npx -p @storybook/cli sb init //通过这个方法初始化，心情也激动，mac也嗡嗡响。他可以自动判断是inject还是script的项目

```

> few moments later(1小时后)，发现当前工作区内多了2个文件夹，

- /.storybook (storybook的配置)
- /src/stories(storybook的demo)



> ´🙃  我们先看一下src/stories里面的2个文件，看不看都一样。

[语法介绍](https://storybook.js.org/docs/basics/writing-stories/)


- 第一种是最新的写法。
- 第二种是稳定的内置的api

![image.png](https://cdn.nlark.com/yuque/0/2020/png/278371/1596969364631-dab35862-77eb-4649-a962-dc53a6dabaac.png#align=left&display=inline&height=76&margin=%5Bobject%20Object%5D&name=image.png&originHeight=182&originWidth=744&size=33543&status=done&style=none&width=310)
官方已经表明，第二种更语义化，更符合未来es标准，因此我们使用第一种。


```javascript
import { storiesOf } from "@storybook/react";
storiesOf("xdf uilab", module).add(  //定义一级目录名，module固定添加，没有其他用处
  "welcome",       // add是增加二级目录，后面添加渲染函数，第三个是展示文档的配置项。
  () => {          // addDecorator 添加装饰器，也就是插件。
    return (       // addParameters 添加当前一级目录下面所有的内容配置参数
      <>
        <h1>欢迎来到新东方ui组件库</h1>
        <code>npm install vantd-xdflib -S</code>
      </>
    );
  },
  {
    info: {
      disable: true,
    },
  }
);
```

> 🙄  是不是文档已经会写了



   修改/.storybook/main.js

```javascript
module.exports = {
  stories: ["../src/**/*.stories.tsx"], //修改js为tsx
  addons: [
    "@storybook/preset-create-react-app",
    "@storybook/addon-actions",
    "@storybook/addon-links",
  ],
  webpackFinal: async (config) => {
    config.module.rules.push({
      test: /\.(ts|tsx)$/,
      use: [
        {
          loader: require.resolve("babel-loader"),
          options: {
            presets: [["react-app", { flow: false, typescript: true }]],
          },
        },
        {
          loader: require.resolve("react-docgen-typescript-loader"),
          options: {
            shouldExtractLiteralValuesFromEnum: true,
            propFilter: (prop) => {
              if (prop.parent) {
                return !prop.parent.fileName.includes("node_modules");
              }

              return true;
            },
          },
        },
      ],
    });
    config.resolve.extensions.push(".ts", ".tsx");
    return config;
  },
};
```


创建/.storybook/preview.tsx

```javascript
import React from "react";
import { addDecorator, addParameters, configure } from "@storybook/react";
import { withInfo } from "@storybook/addon-info";
import { library } from "@fortawesome/fontawesome-svg-core";
import { fas } from "@fortawesome/free-solid-svg-icons";
import "../src/styles/index.scss";
library.add(fas); //在文档中也需要单独引入组件库的模块
const styles: React.CSSProperties = {
  textAlign: "left",
  padding: "20px 40px",
};
// 添加整体配置，这里不很一级二级
const CenterDecorator = (storyFn: any) => (
  <div style={styles}>
    <h3>组件演示</h3>
    {storyFn()}
  </div>
);
addDecorator(withInfo);

addDecorator(CenterDecorator);

addParameters({ info: { inline: true, header: false } });
// 定义加载顺序
const loaderFn = () => {
  const allExports = [require("../src/stories/Welcome.stories.tsx")];
  const req = require.context("../src/components", true, /\.stories\.tsx$/);
  req.keys().forEach((fname) => allExports.push(req(fname)));
  return allExports;
};

configure(loaderFn, module);

```

> 如果能展示源代码的详细信息，需要使用一个插件。

```javascript
npm i @types/storybook__addon-info @storybook/addon-info -D
```


> react-docgen-typescript-loader这个loader目前是正在试用来解决下面问题的。

### readme-当前遗留问题

- 自动生成文档中，有一个配置属性，不需要添加，原生事件非自己定义之外的类型，正在找解决办法。
- ![image.png](https://cdn.nlark.com/yuque/0/2020/png/278371/1596970943315-a9f52ebe-3406-434b-8fc4-4e6ca9cce223.png#align=left&display=inline&height=110&margin=%5Bobject%20Object%5D&name=image.png&originHeight=526&originWidth=1170&size=81499&status=done&style=none&width=244)



```javascript
通过npm run storybook可以在本地运行调试文档库
通过npm run build-storybook可以打包本地的文档静态资源，发送站点到服务器即可完成线上静态站点的部署。
```