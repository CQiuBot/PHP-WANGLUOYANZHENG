# 网络验证系统

基于 PHP + MySQL 的轻量级网络验证系统，适用于各类桌面软件的卡密授权管理。

## 功能特性

- 多应用隔离（一个后台管理多个软件，卡密互不干扰）
- 卡密登录 + 设备绑定（防止一卡多机）
- 5分钟心跳检测（断网、过期、冻结即时踢出）
- 剩余时间倒计时（精确到分钟，服务端裁决）
- API 远程制卡（支持第三方系统调用）
- 后台手动解绑（卡密更换设备）
- 163 邮箱自动备份（懒触发，无需计划任务）
- 自动清理过期卡密与垃圾文件
- 请求签名（时间戳 + SHA256，防篡改、防重放）
- 兼容 PHP 5.2 ~ 7.4，MySQL 5.5

## 环境要求

- PHP 5.6+（推荐 7.4）
- MySQL 5.5+
- Web 服务器：IIS / Apache / Nginx
- 扩展：PDO_MySQL、OpenSSL（用于邮件备份）

## 目录结构

    api.php             客户端验证接口
    admin.php           管理后台
    index.php           API 文档中心
    backup.php          数据库备份与邮件发送
    auto_backup.php     懒触发自动备份
    config.php          数据库配置（不上传 Git，需手动创建）
    database.sql        数据库结构
    README.md           说明文档

## 快速开始

### 1. 导入数据库

在 phpMyAdmin 中导入 database.sql，会自动创建以下表：

- admin       管理员账号
- apps        应用列表
- cards       卡密列表
- settings    系统配置

默认管理员：admin / admin888（首次登录后请立即修改）

### 2. 创建配置文件

在网站根目录新建 config.php，内容如下：

    <?php
    define('DB_HOST', 'localhost');
    define('DB_USER', '你的数据库用户名');
    define('DB_PASS', '你的数据库密码');
    define('DB_NAME', '你的数据库名');

### 3. 上传文件

将除 config.php 外的所有文件通过 FTP 上传到网站根目录。

### 4. 登录后台

访问 http://你的域名/admin.php ，用默认账号登录并修改密码。

### 5. 创建应用

在应用管理中创建应用，获取 AppID 和 AppSecret。

### 6. 配置邮箱备份（可选）

在系统设置中填写 163 邮箱、授权码、收件邮箱。

## 客户端对接

签名算法（所有接口通用）：

1. 将所有 POST 参数（除 sign 外）按参数名 ASCII 排序
2. 拼接为 key1=value1&key2=value2&...
3. 末尾拼接 &secret=你的AppSecret
4. SHA256 加密后转大写

### 主要接口

| action | 说明 | 参数 |
|--------|------|------|
| login | 卡密登录 | appid, timestamp, card, device_id, sign |
| heartbeat | 心跳检测 | appid, timestamp, card, device_id, sign |
| query | 查询卡密 | appid, timestamp, card, sign |
| app_info | 应用信息 | appid, timestamp, sign |
| create_card | 远程制卡 | appid, timestamp, count, duration, sign |
| unbind | 解绑卡密 | appid, timestamp, card, admin_token, sign |
| stats | 卡密统计 | appid, timestamp, admin_token, sign |
| card_list | 卡密列表 | appid, timestamp, admin_token, page, limit, status, sign |
| card_action | 卡密操作 | appid, timestamp, card, op, admin_token, sign |

详细接口文档请登录后台后访问 index.php 查看，包含 Python、C#、VB.NET、VB6、Delphi、易语言、按键精灵等语言的完整示例。

## 安全建议

- config.php 永远不要上传到 Git 或公开目录
- 仓库必须设为私有
- 客户端代码编译后请用 VMP、UPX 加壳，防止 AppSecret 被逆向提取
- 定期在后台清理过期卡密（数据库仅 10MB 时尤其重要）

## 许可证

仅供学习交流使用，请勿用于非法用途。
