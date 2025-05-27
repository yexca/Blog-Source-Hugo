---
slug: 248
title: 'Pixiv 下载器重构记：从乱写到理解混乱'
# draft: true
author: yexca
date: '2025-05-18T16:20:33+09:00'
categories:
    - 开发实践
tags:
    - Pixiv
    - Python
    - PyQt6
---

本来想着随便写个小东西来着，用个两天就忘记 (以往大多数都是这样)，但在不出错运行的情况下节省了我好多时间，越用越顺手

慢慢就激发了自己之前的那个 “为什么不用 SQLite 呢” 的想法，确实，每次都开个 MySQL 着实有点过于麻烦了，于是这一版应运而生了，终于不用每次都开数据库服务了 ~~(也终于是人类能用的了)~~

## 使用方法

项目地址: <https://github.com/yexca/PixivDownloader-SQLite>

GUI 与上一代类似，可访问 <https://blog.yexca.net/archives/211/> 了解

### 配置说明

使用前需要先配置

* `refresh token` (Pixiv 登录验证，参考: [Pixiv OAuth Flow](https://gist.github.com/ZipFile/c9ebedb224406f4f11845ab700124362))
* 下载地址 (默认 `D:\Downloads`)

### 下载说明

之后使用只需要输入

* 画师 ID，或者
* 作品 ID (如果两个都填，只会用画师 ID)

点击下载即可下载所有作品并记录到数据库 (对于无数据记录的画师是全部作品，对于有记录的画师只会下载未下载的作品)

### 错误处理

关于程序错误，只做了爬取错误处理，如果出现错误弹窗，可能有以下问题

* 未配置 `refresh token` 或已失效
* 画师账号不存在
* 作品不存在

我未作具体错误信息提示，出错请检查这三项

对于其他错误 (直接软件退出的那种)，可以将 `程序根目录/logs/app_*-*-*.log` 最新的文件发给我并说明问题

联系方式: `PixivDownloader#yexca.net` (替换 @)

## 新特性: 从 MySQL 到 SQLite

最大修改就是不用自己整个 MySQL 了，采用了轻量的 SQLite

然后就去除了不需要的数据库配置，将设置与 Pixiv 的认证 Token 放一起了

加了个 icon (随便找 ChatGPT 画了个)，部分 UI 样式调整，改变不大

代码初步架构化，虽然写到最后又乱了 ~~不过说不定哪天我就又重新看看了呢~~

## 从旧版 MySQL 数据库迁移方法

虽然我觉得上一个没人会用，不过我还是要说明一下，因为数据库结构问题，最好的方式是查询导出直接用，查询语句

```sql
SELECT ID, name, downloadedDate, lastDownloadID, url
FROM pic 
WHERE platform = 'pixiv';
```

将结果导出为 SQL 语言 INSERT 格式 (我使用 Dataflare 支持该功能)

然后整一个 Python 文件，编入以下内容

```python
import sqlite3

conn = sqlite3.connect("pixiv.db")
cursor = conn.cursor()

cursor.execute('''
        Create Table If Not Exists pic (
               ID TEXT PRIMARY KEY,
               name TEXT,
               downloadedDate TEXT,
               lastDownloadID TEXT,
               url TEXT
               )               
               ''')

conn.commit()

cursor.execute("""
INSERT INTO `pic` (`ID`, `name`, `downloadedDate`, `lastDownloadID`, `url`) VALUES
('123', 'name1', '2024-06-08 00:00:00', '1234', 'https://www.pixiv.net/users/123'),
('234', 'name2', '2025-02-12 00:05:36', '2345', 'https://www.pixiv.net/users/234'),
('345', 'name3', '2025-01-11 17:26:17', '3456', 'https://www.pixiv.net/users/345');
"""
)

conn.commit()

cursor.execute("""
Select * from pic
"""
)

rows = cursor.fetchall()
for row in rows:
    print(row)

conn.close()
```

其中 `cursor.execute` 的内容换成您数据库的备份，这里我留了三条数据作为例子

然后将生成的数据库文件 `pixiv.db` 放到 `程序根目录/resources` 就行

## 一点开发感想: 从 "乱写" 到 "理解混乱"

说起来这次开发实际上还想着上次写的很乱，觉得必须要整改一下，结果改着改着我就知道为什么当时写的那么乱了😂

不如说这次改的反而更乱了，写到一半就是想着不重构了，直接复制过来吧，就造成了现在的驼峰命名和下划线命名混在一起，咱也不想动了，唉

大抵还是个可以运行的半成品，不过能用就行了吧~

![yexca-248](https://count.getloli.com/@yexca-248)
