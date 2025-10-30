# 编写SQLite管理器

SQLite是一个轻量级的嵌入式关系型数据库管理系统，广泛应用于Web应用和移动应用中。它不需要独立的服务器进程，数据库直接存储在磁盘文件中。本教程将指导您如何为Express应用创建一个SQLite管理器，实现数据库的初始化、点赞事件记录和查询功能，并在主程序中注册数据库服务。

## SQLite数据库基础

SQLite与其他数据库系统相比具有以下特点[1]：
1. **零配置**：无需安装和配置数据库服务器
2. **轻量级**：占用资源少，适合小型应用
3. **跨平台**：数据库文件可在不同操作系统间共享
4. **事务支持**：支持ACID事务
5. **SQL标准**：支持大部分SQL标准语法

在Node.js应用中使用SQLite需要安装相应的驱动程序。

## 安装SQLite依赖

首先，我们需要安装SQLite相关的npm包：

```bash
npm install sqlite3
npm install --save-dev @types/sqlite3  # 如果使用TypeScript
```

`sqlite3`包提供了Node.js操作SQLite数据库的API。

## 数据库初始化

数据库初始化是创建数据库连接、建立表结构的过程。我们将创建一个专门的模块来管理数据库连接和初始化。

### 创建数据库管理器目录结构

首先创建用于存放数据库相关文件的目录：

```bash
mkdir database
```

### 创建数据库管理器文件

在`database`目录下创建`dbManager.js`文件：

