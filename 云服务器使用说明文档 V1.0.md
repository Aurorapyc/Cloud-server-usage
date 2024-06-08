# 云服务器使用说明文档 V1.0

> 注意: 本文档适用于个人开发云服务器部署和使用

## 安装系统

首先购买一个云服务器,如:

![image-20211119203444037](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211119203444037.png)

点击右侧更多选择重装系统

![image-20211120113746879](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211120113746879-20220716123136141.png)

![image-20211120113830446](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211120113830446.png)

下一步

![image-20211120114101235](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211120114101235.png)

> 注意: 推荐选择使用 ubuntu系统 默认用户名为 ubuntu 密码为自己指定一定要记住密码!

接下来配置安全组开放 22 端口,有些云服务器默认开放

![image-20211120120551124](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211120120551124.png)

![image-20211120120440471](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211120120440471.png)

## DNS 解析

接下来为了更好配置使用云服务器,最好将云服务公网 ip 与 对应域名进行映射,这样日后可以直接使用域名操作对应服务,如果没有域名这步可以忽略。首先获取公网 IP

![image-20211120114415891](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211120114415891.png)

找到 DNS 解析配置,进行域名解析配置

![image-20211120115913192](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211120115913192.png)

配置好了接下来操作,所有需要公网 ip 地方全部可以使用对应域名进行操作~~

## 网页登录

> 注意: 腾讯云服务器的 ubuntu 系统默认用户名不是 root 是 ubuntu 为用户名

![image-20211120121217434](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211120121217434.png)

![image-20211120121149976](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211120121149976.png)

创建自己的用户

```shell
$ sudo adduser 用户名
```

![image-20211120121450995](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211120121450995.png)

配置用户使用 sudo 命令

```shell
$ sudo vim /etc/sudoers
```

![image-20211120121748136](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211120121748136.png)

