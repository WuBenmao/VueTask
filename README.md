# Vue3 期末作业

**gitee仓库**：https://gitee.com/WuJiaCong/vue-task.git

**github仓库**：https://github.com/WuBenmao/VueTask.git

**测试地址**：[直达网页](http://123.249.36.233:1234)

## 一、 项目简介

本项目是一个基于 **Spring Boot** (后端) 和 **Vue 3** (前端) 的移动端风格 Web 应用。采用了当下主流的“前后端分离”架构。

**主要功能：**

1. **用户注册**：包含账号查重、密码存储。
2. **用户登录**：基于数据库验证身份。
3. **个人中心**：展示登录用户的详细信息（ID、邮箱、注册时间等）。
4. **移动端 UI**：采用极简风格，适配手机屏幕布局。

## 二、 技术栈选型

### 1. 后端 (Server)

- **开发语言**：Java 17
- **核心框架**：Spring Boot 3.4.0
- **ORM 框架**：MyBatis 3.0.4 (配合注解开发)
- **数据库**：MySQL 8.0/5.7
- **工具库**：Lombok (简化实体类代码)

### 2. 前端 (Client)

- **框架**：Vue 3 (Composition API)
- **构建工具**：Vite
- **路由管理**：Vue Router 4
- **HTTP 客户端**：Axios
- **样式**：原生 CSS Flex 布局 (极简主义)

## 三、 数据库设计

表名：user

设计思路：主键自增，用户名唯一索引。

```mysql
CREATE TABLE user (
    id INT AUTO_INCREMENT PRIMARY KEY COMMENT '唯一标识',
    username VARCHAR(50) NOT NULL UNIQUE COMMENT '用户名',
    password VARCHAR(255) NOT NULL COMMENT '密码',
    email VARCHAR(100) COMMENT '邮箱',
    registration_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '注册时间'
);
```

## 四、 开发流程与实现

### 1. 后端开发 (Spring Boot)

采用标准的分层架构：

- **Entity 层**：`User` 类，对应数据库表结构，使用 `@Data` 注解。
- **Mapper 层**：`UserMapper` 接口，使用 `@Select` 和 `@Insert` 注解直接编写 SQL，无需 XML 配置。
- **Service 层**：`UserService`，封装业务逻辑（如：注册前先查重）。
- **Controller 层**：`UserController`，提供 RESTful 接口 (`/login`, `/register`)，使用 `@RequestBody` 接收 JSON 数据。

### 2. 前端开发 (Vue 3)

- **脚手架搭建**：使用 `npm create vue@latest` 初始化项目。
- **网络配置**：配置 `vite.config.js` 中的 `server.proxy`，解决开发环境跨域问题 (Port 5173 -> 8080)。
- **页面实现**：
  - `Login.vue`：登录成功后将用户信息存入 `localStorage`。
  - `Register.vue`：完成注册逻辑。
  - `Home.vue`：实现底部导航栏 Tab 切换 ("首页" / "我的")，读取本地缓存显示用户信息。

## 五、 遇到的问题与解决方案 (Troubleshooting)

在开发过程中，遇到了一些典型的环境和代码错误，以下是详细的排查与解决记录：

### 1. MySQL 版本兼容性问题

- **错误现象**：执行建表语句时报错 `Invalid default value for 'registration_time'`。
- **原因**：旧版本 MySQL 中，`DATETIME` 类型不支持直接使用 `DEFAULT CURRENT_TIMESTAMP`。
- **解决方案**：将字段类型修改为 **`TIMESTAMP`**，问题解决。

### 2. Maven 依赖版本冲突 (NoSuchMethodError)

- **错误现象**：运行 JUnit 测试时报错 `java.lang.NoSuchMethodError ... org.junit.platform...`。
- **原因**：
  1. `pom.xml` 中 Spring Boot 版本误写为 `4.0.0` (尚不存在)。
  2. 缺少 `spring-boot-starter-test` 核心测试库。
  3. IDEA 的 JUnit 运行器与项目依赖版本不匹配。
- **解决方案**：
  1. 将 Spring Boot 版本降级为稳定的 **`3.4.0`**。
  2. MyBatis 版本调整为适配 Spring Boot 3 的 **`3.0.4`**。
  3. 引入 `spring-boot-starter-test` 和 `junit-platform-launcher` 依赖。

### 3. SQL 语法拼写错误

- **错误现象**：后端测试报错 `BadSqlGrammarException`，提示 `near '= 'benmao01''`。
- **日志分析**：控制台输出的 SQL 为 `select * from userwhere username = ?`。
- **原因**：在 `@Select` 注解中，表名 `user` 和关键字 `where` 之间少写了一个空格。
- **解决方案**：修改注解为 `select * from user where ...`。

### 4. PowerShell 脚本权限限制

- **错误现象**：在 VS Code 终端运行 `npm` 命令时，提示“无法加载文件...在此系统上禁止运行脚本”。

- **原因**：Windows PowerShell 默认的安全策略限制。

- **解决方案**：以管理员身份打开 PowerShell，执行命令：

  ```
  set-executionpolicy remotesigned
  ```

### 5. Nginx 部署反向代理配置

- **场景**：上线部署时，前端运行在 1234 端口，后端 Java 运行在 6666 端口。

- **问题**：

  1. Vue 路由刷新页面报 404 错误。
  2. 前端无法访问后端接口 (跨域/连接失败)。

- **解决方案 (Nginx 配置关键点)**：

  ```
  server {
      listen 1234; # 前端端口
      # ...
  
      # 解决 Vue 路由 404
      location / {
          root /path/to/dist;
          try_files $uri $uri/ /index.html; 
      }
  
      # 解决接口转发 (反向代理)
      location /api/ {
          proxy_pass http://localhost:6666/; # 转发给后端
      }
  }
  ```

## 六、 部署总结

项目最终成功部署于宝塔面板。

1. **数据库**：导入 SQL 文件。
2. **后端**：打包为 Jar 包，通过宝塔“Java 项目”管理器启动，运行端口 `6666`。
3. **前端**：打包为 Dist 静态文件，通过 Nginx 部署，运行端口 `1234`。

通过本次实战，完整掌握了从数据库设计、后端 API 开发、前端页面交互到服务器部署的全栈开发流程。