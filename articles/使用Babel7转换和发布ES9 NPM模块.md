# 使用Babel7转换和发布ES9 NPM模块

对一个Javascript开发者来说，2018年毫无疑问是伟大的一年。特别是，我们已经看到AWS Lambda和Google Cloud Functions采用了Node.js8运行时。ES8原生异步方法async/await已经备受期待，对于使用服务端Javascript的人来说，这无疑是非常受欢迎的进步。
但是，随着ES2018（ES9）的发布，现在有更多很酷的特性可以添加到开发人员工具包中，例如：异步迭代和对象扩展属性。不幸的是，对这些功能的支持需要Node v10+。许多开发人员可以随心所欲地使用任何版本的Node，但对于其他人（例如使用AWS Lambda的），要求更加严格。

想象一下，如果您是开发人员，并且您有一个模块，您希望通过NPM与世界分享。您喜欢使用最新最好的Javascript，所以您的软件包自然会包含一些ES9特性。您如何确保您的NPM包可以被尽可能广泛的受众使用？

答案是：Babel。

Babel近年来已经陷入了相当多的困境，在某些情况下，有充分的理由。使用Babel进行调试会变得更加困难，并且它的一些功能（例如缩小）通常对服务端不利。但是，JS语言的新版本将继续发布。

在我看来，为您的项目建立一个工作流，允许您利用新功能而不强迫您的消费者升级他们的Node版本是一件好事。

幸运的是，使用Babel 7设置工作流以转换ES9代码非常简单。以下步骤将帮助您开始使用项目的框架，该项目的框架允许您使用ES9代码创建向后兼容的NPM包。

我假设您从头开始，所以第一步是创建项目文件夹并设置NPM：
```
  mkdir myProject
  cd myProject
  npm init
```
现在安装babel：
```
  npm install --save-dev @babel/cli @babel/core @babel/preset-env @babel/register
```
通过.babelrc使用以下命令在根目录中创建文件来配置Babel：
```
  {
    "presets": [
      [
        "@babel/preset-env", {
          "targets": {
            "node": "current"
          }
        }
      ]
    ]
  }
```
此配置文件告诉Babel要转换的Node版本。出于本练习的目的，我们假设您运行的是不支持ES9的Node版本，因此我们将定位当前版本。不过，您可以根据需要指定特定版本。

在项目文件夹中，创建一个src和lib文件夹：
```
  mkdir lib src
```
src文件夹将包含您的ES9代码，并将保留在您的版本控制中（例如：Git）。lib目录将包含转换后的代码，并将上传到NPM。

设置构建脚本以输出到您的lib目录：
```
  "scripts": {
    ...
    "build": "./node_modules/.bin/babel src --out-dir lib",
    ...
  }
```
NPM还支持在打包和发布包之前运行的准备脚本。您可以使用它来确保在发布时模块已经被转换了：
```
  "scripts": {
    ...
    "prepare": "npm run build",
    ...
  }
```
在package.json更新模块的入口点以指向您的lib目录。例如：
```
  {
    "name": "es9-test",
    "main": "lib/index.js",
    ...
  }
```
您也希望能够对代码运行测试（希望如此！）。如果您使用Mocha，您的测试脚本将如下所示：
```
  "scripts": {
    "test": "mocha --require @babel/register test/*.test.js",
    ...
  }
```
其中的关键部分是通过require钩子包含@babel/register。

为了防止将非转换的ES9代码上传到NPM，您还可以添加一个.npmignore排除/src/目录下的文件。同样，您可能希望将lib目录添加到gitignore来排除已转换的代码被检入版本控制。

您package.json现在应该看起来像下面这样：
```
  {
    "name": "...",
    "version": "1.0.0",
    "description": "...",
    "main": "lib/index.js",
    "scripts": {
      "test": "mocha --require @babel/register test/*.test.js",
      "build": "./node_modules/.bin/babel src --out-dir lib",
      "prepare": "npm run build"
    },
    "dependencies": {
      ...
    },
    "devDependencies": {
      "@babel/cli": "^7.1.5",
      "@babel/core": "^7.1.5",
      "@babel/preset-env": "^7.1.5",
      "@babel/register": "^7.0.0",
      "mocha": "^5.2.0"
    }
  }
```
准备就绪后，您可以通过运行npm publish命令来发布一个ES9特性编写的向后兼容的NPM模块到NPM上。当然，您也可以使用此方法来转换ES6，ES7或ES8代码。

[原文地址](https://blog.vanmulligen.ca/2018/transpiling-es9-node-modules-with-babel-7/)