![image-20211120121928200](https://minioweb.baizhiedu.xin/typora-imgs/2022/07/16/image-20211120121928200.png)

----

## 安装 Docker & Compose

刚创建机器强烈推荐安装 docker 以及 docker-compose 方式管理服务

```shell
# install-docker.sh 

$ curl -fsSL get.docker.com -o get-docker.sh `# 下载安装脚本` \
  && sudo sh get-docker.sh --mirror Aliyun `# 执行安装脚本` \
  && sudo systemctl enable docker `# 加入开机启动` \
  && sudo systemctl start docker `# 启动docker服务` \
  && sudo groupadd -f docker `# 创建docker组` \
  && sudo usermod -aG docker $USER `# 将当前用户加入docker组` \
  && sudo mkdir -p /etc/docker `# 创建配置目录` \
  && sudo newgrp docker `# 更新docker组信息`\
  && sudo echo -e '{\n  "registry-mirrors": ["阿里云镜像加速地址"]\n}' >> /etc/docker/daemon.json `# 设置阿里云镜像加速` \
  && sudo systemctl daemon-reload `# 重新加载所有系统服务配置` \
  && sudo systemctl restart docker `# 重启docker服务` \
  && sudo systemctl enable docker `# 开机启动 docker服务`
```

> **`注意: 使用请修改registry-mirrors的地址`**

```shell
# install-compose.yml

$ sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# 国内用户可以使用以下方式加快下载
$ sudo curl -L https://download.fastgit.org/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

---

## 安装 mysql 服务

```markdown
# 启动 mysql 服务
- docker run -d --name mysql -e "MYSQL_ROOT_PASSWORD=root" mysql:5.7
```

```yml
version: "3.8"

networks:
  dev:
    external: true
  apps:

volumes:
  data:

services:
  mysql:
    image: mysql:5.7
    volumes:
      - data:/var/lib/mysql
    environment:
      - "MYSQL_ROOT_PASSWORD=root"
    networks:
      - dev
      - apps
```

## 安装 sshd 服务

> 注意: sshd 需要开放一个端口映射容器中 sshd 服务

![image-20211120170632208](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211120170632208.png)

```markdown
# 启动 sshd 服务
- docker run -ti -p 2222:22 \
  -e SSH_USERS=chenyn:1000:1000 \
  -e SSH_ENABLE_PASSWORD_AUTH=true \
  -e s \
  -v $(pwd)/entrypoint.d/:/etc/entrypoint.d/ \
  docker.io/panubo/sshd:1.5.0
```

```yaml
version: "3.8"

networks:
  dev:
    external: true
  apps:

volumes:
  data:

services:
  mysql:
    image: mysql:5.7
    volumes:
      - data:/var/lib/mysql
    environment:
      - "MYSQL_ROOT_PASSWORD=root"
    networks:
      - dev
      - apps

  sshd:
    image: docker.io/panubo/sshd:1.5.0
    ports:
      - "33333:22"
    environment:
      - "SSH_USERS=chenyn:1000:1000"
      - "SSH_ENABLE_PASSWORD_AUTH=true"
      - "TCP_FORWARDING=true"
    volumes:
      - ./setpasswd.sh:/etc/entrypoint.d/setpasswd.sh
    networks:
      - dev
      - apps
```

> 注意: 使用 compose 必须创建外部网络!

```sh
# setpasswd.sh

#!/usr/bin/env bash 
echo "blr:root1234" | chpasswd

# 放入对应公钥
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCfcMGrd6hEck84RedBilwz6zpWwCw5gi25xZjKMWV/qCMCPB3Wmk1SKnXXE7OwKFyTfwuzXBkpW16KV2/IlVmhh4CVp4/iGjmxYDUBtjVKHBCMprryaHmd4tb3uazbCbqJ1d6WV03lHv4onjYSl8Qr653pLrbpXRVVjZ2pjRbrzPTxpuo6Y3WsW9eg1zEsbFwoaUy+7NifAmJRotA/105gAoyrDAGIlY7sjVhxLiZZbSm3mnoSVeloghbzK1tx4qdKeWivqbF6bl9IwlYaQylH/JSvSDxUyABJXfPe+nWVAfFsUNGq8bSP5+vq15diCgeNvoNQQCUNxicOVWuMQXYk3RiEpRm4UJbJtW9vy80ALHEzWrnd7aGUqVcVNO96zEHNGUQfkfIqz4O6GtcwAqKeGQ77Mp/1rBSq3S2/dFHLvzurAIEe++wsnQsQDrSeW3LIoK2lFWBEE0aqABMPSUUbt7I8hn1OtQxLMtmEBJi+158erIQka78tRyN761zeQWE= macbookpro" > '/home/chenyn/.ssh/authorized_keys'
```

> 注意: setpasswd.sh需要赋予执行权限

## 安装常用应用

```yml
version: "3.8"

networks:
  dev:
    external: true
  apps:

volumes:
  data:

services:
  mysql:
    image: mysql:5.7
    volumes:
      - data:/var/lib/mysql
    environment:
      - "MYSQL_ROOT_PASSWORD=root"
    networks:
      - dev
      - apps

  redis:
    image: redis:6.2.6
    networks:
      - dev
      - apps

  rabbitmq:
    image: 3.8-management
    environment:
      - "RABBITMQ_DEFAULT_USER=guest"
      - "RABBITMQ_DEFAULT_PASS=guest"
    networks:
      - dev
      - apps


  sshd:
    image: docker.io/panubo/sshd:1.5.0
    ports:
      - "33333:22"
    environment:
      - "SSH_USERS=blr:1000:1000"
      - "SSH_ENABLE_PASSWORD_AUTH=true"
      - "TCP_FORWARDING=true"
    volumes:
      - ./setpasswd.sh:/etc/entrypoint.d/setpasswd.sh
    networks:
      - dev
      - apps
```

> 注意: 使用时通过 ssh 方式进行本地转发

ssh 本地转发如下:

```markdown
- ssh -L 15672:rabbitmq:15672 -p 33333 -N chenyn@tx.chenyn.cn
- -L 本地端口:转发远程端口
  -p 指定 ssh 端口
  -N 不进入终端
 	chenyn@tx.chenyn.cn 服务器

- 如:
	ssh -L 15672:rabbitmq:15672 -p 33333 -N chenyn@tx.chenyn.cn
	ssh -L 5672:rabbitmq:5672 -p 33333 -N chenyn@tx.chenyn.cn
	ssh -L 6379:redis:6379 -p 33333 -N chenyn@tx.chenyn.cn
	ssh -L 3306:rabbitmq:3306 -p 33333 -N chenyn@tx.chenyn.cn
```

---

## ssh 本地端口转发

编写本地转发脚本:

```sh
#!/bin/sh

set -e

ssh -L 15672:rabbitmq:15672 -p 33333 -N chenyn@tx.chenyn.cn &
echo "$!" > ./pids
ssh -L 5672:rabbitmq:5672 -p 33333 -N chenyn@tx.chenyn.cn &
echo "$!" >> ./pids
ssh -L 6379:redis:6379 -p 33333 -N chenyn@tx.chenyn.cn &
echo "$!" >> ./pids
ssh -L 3306:mysql:3306 -p 33333 -N chenyn@tx.chenyn.cn &
echo "$!" >> ./pids
```

编写杀死 shell:

```sh
#!/bin/sh

cat ./pids | xargs kill -9
```

---

## Traefik 代理工具

创建本地网络:

```sh
$ docker network create  proxy
```

编写traefik的docker-compose.yml:

```yml
version: '3.8'

networks:
  proxy:
    external: true

volumes:
  acme:

services:
  traefik:
    # The official Traefik docker image
    image: traefik:v2.5.4
    networks:
      - proxy
    ports:
      # The HTTP port
      - "80:80"
      # The HTTPS port
      - "443:443"
    volumes:
      # 时区
      - /etc/timezone:/etc/timezone
      - /etc/localtime:/etc/localtime
      # So that Traefik can listen to the Docker events
      # 使 Traefik 能够监听 Docker 事件
      - /var/run/docker.sock:/var/run/docker.sock
      # 持久化 ACME 生成的证书
      - acme:/etc/acme
    env_file:
      - ./.alidns.env
    # Enables the web UI and tells Traefik to listen to docker
    command:
      - --api.insecure=true
      - --providers.docker
      - --providers.docker.network=proxy
      - --providers.docker.exposedByDefault=false
      # Web
      - --entryPoints.web.address=:80
      - --entrypoints.web.http.redirections.entrypoint.permanent=true
      - --entrypoints.web.http.redirections.entrypoint.scheme=https
      - --entrypoints.web.http.redirections.entrypoint.to=websecure
      # Web Secure
      - --entryPoints.websecure.address=:443
      - --entrypoints.websecure.http.tls=true
      - --entrypoints.websecure.http.tls.certresolver=ali
      - --entrypoints.websecure.http.tls.domains[0].main=tx.chenyn.cn
      - --entrypoints.websecure.http.tls.domains[0].sans=*.tx.chenyn.cn
      # Let's Encrypt
      - --certificatesresolvers.ali.acme.dnschallenge.provider=alidns
      - --certificatesresolvers.ali.acme.email=chenynqc@163.com
      - --certificatesresolvers.ali.acme.storage=/etc/acme/acme.json
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.entrypoints=web, websecure"
      - "traefik.http.routers.traefik.rule=Host(`proxy.tx.chenyn.cn`)"
      - "traefik.http.services.traefik.loadbalancer.server.port=8080"
```

阿里云accesskey 秘钥:

```env
ALICLOUD_ACCESS_KEY=aaaa
ALICLOUD_SECRET_KEY=bbbb
```

> 注意: 必须是域名所在阿里云accesskey 和 秘钥

---

## Bitwarden

```yml
# 参考: https://docs.docker.com/compose/compose-file/compose-file-v3/
# 固定值, 无需修改
version: '3.8'

# yaml 引用
# 声明可能复用的配置段, 可以使用 * 号引用 & 号标记的内容
# 参考: http://www.ruanyifeng.com/blog/2016/07/yaml.html

# 声明网络
# 在同一网络内的容器可以相互访问
# 在 proxy 网络中的容器可以被 反向代理服务 traefik 访问
networks:
  proxy:
    external: true # 声明该网络为外部网络, 不需要 docker-compose 或 docker swarm 管理

# 声明数据卷
volumes:
  data:

services:
  bitwarden:
    image: vaultwarden/server:1.21.0-alpine
    volumes:
    - data:/data/
    - /etc/localtime:/etc/localtime
    - /etc/timezone:/etc/timezone
    networks:
      - proxy # 使用已声明的网络
    env_file:
      - ./.env
    labels:
      # 声明 traefik 反向代理的配置
      # 模板:
      # - "traefik.enable=true"
      # - "traefik.http.routers.<服务名(自定, 不可重复)>.entrypoints=web, websecure"
      # - "traefik.http.routers.<SERVICE>.rule=Host(`<多级域名>.dev.baizhiedu.cn`)"
      # - "traefik.http.services.<SERVICE>.loadbalancer.server.port=<服务端口号>"
      # region traefik 配置
      - "traefik.enable=true"
      - "traefik.http.routers.bitwarden.entrypoints=web, websecure"
      - "traefik.http.routers.bitwarden.rule=Host(`bw.tx.chenyn.cn`)"
      - "traefik.http.services.bitwarden.loadbalancer.server.port=80"
      # endregion traefik 配置
```

需要的 env 文件:

```env
ADMIN_TOKEN=GnBt7MlyqFBD6BSwvQZJECNbtWbEP
SIGNUPS_ALLOWED=false
WEBSOCKET_ENABLED=true
DOMAIN=https://bw.tx.chenyn.cn
```

> 更多配置参考: https://github.com/dani-garcia/vaultwarden/wiki/Configuration-overview

启动服务打开管理界面:

https://bw.tx.chenyn.cn/admin

输入之前的 token 令牌:

![image-20211123161026893](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211123161026893.png)

配置邮件通知:

![image-20211123165506364](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211123165506364.png)

![image-20211123162736301](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211123162736301.png)

注册用户:

![image-20211123162850751](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211123162850751.png)

客户端使用:

> 访问网站进行下载: https://bitwarden.com/download/

![image-20211123163043551](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211123163043551.png)

登录客户端:

![image-20211123163144712](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211123163144712.png)

------

## NPS 内网穿透

> NPS 是一个非常强大的内网穿透工具,非常实用

编写 docker-compose.yml 文件:

```yml
version: '3.8'

networks:
  proxy:
    external: true

services:
  nps:
    image: ffdfgdfg/nps:v0.26.9
    networks: 
      - proxy
    volumes:
      - /etc/timezone:/etc/timezone
      - /etc/localtime:/etc/localtime
      - ./conf:/conf
    ports:
      # 网桥端口, 用于客户端与服务器通信
      - "8024:8024"
      # tcp 端口, 用于 tcp 隧道  ./conf/nps.conf: allow_ports
      - "14100-14115:14100-14115"
      - "14100-14115:14100-14115/udp"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nps-web.entrypoints=web, websecure"
      - "traefik.http.routers.nps-web.rule=Host(`nps.tx.chenyn.cn`)"
      # 一个容器存在多个服务时, 需要显式声明 router 对应的 service
      - "traefik.http.routers.nps-web.service=nps-web"
      - "traefik.http.services.nps-web.loadbalancer.server.port=8080"
      - "traefik.http.routers.nps-http.entrypoints=web"
      - "traefik.http.routers.nps-http.rule=HostRegexp(`{subdomain:[A-Za-z0-9]+}.nps.tx.chenyn.cn`)"
      - "traefik.http.routers.nps-http.service=nps-http"
      - "traefik.http.routers.nps-https.entrypoints=websecure"
      - "traefik.http.routers.nps-https.rule=HostRegexp(`{subdomain:[A-Za-z0-9]+}.nps.tx.chenyn.cn`)"
      - "traefik.http.routers.nps-https.service=nps-http"
      - "traefik.http.routers.nps-https.tls.certResolver=ali"
      - "traefik.http.routers.nps-https.tls.domains[0].main=nps.tx.chenyn.cn"
      - "traefik.http.routers.nps-https.tls.domains[0].sans=*.nps.tx.chenyn.cn"
      - "traefik.http.services.nps-http.loadbalancer.server.port=80"
```

在 docker-compose.yml 所在目录放入conf 目录:

![image-20211124142143859](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124142143859.png)

修改 conf 目录中配置文件

![image-20211124142641115](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124142641115.png)

调整域名配置:

![image-20211124143047825](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124143047825.png)

指定端口:

![image-20211124143124123](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124143124123.png)

![image-20211124143202852](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124143202852.png)

开放安全组策略:

![image-20211124143341225](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124143341225.png)

登录 nps 客户端:

![image-20211124143509929](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124143509929.png)

创建客户端:

![image-20211124143932084](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124143932084.png)

![image-20211124143957777](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124143957777.png)

创建隧道:

![image-20211124144442931](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124144442931.png)

![image-20211124144505430](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124144505430.png)

下载客户端启动:

![image-20211124144532137](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124144532137.png)

![image-20211124144803425](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124144803425.png)

![image-20211124144941003](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211124144941003.png)

---

## Gogs

> Gogs 是一款极易搭建的自助 Git 服务。

编写 docker-compose.yml

```yml
version: '3.8'

networks:
  proxy:
    external: true

volumes:
  data:

services:
  gogs:
    image: gogs/gogs:0.12.2
    networks: 
      - proxy
    volumes:
			- /etc/timezone:/etc/timezone
      - /etc/localtime:/etc/localtime
			- data:/data
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.gogs.service=gogs"
      - "traefik.http.routers.gogs.rule=Host(`gogs.tx.chenyn.cn`)"
      - "traefik.http.services.gogs.loadbalancer.server.port=3000"

      - "traefik.tcp.routers.gogs-ssh.service=gogs-ssh"
      - "traefik.tcp.routers.gogs-ssh.entrypoints=gogs"
      - "traefik.tcp.routers.gogs-ssh.rule=HostSNI(`*`)"
      - "traefik.tcp.services.gogs-ssh.loadbalancer.server.port=10022"
```

开放端口

![image-20211210225553785](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211210225553785.png)

启动服务&访问地址

```http
https://gogs.tx.chenyn.cn
```

配置 gogs

![image-20211210230102648](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211210230102648.png)

![image-20211212193044905](https://typora-1993.oss-cn-beijing.aliyuncs.com/imgs/image-20211212193044905.png)

----

## Drone CI 工具

> 简介: 非常强大的 CI 工具,方便构建自己的应用

```yml
version: '3.8'

networks:
  proxy:
    external: true

volumes:
  data:

services:
  drone-server:
    image: drone/drone:2
    networks:
      - proxy
    volumes:
      - /etc/timezone:/etc/timezone
      - /etc/localtime:/etc/localtime
      - data:/data
    environment:
      - DRONE_PROVIDER=gogs
      - DRONE_GOGS=true
      - DRONE_AGENTS_ENABLED=true
      - DRONE_GOGS_SERVER=https://gogs.baizhiedu.xin
      - DRONE_RPC_SECRET=6d26eacdb47351b7b579678af33f9d04
      - DRONE_SERVER_HOST=server.drone.chenyn.cn
      - DRONE_SERVER_PROTO=https
      - DRONE_USER_CREATE=username:chenyn,admin:true
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.drone-server.entrypoints=web, websecure"
      - "traefik.http.routers.drone-server.rule=Host(`server.drone.chenyn.cn`)"
      - "traefik.http.services.drone-server.loadbalancer.server.port=80"



  drone-agent:
    image: drone/drone-runner-docker:1
    networks:
      - proxy
    volumes:
      - /etc/timezone:/etc/timezone
      - /etc/localtime:/etc/localtime
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_RPC_PROTO=https
      - DRONE_RPC_HOST=server.drone.chenyn.cn
      - DRONE_RPC_SECRET=6d26eacdb47351b7b579678af33f9d04
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME=drone-runner
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.drone-agent.entrypoints=web, websecure"
      - "traefik.http.routers.drone-agent.rule=Host(`agent.drone.chenyn.cn`)"
      - "traefik.http.services.drone-agent.loadbalancer.server.port=3000"
```

> `注意:这里使用是 gogs 自定义版本库实现的`

## MinIO

> 简介: 强大的oss 对象存储

```yml
version: '3.8'

x-timezone:
  timezone: &timezone
    source: timezone
    target: /etc/timezone
  localtime: &localtime
    source: localtime
    target: /etc/localtime

networks:
  proxy:
    external: true

configs:
  timezone:
    external: true
  localtime:
    external: true

volumes:
  data:

services:
  minio:
    image: minio/minio:RELEASE.2020-12-10T01-54-29Z
    networks:
      - proxy
    configs:
      - *timezone
      - *localtime
    volumes:
      - data:/data
    env_file:
      - ./.env
    command: server /data
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://localhost:9000/minio/health/live" ]
      interval: 30s
      timeout: 20s
      retries: 3
    deploy:
      replicas: 1
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.minio.entrypoints=web, websecure"
        - "traefik.http.routers.minio.rule=Host(`minio.baizhiedu.xin`)"
        - "traefik.http.services.minio.loadbalancer.server.port=9000"

  minio-nginx:
    image: nginx:1.19-alpine
    networks:
      - proxy
    configs:
      - *timezone
      - *localtime
    volumes:
      - data:/data
      - ./default.conf:/etc/nginx/conf.d/default.conf
    deploy:
      replicas: 1
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.minio-web.entrypoints=web, websecure"
        - "traefik.http.routers.minio-web.rule=Host(`minioweb.baizhiedu.xin`)"
        - "traefik.http.services.minio-web.loadbalancer.server.port=80"
```

编写`nginx`配置文件

```conf
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /data/;
        #index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

创建`.env`环境文件

```env
MINIO_ACCESS_KEY=dev
MINIO_SECRET_KEY=v3jHyxeeYyVeQT6fdLUAsijhOlWGjCjJ
```

## YAPI

> 简介: 强大的接口管理工具

```yml
version: '3.8'

networks:
  proxy:
    external: true
  yapi: {}

volumes:
  data:

services:
  yapi:
    image: jayfong/yapi:latest
    networks: 
      - proxy
      - yapi
    environment:
      YAPI_DB_SERVERNAME: mongo
      YAPI_DB_PORT: "27017"
      YAPI_DB_DATABASE: &database yapi
    env_file: 
      - ./.env
    deploy:
      replicas: 1
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.yapi.service=yapi"
        - "traefik.http.routers.yapi.entrypoints=web, websecure"
        - "traefik.http.routers.yapi.rule=Host(`yapi.baizhiedu.xin`)"
        - "traefik.http.services.yapi.loadbalancer.server.port=3000"

  mongo:
    image: mongo:4.4.1-bionic
    networks: 
      - yapi
    volumes:
      - data:/data/db
    command: --wiredTigerCacheSizeGB 0.25
    environment: 
      MONGO_INITDB_DATABASE: *database
    deploy:
      replicas: 1
```

创建 `.env` 文件

```env
YAPI_ADMIN_ACCOUNT=chenynqc@163.com
YAPI_ADMIN_PASSWORD=mima
YAPI_CLOSE_REGISTER=true
YAPI_MAIL_ENABLE=false
YAPI_LDAP_LOGIN_ENABLE=false
# YAPI_PLUGINS="[]"
```

