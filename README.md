
# 115VideoTutor

本教程演示如何使用 cdn2、alist、Emby、Nginx 和 auto_symlink 组合，无缝播放存储在 115.com 上的视频。学习如何设置一个 302 重定向系统，让你可以通过用户友好的界面访问你的 115 视频，绕过直接播放可能遇到的限制或复杂性。

## FAQ

### 配置要求

1C2G,建议使用Ubuntu

### 路径
cd2挂载路径为 /home/media

emby读取路径为 /home/strm

### 如何安装

#### Ubuntu


```bash
apt update
apt install docker.io
apt install docker-compose
cd /root
vi docker-compose.yaml
//将下文合集安装内容复制粘贴
docker-compose up -d
```
## 合集安装

```bash
version: "4.8"  # 使用最高的版本号

services:
  alist:
    image: 'xhofe/alist:latest'
    container_name: alist
    volumes:
      - 'root/alist/data:/opt/alist/data'
    ports:
      - '5244:5244'
    environment:
      - PUID=0
      - PGID=0
      - UMASK=022
    restart: unless-stopped

  auto_symlink:
    image: shenxianmq/auto_symlink:latest
    container_name: auto_symlink
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - /home/media:/home/media:rslave
      - /home/strm:/home/strm
      - /root/link/config:/app/config
    ports:
      - "8095:8095"
    user: "0:0"

  clouddrive2:
    image: cloudnas/clouddrive2
    container_name: clouddrive2
    environment:
      - TZ=Asia/Shanghai
      - CLOUDDRIVE_HOME=/Config
    volumes:
      - /home/media:/media:shared
    devices:
      - /dev/fuse:/dev/fuse
    restart: unless-stopped
    pid: "host"
    privileged: true
    network_mode: "host"

  emby-server:
    image: amilys/embyserver:beta
    container_name: emby-server
    restart: always
    privileged: true
    ports:
      - "8096:8096"
    volumes:
      - /home/strm:/strm
      - /home/media:/home/media:rslave
      - /root/emby/config:/config

  medialinker:
    restart: always
    volumes:
      - '/root/mediaLinker/opt/:/opt/'
    environment:
      - AUTO_UPDATE=false
      - SERVER=emby
      - NGINX_PORT=8091
      - NGINX_SSL_PORT=8095
      - NGINX_LOG_LEVEL=error
    container_name: medialinker
    image: 'thsrite/medialinker:latest'
    network_mode: "host"

  nginx-proxy-manager: # 更名避免与其他服务冲突
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - /root/nginx/data:/data
      - /root/nginx/letsencrypt:/etc/letsencrypt

  portainer:
    image: outlovecn/portainer-cn:latest
    container_name: portainer
    restart: always
    ports:
      - "9000:9000"
      - "8000:8000"
    volumes:
      - /root/portainer/dockerconfig/portainer:/data
      - /var/run/docker.sock:/var/run/docker.sock

```

## 独立安装

Alist 

```bash
version: '3.3'
services:
    alist:
        image: 'xhofe/alist:latest'
        container_name: alist
        volumes:
            - 'root/alist/data:/opt/alist/data'
        ports:
            - '5244:5244'
        environment:
            - PUID=0
            - PGID=0
            - UMASK=022
        restart: unless-stopped
```

Auto_symlink
```bash
version: "3.9"
services:
  auto_symlink:
    image: shenxianmq/auto_symlink:latest
    container_name: auto_symlink
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - /home/media:/home/media:rslave
      - /home/strm:/home/strm
      - /root/link/config:/app/config
    ports:
      - "8095:8095"
    user: "0:0"

```

CloudDrive2
```bash
version: "2.1"
services:
  cloudnas:
    image: cloudnas/clouddrive2
    container_name: clouddrive2
    environment:
      - TZ=Asia/Shanghai
      - CLOUDDRIVE_HOME=/Config
    volumes:
      - /home/media:/media:shared
    devices:
      - /dev/fuse:/dev/fuse
    restart: unless-stopped
    pid: "host"
    privileged: true
    network_mode: "host"

```

Emby
```bash
version: "3.9"
services:
  emby-server:
    image: amilys/embyserver:beta
    container_name: emby-server
    restart: always
    privileged: true
    ports:
      - "8096:8096"
    volumes:
      - /home/strm:/strm
      - /home/media:/home/media:rslave
      - /root/emby/config:/config

```

MediaLinker
```bash
version: '3'
services:
    medialinker:
        restart: always
        volumes:
            - '/root/mediaLinker/opt/:/opt/'
        environment:
            - AUTO_UPDATE=false
            - SERVER=emby
            - NGINX_PORT=8091
            - NGINX_SSL_PORT=8095
            - NGINX_LOG_LEVEL=error

        container_name: medialinker
        image: 'thsrite/medialinker:latest'
        network_mode: "host"

```

Nginx-Proxy-Manager
```bash
version: '4.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - /root/nginx/data:/data
      - /root/nginx/letsencrypt:/etc/letsencrypt

```

Portainer
```bash
version: "2.1"
services:
  portainer:
    image: outlovecn/portainer-cn:latest
    container_name: portainer
    restart: always
    ports:
      - "9000:9000"
      - "8000:8000"
    volumes:
      - /root/portainer/dockerconfig/portainer:/data
      - /var/run/docker.sock:/var/run/docker.sock

```

