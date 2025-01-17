# wechat_spider 微信爬虫

基于 Node.js 的微信爬虫，通过中间人代理的原理，批量获取微信文章数据，包括阅读量、点赞量、在看数、评论和文章正文等数据，并且支持根据阅读数量进行排序

使用代理模块 AnyProxy。代码已支持 AnyProxy 4 版本。

支持 Docker 部署。

项目可运行在个人电脑上，也可部署在服务器上。

## 开始

### 安装前准备

- 安装 Node，推荐版本 16
- 安装 MongoDB，最近版本即可
- 安装 Redis，最近版本即可

### 安装

```bash
git clone https://github.com/lqqyt2423/wechat_spider.git
cd wechat_spider
npm install
```

本项目基于代理模块 AnyProxy，解析微信 HTTPS 请求需在电脑和手机上都安装证书。可参考：https://blog.csdn.net/chencangui/article/details/133976042?spm=1001.2014.3001.5501

### 通过 Docker 部署

```bash
git clone https://github.com/lqqyt2423/wechat_spider.git
cd wechat_spider
# build image
docker-compose build
# 运行实例（mongo数据存储地址需通过环境变量MONGO_PATH传入）
MONGO_PATH=/data/mongo docker-compose up
# 终止运行
docker-compose down
```

- `Dockerfile` 中已经设置了在 `Linux` 环境的 Docker 中添加根证书的操作步骤，所以接下来仅需在手机上安装 https 证书即可。
- 最终手机上设置的代理 ip 还是需要以自己电脑上的 ip 为准，需忽略 Docker 实例中打印的 ip 地址
- 可编辑 `Dockerfile` 和 `docker-compose.yml` 改变部署规则

## 使用
```
1、开启anyproxy: anyproxy -i
2、开启redis：redis-server.exe
3、打开mongodb
```

```bash
cd wechat_spider
npm start
```

1. 确保电脑和手机连接同一 WIFI，`npm start` 之后，命令行输出`请配置代理: xx.xx.xx.xx:8101` 类似语句，手机设置代理为此 IP 和端口
2. 手机上测试打开任一公众号历史文章详情页和文章页，观察电脑命令行的输出，查看数据是否保存至 MongoDB

> - 如需测试自动翻页，可先多次分别打开不同的公众号的历史详情页，等数据库中有了翻页的基础公众号信息之后，再随便进入历史页等待翻页跳转
> - 翻页逻辑仅支持公众号历史页面跳公众号历史页面，微信文章页面跳微信文章页面，两个不同页面不能互相跳转

### 针对微信新版需注意

1. 历史页面可自行拼接后发送至微信中打开，拼接规则为：

```javascript
var biz = 'MzI4NjQyMTM2Mw==';
var history_page = 'https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=' + biz + '&scene=124#wechat_redirect';
// https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=MzI4NjQyMTM2Mw==&scene=124#wechat_redirect
```
上面的 biz 的获取方式如下：
第一步，从你需要采集的公众号里面随便找一篇文章，然后用浏览器打开，打开有右击鼠标，选择查看网页源码，

第二步，打开源码后，按 Ctrl+F 搜索关键词 __biz= 查看第一个搜索结果，如下图的这一行

__biz=后面的一直到&符号结束的字符串复制一下就是了。最后替换上面的 biz 就可以了。

再将后面这个链接复制发送给文件传输助手，打开就能看到历史消息界面了

2. 进入微信文章页面先刷新一下

### 自定义配置

可编辑 `config.js` 文件进行自定义配置，文件中每个配置项都有详细的说明。

可配置项举例如下：

- 控制是否开启文章或历史详情页自动跳转
- 控制跳转时间间隔
- 根据文章发布时间控制抓取范围
- 是否保存文章正文内容
- 是否保存文章评论

需注意，本项目修改了 AnyProxy 的默认端口。连接代理的端口改为 8101，AnyProxy 管理界面的端口改为 8102，且仅在 `NODE_ENV=development` 时才会开启 AnyProxy 的管理界面功能。如需修改，可编辑 `config.js`。

### 可视化界面

前端页面已打包好，启动项目后，如无修改默认 `server port` 配置，浏览器直接访问 `http://localhost:8104` 即可。检测数据有无抓取保存直接刷新此页面即可。

![可视化界面](./imgs/posts_screenshot.png)

前端页面由 `React` 编写，如需修改，可编辑 `client` 文件中的代码。

### MongoDB 数据信息

数据库 database: wechat_spider

数据表 collections:

- posts - 文章数据
- profiles - 公众号数据
- comments - 评论数据

### 从 MongoDB 导出数据

#### 命令行直接导出数据

```bash
mongoexport --db wechat_spider --collection posts --type=csv --fields title,link,publishAt,readNum,likeNum,likeNum2,msgBiz,msgMid,msgIdx,sourceUrl,cover,digest,isFail --out ~/Desktop/posts.csv
```

#### 脚本导出

可参考文件 `/test/exportData.js` 。

## 感谢

感谢此文章提供思路：[微信公众号文章批量采集系统的构建](https://zhuanlan.zhihu.com/p/24302048)

### 赞助我

如果你觉得这个项目对你有帮助，不妨考虑给我买杯咖啡。

赞助时可备注来源 wechat spider，我会将你添加至下面的赞助列表中。

<div align="center">
	<img alt="sponsorme" src="./imgs/sponsor-me.jpg" style="width: 300px" />
</div>

感谢以下赞助者：

暂无

## License

[MIT](LICENSE)
