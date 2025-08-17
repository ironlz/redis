[![codecov](https://codecov.io/github/redis/redis/graph/badge.svg?token=6bVHb5fRuz)](https://codecov.io/github/redis/redis)

本 README 只是一个快速*入门*文档。你可以在 [redis.io](https://redis.io) 找到更详细的文档。

什么是 Redis？
---

Redis 通常被称为*数据结构*服务器。这意味着 Redis 通过一组命令提供对可变数据结构的访问，这些命令通过*客户端-服务器*模型、TCP 套接字和简单协议进行发送。因此，不同进程可以以共享的方式查询和修改相同的数据结构。

Redis 实现的数据结构具有以下几个特殊属性：

* Redis 会将它们存储到磁盘上，即使它们总是在服务器内存中被服务和修改。这意味着 Redis 很快，但也具备非易失性。
* 数据结构的实现强调内存效率，因此 Redis 内部的数据结构通常比用高级编程语言建模的同类数据结构占用更少的内存。
* Redis 提供了许多数据库常见的功能，如复制、可调节的持久化级别、集群和高可用性。

另一个很好的例子是将 Redis 看作是 memcached 的更复杂版本，操作不仅仅是 SET 和 GET，还可以对复杂数据类型如列表、集合、有序数据结构等进行操作。

如果你想了解更多，以下是一些推荐的起点：

* Redis 数据类型简介：https://redis.io/docs/latest/develop/data-types/
* Redis 命令全集：https://redis.io/commands
* 更多内容请见官方 Redis 文档：https://redis.io/documentation

什么是 Redis Community Edition？
---

自 v7.4 版本起，Redis OSS 被重命名为 Redis Community Edition（CE）。

Redis Ltd. 还提供 [Redis Software](https://redis.io/enterprise/)，这是一款自管理软件，具备企业级扩展所需的合规性、可靠性和弹性增强功能；
以及 [Redis Cloud](https://redis.io/cloud/)，这是一个与 Google Cloud、Azure 和 AWS 集成的全托管服务，适用于生产级应用。

关于 Redis Community Edition 与 Redis 其他版本的区别，请参见 [这里](https://redis.io/comparisons/oss-vs-enterprise/)。

构建 Redis
---

Redis 可以在 Linux、OSX、OpenBSD、NetBSD、FreeBSD 上编译和使用。
我们支持大端和小端架构，以及 32 位和 64 位系统。

它也许可以在 Solaris 衍生系统（如 SmartOS）上编译，但我们对该平台的支持是*尽力而为*，不能保证像在 Linux、OSX 和 \*BSD 上那样稳定。

编译非常简单：

    % make

如需构建带 TLS 支持的版本，你需要 OpenSSL 开发库（如 Debian/Ubuntu 下的 libssl-dev），然后运行：

    % make BUILD_TLS=yes

如需构建带 systemd 支持的版本，你需要 systemd 开发库（如 Debian/Ubuntu 下的 libsystemd-dev 或 CentOS 下的 systemd-devel），然后运行：

    % make USE_SYSTEMD=yes

如需为 Redis 程序名添加后缀，使用：

    % make PROG_SUFFIX="-alt"

如需构建 32 位 Redis 二进制文件，使用：

    % make 32bit

编译完成后，建议运行测试：

    % make test

如果构建了 TLS，需启用 TLS 运行测试（你需要安装 `tcl-tls`）：

    % ./utils/gen-test-certs.sh
    % ./runtest --tls

修复依赖或缓存构建选项导致的构建问题
---

Redis 有一些依赖项，包含在 `deps` 目录中。
`make` 不会自动重建依赖项，即使依赖项的源码发生了变化。

当你用 `git pull` 更新源码或以其他方式修改了依赖树中的代码时，请务必使用以下命令彻底清理并重新构建：

    % make distclean

这会清理 jemalloc、lua、hiredis、linenoise 及其他依赖项。

如果你强制使用了某些构建选项（如 32 位目标、无 C 编译器优化用于调试等），这些选项会被无限期缓存，直到你执行 `make distclean`。

修复构建 32 位二进制文件的问题
---

如果你在构建 32 位目标后需要重新构建 64 位目标，或反之，需要在 Redis 根目录下执行 `make distclean`。

如在构建 32 位 Redis 二进制文件时遇到错误，尝试以下步骤：

* 安装 libc6-dev-i386（也可尝试 g++-multilib）。
* 尝试用如下命令替代 `make 32bit`：
  `make CFLAGS="-m32 -march=native" LDFLAGS="-m32"`

内存分配器
---

选择非默认内存分配器可通过设置 `MALLOC` 环境变量实现。默认情况下，Redis 在 Linux 下使用 jemalloc，其它系统下使用 libc malloc。选择 jemalloc 是因为其内存碎片问题较少。

强制使用 libc malloc：

    % make MALLOC=libc

在 Mac OS X 下使用 jemalloc：

    % make MALLOC=jemalloc

单调时钟
---

默认情况下，Redis 使用 POSIX 的 clock_gettime 作为单调时钟源。在大多数现代系统上，可以使用内部处理器时钟以提升性能。相关注意事项见：http://oliveryang.net/2015/09/pitfalls-of-TSC-usage/

如需支持处理器内部指令时钟，使用：

    % make CFLAGS="-DUSE_PROCESSOR_CLOCK"

详细构建
---

Redis 默认以用户友好的彩色输出构建。
如需更详细的输出，使用：

    % make V=1

运行 Redis
---

使用默认配置运行 Redis：

    % cd src
    % ./redis-server

如需指定配置文件运行：

    % cd src
    % ./redis-server /path/to/redis.conf

也可以通过命令行参数直接修改配置。例如：

    % ./redis-server --port 9999 --replicaof 127.0.0.1 6379
    % ./redis-server /etc/redis/6379.conf --loglevel debug

redis.conf 中的所有选项都可以通过命令行参数指定，名称完全一致。

使用 TLS 运行 Redis
---

请参见 [TLS.md](TLS.md) 文件，了解如何使用 TLS。

玩转 Redis
---

你可以使用 redis-cli 与 Redis 交互。启动 redis-server 后，在另一个终端尝试：

    % cd src
    % ./redis-cli
    redis> ping
    PONG
    redis> set foo bar
    OK
    redis> get foo
    "bar"
    redis> incr mycounter
    (integer) 1
    redis> incr mycounter
    (integer) 2
    redis>

所有可用命令见：https://redis.io/commands

安装 Redis
---

如需将 Redis 二进制文件安装到 /usr/local/bin，使用：

    % make install

如需安装到其他目录：

    % make PREFIX=/some/other/directory install

`make install` 只会安装二进制文件，不会配置启动脚本和配置文件。如果你只是想体验 Redis，这样就够了；但如果要在生产环境中安装，我们提供了一个适用于 Ubuntu 和 Debian 的脚本：

    % cd utils
    % ./install_server.sh

_注意_：`install_server.sh` 不适用于 Mac OSX，仅适用于 Linux。

该脚本会询问你一些问题，并完成后台守护进程的配置，使其能随系统重启自动启动。

你可以通过 `/etc/init.d/redis_<portnumber>` 脚本启动和停止 Redis，例如 `/etc/init.d/redis_6379`。

代码贡献
---

向 Redis 项目贡献代码（包括通过 GitHub 提交 PR、通过邮件或讨论组发送代码片段或补丁），即表示你同意在 [Redis 软件授权与贡献者许可协议][1] 下发布你的代码。Redis 软件包含对原始 Redis 核心项目的贡献，这些贡献归贡献者所有，并遵循 3BSD 许可证。本仓库中的该许可证副本仅适用于这些贡献。自 7.4.x 版本起，Redis Community Edition 采用 RSALv2/SSPL 双许可证，详见 [LICENSE.txt][2]。

更多信息请参见本仓库的 [CONTRIBUTING.md][1]。安全漏洞请参见 [SECURITY.md][3]。

[1]: https://github.com/redis/redis/blob/unstable/CONTRIBUTING.md
[2]: https://github.com/redis/redis/blob/unstable/LICENSE.txt
[3]: https://github.com/redis/redis/blob/unstable/SECURITY.md

Redis 商标
---

商标的目的是识别某人或公司的商品和服务，避免混淆。作为名称和标志的注册所有者，Redis 接受对其商标的有限使用，但必须遵守其商标指南，详见：https://redis.com/legal/trademark-guidelines/

Redis 内部原理
===

如果你正在阅读本 README，说明你可能正在浏览 GitHub 页面，或刚刚解压了 Redis 源码包。无论哪种情况，你都离源码只有一步之遥。下面我们介绍 Redis 源码布局、各文件内容、最重要的函数和结构等。我们保持高层次讨论，不深入细节（否则文档会非常庞大且代码持续变化），但这些内容足以作为理解的起点。大部分代码都有详细注释，易于跟踪。

源码布局
---

Redis 根目录包含本 README、Makefile（实际调用 `src` 目录下的 Makefile）以及 Redis 和 Redis Sentinel 的示例配置。还有一些 shell 脚本用于执行 Redis、Redis Cluster 和 Redis Sentinel 的单元测试，测试实现位于 `tests` 目录。

根目录下主要有以下重要目录：

* `src`：Redis 的 C 语言实现。
* `tests`：单元测试，使用 Tcl 实现。
* `deps`：Redis 使用的第三方库。所有编译所需依赖都在此目录下，系统只需提供 `libc`、POSIX 兼容接口和 C 编译器。特别地，`deps` 包含 jemalloc（Linux 下默认分配器）。注意，`deps` 下也有一些最初由 Redis 项目发起，但主仓库并非 `redis/redis` 的内容。

还有一些其他目录，但对我们的目标不是很重要。我们主要关注 `src`，即 Redis 实现所在，按逻辑顺序逐步介绍各文件，便于逐层理解复杂性。

注意：最近 Redis 进行了较大重构，函数和文件名有所变化，本说明更贴近 `unstable` 分支。例如，Redis 3.0 中的 `server.c` 和 `server.h` 过去叫 `redis.c` 和 `redis.h`，但整体结构一致。所有新开发和 PR 应提交到 `unstable` 分支。

server.h
---

理解一个程序最简单的方法是理解其使用的数据结构。我们从 Redis 的主头文件 `server.h` 开始。

所有服务器配置和共享状态都定义在一个全局结构 `server`（类型为 `struct redisServer`）中。该结构中几个重要字段：

* `server.db`：Redis 数据库数组，存储数据。
* `server.commands`：命令表。
* `server.clients`：已连接客户端链表。
* `server.master`：主节点客户端（如果本实例为从节点）。

还有许多其他字段，结构体定义中有详细注释。

另一个重要结构是客户端结构体。过去叫 `redisClient`，现在叫 `client`。主要字段如下：
```c
struct client {
    int fd;
    sds querybuf;
    int argc;
    robj **argv;
    redisDb *db;
    int flags;
    list *reply;
    // ... 其他字段 ...
    char buf[PROTO_REPLY_CHUNK_BYTES];
}
```
该结构定义了一个*已连接客户端*：

* `fd`：客户端套接字文件描述符。
* `argc` 和 `argv`：当前客户端执行命令的参数，命令实现函数可读取参数。
* `querybuf`：累积客户端请求，Redis 服务器按协议解析并执行。
* `reply` 和 `buf`：动态和静态缓冲区，累积服务器对客户端的回复，缓冲区在文件描述符可写时逐步写入套接字。

如上所示，命令参数为 `robj` 结构。`robj` 定义如下：

```c
struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* LRU 时间或 LFU 数据 */
    int refcount;
    void *ptr;
};
```

该结构可表示所有基本 Redis 数据类型（字符串、列表、集合、有序集合等）。`type` 字段表示对象类型，`refcount` 支持多处引用同一对象，`ptr` 指向实际数据（同类型对象的实现可能不同，取决于 `encoding`）。

Redis 对象在内部广泛使用，但为避免间接访问开销，最近很多地方直接用动态字符串而非 Redis 对象。

server.c
---

这是 Redis 服务器的入口，定义了 `main()`。启动流程主要步骤：

* `initServerConfig()`：设置 `server` 结构的默认值。
* `initServer()`：分配所需数据结构，设置监听套接字等。
* `aeMain()`：启动事件循环，监听新连接。

事件循环中有两个特殊函数：

1. `serverCron()`：按 `server.hz` 频率周期调用，执行定时任务（如检查超时客户端）。
2. `beforeSleep()`：每次事件循环处理完请求、返回循环前调用。

server.c 还包含其他核心功能：

* `call()`：在指定客户端上下文中调用命令。
* `activeExpireCycle()`：处理设置了过期时间的键的淘汰。
* `performEvictions()`：当写入命令到来且内存超限时，根据 `maxmemory` 策略淘汰键。
* 全局变量 `redisCommandTable` 定义所有 Redis 命令，包括命令名、实现函数、参数数量及其他属性。

commands.c
---
该文件由 utils/generate-command-code.py 自动生成，内容基于 src/commands 目录下的 JSON 文件。这些 JSON 文件是 Redis 命令及其元数据的唯一数据源。不要直接使用这些 JSON 文件，相关元数据可通过 `COMMAND` 命令获取。

networking.c
---

该文件定义了所有与客户端、主节点、从节点的 I/O 函数（主从节点本质上也是特殊客户端）：

* `createClient()`：分配并初始化新客户端。
* `addReply*()` 系列函数：命令实现用来向客户端结构追加回复数据，最终发送给客户端。
* `writeToClient()`：将输出缓冲区数据写入客户端，由可写事件处理器 `sendReplyToClient()` 调用。
* `readQueryFromClient()`：可读事件处理器，将客户端数据读入查询缓冲区。
* `processInputBuffer()`：按 Redis 协议解析客户端查询缓冲区，准备好命令后调用 `processCommand()`（定义在 server.c）实际执行命令。
* `freeClient()`：释放、断开并移除客户端。

aof.c 和 rdb.c
---

顾名思义，这两个文件实现了 Redis 的 RDB 和 AOF 持久化。Redis 的持久化模型基于 `fork()`，创建一个拥有主进程内存快照的子进程，由子进程将内存内容写入磁盘。`rdb.c` 用于生成快照，`aof.c` 用于 AOF 重写。

aof.c 还实现了 API，允许命令在执行时将新命令追加到 AOF 文件。

server.c 中的 `call()` 负责调用写入 AOF 的相关函数。

db.c
---

某些 Redis 命令针对特定数据类型，另一些则是通用命令（如 `DEL`、`EXPIRE`）。这些通用命令定义在 db.c。

此外，db.c 实现了操作 Redis 数据集的 API，避免直接访问内部数据结构。

常用函数有：

* `lookupKeyRead()` 和 `lookupKeyWrite()`：根据键获取值指针，不存在则返回 NULL。
* `dbAdd()` 及高级接口 `setKey()`：在数据库中创建新键。
* `dbDelete()`：删除键及其值。
* `emptyData()`：清空单个或全部数据库。

文件其余部分实现了对客户端暴露的通用命令。

object.c
---

前文已介绍 `robj` 结构。object.c 包含所有操作 Redis 对象的基础函数，如分配新对象、引用计数等。主要函数有：

* `incrRefCount()` 和 `decrRefCount()`：增加或减少对象引用计数，降为 0 时释放对象。
* `createObject()`：分配新对象。还有专门为特定内容分配字符串对象的函数，如 `createStringObjectFromLongLong()` 等。

本文件还实现了 `OBJECT` 命令。

replication.c
---

这是 Redis 最复杂的文件之一，建议熟悉其他代码后再阅读。该文件实现了 Redis 的主从角色。

最重要的函数之一是 `replicationFeedSlaves()`，用于将命令写入代表从节点的客户端，使其数据集与主节点保持同步。

本文件还实现了 `SYNC` 和 `PSYNC` 命令，用于主从初次同步或断线重连后的继续同步。

脚本
---

脚本单元由 3 个部分组成：
* `script.c` - 脚本与 Redis 的集成（命令执行、设置复制/RESP 等）
* `script_lua.c` - 负责执行 Lua 代码，使用 `script.c` 与 Redis 交互
* `function_lua.c` - Lua 引擎实现，使用 `script_lua.c` 执行 Lua 代码
* `functions.c` - Redis Functions 实现（`FUNCTION` 命令），如需 Lua 引擎则调用 `functions_lua.c`
* `eval.c` - `eval` 命令实现，使用 `script_lua.c` 调用 Lua 代码

其他 C 文件
---

* `t_hash.c`、`t_list.c`、`t_set.c`、`t_string.c`、`t_zset.c`、`t_stream.c`：实现各数据类型的 API 及客户端命令。
* `ae.c`：事件循环实现，是一个自包含、易读的库。
* `sds.c`：Redis 字符串库，详见 https://github.com/antirez/sds
* `anet.c`：简化 POSIX 网络编程的库。
* `dict.c`：非阻塞哈希表实现，支持渐进式 rehash。
* `cluster.c`：Redis 集群实现，建议熟悉其他代码后再阅读。阅读前建议先看 [Redis Cluster 规范][4]。

[4]: https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/

Redis 命令剖析
---

所有 Redis 命令定义如下：

```c
void foobarCommand(client *c) {
    printf("%s",c->argv[1]->ptr); /* 处理参数 */
    addReply(c,shared.ok); /* 回复客户端 */
}
```

命令函数由 JSON 文件引用，并包含元数据，详见上文 commands.c。命令标志详见 `server.h` 中 `struct redisCommand` 的注释。更多细节请参考 `COMMAND` 命令：https://redis.io/commands/command/

命令执行后，通常通过 `addReply()` 或 networking.c 中的类似函数回复客户端。

Redis 源码中有大量命令实现可供参考（如 pingCommand）。编写一些简单命令有助于熟悉代码。

还有许多文件未在此描述，但没必要全部覆盖。希望这些内容能帮助你迈出第一步，最终你会找到 Redis 代码库的方向 :-)

祝你玩得愉快！