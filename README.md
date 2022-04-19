# docker-install-everything

> install all environments using docker.
> 使用 docker 安装各种服务。

## 特色

- 仅依赖 docker 和 docker-compose，无需本地复杂环境。
- 支持软件和服务多，并且在持续新增。
- 每个文件夹一组(套)服务，根据需要安装即可。
- 所有的文件夹相互独立，无互相依赖，降低使用难度。

## 使用方法

```
git clone https://github.com/FX-Max/docker-install-everything.git
cd docker-install-everything
# 进入里想要安装的服务文件夹后，按照文件夹内的 README 文件介绍使用。
```

## 支持列表

- beanstalkd

    简要说明: 高性能，轻量级的分布式内存队列。

- wikijs

    简要说明： 自建开源的wiki/文档管理系统 wiki.js，[官网](https://js.wiki/)

- wordpress

    简要说明: 快速搭建 wordpress 系统。
