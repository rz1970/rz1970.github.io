---
title: 使用Jekyll的docker在本地部署GitHub Pages
tags: [jekyll, github pages, docker]
---

## 准备
1. jekyll的docker镜像
2. docker & docker-compose

## 制作github pages的本地镜像

基于jekyll的镜像，通过dockerfile制作github pages的镜像。由于GitHub Pages本身即为基于Jekyll开发的，所以只需要在本地安装github pages的gem即可。为了便于管理，制作docker镜像时采用了dockerfile的形式。dockfile如下：
```
FROM jekyll/jekyll
ADD Gemfile /srv/jekyll
RUN ["bundle", "install"]
```
Gemfile 文件内容如下：
```
source 'https://rubygems.org'
gem 'github-pages', group: :jekyll_plugins
```
将Dockerfile文件和Gemfile文件放置在同一目录下，执行如下命令，生成名为github-pages的本地GitHub Pages镜像
```
docker build -t github-pages .
```

## 使用docker-compose 运行 github pages
使用上文制作的github-pages镜像，结合自己的github的repo(准备作为GitHub Pages公开展示的repo),在本地运行GitHub Pages。docker-compose.yaml文件内容参考如下：
```
version: '3.2'

services:
  jekyll:
    container_name: github_pages
    image: github-pages
    volumes:
       - type: bind
         source: ./
         target: /srv/jekyll
    ports:
      - 9999:4000
    entrypoint: ["jekyll", "serve"]
```
在本文中，docker-compose.yaml 文件放置在GitHub Pages Repo的根目录下，所以source的目录指代为当前的目录。然后通过`docker-compose up`即可起来。

关于如何使用GitHub Pages，可以参考Github官方文档和Jekyll的官方文档：
1. [GitHub Pages](https://pages.github.com/)
2. [Jekyll](https://jekyllrb.com/docs/)