---
title: github actions 自动构建部署hexo blog
date: 2019-09-04 23:53:32
tags: 
  - github_actions
  - CI/CD
categories: CI/CD
---

## 配置文件

```yml
name: Deploy to gh-pages branch
on:
  push:
    branches: master

jobs:
  Build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@master

      - name: Deploy...
        uses: JamesIves/github-pages-deploy-action@master
        env:
          ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
          BRANCH: gh-pages
          FOLDER: public
          BUILD_SCRIPT: npm install && npm install hexo-cli -g && hexo generate

```