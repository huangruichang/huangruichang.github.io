# 使用 eslint 和 prettier 统一代码风格

## 前言

最近在给团队的前端项目接入代码检查工具，使用了 eslint 检查代码，prettier 格式化代码，本文记录过程中的一些总结。

## 什么是 eslint 和 prettier

ESLint 最初是由 Nicholas C. Zakas 于 2013 年 6 月创建的开源项目。它的目标是提供一个插件化的 javascript 代码检测工具。

Prettier 是一个固定的代码格式化程序，支持：

- JavaScript，包括 ES2017
- JSX
- Flow
- TypeScript
- CSS, Less 和 SCSS
- JSON
- GraphQL
- Markdown, 包含 GFM(GitHub Flavored Markdown Spec)

## 为什么接入

代码风格参差不齐，会带来以下问题：

1.  代码不美观，难以解读
2.  部分非主流写法会存在风险和性能问题，代码健壮性不能保证
3.  不同编辑环境下，代码格式化存在差异
4.  强迫症忍受不了

## eslint 和 prettier 的使用

eslint 是一个 npm 包，直接运行

```
npm install eslint --save-dev
```

eslint 的检测是基于检测规则(rules)的，本身是插件化的工具，自带一些规则，并且可以在官网文档[查询][1]，非常容易扩展新的规则。其规则和检测选项可以在官方推荐的几种配置文件格式中配置。(比如 .eslintrc.js)

比如我需要检测 react 代码，只需要

```
npm install eslint-plugin-react --save-dev
```

然后在配置文件的 extends(有时是 plugin) 上填上对应的规则插件名即可

```
/* 这是 .eslintrc.js */

module.exports = {
  extends: [
    "eslint:recommended",
    "plugin:react/recommended"
  ],
};
```

eslint 配置十分繁琐，建议直接使用别人已经配置好的规则，或在他们的基础上覆盖一些规则，如：[standard][2]、[eslint-config-alloy][3]

prettier 有多种用法，可以在主流编辑器(vscode、sublime)上找到对应插件直接使用，但是只能针对单文件，满足不了旧项目批量格式化。但是 prettier 提供函数库和命令行调用，只需要根据官方文档操作即可，如

```
/* 先安装 prettier 的命令行 */
npm install prettier -g

/* .prettierrc 是配置文件，可以配置格式化规则，比如不要分号 */
prettier --config ./src/.prettierrc --write ./src/*.js
```

如果选用了 standard 规则，还可以直接使用 prettier-standard 这个库来批量格式化代码，并且直接支持 .eslintrc.js 配置文件，就不需要再另外配置 .prettierrc 了。

## eslint 和 prettier 的差异

eslint --fix 可以修改部分代码以通过代码检测，那 prettier 跟 eslint 有什么区别？

这是最常见的问题之一，简明的回答是 elinst 只是一个代码质量工具(确保没有未使用的变量、没有全局变量，等等)。而 Prettier 只关心格式化文件(最大长度、混合标签和空格、引用样式等)。你可以将 eslint 和 prettier 结合起来使用，以获得双赢的组合。

## eslint 检测

首先接入 eslint，假设我们使用 webpack 构建前端代码，则

```
npm install eslint eslint-loader --save-dev
```

然后在项目根目录(package.json 那一级)新建和配置 .eslintrc.js 文件和 .eslintignore(忽略哪些文件)。

然后再 webpack 的构建配置文件里加入 eslint-loader 即可

```
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: [
          "babel-loader",
          "eslint-loader",
        ],
      },
    ],
  },
  // ...
}
```

假设我们基于 standard 规则来确立代码风格，首先我们应该在 eslint 上配置好 standard 检测规则。

```
npm install eslint-config-standard --save-dev
```

然后再 .eslintrc.js 里的 extends 增加 standard 即可

重新运行构建脚本，会发现原来的代码有大量报错，并且会带上规则名

![在这里输入图片描述][4]

## prettier 格式化

接下来就该 prettier 登场了。上面提到，可以使用命令行进行批量格式化，使用 prettier-eslint 或 prettier-standard 都可以，都直接识别 .eslintrc.js 进行代码格式化。

格式化过后，可能仍然存在少量报错，这时就需要根据规则名到 standard 或 eslint 官方规则页面里查询规则意义及配置方法，通过规则的取舍满足代码风格的定制化。如果有一些代码不方便改动，可以先把规则关闭，在日后处理。

满足了格式化后，结合编辑器的 prettier 插件，比如在 vscode 中，在 settings.json 里配置 prettier.eslintIntegration 选项，也可以直接根据 .eslintrc.js 文件进行代码格式化

```
{
    "editor.fontSize": 14,
    "editor.minimap.enabled": false,
    "explorer.autoReveal": false,
    "window.zoomLevel": 0,
    "prettier.eslintIntegration": true
}
```

如此一来，就能满足日常开发中随手格式化代码了。

[1]: https://cn.eslint.org/docs/rules/
[2]: https://github.com/standard/standard/blob/master/docs/README-zhcn.md
[3]: https://github.com/AlloyTeam/eslint-config-alloy
[4]: https://user-images.githubusercontent.com/4167510/82729511-af181600-9d2a-11ea-99c5-5bb156d53259.png
