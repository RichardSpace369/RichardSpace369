---
title: 前端工程化-npm 安装机制
author: richard
toc: true
excerpt: 掌握了解npm 安装机制...
categories:
- 前端工程化
---

## npm 内部机制和核心原理


### npm 的安装机制和背后思想

npm 的安装机制非常值得探究。Ruby 的 Gem、Python 的 pip 都是全局安装，但是 npm 的安装机制秉承了不同的设计哲学。

它会优先安装依赖包到当前项目目录，使得不同应用项目的依赖各成体系，同时还减轻了包作者的 API 兼容性压力，但这样做的缺陷也很明显：如果我们的项目 A 和项目 B，都依赖了相同的公共库 C，那么公共库 C 一般都会在项目 A 和项目 B 中，各被安装一次。这就说明，同一个依赖包可能在我们的电脑上进行多次安装。

当然，对于一些工具模块比如 supervisor 和 gulp，你仍然可以使用全局安装模式，这样方便注册 path 环境变量，我们可以在任何地方直接使用 supervisor、 gulp 这些命令。（不过，一般还是建议不同项目维护自己局部的 gulp 开发工具以适配不同项目需求。）


![npm安装机制](/img/工程化/npm安装机制.jpg)

npm install 执行之后，首先，检查并获取 npm 配置，这里的优先级为：**项目级的 .npmrc 文件 > 用户级的 .npmrc 文件> 全局级的 .npmrc 文件 > npm 内置的 .npmrc 文件。**

然后检查项目中是否有 package-lock.json 文件。

如果有，则检查 package-lock.json 和 package.json 中声明的依赖是否一致：

- 一致，直接使用 package-lock.json 中的信息，从缓存或网络资源中加载依赖；

- 不一致，按照 npm 版本进行处理（不同 npm 版本处理会有不同，具体处理方式如图所示）。

如果没有，则根据 package.json 递归构建依赖树。然后按照构建好的依赖树下载完整的依赖资源，在下载时就会检查是否存在相关资源缓存：

- 存在，则将缓存内容解压到 node_modules 中；

- 否则就先从 npm 远程仓库下载包，校验包的完整性，并添加到缓存，同时解压到 node_modules。

最后生成 package-lock.json。

构建依赖树时，当前依赖项目不管其是直接依赖还是子依赖的依赖，都应该按照**扁平化原则**，优先将其放置在 node_modules 根目录（最新版本 npm 规范）。在这个过程中，遇到相同模块就判断已放置在依赖树中的模块版本是否符合新模块的版本范围，如果符合则跳过；不符合则在当前模块的 node_modules 下放置该模块（最新版本 npm 规范）。

**同一个项目成员应该保持一致的npm 版本**

前端工程中，依赖嵌套依赖，一个中型项目中 node_moduels 安装包可能就已经是海量的了。如果安装包每次都通过网络下载获取，无疑会增加安装时间成本。对于这个问题，缓存始终是一个好的解决思路，我们接下来看看 npm 自己的缓存机制。



### npm 缓存机制
对于一个依赖包的同一版本进行本地化缓存，是当代依赖包管理工具的一个常见设计。使用时要先执行以下命令：
```
npm config get cache
```
我们可以使用以下命令清除 /Users/cehou/.npm/_cacache 中的文件：

```
 npm cache clean --force

```
![npm cache](/img/工程化/npm-cache.jpg)

接下来打开_cacache文件，看看 npm 缓存了哪些东西，一共有 3 个目录：

- content-v2
- index-v5
- tmp

其中 content-v2 里面基本都是一些二进制文件。为了使这些二进制文件可读，我们把二进制文件的扩展名改为 .tgz，然后进行解压，得到的结果其实就是我们的 npm 包资源。

而 index-v5 文件中，我们采用跟刚刚一样的操作就可以获得一些描述性的文件，事实上这些内容就是 content-v2 里文件的索引。

这些缓存如何被储存并被利用的呢？

这就和 npm install 机制联系在了一起。当 npm install 执行时，通过pacote把相应的包解压在对应的 node_modules 下面。npm 在下载依赖时，先下载到缓存当中，再解压到项目 node_modules 下。pacote 依赖npm-registry-fetch来下载包，npm-registry-fetch 可以通过设置 cache 属性，在给定的路径下根据IETF RFC 7234生成缓存数据。

接着，在每次安装资源时，根据 package-lock.json 中存储的 integrity、version、name 信息生成一个唯一的 key，这个 key 能够对应到 index-v5 目录下的缓存记录。如果发现有缓存资源，就会找到 tar 包的 hash，根据 hash 再去找缓存的 tar 包，并再次通过pacote把对应的二进制文件解压到相应的项目 node_modules 下面，省去了网络下载资源的开销。

**注意，这里提到的缓存策略是从 npm v5 版本开始的。在 npm v5 版本之前，每个缓存的模块在 ~/.npm 文件夹中以模块名的形式直接存储，储存结构是：{cache}/{name}/{version}。**

## npm 不完全指南


### npm link 

从工作原理上总结，npm link 的本质就是软链接，它主要做了两件事：

- 为目标 npm 模块（npm-package-1）创建软链接，将其链接到全局 node 模块安装路径 /usr/local/lib/node_modules/ 中；

- 为目标 npm 模块（npm-package-1）的可执行 bin 文件创建软链接，将其链接到全局 node 命令安装路径 /usr/local/bin/ 中。
### npx 的作用

直接执行 node_modules/.bin 文件夹下的文件。在运行命令时，npx 可以自动去 node_modules/.bin 路径和环境变量 $PATH 里面检查命令是否存在，而不需要再在 package.json 中定义相关的 script。

**npx 另一个更实用的好处是：npx 执行模块时会优先安装依赖，但是在安装执行后便删除此依赖，这就避免了全局安装模块带来的问题。**

### npm 多源镜像和企业级部署私服原理
npm 中的源（registry），其实就是一个查询服务。以 npmjs.org 为例，它的查询服务网址是 [https://registry.npmjs.org/](https://registry.npmjs.org/)。这个网址后面跟上模块名，就会得到一个 JSON 对象，里面是该模块所有版本的信息。比如，访问 [react](https://registry.npmjs.org/react)，就会看到 react 模块所有版本的信息。

现在社区上主要有 3 种工具来搭建 npm 私服：nexus、verdaccio 以及 cnpm。



## 参考链接
[聊聊 NPM 镜像那些险象环生的坑](https://mp.weixin.qq.com/s/2ntKGIkR3Uiy9cQfITg2NQ)