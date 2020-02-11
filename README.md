# electron-tutorial-demo
[Electron](https://www.electronjs.org/docs/tutorial/about)是由Github开发，用HTML，CSS和JavaScript来构建跨平台桌面应用程序的一个开源库。
本demo意在展示Electron运行原生web(HTML+CSS+JS或其他框架如Vue/React)项目，并可能将其打包成在windows、linux以及macOS操作系统下的可执行应用。为今后RR可视化编程工具在linux/win平台上运行提供方案。
## Getting Start
Electron基于node.js环境，使用该库时请先[下载安装node.js](https://nodejs.org/en/)环境。
安装完成node环境后，clone本库到本地，进入项目根目录，执行
**`npm install`**
(更推荐使用`yarn`指令替代`npm`，安装依赖速度更快)。
## 简易项目结构

```sh
.
├── index.html # demo页面之一
├── next.html # demo页面之一
├── main.js # 程序主入口，是控制应用程序生命周期和创建本机浏览器窗口的模块
├── preload.js # 预加载模块，所有node.js的API可以在此模块中预加载使用。
├── css
|   └──  index.css # css样式控制文件
├── js
|   └──  index.js # js脚本运行文件
├──package.json # 配置文件，其中包含打包指令及其参数配置
├──debian.json # 配置文件，其中包含在linux平台下打包成.deb格式的配置参数
├──release-builds-linux # linux二进制文件打包输出目录(进行打包前没有该文件夹)
├──release-builds-win # .exe文件打包输出目录(进行打包前没有该文件夹)
└──release-builds-debian # .deb文件打包输出目录(进行打包前没有该文件夹)
```
## main.js说明
```js
// 控制应用程序生命周期和创建本机浏览器窗口的模块
const {app， BrowserWindow} = require('electron')
const path = require('path')

function createWindow () {  // 创建应用窗口主函数
  const mainWindow = new BrowserWindow({
    width: 800， // 窗口原始宽度
    height: 600, // 窗口原始高度
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  // 加载应用主页
  mainWindow.loadFile('index.html')

  // 打开开发者模式(可不选)
  // mainWindow.webContents.openDevTools()
}

//此方法将在Electron加载完成后调用，初始化并准备创建浏览器窗口。

app.on('ready', createWindow)

// 关闭全部窗口时退出进程
app.on('window-all-closed', function () {
//在macOS上，通常使用应用程序及其菜单栏，在用户使用Cmd+Q显式退出之前保持活动状态。
  if (process.platform !== 'darwin') app.quit()
})

// 可创建多窗口
app.on('activate', function () {
  if (BrowserWindow.getAllWindows().length === 0) createWindow()
})


```
## package.json配置说明
```
{
  "name": "electron-tutorial-app",
  "version": "1.0.0",
  "description": "A minimal Electron application",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "package-win": "electron-packager .  electron-tutorial-app --overwrite --asar=true --platform=win32 --arch=x64  --prune=true --out=release-builds-win",
    "package-linux": "electron-packager .  electron-tutorial-app --overwrite --asar=true --platform=linux --arch=x64  --prune=true --out=release-builds-linux"
  },

  "keywords": [
    "Electron",
    "tutorial",
    "demo"
  ],
  "author": "",
  "license": "",
  "devDependencies": {
    "electron": "^8.0.0",
    "electron-packager": "^14.2.1"
  }
}
```
electron和electron-packager负责运行和打包原生web应用。
`"package-win":"..."`和`"package-linux":"..."`后的参数说明:
* `electron-packager`: 打包工具库。
* `--overwrite`: 打包时替换任何现有的输出目录。
* `--asar`: app.asar文件是Electron加密打包时的中间产物，electron调用resources文件夹下的app.asar从而实现不用解压缩而直接读取文件内容,使之提高效率。
* `--platform`: 平台(linux/windows/macOS)
* `--arch`: 架构(x64/x86)
* `--prune`: 打包应用程序之前会运行`npm prune –production`, 它会删除不必要的软件包。
* `--out`: 打包结果的输出路径
还有其他参数如 `--icon:"..."`, 为打包好应用的添加logo，如果没有专门配置则采用默认logo。
## Windows下的运行和打包
安装好全部依赖后，在项目根目录执行`npm start`，将会弹出一个应用窗口。(展示作用，并非登录实际功能)
![win-app](https://s2.ax1x.com/2020/02/12/171TG6.png)
退出窗口后运行`npm run-script package-win`, 将会看到项目文件夹中多出一个release-builds-linux文件夹，进入后可找到已打包好的windows可执行程序**electron-tutorial-app.exe**

![win-app](https://s2.ax1x.com/2020/02/12/1717RK.png)
## Linux(ubuntu)下的打包运行
安装好全部依赖后，运行`npm run-script package-linux`，将会看到项目文件夹中多出一个release-builds-linux文件夹，此时内部的**electron-tutorial-app**为二进制文件，不可直接执行，需要再次编译将其转换为linux下的可执行程序 **.deb**格式。

#### Step 1. 安装lectron-installer-debian
`npm install -g electron-installer-debian`或`yarn global add electron-installer-debian`

#### Step 2. 创建配置文件debian.json
```
{
  "dest": "release-builds-debian/",
  "categories": [
    "Utility"
  ],
  "lintianOverrides": [
    "changelog-file-missing-in-native-package"
  ]
}
```
参数说明:
`dest`: **.deb**包的保存位置。
`categories`:设置菜单中显示应用程序的类别。(可以查看应用的[可用类别](https://specifications.freedesktop.org/menu-spec/latest/apa.html))
`lintianOverrides`:debian软件包检查器。
#### Step 3. 打包应用
输入指令`electron-installer-debian --src release-builds/electron-tutorial-app-linux-x64/ --arch amd64 --config debian.json`
参数说明：
`--src`: 指向打包程序保存应用程序的文件夹。
`--arch`: 告诉electron-installer-debian构建哪种架构。
`--config`:指向在Step 2中定义的配置文件。

在Terminal看到Successfully created package at release-builds-debian/则说明 **.deb**应用打包成功。
![linux-app](https://s2.ax1x.com/2020/02/12/171HxO.png)
此时在release-builds-debian文件夹下可找到引用安装包，点击安装。
![linux-app](https://s2.ax1x.com/2020/02/12/171qMD.png)
安装完成后，在应用界面找到该应用，即可跑起。

![linux-app](https://s2.ax1x.com/2020/02/12/171Lse.png)




