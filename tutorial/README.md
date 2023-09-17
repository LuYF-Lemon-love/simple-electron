# Tutorial

源教程地址：

1. [Prerequisites](https://www.electronjs.org/docs/latest/tutorial/tutorial-prerequisites)
2. [Building your First App](https://www.electronjs.org/docs/latest/tutorial/tutorial-first-app)
3. [Using Preload Scripts](https://www.electronjs.org/docs/latest/tutorial/tutorial-preload)

## Prerequisites

Electron is a framework for building desktop applications using JavaScript, HTML, and CSS. By embedding Chromium and Node.js into a single binary file, Electron allows you to create cross-platform apps that work on Windows, macOS, and Linux with a single JavaScript codebase.

This tutorial will guide you through the process of developing a desktop application with Electron and distributing it to end users.

### Goals

This tutorial starts by guiding you through the process of piecing together a minimal Electron application from scratch, then teaches you how to package and distribute it to users using Electron Forge.

If you prefer to get a project started with a single-command boilerplate, we recommend you start with Electron Forge's `create-electron-app` command.

### Assumptions

Electron is a native wrapper layer for web apps and is run in a Node.js environment. Therefore, this tutorial assumes you are generally familiar with Node and front-end web development basics. If you need to do some background reading before continuing, we recommend the following resources:

1. [Getting started with the Web (MDN Web Docs)](https://developer.mozilla.org/en-US/docs/Learn/)
2. [Introduction to Node.js](https://nodejs.dev/en/learn/)

### Required tools

**Code editor**

You will need a text editor to write your code. We recommend using `Visual Studio Code`, although you can choose whichever one you prefer.

**Command line**

Throughout the tutorial, we will ask you to use various command-line interfaces (CLIs). You can type these commands into your system's default terminal:

- Windows: Command Prompt or PowerShell
- macOS: Terminal
- Linux: varies depending on distribution (e.g. GNOME Terminal, Konsole)

Most code editors also come with an integrated terminal, which you can also use.

**Git and GitHub**

Git is a commonly-used version control system for source code, and GitHub is a collaborative development platform built on top of it. Although neither is strictly necessary to building an Electron application, we will use GitHub releases to set up automatic updates later on in the tutorial. Therefore, we'll require you to:

- Create a GitHub account
- Install Git

If you're unfamiliar with how Git works, we recommend reading GitHub's Git guides. You can also use the GitHub Desktop app if you prefer using a visual interface over the command line.

We recommend that you create a local Git repository and publish it to GitHub before starting the tutorial, and commit your code after every step.

>INSTALLING GIT VIA GITHUB DESKTOP
>
>GitHub Desktop will install the latest version of Git on your system if you don't already have it installed.

**Node.js and npm**

To begin developing an Electron app, you need to install the Node.js runtime and its bundled npm package manager onto your system. We recommend that you use the latest long-term support (LTS) version.

>Please install Node.js using pre-built installers for your platform. You may encounter incompatibility issues with different development tools otherwise. If you are using macOS, we recommend using a package manager like Homebrew or nvm to avoid any directory permission issues.

To check that Node.js was installed correctly, you can use the `-v` flag when running the `node` and `npm` commands. These should print out the installed versions.

```shell
$ node -v
v16.14.2
$ npm -v
8.7.0
```

>CAUTION
>
>Although you need Node.js installed locally to scaffold an Electron project, **Electron does not use your system's Node.js installation to run its code.** Instead, it comes bundled with its own Node.js runtime. This means that your end users do not need to install Node.js themselves as a prerequisite to running your app.
>
>To check which version of Node.js is running in your app, you can access the global `process.versions` variable in the main process or preload script. You can also reference https://releases.electronjs.org/releases.json.

## Building your First App

### Learning goals

In this part of the tutorial, you will learn how to set up your Electron project and write a minimal starter application. By the end of this section, you should be able to run a working Electron app in development mode from your terminal.

### Setting up your project

>AVOID WSL
>
>If you are on a Windows machine, please do not use Windows Subsystem for Linux (WSL) when following this tutorial as you will run into issues when trying to execute the application.

**Initializing your npm project**

Electron apps are scaffolded using npm, with the package.json file as an entry point. Start by creating a folder and initializing an npm package within it with `npm init`.

```shell
mkdir my-electron-app && cd my-electron-app
npm init
```

This command will prompt you to configure some fields in your package.json. There are a few rules to follow for the purposes of this tutorial:

- `entry point` should be `main.js` (you will be creating that file soon).
- `author`, `license`, and `description` can be any value, but will be necessary for packaging later on.

Then, install Electron into your app's devDependencies, which is the list of external development-only package dependencies not required in production.

>WHY IS ELECTRON A DEVDEPENDENCY?
>
>This may seem counter-intuitive since your production code is running Electron APIs. However, packaged apps will come bundled with the Electron binary, eliminating the need to specify it as a production dependency.

```shell
npm install electron --save-dev
```

Your package.json file should look something like this after initializing your package and installing Electron. You should also now have a `node_modules` folder containing the Electron executable, as well as a `package-lock.json` lockfile that specifies the exact dependency versions to install.

```json
{
  "name": "my-electron-app",
  "version": "1.0.0",
  "description": "Hello World!",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Jane Doe",
  "license": "MIT",
  "devDependencies": {
    "electron": "23.1.3"
  }
}
```

>ADVANCED ELECTRON INSTALLATION STEPS
>
>If installing Electron directly fails, please refer to our Advanced Installation documentation for instructions on download mirrors, proxies, and troubleshooting steps.

**Adding a .gitignore**

The `.gitignore` file specifies which files and directories to avoid tracking with Git. You should place a copy of [GitHub's Node.js gitignore template](https://github.com/github/gitignore/blob/main/Node.gitignore) into your project's root folder to avoid committing your project's `node_modules` folder.

### Running an Electron app

>FURTHER READING
>
>Read Electron's process model documentation to better understand how Electron's multiple processes work together.

The `main` script you defined in package.json is the entry point of any Electron application. This script controls the `main process`, which runs in a Node.js environment and is responsible for controlling your app's lifecycle, displaying native interfaces, performing privileged operations, and managing renderer processes (more on that later).

Before creating your first Electron app, you will first use a trivial script to ensure your main process entry point is configured correctly. Create a `main.js` file in the root folder of your project with a single line of code:

```js
console.log('Hello from Electron 👋')
```

Because Electron's main process is a Node.js runtime, you can execute arbitrary Node.js code with the electron command (you can even use it as a REPL). To execute this script, add `electron .` to the `start` command in the scripts field of your package.json. This command will tell the Electron executable to look for the main script in the current directory and run it in dev mode.

```json
{
  "name": "my-electron-app",
  "version": "1.0.0",
  "description": "Hello World!",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Jane Doe",
  "license": "MIT",
  "devDependencies": {
    "electron": "23.1.3"
  }
}
```

```shell
npm run start
```

Your terminal should print out Hello from Electron 👋. Congratulations, you have executed your first line of code in Electron! Next, you will learn how to create user interfaces with HTML and load that into a native window.

### Loading a web page into a BrowserWindow

In Electron, each window displays a web page that can be loaded either from a local HTML file or a remote web address. For this example, you will be loading in a local file. Start by creating a barebones web page in an `index.html` file in the root folder of your project:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta
      http-equiv="Content-Security-Policy"
      content="default-src 'self'; script-src 'self'"
    />
    <meta
      http-equiv="X-Content-Security-Policy"
      content="default-src 'self'; script-src 'self'"
    />
    <title>Hello from Electron renderer!</title>
  </head>
  <body>
    <h1>Hello from Electron renderer!</h1>
    <p>👋</p>
  </body>
</html>
```

Now that you have a web page, you can load it into an Electron `BrowserWindow`. Replace the contents of your `main.js` file with the following code. We will explain each highlighted block separately.

```js
const { app, BrowserWindow } = require('electron')

const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600
  })

  win.loadFile('index.html')
}

app.whenReady().then(() => {
  createWindow()
})
```

**Importing modules**

```js
const { app, BrowserWindow } = require('electron')
```

In the first line, we are importing two Electron modules with CommonJS module syntax:

- `app`, which controls your application's event lifecycle.
- `BrowserWindow`, which creates and manages app windows.

>CAPITALIZATION CONVENTIONS
>
>You might have noticed the capitalization difference between the app and BrowserWindow modules. Electron follows typical JavaScript conventions here, where PascalCase modules are instantiable class constructors (e.g. BrowserWindow, Tray, Notification) whereas camelCase modules are not instantiable (e.g. app, ipcRenderer, webContents).
>
>ES MODULES IN ELECTRON
>
>ECMAScript modules (i.e. using import to load a module) are currently not directly supported in Electron. You can find more information about the state of ESM in Electron in electron/electron#21457.

**Writing a reusable function to instantiate windows**

The `createWindow()` function loads your web page into a new BrowserWindow instance:

```js
const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600
  })

  win.loadFile('index.html')
}
```

**Calling your function when the app is ready**

```js
app.whenReady().then(() => {
  createWindow()
})
```

Many of Electron's core modules are Node.js event emitters that adhere to Node's asynchronous event-driven architecture. The app module is one of these emitters.

In Electron, BrowserWindows can only be created after the app module's ready event is fired. You can wait for this event by using the `app.whenReady()` API and calling `createWindow()` once its promise is fulfilled.

>INFO
>You typically listen to Node.js events by using an emitter's .on function.
>
>```js
>+ app.on('ready', () => {
>- app.whenReady().then(() => {
>  createWindow()
>})
>```
>
>However, Electron exposes `app.whenReady()` as a helper specifically for the `ready` event to avoid subtle pitfalls with directly listening to that event in particular. See electron/electron#21972 for details.

At this point, running your Electron application's start command should successfully open a window that displays your web page!

Each web page your app displays in a window will run in a separate process called a renderer process (or simply renderer for short). Renderer processes have access to the same JavaScript APIs and tooling you use for typical front-end web development, such as using webpack to bundle and minify your code or React to build your user interfaces.

### Managing your app's window lifecycle

Application windows behave differently on each operating system. Rather than enforce these conventions by default, Electron gives you the choice to implement them in your app code if you wish to follow them. You can implement basic window conventions by listening for events emitted by the app and BrowserWindow modules.

>PROCESS-SPECIFIC CONTROL FLOW
>
>Checking against Node's `process.platform` variable can help you to run code conditionally on certain platforms. Note that there are only three possible platforms that Electron can run in: `win32` (Windows), `linux` (Linux), and `darwin` (macOS).

**Quit the app when all windows are closed (Windows & Linux)**

On Windows and Linux, closing all windows will generally quit an application entirely. To implement this pattern in your Electron app, listen for the app module's `window-all-closed` event, and call `app.quit()` to exit your app if the user is not on macOS.

```js
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})
```

**Open a window if none are open (macOS)**

In contrast, macOS apps generally continue running even without any windows open. Activating the app when no windows are available should open a new one.

To implement this feature, listen for the app module's `activate` event, and call your existing `createWindow()` method if no BrowserWindows are open.

Because windows cannot be created before the ready event, you should only listen for `activate` events after your app is initialized. Do this by only listening for activate events inside your existing `whenReady()` callback.

```js
app.whenReady().then(() => {
  createWindow()

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})
```

### Final starter code

`main.js`

```js
const { app, BrowserWindow } = require('electron')

