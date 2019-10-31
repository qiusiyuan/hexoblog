---
title: gitpod and nodejs example
date: 2019-10-31 12:24:07
categories: 
- Experience
tags:
- nodejs
- gitpod
- react
- docker
---

## What is gitpod

Gitpod is a cloud editor integrated with Github naturally. It's using github.io for its service. It's using docker image for environment and workspace setup. The IDE is open-source and based on Eclipse Theia. Theia is highly extensible and builds upon mature technologies such as TypeScript, VS Code, Webpack, and Node.js. There are many interesting facts about Gitpod, please refer to [Gitpod - Introduction](https://www.gitpod.io/docs/) for more info.

My first view of this online editor is WOW! Imagine that you have your github source code and at one time you switched to a new computer then you have to setup the whole development environment for your project. Which with this service, you just need a few settings for your environment and a single-click, you will get a ready-to-code dev environment everywhere you have a browser.

Keyword for this service:
* VS code
* github
* Docker

**To use gitpod**, you are a big fan to VS code among all the other editors, or you would like to switch to simple but powerful editor supporting multiple popular languages including C++, Java, Javascript, Python, go and etc. And you think direct editing on github is troublesome and you definitely would like to edit your project online with proper devlopment environment setup so that you can test your changes. Then gitpod is for you. It's recommended but not required that you know how to setup a docker image.

## How does it look like?
![gitpod example](https://user-images.githubusercontent.com/17970730/67916891-a339ec00-fb6d-11e9-9094-6c81afc7fc9d.png)

Yes, it looks just like VS code, but it shows in your browser! 

To open a workspace for your github project, you just need to  prefix the URL in the address bar of your browser with https://gitpod.io/# for example, https://gitpod.io/#https://github.com/{you}/{project}. Really simple right? But I recommend you to install browser extension for gitpod, so that you just need a single-click to start it. [Instruction on Extension](https://www.gitpod.io/docs/20_browser_extension/)

## Dev Environment Setup
Once you click a project, it will launch gitpod for your project. By default without any configuration, it will analyze your project and prepare a IDE for you. For my node.js project without any configuration, they gives me a workspace with `Node` and `npm` installed. Here's what I checked for those environment,
```bash
$ node -v
v13.0.1

$ npm -v
6.12.0

```
Notice that I didn't make any configuration for gitpod so far. This is really smart and convenient. And this is why if you don't know docker you can still use gitpod.

Here, I would like to show a example of my configuration for my React project and I think it's generic for other nodejs project.

## Example
### Add .gitpod.yml
Gitpod use .gitpod.yml for configuration. For my React project  [GitHub - qiusiyuan/TRUX-Ncrypter](https://github.com/qiusiyuan/TRUX-Ncrypter),
```yml
# The Docker image to run your workspace in. Defaults to gitpod/workspace-full
image: node:alpine
# Command to start on workspace startup (optional)
tasks:
  - init: npm install

# Ports to expose on workspace startup (optional)
ports:
  - port: 3000
    onOpen: open-preview
    
```
I choose `node：alpine` for my base image, this is enough for me. And `alpine` is known for it's small size and no redundency. If you want to build your own image, you can use,
```yml
image:
  file: .gitpod.dockerfile
  
```
For task, right after workspace is loaded, I ask for a `npm install` to install dependency. Notice that using `- init` will only be run during initialization. [Here](https://www.gitpod.io/docs/44_config_start_tasks/#start-tasks) are other command that you can specify.

I also specify my exposing port to `3000` which is my React app will listen to. You **don't** have specify this since once a service is started on a port, gitpod will automatically notify you.

### Run my app
After initialization is done, dependencies are properly installed. Then I start my React app in the VS code terminal,
```bash
$ npm run react-start

```
A preview window opened once the service is started and shows the content of my app.
![67915010-25271680-fb68-11e9-95c7-51ca5a3f9e83.png](https://user-images.githubusercontent.com/17970730/67915010-25271680-fb68-11e9-95c7-51ca5a3f9e83.png)

See this is awesome!

### Git commit and push
To do git commit and push, you can just use the terminal for commanding. Or just use VS code git tool. Notice that you may be asked for grant write permissions to your github.

## Conclustion
Gitpod is really convenient and powerful, and the configuration is direct and simple. This is benefial for not only individual programmers but also for enterprises or a group. Imagine that a group of people work on the same project but with different asset( Windows, Linux, Mac). People and new comers used to have to spend a lot of time on dev environment setup. But for now, it's just a single-click for everyone. I think such a cloud editor like git pod is the future. VS code is gathering its large community, docker and kubernete is very popular in current cloud industry, github is one of the most popular version control service. Gitpod combine those together, this is the trend. Modern technologies are mature and convenient. Programmers are tend to build better application by using those technologies instead of building from wheels. Programmers are changing their programming style and work environments. Programming should become more convenient and simple. I'm really eager to see future development of such an application. 

