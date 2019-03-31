---
title: 项目中使用husky、lint-staged检测代码规范性
date: 2019-03-31 11:22:29
tags:
 - git
 - 规范
 - 工具
categories: 工具
---

### 工具
#### husky
我们每次执行`git ...`相关操作时，都会触发一些`钩子`，它们存放在.git/hooks目录下，我们可以看到目录下有`commit-msg.sample`、`pre-commit.sample`等文件，这些都是案例文件，不会执行，要想执行的话把后面的`.sample`后缀去掉就可以了。

但是钩子默认是不继承的，clone下来的项目不会带有这些钩子。所以使用`husky`在 .git/hooks 中写入 pre-commit 等脚本激活钩子，在 Git 操作时触发(安装husky后会自动激活，不需要其他手动操作)

这里主要用到 pre-commit这个 hook，在执行 commit 之前，运行一些自定义操作

#### lint-staged
`参考了Git中staged 暂存区概念，在每次提交时只检查本次提交的文件。

### 为什么使用`lint-staged`？
本来在项目的package.json文件中已经有
```
"scripts": {
    "lint": "eslint --ext .js,.vue src" // 假设已经在项目中配置过eslint
 }
```
我们可以使用pre-commit这个 hook，在执行 commit 之前，运行`npm run lint`。
```
"husky": {
    "hooks": {
      "pre-commit": "npm run lint"
    }
  },
```
但是这会检查`src`目录下所有的所有`.js`和`.vue`文件（包括已经提交的），这样是性价比不高的操作，而使用`lint-staged`只会检查本次提交的代码（暂存区代码）


### 步骤

* 安装

> `$ cnpm i husky lint-staged -D`

在package.json中配置

```


 "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{js, vue}": [ 
        "eslint --fix", // 自动修复代码
        "git add" // 修复完成后执行git add
      ]
  },

  "dependencies": {
      ...
   }
```

