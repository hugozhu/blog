---
date: 2020-03-20
layout: post
title: 在iPad上写Blog
description: Write your Blog on your iPad
categories:
- Blog
tags:
- iPad

---

{:toc}

iPad的创作能力越来越强，直接修改Github上的文件就可以写Blog了，方法是在Github上建一个自动更新网站的Workflow

```
name: Blog of hugozhu.myalert.info

on:
  push:
    branches: [ master ]
jobs:
  checks:
    name: run
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        token: ${{ secrets.TOKEN }}
    - name: Set up Go
      uses: actions/setup-go@v2
      with:
        go-version: 1.16
    - name: Install gor
      run: |
        go version
        go install github.com/hugozhu/gor/gor@latest
    - name: Clone blog
      run: |
        mkdir compiled
        git clone https://github.com/hugozhu/hugozhu.github.com compiled
    - name: Compile blog
      run: |
        gor compile
    - name: Publish blog
      run: |
        cd compiled
        git status
        git config --global user.name "Hugo Zhu"
        git config --global user.email "hugozhu@gmail.com"
        git remote set-url origin https://x-access-token:${{ secrets.TOKEN }}@github.com/hugozhu/hugozhu.github.com        
        git add *
        git commit -m "Updated ${{ env.GITHUB_RUN_ID }}" .
        git push origin
```        