```javascript
const sqlite3 = require('sqlite3').verbose();
const path = require('path');

// 数据库文件路径
const dbPath = path.join(__dirname, '..', 'database.db');

// 创建数据库实例
const db = new sqlite3.Database(dbPath, (err) => {
  if (err) {
    console.error('数据库连接失败:', err.message);
  } else {
    console.log('数据库连接成功:', dbPath);
  }
});

// 初始化数据库表
function initializeDatabase() {
  // 创建点赞记录表
  const createThumbsupTable = `
    CREATE TABLE IF NOT EXISTS thumbsup_records (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      ip_address TEXT NOT NULL,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
  `;

  // 执行表创建语句
  db.serialize(() => {
    db.run(createThumbsupTable, (err) => {
      if (err) {
        console.error('创建点赞记录表失败:', err.message);
      } else {
        console.log('点赞记录表创建成功');
      }
    });
  });
}

// 导出数据库实例和初始化函数
module.exports = {
  db,
  initializeDatabase
};
```

代码解释：
1. `require('sqlite3').verbose()`：引入sqlite3模块并启用详细模式
2. `new sqlite3.Database(dbPath, callback)`：创建数据库连接实例
3. `CREATE TABLE IF NOT EXISTS`：创建表，如果表已存在则不执行创建
4. `thumbsup_records`表包含三个字段：
   - `id`：主键，自动递增
   - `ip_address`：记录用户IP地址
   - `created_at`：记录创建时间戳
5. `db.serialize()`：确保SQL语句按顺序执行
6. `db.run()`：执行SQL语句

## 实现点赞功能

我们将实现两个核心功能：记录点赞事件和查询指定IP的点赞记录。

### 创建点赞操作模块

在`database`目录下创建`thumbsupModel.js`文件：

```
const { db } = require('./dbManager');

// 记录点赞事件
function addThumbsup(ipAddress, callback) {
  const sql = `INSERT INTO thumbsup_records (ip_address) VALUES (?)`;
  
  db.run(sql, [ipAddress], function(err) {
    if (err) {
      callback(err, null);
    } else {
      callback(null, { id: this.lastID, ip_address: ipAddress, created_at: new Date().toISOString() });
    }
  });
}

// 根据IP地址查询点赞记录
function getThumbsupByIP(ipAddress, callback) {
  const sql = `SELECT * FROM thumbsup_records WHERE ip_address = ? ORDER BY created_at DESC`;
  
  db.all(sql, [ipAddress], (err, rows) => {
    if (err) {
      callback(err, null);
    } else {
      callback(null, rows);
    }
  });
}

// 获取所有点赞记录
function getAllThumbsup(callback) {
  const sql = `SELECT * FROM thumbsup_records ORDER BY created_at DESC`;
  
  db.all(sql, [], (err, rows) => {
    if (err) {
      callback(err, null);
    } else {
      callback(null, rows);
    }
  });
}

// 获取指定时间范围内的点赞记录数
function getThumbsupCount(since, callback) {
  const sql = `SELECT COUNT(*) as count FROM thumbsup_records WHERE created_at >= ?`;
  
  db.get(sql, [since], (err, row) => {
    if (err) {
      callback(err, null);
    } else {
      callback(null, row ? row.count : 0);
    }
  });
}

// 导出所有点赞操作函数
module.exports = {
  addThumbsup,
  getThumbsupByIP,
  getAllThumbsup,
  getThumbsupCount
};
```

### 函数详细说明

1. **addThumbsup(ipAddress, callback)**：
   - 功能：记录用户的点赞事件
   - 参数：[ipAddress] - 用户的IP地址
   - 回调：成功时返回包含记录ID、IP地址和创建时间的对象，失败时返回错误信息

2. **getThumbsupByIP(ipAddress, callback)**：
   - 功能：查询指定IP地址的所有点赞记录
   - 参数：[ipAddress] - 要查询的IP地址
   - 回调：成功时返回该IP的所有点赞记录数组，失败时返回错误信息

3. **getAllThumbsup(callback)**：
   - 功能：获取所有点赞记录
   - 回调：成功时返回所有点赞记录数组，失败时返回错误信息

4. **getThumbsupCount(since, callback)**：
   - 功能：获取指定时间之后的点赞记录总数
   - 参数：[since] - 时间戳，查询此时间之后的记录数
   - 回调：成功时返回记录数，失败时返回错误信息

## 在主程序中注册服务

为了让数据库服务在整个应用中可用，我们需要在主程序中初始化数据库并注册相关服务。

### 更新主应用文件

修改项目根目录下的`index.js`文件：

```javascript
const express = require('express');
const path = require('path');
const timeRoutes = require('./routes/api/time');
const { initializeDatabase } = require('./database/dbManager');

const app = express();
const port = 3000;

// 初始化数据库
initializeDatabase();

// 中间件：解析JSON请求体
app.use(express.json());

// 提供静态文件服务
app.use(express.static('public'));

// 添加API路由
app.use('/api', timeRoutes);

// 处理根路径请求，返回主页
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

// 错误处理中间件
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: '服务器内部错误' });
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});

// 应用关闭时关闭数据库连接
process.on('exit', () => {
  console.log('应用关闭，数据库连接已断开');
});
```

### 创建点赞API路由

为了测试数据库功能，我们需要创建API路由来处理点赞相关操作。在`routes/api/`目录下创建`thumbsup.js`文件：

```javascript
const express = require('express');
const router = express.Router();
const { 
  addThumbsup, 
  getThumbsupByIP,
  getAllThumbsup,
  getThumbsupCount
} = require('../../database/thumbsupModel');

// 点赞接口
router.post('/thumbsup', (req, res) => {
  // 获取客户端IP地址
  const ipAddress = req.headers['x-forwarded-for'] || 
                   req.connection.remoteAddress || 
                   req.socket.remoteAddress ||
                   (req.connection.socket ? req.connection.socket.remoteAddress : null) ||
                   'unknown';
  
  // 记录点赞事件
  addThumbsup(ipAddress, (err, record) => {
    if (err) {
      return res.status(500).json({ error: '记录点赞失败' });
    }
    res.status(201).json({ message: '点赞成功', record });
  });
});

// 查询指定IP的点赞记录
router.get('/thumbsup/:ip', (req, res) => {
  const ipAddress = req.params.ip;
  
  getThumbsupByIP(ipAddress, (err, records) => {
    if (err) {
      return res.status(500).json({ error: '查询点赞记录失败' });
    }
    res.json({ ip_address: ipAddress, records });
  });
});

// 获取所有点赞记录（仅用于测试）
router.get('/thumbsup', (req, res) => {
  getAllThumbsup((err, records) => {
    if (err) {
      return res.status(500).json({ error: '获取点赞记录失败' });
    }
    res.json(records);
  });
});

// 获取24小时内的点赞总数
router.get('/thumbsup-count', (req, res) => {
  const since = new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString();
  
  getThumbsupCount(since, (err, count) => {
    if (err) {
      return res.status(500).json({ error: '获取点赞数失败' });
    }
    res.json({ count, since });
  });
});

// 导出路由器
module.exports = router;
```

### 更新主应用以包含点赞API

更新`index.js`文件以包含新的点赞API路由：

```javascript
const express = require('express');
const path = require('path');
const timeRoutes = require('./routes/api/time');
const thumbsupRoutes = require('./routes/api/thumbsup');
const { initializeDatabase } = require('./database/dbManager');

const app = express();
const port = 3000;

// 初始化数据库
initializeDatabase();

// 中间件：解析JSON请求体
app.use(express.json());

// 提供静态文件服务
app.use(express.static('public'));

// 添加API路由
app.use('/api', timeRoutes);
app.use('/api', thumbsupRoutes);

// 处理根路径请求，返回主页
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

// 错误处理中间件
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: '服务器内部错误' });
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});

// 应用关闭时关闭数据库连接
process.on('exit', () => {
  console.log('应用关闭，数据库连接已断开');
});
```

## 测试点赞功能

### 启动服务器

确保所有文件都已正确配置后，启动Express应用：

```bash
npm start
```

### 测试点赞API

使用curl命令或Postman测试点赞API：

1. **点赞**：
```
curl -X POST http://localhost:3000/api/thumbsup
```

2. **查询指定IP的点赞记录**：
```
curl http://localhost:3000/api/thumbsup/::1
```

3. **获取所有点赞记录**：
```
curl http://localhost:3000/api/thumbsup
```

4. **获取24小时内的点赞总数**：
```
curl http://localhost:3000/api/thumbsup-count
```

## 数据库管理器最佳实践

### 1. 连接管理

- 在应用启动时初始化数据库连接
- 在应用关闭时正确关闭数据库连接
- 使用连接池处理高并发场景

### 2. 错误处理

- 捕获并处理数据库操作异常
- 返回有意义的错误信息给客户端
- 记录错误日志便于调试

### 3. 数据验证

- 在插入数据前进行验证
- 使用参数化查询防止SQL注入
- 实施数据完整性约束

### 4. 性能优化

- 为常用查询字段添加索引
- 避免N+1查询问题
- 合理使用分页处理大量数据

### 5. 安全性

- 不在代码中硬编码敏感信息
- 实施适当的身份验证和授权机制
- 定期备份重要数据

## 总结

通过本教程，我们学习了如何：

1. 安装和配置SQLite数据库
2. 创建数据库管理器实现初始化功能
3. 实现点赞事件记录和查询功能
4. 在主程序中注册数据库服务
5. 创建API路由测试数据库功能
6. 应用数据库操作的最佳实践

这些技能为构建数据驱动的Web应用奠定了基础，使您的应用能够持久化存储和管理点赞数据。

## 参考文献

[1] SQLite, "SQLite Database Management System," 2023. [Online]. Available: https://www.sqlite.org/index.html. [Accessed: Oct. 30, 2025].