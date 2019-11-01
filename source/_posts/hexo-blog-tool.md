---
title: hexo blog tool
date: 2019-11-01 19:44:01
categories:
- StudyNotes
tags:
- nodejs
- hexo
- blog
---

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
