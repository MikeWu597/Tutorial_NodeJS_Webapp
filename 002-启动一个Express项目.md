# 启动一个Express项目

Express是一个基于Node.js的快速、开放、极简的Web应用框架[1]。它为构建Web应用和API提供了强大的功能集，同时保持了简洁和灵活性。本教程将指导您如何从零开始创建一个Express项目。

## 初始化项目

在开始编写代码之前，我们需要初始化一个新的Node.js项目。这个过程会创建一个package.json文件，它是Node.js项目的配置文件，包含项目元数据、依赖信息和脚本命令。

### 创建项目目录

首先，创建一个新的项目目录并进入该目录：

```bash
mkdir my-express-app
cd my-express-app
```

### 初始化package.json文件

在项目目录中运行以下命令来初始化项目：

```bash
npm init
```

执行该命令后，系统会提示您输入一些项目信息：
- 项目名称（name）：项目的标识符
- 版本（version）：项目的版本号，遵循语义化版本控制规范
- 描述（description）：项目的简要描述
- 入口文件（entry point）：应用程序的主文件，默认为index.js
- 测试命令（test command）：运行测试的命令
- Git仓库（git repository）：项目的Git仓库地址
- 关键词（keywords）：项目的关键词，有助于在npm中搜索
- 作者（author）：项目的作者信息
- 许可证（license）：项目的开源许可证

如果您希望使用默认值快速创建package.json文件，可以使用以下命令：

```bash
npm init -y
```

参数`-y`表示接受所有默认设置，无需手动输入。

### package.json文件的作用

package.json文件在Node.js项目中扮演着重要角色[2]：
1. **项目元数据存储**：保存项目的基本信息，如名称、版本、描述等
2. **依赖管理**：记录项目所需的各种依赖包及其版本
3. **脚本管理**：定义常用的命令行脚本，如启动、测试、构建等
4. **版本控制**：确保团队成员使用相同的依赖版本

## 安装Express框架

初始化项目后，我们需要安装Express框架。有两种安装方式：作为生产依赖和作为开发依赖。

### 作为生产依赖安装

运行以下命令将Express安装为生产依赖：

```bash
npm install express
```

或者使用简写形式：

```bash
npm i express
```

这条命令会执行以下操作：
1. 下载Express包及其依赖项
2. 将Express添加到package.json的dependencies部分
3. 在node_modules目录中安装所有文件
4. 生成或更新package-lock.json文件以锁定依赖版本

### dependencies与devDependencies的区别

Node.js项目中的依赖分为两类[3]：
- **dependencies**：生产环境所需的依赖，这些包在应用程序运行时是必需的
- **devDependencies**：仅在开发过程中需要的依赖，如测试框架、构建工具等

### package-lock.json的作用

package-lock.json文件确保在不同环境中安装完全相同的依赖版本[4]。它记录了：
- 每个依赖的确切版本号
- 依赖的依赖（嵌套依赖）的版本
- 依赖树的完整结构

## 创建第一个Express应用

现在我们创建一个简单的Express应用来测试安装是否成功。

### 创建入口文件

创建一个名为index.js的文件（或package.json中指定的入口文件），并添加以下代码：

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

代码解释：
1. `const express = require('express');`：引入Express模块
2. `const app = express();`：创建Express应用实例
3. `const port = 3000;`：定义服务器监听的端口号
4. `app.get('/', ...)`：定义一个路由处理器，处理对根路径的GET请求
5. `res.send('Hello World!');`：发送响应内容
6. `app.listen(port, ...)`：启动服务器并监听指定端口

## 配置npm脚本

为了方便启动应用，我们可以配置npm脚本。

### 修改package.json

打开package.json文件，在scripts部分添加启动脚本：

```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

### npm start命令的作用

通过配置scripts中的start字段，我们可以使用以下命令启动应用：

```bash
npm start
```

npm脚本的优势：
1. **标准化**：遵循npm生态系统约定，其他开发者能快速了解如何运行项目
2. **环境封装**：在正确的Node.js上下文中执行命令
3. **简化命令**：避免每次都输入完整的命令路径

## 运行Express应用

现在我们可以运行Express应用了。

### 启动服务器

执行以下命令启动服务器：

```bash
npm start
```

如果一切正常，您将看到控制台输出：

```
Example app listening at http://localhost:3000
```

### 测试应用

打开浏览器并访问http://localhost:3000，您应该看到页面显示"Hello World!"。

## 理解Express应用的工作流程

Express应用的基本工作流程包括以下几个步骤[1]：
1. **创建应用实例**：通过express()函数创建应用实例
2. **定义路由**：使用app.get()、app.post()等方法定义路由处理器
3. **启动服务器**：调用app.listen()方法监听指定端口
4. **处理请求**：当收到匹配的HTTP请求时，执行相应的路由处理器
5. **发送响应**：通过res对象发送响应给客户端

## 项目结构最佳实践

随着项目的发展，合理的项目结构非常重要。一个典型的Express项目结构如下：

```
my-express-app/
├── node_modules/
├── public/
│   ├── css/
│   ├── js/
│   └── images/
├── routes/
├── views/
├── app.js
├── index.js
├── package.json
└── package-lock.json
```

各目录的作用：
- **node_modules/**：存放项目依赖的第三方模块
- **public/**：存放静态资源文件，如CSS、JavaScript、图片等
- **routes/**：存放路由定义文件
- **views/**：存放视图模板文件
- **app.js**：应用配置和中间件设置
- **index.js**：应用入口文件

## 总结

通过本教程，我们学习了如何：
1. 初始化Node.js项目并创建package.json文件
2. 安装Express框架作为生产依赖
3. 理解package.json和package-lock.json的作用
4. 创建简单的Express应用
5. 配置和使用npm脚本
6. 启动和测试Express应用

这些步骤构成了创建Express Web应用的基础，为后续的开发工作奠定了基础。

## 参考文献

[1] Express.js, "Express - Node.js web application framework," 2023. [Online]. Available: https://expressjs.com/. [Accessed: Oct. 30, 2025].

[2] npm, Inc., "package.json," npm Docs, 2023. [Online]. Available: https://docs.npmjs.com/cli/v9/configuring-npm/package-json. [Accessed: Oct. 30, 2025].

[3] npm, Inc., "npm-install," npm Docs, 2023. [Online]. Available: https://docs.npmjs.com/cli/v9/commands/npm-install. [Accessed: Oct. 30, 2025].

[4] npm, Inc., "package-lock.json," npm Docs, 2023. [Online]. Available: https://docs.npmjs.com/cli/v9/configuring-npm/package-lock-json. [Accessed: Oct. 30, 2025].