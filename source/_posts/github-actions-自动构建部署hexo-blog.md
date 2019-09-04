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
name: Deploy
on:
  push:
    branches: master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - name: Use Node.js 10.x
        uses: actions/setup-node@v1
        with:
          node-version: '10.x'
      - name: Install Dependencies
        run: npm install
      - name: Install hexo-cli -g
        run: npm install hexo-cli -g
      - name: Hexo generate
        run: hexo generate
      - name: GitHub Action for hexo
        uses: heowc/action-hexo@1.0.2


```