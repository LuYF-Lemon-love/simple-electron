# Quick Start

源教程地址：https://www.electronjs.org/docs/latest/tutorial/quick-start

>以管理员模型运行 Node.js 命令行终端。 

## Create your application

### Scaffold the project

1. 初始化：

```shell
mkdir my-electron-app && cd my-electron-app
npm init
```

`entry point` should be `main.js`.

`package.json`:

```json
{
  "name": "my-electron-app",
  "version": "1.0.0",
  "description": "Hello World!",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "luyanfeng",
  "license": "MIT"
}
```

2. 安装 `electron`：

```shell
npm install --save-dev electron
```

3. Finally, you want to be able to execute Electron. In the `scripts` field of your `package.json` config, add a `start` command like so:

```json
{
  "scripts": {
    "start": "electron ."
  }
}
```

4. This `start` command will let you open your app in development mode.

```shell
npm start
```

>Note: This script tells Electron to run on your project's root folder. At this stage, your app will immediately throw an error telling you that it cannot find an app to run.

### Run the main process

1. 创建一个空文件 `main.js`。

>Note: If you run the start script again at this point, your app will no longer throw any errors! However, it won't do anything yet because we haven't added any code into main.js.

### Create a web page

1. 创建一个文件 `index.html`。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.
  </body>
</html>
```

>Note: Looking at this HTML document, you can observe that the version numbers are missing from the body text. We'll manually insert them later using JavaScript.

### Opening your web page in a browser window

Now that you have a web page, load it into an application window. To do so, you'll need two Electron modules:

- The `app` module, which controls your application's event lifecycle.
- The `BrowserWindow` module, which creates and manages application windows.

Because the main process runs Node.js, you can import these as CommonJS modules at the top of your file:

```js
const { app, BrowserWindow } = require('electron')
```

Then, add a `createWindow()` function that loads `index.html` into a new BrowserWindow instance.

```js
const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600
  })

  win.loadFile('index.html')
}
```

Next, call this `createWindow()` function to open your window.

In Electron, browser windows can only be created after the `app` module's ready event is fired. You can wait for this event by using the `app.whenReady()` API. Call `createWindow()` after `whenReady()` resolves its Promise.

```js
app.whenReady().then(() => {
  createWindow()
})
```

>Note: At this point, your Electron application should successfully open a window that displays your web page!

### Manage your window's lifecycle

**Quit the app when all windows are closed (Windows & Linux)**

On Windows and Linux, exiting all windows generally quits an application entirely.

To implement this, listen for the `app` module's 'window-all-closed' event, and call `app.quit()` if the user is not on macOS (darwin).

```js
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})
```

**Open a window if none are open (macOS)**

Whereas Linux and Windows apps quit when they have no windows open, macOS apps generally continue running even without any windows open, and activating the app when no windows are available should open a new one.

To implement this feature, listen for the `app` module's `activate` event, and call your existing `createWindow()` method if no browser windows are open.

Because windows cannot be created before the `ready` event, you should only listen for `activate` events after your app is initialized. Do this by attaching your event listener from within your existing `whenReady()` callback.

```js
app.whenReady().then(() => {
  createWindow()

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})
```

>Note: At this point, your window controls should be fully functional!

### Access Node.js from the renderer with a preload script

1. 创建一个脚本 `preload.js`:

```js
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    if (element) element.innerText = text
  }

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency])
  }
})
```

The above code accesses the Node.js `process.versions` object and runs a basic `replaceText` helper function to insert the version numbers into the HTML document.

To attach this script to your renderer process, pass in the path to your preload script to the `webPreferences.preload` option in your existing `BrowserWindow` constructor.

```js
const { app, BrowserWindow } = require('electron')
// include the Node.js 'path' module at the top of your file
const path = require('node:path')

// modify your existing createWindow() function
const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  win.loadFile('index.html')
}
// ...
```

There are two Node.js concepts that are used here:

- The `__dirname` string points to the path of the currently executing script (in this case, your project's root folder).
- The `path.join` API joins multiple path segments together, creating a combined path string that works across all platforms.

We use a path relative to the currently executing JavaScript file so that your relative path will work in both development and packaged mode.

### Bonus: Add functionality to your web contents

At this point, you might be wondering how to add more functionality to your application.

For any interactions with your web contents, you want to add scripts to your renderer process. Because the renderer runs in a normal web environment, you can add a `<script>` tag right before your `index.html` file's closing `</body>` tag to include any arbitrary scripts you want:

```html
<script src="./renderer.js"></script>
```

The code contained in `renderer.js` can then use the same JavaScript APIs and tooling you use for typical front-end development, such as using webpack to bundle and minify your code or React to manage your user interfaces.

### Recap

The full code is available below:

```js
// main.js

// Modules to control application life and create native browser window
const { app, BrowserWindow } = require('electron')
const path = require('node:path')

const createWindow = () => {
  // Create the browser window.
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  // and load the index.html of the app.
  mainWindow.loadFile('index.html')

  // Open the DevTools.
  // mainWindow.webContents.openDevTools()
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.whenReady().then(() => {
  createWindow()

  app.on('activate', () => {
    // On macOS it's common to re-create a window in the app when the
    // dock icon is clicked and there are no other windows open.
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

// Quit when all windows are closed, except on macOS. There, it's common
// for applications and their menu bar to stay active until the user quits
// explicitly with Cmd + Q.
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})

// In this file you can include the rest of your app's specific main process
// code. You can also put them in separate files and require them here.
```

```js
// preload.js

// All the Node.js APIs are available in the preload process.
// It has the same sandbox as a Chrome extension.
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    if (element) element.innerText = text
  }

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency])
  }
})
```

```html
<!--index.html-->

<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.

    <!-- You can also require other files to run in this process -->
    <script src="./renderer.js"></script>
  </body>
</html>
```

To summarize all the steps we've done:

- We bootstrapped a Node.js application and added Electron as a dependency.
- We created a `main.js` script that runs our main process, which controls our app and runs in a Node.js environment. In this script, we used Electron's `app` and `BrowserWindow` modules to create a browser window that displays web content in a separate process (the renderer).
- In order to access certain Node.js functionality in the renderer, we attached a preload script to our `BrowserWindow` constructor.

### Package and distribute your application

The fastest way to distribute your newly created app is using `Electron Forge`.

1. Add Electron Forge as a development dependency of your app, and use its `import` command to set up Forge's scaffolding:

```shell
npm install --save-dev @electron-forge/cli
npx electron-forge import

✔ Checking your system
✔ Initializing Git Repository
✔ Writing modified package.json file
✔ Installing dependencies
✔ Writing modified package.json file
✔ Fixing .gitignore

We have ATTEMPTED to convert your app to be in a format that electron-forge understands.

Thanks for using "electron-forge"!!!
```

2. Create a distributable using Forge's `make` command:

```shell
npm run make

> my-electron-app@1.0.0 make /my-electron-app
> electron-forge make

✔ Checking your system
✔ Resolving Forge Config
We need to package your application before we can make it
✔ Preparing to Package Application for arch: x64
✔ Preparing native dependencies
✔ Packaging Application
Making for the following targets: zip
✔ Making for target: zip - On platform: darwin - For arch: x64
```

3. Electron Forge creates the `out` folder where your package will be located:

```shell
// Example for macOS
out/
├── out/make/zip/darwin/x64/my-electron-app-darwin-x64-1.0.0.zip
├── ...
└── out/my-electron-app-darwin-x64/my-electron-app.app/Contents/MacOS/my-electron-app
```