const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600
  })

  win.loadFile('index.html')
}

app.whenReady().then(() => {
  createWindow()

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow()
    }
  })
})

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})
```

`index.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta
      http-equiv="Content-Security-Policy"
      content="default-src 'self'; script-src 'self'"
    />
    <meta
      http-equiv="X-Content-Security-Policy"
      content="default-src 'self'; script-src 'self'"
    />
    <title>Hello from Electron renderer!</title>
  </head>
  <body>
    <h1>Hello from Electron renderer!</h1>
    <p>👋</p>
    <p id="info"></p>
  </body>
  <script src="./renderer.js"></script>
</html>
```

### Summary

Electron applications are set up using npm packages. The Electron executable should be installed in your project's devDependencies and can be run in development mode using a script in your package.json file.

The executable runs the JavaScript entry point found in the main property of your package.json. This file controls Electron's `main process`, which runs an instance of Node.js and is responsible for your app's lifecycle, displaying native interfaces, performing privileged operations, and managing renderer processes.

`Renderer processes` (or renderers for short) are responsible for displaying graphical content. You can load a web page into a renderer by pointing it to either a web address or a local HTML file. Renderers behave very similarly to regular web pages and have access to the same web APIs.

In the next section of the tutorial, we will be learning how to augment the renderer process with privileged APIs and how to communicate between processes.

## Using Preload Scripts