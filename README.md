DGM（Docker + Go + MySQL + Redis）一键生成golang常用环境


> 使用前最好提前阅读一遍[目录](#目录)，以便快速上手，遇到问题也能及时排除。

[**[GitHub地址]**](https://github.com/yeszao/dnmp)
DGM项目特点：
1. `100%`开源
2. `100%`遵循Docker标准
3. 所有镜像源于[Docker官方仓库](https://hub.docker.com)，安全可靠
4. 一次配置，**Windows、Linux、MacOs**皆可用

# 目录
- [1.目录结构](#1目录结构)
- [2.快速使用](#2快速使用)
- [3.管理命令](#3管理命令)
- [4.使用Log](#4使用Log)
- [5.常见问题](#5常见问题)


## 1.目录结构

```
/
├── data                        数据库数据目录
│   ├── mysql                   MySQL8 数据目录
├── services                    服务构建文件和配置文件目录
│   ├── mysql                   MySQL 配置文件目录
│   ├── go                     golang 配置目录
│   └── redis                   Redis 配置目录
├── logs                        日志目录
├── docker-compose.sample.yml   Docker 服务配置示例文件
├── env.smaple                  环境配置示例文件
└── code                        代码目录
```

## 2.快速使用
1. 本地安装
    - `git`
    - `Docker`(系统需为Linux，Windows 10 Build 15063+，或MacOS 10.12+，且必须要`64`位）
    - `docker-compose 1.7.0+`
2. `clone`项目：
    ```
    $ git clone https://github.com/yeszao/dnmp.git
    ```
3. 如果不是`root`用户，还需将当前用户加入`docker`用户组：
    ```
    $ sudo gpasswd -a ${USER} docker
    ```
4. 拷贝并命名配置文件（Windows系统请用`copy`命令），启动：
    ```
    $ cd dnmp                                           # 进入项目目录
    $ cp env.sample .env                                # 复制环境变量文件
    $ cp docker-compose.sample.yml docker-compose.yml   # 复制 docker-compose 配置文件。
                                                        # 默认启动3个服务：golang、redis、MySQL。
    ```
## 3.管理命令
### 3.1 服务器启动和构建命令
如需管理服务，请在命令后面加上服务器名称，例如：
```bash
$ docker-compose up                         # 创建并且启动所有容器
$ docker-compose up -d                      # 创建并且后台运行方式启动所有容器
$ docker-compose up redis go mysql         # 创建并且启动多个容器
$ docker-compose up -d redis go mysql     # 创建并且已后台运行的方式启动容器


$ docker-compose start mysql                  # 启动服务
$ docker-compose stop mysql                   # 停止服务
$ docker-compose restart mysql                # 重启服务
$ docker-compose build mysql                  # 构建或者重新构建服务

$ docker-compose rm mysql                     # 删除并且停止php容器
$ docker-compose down                       # 停止并删除容器，网络，图像和挂载卷
```

### 3.2 查看docker网络
```sh
ifconfig docker0
```
用于填写`extra_hosts`容器访问宿主机的`hosts`地址

## 4.使用Log
Log文件生成的位置依赖于conf下各log配置的值。
### 4.1 MySQL日志
因为MySQL容器中的MySQL使用的是`mysql`用户启动，它无法自行在`/var/log`下的增加日志文件。所以，我们把MySQL的日志放在与data一样的目录，即项目的`mysql`目录下，对应容器中的`/var/lib/mysql/`目录。
```bash
slow-query-log-file     = /var/lib/mysql/mysql.slow.log
log-error               = /var/lib/mysql/mysql.error.log
```
以上是mysql.conf中的日志文件的配置。



## 5 常见问题
### 5.1 Docker容器时间
容器时间在.env文件中配置`TZ`变量，所有支持的时区请看[时区列表·维基百科](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)或者[PHP所支持的时区列表·PHP官网](https://www.php.net/manual/zh/timezones.php)。

### 5.2 Docker使用cron定时任务 
[Docker使用cron定时任务](https://www.awaimai.com/2615.html)

### 5.3 如何连接MySQL和Redis服务器
这要分两种情况，

第一种情况，在**代码中**。
容器与容器是`expose`端口联通的，而且在同一个`networks`下，所以连接的`host`参数直接用容器名称，`port`参数就是容器内部的端口。更多请参考[《docker-compose ports和expose的区别》](https://www.awaimai.com/2138.html)。

第二种情况，**在主机中**通过**命令行**或者**Navicat**等工具连接。主机要连接mysql和redis的话，要求容器必须经过`ports`把端口映射到主机了。以 mysql 为例，`docker-compose.yml`文件中有这样的`ports`配置：`3306:3306`，就是主机的3306和容器的3306端口形成了映射，所以我们可以这样连接：
```bash
$ mysql -h127.0.0.1 -uroot -p123456 -P3306
$ redis-cli -h127.0.0.1
```
这里`host`参数不能用localhost是因为它默认是通过sock文件与mysql通信，而容器与主机文件系统已经隔离，所以需要通过TCP方式连接，所以需要指定IP。

## License
MIT


