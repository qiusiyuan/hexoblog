---
title: hexo blog tool
date: 2019-11-01 19:44:01
lang: en
categories:
- StudyNotes
tags:
- nodejs
- hexo
- blog
---

[Full doc](https://hexo.io/docs/)

## Environment requirement
* nodejs
* git

## Install hexo
```bash
npm install -g hexo-cli

```
## Commands
Init project
```bash
hexo init <folder>
cd <folder>
npm install

```

New page(about me)
```bash
hexo new page --path about/me "About me"

```

New post
```bash
hexo new post "post name"

```

Generate
Generates static files.
```bash
hexo generate
hexo g

```
Clean
```bash
hexo Clean

```
Deploy
```bash
hexo d
hexo deploy

```
Serve local
```bash
hexo server

```
`http://localhost:4000/`

## Configure
All configuration in `_config.yml`

`url`: url of you blog website
`theme`: theme you want to use
`deploy`:
    `type`: git
    `repo`: git@github.com: user/user.github.io.git
    `branch`: master

## Theme
refer to [theme](https://hexo.io/docs/themes)

## Deploy
Command line
```bash
hexo Clean
hexo generate
hexo deploy

```
Travis:
https://hexo.io/docs/github-pages

## Permalinks
To create a multi-language site, you can modify the new_post_name and permalink settings like this:
```yml
new_post_name: :lang/:title.md
permalink: :lang/:title/

```
When you create a new post, the post will be saved to:
```bash
hexo new "Hello World" --lang tw
# => source/_posts/tw/Hello-World.md

```

and the URL will be:
```
http://localhost:4000/tw/hello-world/

```
