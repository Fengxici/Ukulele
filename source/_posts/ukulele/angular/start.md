---
title: 第一章 Get Start
date: 2019-10-16 17:04:15
tags:
- 部署
- 开发
categories:
- Ukulele
- Angular
---
## 前置条件
1. 安装[node](https://nodejs.org)
2. 安装[yarn](https://yarn.bootcss.com)

## 获取代码

代码位置：https://github.com/Fengxici/Ukulele-Angular

使用git工具克隆代码

## 安装依赖
进入项目跟路径（Ukulele-Angular)，打开命令行，执行以下命令
> yarn

此时会下载以来，可能会耗费比较长时间（视网络情况），如果有失败，可以多尝试几次。

完成之后根路径下会多出一个**node_modules**目录，该目录即是所有以来的存储位置。不同于maven的本地仓库，前端的依赖都是安装在项目根路径下的。

## 运行
仍然在项目根路径下，执行：
> npm start

命令执行完后，一般会自动打开默认浏览器（http://localhost:4200）进入登陆界面。
![登陆界面](/images/ukulele/spring-cloud/angular-login.png)

注：请确保**proxy.conf.json**文件中配置的代理地址是您后台服务的地址。
``` json
{
  "/auth": {
    "target": "http://127.0.0.1:9090",
    "secure": false
  },
  "/api/**":{
    "target": "http://127.0.0.1:10000",
    "secure": false
  }
}
```
- **/auth**配置的是权限服务（Ukulele-Auth)的地址，
- **/api**是网关（Ukukeke-gateway)的地址。

## 打包
依然在根目录下，执行：
>npm run-script build

完成之后，根目录下会增加一个**dist**目录，此即为打包后的程序位置。

可使用ngnix等容器进行部署。



