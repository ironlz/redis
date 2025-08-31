**核心服务初始化**

1. **initServer 主要作用与时序图**

待办清单
- 概要说明 `initServer` 的主要职责。
- 列出 `initServer` 内的关键子步骤与目的。
- 给出主启动流程中 `initServer` 的时序图（ASCII）。

主要职责（简要）
- 将全局 `server` 状态和各种子系统从“配置/默认”过渡到“运行就绪”。
- 初始化信号处理、线程基础设施与日志系统。
- 分配并初始化大量全局数据结构（clients 列表、数据库数组、索引、统计、复制/AOF/RDB 状态等）。
- 初始化脚本/Lua、functions、慢日志、延迟监控、ACL 等子系统。
- 准备监听器相关的数据结构（真正的 bind/listen 在 `initListeners()` 中进行）。
- 将某些必须在模块加载或线程创建后执行的初始化延后到 `InitServerLast()`，以避免 dlopen/线程 TLS 的竞态问题。

关键子步骤（典型顺序与目的）
1. 忽略或注册信号：`signal(SIGHUP, SIG_IGN); signal(SIGPIPE, SIG_IGN); setupSignalHandlers();` — 避免不期望的进程中止/PIPE 中断，并安装必要的信号处理器。
2. 线程与取消策略：`ThreadsManager_init(); makeThreadKillable();` — 初始化线程管理并设置线程可取消性。
3. 日志与 syslog：若启用则 `openlog()`。
4. 初始化 `server.*` 字段与容器：runid、hz、lists/rax/dicts、统计计数器、复制与持久化默认状态等（并调用 `resetServerStats()`、`resetReplicationBuffer()` 等）。
5. 初始化脚本/函数环境：`luaEnvInit(); scriptingInit(); functionsInit();` — 初始化 Lua 与 Functions，失败则退出。
6. 初始化监控子系统：`slowlogInit(); latencyMonitorInit();`。
7. 安全与 ACL：`ACLUpdateDefaultUserPassword(server.requirepass);` 等。
8. 准备监听器配置（填充 `server.listeners` 等），但不执行 `listen()`（由 `initListeners()` 完成）。
9. 返回主流程，由主流程继续执行 daemonize、模块加载、`initListeners()`、`InitServerLast()` 与数据加载等步骤。

主启动流程时序（简化 ASCII 图）

```
main
  |
  |-- 解析命令行与载入配置
  |
  |-- initServerConfig()    // 设置默认配置
  |
  |-- initServer()          // <-- 主要初始化（下列为内部要点）
  |     |-- signal(...)
  |     |-- setupSignalHandlers()
  |     |-- ThreadsManager_init()/makeThreadKillable()
  |     |-- openlog() (若启用)
  |     |-- 初始化 server.* 字段、容器、统计（resetServerStats 等）
  |     |-- luaEnvInit()/scriptingInit()/functionsInit()
  |     \-- slowlogInit()/latencyMonitorInit()/ACL 初始化
  |
  |-- daemonize / createPidFile / set proc title / ascii art
  |
  |-- checkTcpBacklogSettings()
  |
  |-- moduleInit / moduleLoad* (若非 sentinel)
  |
  |-- ACLLoadUsersAtStartup()
  |
  |-- initListeners()       // 执行 socket bind/listen, 创建 accept handlers
  |
  |-- InitServerLast()      // 延后初始化（bioInit(), initThreadedIO(), jemalloc bg thread 等）
  |
  |-- aofLoadManifestFromDisk()
  |-- loadDataFromDisk()    // 加载 RDB/AOF 到内存
  |
  \-- aeMain(server.el)     // 进入事件循环，开始对外服务
```

注意与提示
- `initServer()` 是“准备就绪”的核心，但真实的网络监听由 `initListeners()` 完成，因此要分清两者边界。
- 某些初始化被推迟到 `InitServerLast()`，这是为了解决 dlopen/线程本地存储的竞态问题（模块加载与线程创建的先后依赖）。
- 如果需要，我可以把上述时序图导出为 PlantUML 或把每个步骤对应的源码行号加入备注，以便精确追踪实现位置。