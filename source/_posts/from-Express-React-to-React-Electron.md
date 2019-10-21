---
title: from Express-React to React-Electron
date: 2019-10-12 08:29:46
categories:
- Experience
tags:
- react
- electron
---

## Why Electron
[Electron](https://electronjs.org/) is an open-source framework that can be used to build cross platform desktop apps with JavaScript, HTML and CSS. And most importantly, you can have all the Node.js api within an Electron app so that you use plenty of poweful Node modules such as `fs`.

[React](https://reactjs.org/) is a very popular frontend framework, that make interactive UIs really easy to implement.

Express-React is currently a popular model for web application. It's acing for its simplicity and easiness to deploy. Not only production but also individuals are building a Express-React app for self-usage. In many cases, the usages are very small and for simple tasks. Imagine that if we could make them desktop apps, so that there's no server, and you only need a simple double-click and the application will start. 

And here's where the Electron framework comes in.

## Precondition
Firstly, suppose now we have an Express-React app. And our Express server exposes REST api for communication with React app. And suppose we have a good programming style that we separate our **utility** functions which do real functionality from our controllers and routers. This way, we can later ask our Electron app to directly use those functions since we are replacing Express with Electron.

## Put in Electron
To put such a project into Electron app is simple, you simply add an `electron.js` into the React app root folder with following code
```js
const { app, BrowserWindow } = require('electron');
let win = null;

function createWindow () {
  // Create the browser window.
  win = new BrowserWindow({
    width: 1296,
    height: 720,
    webPreferences: {
      nodeIntegration: true,
    }
  })
  // and load the index.html of the app.
  win.loadFile(
  "build/index.html"
  );

  // Emitted when the window is closed.
  win.on('closed', () => {
    // Dereference the window object, usually you would store windows
    // in an array if your app supports multi windows, this is the time
    // when you should delete the corresponding element.
    win = null
  })
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.on('ready', createWindow)

// Quit when all windows are closed.
app.on('window-all-closed', () => {
  // On macOS it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  // On macOS it's common to re-create a window in the app when the
  // dock icon is clicked and there are no other windows open.
  if (win === null) {
    createWindow()
  }
})

```

Notice that we ask window to load a html file from 
```js
win.loadFile(
  "build/index.html"
  );
  
```
This is where the React build file should locate and this is default by calling `react-scripts build`

And here's the **scripts** in `package.json` that I use,
```js
"scripts": {
    "react-start": "react-scripts start",
    "react-build": "react-scripts build",
    "react-test": "react-scripts test",
    "react-eject": "react-scripts eject",
    "electron-start": "electron .",
    "electron-build": "electron-builder",
    "release": "npm run react-build && electron-builder --publish=always",
    "build": "npm run react-build  && npm run electron-build",
    "file-start": "npm run react-build && npm run electron-start"
  }

```
Also add two fields into `package.json` for electron:
```js
"main": "./electron.js",
"homepage": "./",

```

I used `electron-builder` module to build electron apps. Run `npm i -save electron-builder` to install it.

To start the App for development, run `npm run file-start`. From above script, this will **build react app** first and then **start the electron app**

Notice that your App might shows nothing or can't function properly as there are still something to care about.

## BroswerRouter
Your React app, as it is transformed from an web app, must have **Routes** for multiple pages. 

If you are using `react-router-dom`, you might have problem that your page is blank when you start electron if you are using **BrowserRouter**. 

In this case we should replace **BrowserRouter** with **HashRouter**.

**BrowserRouter** will send request with the whole routes, for example, `http://myexample.com/routes/path` to server. While **HashRouter** instead, will handle the  routes in the client side. Remember that we let the electron.js to load html from file, which means there won't be a server to serve this.
```js
win.loadFile(
  "build/index.html"
  );
  
```
So we need the client to handle the routes change in this case, and we replace the **BrowserRouter** with the **HashRouter** here. 
[more info about HashRouter](https://reacttraining.com/react-router/web/api/HashRouter)

## Communication between backend
Your react app might have communication between the Express backend(or other backend) through http request.

For example, in my project, I send a http request from frontend and get the data sent back from my backend and use it to update states of my component. My backend, which also setup in my local, once get the request, will talk to my local file system, fetch the data, compose the response and then send it back to frontend. 

If your app is the similar to mine, here are a few points to make changes to fulfill our pure offline desktop app.

### IPC
Electron allows the main process talk with renderer processes through IPC (Inter-process communication) using [ipcMain](https://electronjs.org/docs/api/ipc-main) and [ipcRenderer](https://electronjs.org/docs/api/ipc-renderer).

Here, our main process is the Electron app. And the renderer process is our React app.

#### 1.To add `ipcRenderer`
If you have tried your own, you might find that below code can not be compiled within React,
```js
const ipcRenderer = require('electron').ipcRenderer

```

To use ipcRenderer, we have a workaround for React app.
Firstly, we need a new file `preload.js` with content,
```js
window.ipcRenderer = require('electron').ipcRenderer;

```
then add it in the preload field when creating window in `electron.js`,
```diff
win = new BrowserWindow({
  width: 1296,
  height: 720,
  webPreferences: {
  nodeIntegration: true,
+  preload: __dirname + '/preload.js'
 }
})

```

Now within your React code, you can access `ipcRenderer` by using **window.ipcRenderer**.

#### 2. To use `ipcMain`
Create `main_process.js` under the same folder with `electron.js`.

`ipcMain` as said in the documentation, is an instance of the EventEmitter class. We want to use it as a backend so that it should talk to the file system, fetch the data or change the data and compose the response and send it back.

Here's an example of my use of `ipcMain`, within `main_process.js`:
```js
const {ipcMain} = require('electron');
const dataUtils = require('./server/controllers/utils/dataUtils');

ipcMain.on('data', (event, arg) => {
    let data = dataUtils.list(arg);
    let args = {
        success: true,
        data: data
    }
    event.sender.send("data-reply", args);
});

```
`dataUtils` is to reuse my utility code within my Express backend. Remember in [Precondition](#Precondition) I mentioned that if you separate your utility functions which do real functionality from the controllers and routers, you can reuse those functions for your electron app.

We use `ipcMain` to handle asynchronous messages from `renderer` process, and send back args using another event. You can also check for errors and other condition handling and send corresponding message back to frontend. I used this just like the REST api that I did in my Express app. I didn't include `statusCode` as I didn't use it. What I mean is that you can use it just like a response of http request and you don't need to modify your code much in the frontend.

And we import this file as the start of our Electron app,
in `electron.js`, add,
```js
require("./main_process.js");

```

#### 3. Use `ipcRender` to talk with `ipcMain`
In React app, I used `axios` to send http request to backend. Now I should change it to use `ipcRenderer`.

Here's an example of my code change. **Before** changing it looks like,
```js
getData(){
  this.setState({
    loading: true
  });
  axios.get('http://localhost:3001/api/data')
  .then(res => {
    this.setState({
      data: res.data.data,
      loading: false,
    });
  })
}

```
**Now** we change it to,
```js
getData(){
  this.setState({
    loading: true
  });
  window.ipcRenderer.send('data', '');
  window.ipcRenderer.once("data-reply", (event, arg) => {
    this.setState({
      data: arg.data,
      loading: false,
    });
  })
}

```
Notice that I have `''` as the `argument` to send with `data` event, because I don't need any arguments in this use case. You can still send any data as the arguments to `ipcMain`, for example, 
```js
window.ipcRenderer.send('getOne', {uid: this.state.uid});

```

Also notice that to get response, I used `window.ipcRenderer.once`, this is important. **once** is a one time `listener` for the event, it will be removed after being invoked. Please be sure to use **once** instead of **on** here. Or it will hang forever since it's trying to waiting for handling all the `data-reply` events. Please consult [ipcRenderer](https://electronjs.org/docs/api/ipc-renderer) for more information.

## Data storage
Your backend might need to store data in the local file system. Remember that your Electron app is a desktop app and it can be installed in some other computers. No one wants a app storing data in some random places. So Electron provides certains places to store it.

By default, Electron stores app data in 
* `%APPDATA%` on Windows
* `$XDG_CONFIG_HOME` or `~/.config` on Linux
* `~/Library/Application Support` on macOS

A directory with your app's name will be created under this path when whether you start the electron app with `electron .` or you install it with the installer.

[app.getPath](https://electronjs.org/docs/api/app#appgetpathname) provides ways to access these paths. It also provides ways  to access other app related places.
The most commonly used and what I am using is **userData**.
Example,
```js
const { app } = require ('electron');
const path = require('path');
const modelPath = path.join(app.getPath("userData"), 'model');
let files = fs.readdirSync(modelPath);
// Do other stuff

```
This code is located in my server side `dao` folder. And utility functions mentioned above will directly call `dao` object to manipulate data in file system.

To customize those paths, you can use [app.setPath](https://electronjs.org/docs/api/app#appsetpathname-path).
Please be sure to go through both APIs before you use them.

## Build and Install
To successfully build the Electron, we need to include files dependencies. Remember we add multiple files for our projects and we need to add them into build dependencies.

Add all the used files into `"build"`,  `"files"` in the `package.json`,
```diff
  "build": {
    "appId": "name",
    "extends": null,
    "files": [
      "electron.js",
      "build/**/*",
      "node_modules/**/*",
+      "server/**/*",
+      "preload.js",
+      "main_process.js"
    ],
     "directories": {
      "buildResources": "assets"
    }
  },
```
`server/**/*` is my backend source file, it includes utility functions and their dependency functions that I reused in the electron app and this is why I put it in the build.

As shown in the `scripts` section in `package.json`, run 
```
npm run build

```
will start build both react and electron app.

As a result, built files will be located in the `dist` folder under project root. You may find a `setUp` application, which is an installer allow you install on other computers.

You may also find an `unpack` folder, in which you can find a ready-to-use application with your app's name. You can now play with it by double clicking it.

There are also many other settings for `electron builder` like setting icons or specifying platform. Please search online and there are plenty of sources for it.

Hope this blog can help you with your Electron-React app.

## References
[Electron \| Build cross platform desktop apps with JavaScript, HTML, and CSS.](https://electronjs.org/)
[React Router: Declarative Routing for React.js](https://reacttraining.com/react-router/web/api/HashRouter)
Other online sources


