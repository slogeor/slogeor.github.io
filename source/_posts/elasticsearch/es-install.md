---
title: Mac 下配置 Elasticsearch
categories: elasticsearch
tags: #文章標籤 可以省略
  - es
---

Elasticsearch 的版本 6.2.4。

### 安装 Elasticsearch

到官网根据不同操作系统下载最新版本的 [Elasticsearch](https://www.elastic.co/downloads/elasticsearch) 压缩包，之后本地解压缩。

#### Elasticsearch下载 x-pack

在 Elasticsearch 的根目录，运行 bin/elasticsearch-plugin 进行安装:

```js
bin/elasticsearch-plugin install x-pack
```

安装成功是这样的

![es1](https://github.com/slogeor/images/blob/master/other/2018/es1.png?raw=true)

安装到最后会跳出选项，选 y 即可。

#### 运行 Elasticsearch

在 Elasticsearch 的根目录，运行bin/elasticsearch：

```js
bin/elasticsearch
```

#### 设置 x-pack 的默认密码

在 Elasticsearch 的根目录，运行 bin/x-pack/setup-passwords interactive 进行安装:

```js
$ bin/x-pack/setup-passwords interactive
```

过一会会跳出选项，选 y 即可，然后设置自己的默认密码。

```js
Initiating the setup of reserved user elastic,kibana,logstash_system passwords.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y

Enter password for [elastic]:
Reenter password for [elastic]:
Enter password for [kibana]:
Reenter password for [kibana]:
Enter password for [logstash_system]:
Reenter password for [logstash_system]:
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [elastic]
```

#### 测试安装成功

启动成功后浏览器打开 http://localhost:9200/?pretty 会发现在安装了 x-pack 之后访问受到限制

![es2](https://github.com/slogeor/images/blob/master/other/2018/es2.png?raw=true)

输入用户名: elastic，密码: 刚才设置的，成功会看到。

```js
{
name: "EeU2Ij8",
cluster_name: "elasticsearch",
cluster_uuid: "KIyC8cYRQmKihcl4VPJX6A",
version: {
number: "6.2.4",
build_hash: "ccec39f",
build_date: "2018-04-12T20:37:28.497551Z",
build_snapshot: false,
lucene_version: "7.2.1",
minimum_wire_compatibility_version: "5.6.0",
minimum_index_compatibility_version: "5.0.0"
},
tagline: "You Know, for Search"
}
```

### 安装 Kibana

到官网根据不同操作系统下载最新版本的 [Kibana](https://www.elastic.co/downloads/kibana) 压缩包，选择和上面的 Elasticsearch 相同版本号的，之后本地解压缩。

#### Kibana 下载 x-pack

在 Kibana 根目录运行 bin/kibana-plugin 进行安装

```js
➜  kibana-6.2.4 bin/kibana-plugin install x-pack
Found previous install attempt. Deleting...
Attempting to transfer from x-pack
Attempting to transfer from https://artifacts.elastic.co/downloads/kibana-plugins/x-pack/x-pack-6.2.4.zip
Transferring 264988487 bytes....................
Transfer complete
Retrieving metadata from plugin archive
Extracting plugin archive
Extraction complete
Optimizing and caching browser bundles...
Plugin installation complete
```

#### 运行 Kibana

在 Kibana 的根目录，运行 bin/kibana

```js
bin/kibana
```

### 测试是否成功

启动成功后浏览器打开 http://localhost:5601/ 会发现在安装了 x-pack 之后访问也受限制，这里默认的用户名: elastic, 密码: 刚才设置的。

![es2](https://github.com/slogeor/images/blob/master/other/2018/es7.png?raw=true)

### Login is currently disabled.

现象

Login is currently disabled. Administrators should consult the Kibana logs for more details.

解决方案

打开 elasticsearch-6.2.4/config/elasticsearch.yml 文件，添加 `xpack.security.enabled: false`。
打开 kibana-6.2.4/config/elasticsearch.yml 文件，添加 `xpack.security.enabled: false`。
