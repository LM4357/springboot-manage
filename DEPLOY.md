# 商品管理系统部署指南

> Fork 自 [zaiyunduan123/springboot-manage](https://github.com/zaiyunduan123/springboot-manage)  
> 部署环境：Linux（Ubuntu 20.04 / CentOS 7）  
> 部署者：[你的名字] | 2026-06-22

---

## 一、环境准备

### 1.1 安装 JDK 8

```bash
# Ubuntu
sudo apt update
sudo apt install openjdk-8-jdk -y

# CentOS
sudo yum install java-1.8.0-openjdk-devel -y

# 验证
java -version
```

### 1.2 安装 Maven

```bash
# Ubuntu
sudo apt install maven -y

# CentOS
sudo yum install maven -y

# 验证
mvn -v
```

### 1.3 安装 MySQL

```bash
# Ubuntu
sudo apt install mysql-server -y
sudo systemctl start mysql
sudo systemctl enable mysql

# CentOS
sudo yum install mysql-server -y
sudo systemctl start mysqld
sudo systemctl enable mysqld
```

### 1.4 安装 Redis

```bash
# Ubuntu
sudo apt install redis-server -y
sudo systemctl start redis
sudo systemctl enable redis

# CentOS
sudo yum install redis -y
sudo systemctl start redis
sudo systemctl enable redis
```

### 1.5（可选）安装 MongoDB

> ⚠️ 不使用图片上传功能可跳过。跳过不影响系统核心功能。

---

## 二、数据库初始化

### 2.1 创建数据库用户

```sql
-- 登录 MySQL
mysql -u root -p

-- 创建应用账号（最小权限原则）
CREATE USER 'shopuser'@'localhost' IDENTIFIED BY 'Shop@123456';
CREATE DATABASE jesper_shop DEFAULT CHARACTER SET utf8;
GRANT ALL PRIVILEGES ON jesper_shop.* TO 'shopuser'@'localhost';
FLUSH PRIVILEGES;
```

### 2.2 导入数据

```bash
mysql -u shopuser -pShop@123456 jesper_shop < sql/jesper_shop.sql
```

---

## 三、项目配置

### 3.1 修改数据库连接

编辑 `src/main/resources/application.yml`：

```yaml
spring:
  datasource:
    username: shopuser
    password: Shop@123456
    url: jdbc:mysql://localhost:3306/jesper_shop?useUnicode=true&characterEncoding=utf-8
```

### 3.2 确认端口

`src/main/resources/application.yml` 中 `server.port: 8080`

---

## 四、打包与启动

```bash
# 打包
mvn clean package -DskipTests

# 启动
java -jar target/jesper-0.0.1-SNAPSHOT.jar
```

### 4.1 验证部署

浏览器访问：`http://服务器IP:8080/user/login`

---

## 五、常见问题排查

| 报错 | 原因 | 排查命令 |
|------|------|---------|
| 端口 8080 被占用 | 旧 Java 进程没关 | `netstat -tlnp | grep 8080` → `kill -9 PID` |
| Access denied for user | 数据库账号/密码错误 | `mysql -u shopuser -pShop@123456` 验证是否能登录 |
| Unknown database | 没导入 SQL 或库名不对 | `mysql -u shopuser -p -e "SHOW DATABASES;"` |
| Redis 连接失败 | Redis 没启动 | `systemctl status redis` |
| java: command not found | JDK 没装或环境变量没配 | `echo $JAVA_HOME` |
| OutOfMemoryError | JVM 内存不足 | 启动加参数：`java -Xmx512m -jar xxx.jar` |

---

## 六、部署记录

| 项目 | 值 |
|------|-----|
| 部署时间 | 2026-06-22 |
| 操作系统 | Ubuntu 20.04 |
| JDK 版本 | 1.8 |
| MySQL 版本 | 8.0 |
| 应用端口 | 8080 |

---

## 七、部署后自检清单

- [ ] 数据库连接正常
- [ ] Redis 正常运行
- [ ] 浏览器能打开登录页
- [ ] 能正常登录注册
- [ ] 商品列表能加载
- [ ] 订单管理页面能打开
- [ ] 日志无严重错误（`tail -f` 确认）
