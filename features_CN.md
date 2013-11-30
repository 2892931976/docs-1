gopm
====

* [总体设计目标](#10)
* [程序结构](#11)
* [Go包版本说明](#20)
* [各命令的目标和作用](#30)
	* [gopm sources](#32)
	* [gopm list](#33)
	* [gopm rm](#35)
	* [gopm search](#36)
	* [gopm doc](#37)
	* [gopm serve](#38)
	* [gopm sync](#39)
	* [gopm import](#40)
	* [gopm test](#44)

<a id="10" name="10"></a>
# 总体设计目标

1. 支持go语言的版本管理
2. 支持文档管理
3. 支持本地源服务器
4. 本地源服务器同时支持公共包和私有包
5. 支持依赖管理
6. 支持从github, code.google.com, gitLab, 等常见的源码托管服务下载 

<a id="11" name="11"></a>
# 最终程序只有一个，但是通过配置，可以有三种模式：

1 独立服务器
2 子服务器
3 客户端（默认）

## 独立服务器

独立服务器就是本身的包都是直接从源服务器中获取的。

## 子服务器

子服务器就是包是从所配置的独立服务器上获取的，而不是直接从github等源服务器获取，在一个局域网中，可以通过架设子服务器来加快包的分发。

## 客户端

默认下载即为客户端模式，客户端默认是从源服务器获取包，如果要从包服务器获取包，则可在配置文件中通过配置即可。

<a id="20" name="20"></a>
#Go包版本说明

版本分为四种：

* []:      表示的是当前最新版本即trunk     
* branch:  表示的是某个分支
* tag:     表示的是某个tag
* commit:  表示的是某个reversion

#配置文件说明

默认没有配置文件，当系统第一次启动时检测homedir/.gopm/config，看是否存在，如果不存在则自动创建此配置文件。
配置文件内容如下：
[sources]
http://gopm.io

[repos]
~/.gopm/repos

#数据库说明
包信息数据采用goleveldb，这是一个key/value数据库。数据库存默认放在~/.gopm/repos下。数据存放规则如下：

* "lastId" : "{lastId}"   			lastId中存放最大的Id，Id为自增

* "index:{packageName}": "{id}"  index:中存放的是包名，value中存放的是这个包的不同版本的id，不同版本用逗号分隔

* “pkg:{id}” : "{pkg}" 某个包的名称

* “ver:{id}” : "{verString1}, {verString2}"  某个包版本对应的内容

* "desc:{id}" : "{desc}"  某个包的最新版本的描述

* "down:{id}" : "{down}"  某个包的下载url

* "deps:{id}" : "{deps}"  某个包的最新版本的描述

* “key:{keyword}:{id}” : ""   关键词及其对应的版本

* “total” :"{total}" 包总数

<a id="30" name="30"></a>
#各命令的目标和作用

<a id="32" name="32"></a>
###gopm sources [add|rm [{url}]]

* []   	   列出当前可用的所有源，默认为http://gopm.io/
* add url 添加一个源到本地
* rm  url 删除一个源到本地，如果没有任何源，则自己成为一个独立的服务器，类似gopm.io

<a id="33" name="33"></a>
###gopm list [{packagename}[:{version}]]

* []   列出所有本地的包
* packagename   显示指定名称的包的详细信息

<a id="35" name="35"></a>
###gopm rm {packagename}[:{version}]

去除一个包，如果不加版本标示，则删除该包的所有版本

<a id="36" name="36"></a>
###gopm search [-e] {keyword}

根据关键词查找包名或者包的描述，如果有-e开关，则完全匹配包名

<a id="37" name="37"></a>
###gopm doc [-b] {packagename}[:{version}]

* []   显示一个包的文档
* -b   在默认浏览器中显示该包的文档

<a id="38" name="38"></a>
###gopm serve [-p {port}]

将本地仓库作为服务对外提供，如果没有-p，则端口为80，如果有，则端口为指定端口，该服务是一个web服务，通过浏览器也可以进行浏览。

<a id="39" name="39"></a>
###gopm sync [-u]

[] 如果当前配置了源，则从可用的源中同步所有的包信息和包内容的最新版本到本地仓库；
    如果当前没有配置任何源，则将所有已有的包从源头进行更新
-u  仅更新本地仓库已经有的包，不包含本地仓库没有的包

<a id="40" name="40"></a>
###gopm import [{url}|{filepath}]

将某个地址或者本地的包导入到本地仓库中，url应为可支持的源码托管站点或者gitLab

<a id="44" name="44"></a>
###gopm test

此命令依赖于go test

调用gopm build在临时文件夹生成可执行的测试文件，并设置程序当前目录为当前目录，并执行
