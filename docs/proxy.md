# 社区应用代理指南



### 问题

Unraid社区应用（Community Applications，CA）插件依赖GitHub（ `github.com` 和 `raw.githubusercontent.com` ）下载数据。

然而，根据[wiki](https://zh.wikipedia.org/wiki/%E5%AF%B9GitHub%E7%9A%84%E5%AE%A1%E6%9F%A5%E5%92%8C%E5%B0%81%E9%94%81)，仅仅修改Hosts的方法已经不够了。本文介绍通过[trojan-gfw](https://trojan-gfw.github.io/trojan/)代理访问CA并下载插件的方法。



### 手动安装CA插件

* 通过PC下载CA插件的本体文件：

  ```
  https://raw.githubusercontent.com/Squidly271/community.applications/master/plugins/community.applications.plg
  ```

* 将其命名为`community.applications.plg`并上传到Unraid下的`/boot/config/plugins/`。

* 在网页端插件页面下，选择安装插件，并从本地安装CA插件。

  此时会卡在下载`community.applications-***.txz`的过程中，记录这个文件的URL，并中断安装。

* 手动下载上述文件并上传到`/boot/config/plugins/community.applications/`。

  （此路径可以通过在`plg`文件中搜索`<FILE`找到）

* 回到网页端插件页面下，再次从本地安装CA插件。




### 通过Docker运行代理

因为Unraid的终端没有包管理器（替代品Nerd Pack也需要通过CA才能安装），我们选择直接用docker运行代理程序。

这个[Dockerfile](https://github.com/ashawkey/trojan-privoxy-client)以trojan为例，如果使用其他工具，需要自己搭建镜像。

* 通过终端设置国内Docker镜像源（阿里云镜像需要填入自己的[ID](https://www.aliyun.com/product/acr?source=5176.11533457&userCode=8lx5zmtu)）。

  推荐贴到`go`文件中以实现持久化。

  ```bash
  # docker mirrors
  mkdir -p /etc/docker
  tee /etc/docker/daemon.json <<- "EOF"
  {
      "registry-mirrors" : [
          "https://[yourid].mirror.aliyuncs.com",
          "https://registry.docker-cn.com",
          "http://hub-mirror.c.163.com"
      ]
  }
  EOF
  ```
  
* 准备trojan的`config.json` 文件，放在`/boot/trojan/`下。

  注意`local_port`需要设置为`1086`。仅供参考的例子：

  ```json
  {
      "run_type": "client",
      "local_addr": "127.0.0.1",
      "local_port": 1086,
      "remote_addr": "[your domain]",
      "remote_port": 443,
      "password": [
          "[yourpasswd]"
      ],
      "log_level": 1,
      "ssl": {
          "verify": false,
          "verify_hostname": false,
          "cert": "",
          "key": "",
          "key_password": "",
          "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
          "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
          "prefer_server_cipher": true,
          "alpn": [
              "http/1.1"
          ],
          "alpn_port_override": {
              "h2": 81
          },
          "reuse_session": true,
          "session_ticket": false,
          "session_timeout": 600,
          "plain_http_response": "",
          "curves": "",
          "dhparam": ""
      },
      "tcp": {
          "prefer_ipv4": false,
          "no_delay": true,
          "keep_alive": true,
          "reuse_port": false,
          "fast_open": false,
          "fast_open_qlen": 20
      },
      "mysql": {
          "enabled": false,
          "server_addr": "127.0.0.1",
          "server_port": 3306,
          "database": "trojan",
          "username": "trojan",
          "password": "",
          "key": "",
          "cert": "",
          "ca": ""
      }
  }
  ```

* 拉取并运行镜像：

  ```bash
  docker pull ashawkey/trojan-privoxy-client
  
  # the default proxy:
  # socks5://127.0.0.1:1086
  # http://127.0.0.1:1087
  docker run -d --name tpc -v /boot/config/trojan:/etc/trojan -p 1086:1086 -p 1087:1087 ashawkey/trojan-privoxy-client
  ```

  在docker界面可以确认其正常运行，并开启自启动。



### 代理设置

* 网页端Apps界面

  根据[代码](https://github.com/Squidly271/community.applications/blob/722f7f489dfbc71382e6dc4a524ee013e29cb344/source/community.applications/usr/local/emhttp/plugins/community.applications/include/helpers.php#L63), CA使用`curl` 下载Apps数据，并且提供了`proxy.cfg`文件用于显式设置`curl`的代理。

  因此，只需要创建文件`/boot/config/plugins/community.applications/proxy.cfg`并写入：

  ```
  port=1087
  tunnel=1
  proxy=http://127.0.0.1
  ```

  `curl`支持http或socks5协议，这里使用http。

  

* 插件安装

  同样根据代码（位于`/usr/local/sbin/plugin`），插件安装采用`wget`下载数据。

  `wget`只支持http代理，所以我们需要修改`go`文件，在运行`emhttp`前加上代理的环境变量，顺便也设置终端的代理。

  ```bash
  # emhttp
  http_proxy="http://127.0.0.1:1087" https_proxy="http://127.0.0.1:1087" /usr/local/sbin/emhttp &
  
  # terminal
  echo "export http_proxy=\"http://127.0.0.1:1087\"" >> /etc/profile
  echo "export https_proxy=\"http://127.0.0.1:1087\"" >> /etc/profile
  ```

  

现在重启Unraid，应该可以正常访问Apps页面，并下载插件了。



