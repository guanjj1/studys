### 基础
<https://www.bilibili.com/video/BV13R4y1v7sP?p=12&vd_source=ac44a38168194f76fada7686c12a4a37>

### 常见命令及使用场景
<https://baijiahao.baidu.com/s?id=1756962745464790137&wfr=spider&for=pc>\
<https://zhuanlan.zhihu.com/p/683564770>\
<https://blog.csdn.net/qq_61323055/article/details/129896694>\
<https://blog.csdn.net/weixin_52372879/article/details/129680584>

### 命令查询
<https://redis.io/docs/latest/commands/>

### redis持久化
#### rdb
RDB 的优点:RDB 是一个非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。 这种文件非常适合用于进行备份： 比如说，你可以在最近的 24 小时内，每小时备份一次 RDB 文件，并且在每个月的每一天，也备份一个 RDB 文件。 这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。RDB 非常适用于灾难恢复（disaster recovery）：它只有一个文件，并且内容都非常紧凑，可以（在加密后）将它传送到别的数据中心，或者亚马逊 S3 中。RDB 可以最大化 Redis 的性能：父进程在保存 RDB 文件时唯一要做的就是 fork 出一个子进程，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作。RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

RDB 的缺点:如果你需要尽量避免在服务器故障时丢失数据，那么 RDB 不适合你。 虽然 Redis 允许你设置不同的保存点（save point）来控制保存 RDB 文件的频率， 但是， 因为RDB 文件需要保存整个数据集的状态， 所以它并不是一个轻松的操作。 因此你可能会至少 5 分钟才保存一次 RDB 文件。 在这种情况下， 一旦发生故障停机， 你就可能会丢失好几分钟的数据。每次保存 RDB 的时候，Redis 都要 fork() 出一个子进程，并由子进程来进行实际的持久化工作。 在数据集比较庞大时， fork() 可能会非常耗时，造成服务器在某某毫秒内停止处理客户端； 如果数据集非常巨大，并且 CPU 时间非常紧张的话，那么这种停止时间甚至可能会长达整整一秒。 虽然 AOF 重写也需要进行 fork() ，但无论 AOF 重写的执行间隔有多长，数据的耐久性都不会有任何损失。

#### aof
AOF 的优点:使用 AOF 持久化会让 Redis 变得非常耐久（much more durable）：你可以设置不同的 fsync 策略，比如无 fsync ，每秒钟一次 fsync ，或者每次执行写入命令时 fsync 。AOF 的默认策略为每秒钟 fsync 一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据（ fsync 会在后台线程执行，所以主线程可以继续努力地处理命令请求）。AOF 文件是一个只进行追加操作的日志文件（append only log）， 因此对 AOF 文件的写入不需要进行 seek ， 即使日志因为某些原因而包含了未写入完整的命令（比如写入时磁盘已满，写入中途停机，等等）， redis-check-aof 工具也可以轻易地修复这种问题。Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。

AOF 的缺点:对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。AOF 在过去曾经发生过这样的 bug ： 因为个别命令的原因，导致 AOF 文件在重新载入时，无法将数据集恢复成保存时的原样。 （举个例子，阻塞命令 BRPOPLPUSH 就曾经引起过这样的 bug 。） 测试套件里为这种情况添加了测试： 它们会自动生成随机的、复杂的数据集， 并通过重新载入这些数据来确保一切正常。 虽然这种 bug 在 AOF 文件中并不常见， 但是对比来说， RDB 几乎是不可能出现这种 bug 的。

### 配置文件
<https://blog.csdn.net/m0_63237640/article/details/138302350>
```conf
#把文件中的daemonize属性改为yes（表明需要在后台运行）,docker中设置为no
daemonize yes

#指定log日志文件的位置 自己建一下文件夹和文件
logfile "/usr/local/redis/logs/redis.log"

# 绑定地址，默认是127.0.0.1，会导致只能在本地访问。修改为0.0.0.0则可以在任意IP访问
bind 0.0.0.0或注释该行

; 指定Redis服务的端口号，默认为6379。
port：6379

# 保护模式，关闭保护模式 设置成 no 允许远程访问
protected-mode no

# 配置持久化文件存放的路径 r
dir /usr/local/redis

# 配置文件中的appendonly修改为yes，开启AOF持久化  其他默认即可
appendonly yes

#配置密码 # requirepass foobared 去除 #，foobared修改为自己的密码  
requirepass 123456

dbfilename：指定RDB持久化的数据库文件名，默认为dump.rdb。
```
```conf
[root@redis-server etc]# grep -Ev "^$|#" redis.conf
# 绑定的主机地址， "127.0.0.1":代表本地地址，访问redis服务只能通过本机的客户端连接，而无法通过远程连接,"0.0.0.0":接受所有来自于可用网络接口的连接
bind “127.0.0.1” -::1

# 是否开启保护模式，默认开启。要是配置里没有指定bind和密码。开启该参数后，redis只会本地进行访问，拒绝外部访问。要是开启了密码和bind，可以开启。否则最好关闭设置为no。
protected-mode yes

# redis进程的端口号
port 6379

# 此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度， 当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值，默认是511，而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。
tcp-backlog 511

# 客户端闲置多长时间后关闭连接，默认此参数为0即关闭此功能
timeout 0

# TCP keepalive，tcp连接保活时长，在linux系统中，客户端发送的最后一个数据包与redis发送的第一个保活探测报文之间的时间间隔。单位是秒。默认维持300s的长连接。
# 服务器端以这个设置的时间对客户端进行查询，每隔设置的时间查询一次，若客户端无响应，则关闭其连接。默认是300秒。如果设置为0，则表示不进行保活检测
tcp-keepalive 300

# redis进程是否以守护进程的方式运行，yes为是，no为否(不以守护进程的方式运行会占用一个终端)。
daemonize no

# 指定redis进程的PID文件存放位置
pidfile /var/run/redis_6379.pid

# redis日志级别，可用的级别有debug.verbose.notice.warning
loglevel notice

# log文件输出位置，如果进程以守护进程的方式运行，此处又将输出文件设置为stdout的话，就会将日志信息输出到/dev/null里面去了
logfile ""

# 设置数据库的数量，默认为0可以使用select <dbid>命令在连接上指定数据库id
databases 16

# 是否总是显示logo
always-show-logo no

# 设置进程标题，该配置选项是用于配置 Redis 服务器是否应该设置操作系统的进程标题。这通常用于在操作系统的进程列表中更容易识别正在运行的 Redis 进程。
set-proc-title yes

# 在更改进程标题时，Redis 使用以下模板来构造修改后的标题：{title} 父进程执行的进程名称，或子进程的类型;{listen-addr} 绑定地址或 ‘*’ 后跟 TCP 或 TLS 端口侦听，或 Unix 套接字（如果可用）;{server-mode} 特殊模式，即“[sentinel]”或“[cluster]”
proc-title-template "{title} {listen-addr} {server-mode}"

# 用于设置用于排序的区域设置。这个配置项通常不需要设置，因为Redis会根据系统的区域设置自动进行排序。
locale-collate ""

# redis的RDB持久化配置。如果为"yes"：redis会创建一个新的后台进程dump rdb。创建快照（硬盘上，产生一个新的rdb文件）需要 20s时间，redis主进程，在这20s内，会继续接受客户端命令，但是，就在这20s内创建快照! 出错了，比如磁盘满了，那么redis会认为：Redis被配置为保存RDB快照，但目前无法在磁盘上保持。但当前无法在磁盘上持久化。那么，redis会，拒绝新的写入，也就是说，它认为你当下持久化数据出现了问题，你就不要再set key啦。
stop-writes-on-bgsave-error yes

# 设置存储至本地数据库时是否压缩数据，默认为 yes，采用 LZF 压缩。通常默认为开启状态，如果设置为no，可以节省 CPU 运行时间，但会使存储的文件变大（巨大）。
rdbcompression yes

# 设置是否进行RDB文件格式校验，该校验过程在写文件和读文件过程均进行。通常默认为开启状态，如果设置为no，可以节约读写性过程约10%时间消耗，但是存储一定的数据损坏风险。
rdbchecksum yes

# 设置本地数据库文件名，默认值为 dump.rdb。通常设置为dump-端口号.rdb。
dbfilename dump.rdb

# 主从进行全量同步时，通过传输 RDB 内存快照文件实现，没有开启 RDB 持久化的实例在同步完成后会删除该文件，通常情况下保持默认即可。
rdb-del-sync-files no

# 指定本地数据存放位置
dir ./


# 当复制副本失去与主机的连接时，或者当复制如果仍在进行中，复制副本可以以两种不同的方式进行操作：1）如果replica serve stale data设置为“yes”（默认值），则复制副本将仍然回复客户端请求，可能包含过期数据，或者如果这是第一次同步，数据集可能只是空的。2）如果replica serve stale data设置为“no”，则replica将用所有类型的命令都出现“正在与主控同步”错误。数据同步过程中，备机是否支持查询。yes可以查询老数据，no必须同步完
replica-serve-stale-data yes

# replica-read-only支持参数[yes,no]，yes 开启只读模式，no 支持读写操作。
replica-read-only yes

# 该参数默认值为no，也即从节点请求全量复制的时候，主节点上不开启无盘复制，先通过bgsave做持久化，然后传输RDB文件到从节点，全量同步的可以通过两种方式进行：基于磁盘的复制：Redis主节点会创建一个新的进程，在磁盘上写入RDB文件。之后，父进程会逐步将文件传输给副本；无磁盘复制：Redis主节点创建的新进程会直接将RDB文件写入副本的套接字，完全不涉及磁盘操作；在使用基于磁盘的复制时，当RDB文件正在生成时，其他的副本可以排队，并能在当前子进程完成RDB文件生成后立刻使用这个文件。在磁盘速度慢而网络速度快（带宽大）的情况下，无磁盘复制表现得更好。这是因为无磁盘复制避免了磁盘I/O操作的瓶颈，可以更快地通过网络直接将数据发送给副本。而基于磁盘的复制方式在传输前需要将数据写入磁盘，这在磁盘性能不佳的情况下可能会成为性能瓶颈。因此，在网络速度快而磁盘速度慢的环境中，选择无磁盘复制通常是一个更好的选择。（该参数主要影响主节点）
repl-diskless-sync yes

# 默认值为5，单位为秒，该参数只有开启了repl-diskless-sync 为yes的时候才会生效。在使用无磁盘复制时，一旦传输开始，新到达的副本会被排队，直到当前传输结束，新的传输才会开始。当使用无磁盘复制时，主节点会等待一个时间段repl-diskless-sync-delay（以秒为单位）再开始传输，这样做的目的是希望多个副本能够在这段时间内到达，从而使传输过程可以并行化。（该参数主要影响主节点）
repl-diskless-sync-delay 5

# 该参数默认值为0；当使用延迟启用无磁盘复制时，如果连接的副本达到预期的最大数量，则可以在达到最大延迟之前启动复制。默认值为0表示没有定义最大值，Redis将等待完整的repl-diskless-sync-delay。（该参数主要影响主节点）
repl-diskless-sync-max-replicas 0

# 该参数默认为disabled。副本可以直接从socket中加载它从复制链接读取的RDB，直接加载到内存或者暂存在本地磁盘，完整地接收主节点的RDB文件之后再加载到内存。在许多情况下，磁盘比网络慢，存储RDB到磁盘和加载RDB文件到内存，可能会增加复制时间。但是，当直接从套接字解析RDB文件时，为了避免数据丢失，只有在新数据集完全加载到内存中时才会刷新当前数据集，这将导致更高的内存使用。出于这个原因，我们有以下选项:"disabled"  ：（从节点）不使用无磁盘加载(首先将rdb文件存储到磁盘)，意思是完整地将RDB写入磁盘文件之后，再从磁盘加载到内存。"swapdb"  ：在直接从套接字解析数据时，将当前数据库内容保存在RAM中。这种模式下的副本可以在复制过程中继续服务当前数据集，除非它们不能识别出主数据集具有来自相同复制历史的数据集。注意，这需要足够的内存（理论上需要双倍的内存，一份是从主节点接收的RDB文件，一份是从RDB文件转换为当前实例的内存），如果没有，将会存在OOM的风险；"on-empty-db"  ：只在当前数据集为空时使用无磁盘加载。这样更安全，避免了在复制过程中同时加载旧数据集和新数据集。
repl-diskless-load disabled

# 在slave和master同步后（发送psync/sync），后续的同步是否设置成TCP_NODELAY；假如设置成yes，则redis会合并小的TCP包从而节省带宽，但会增加同步延迟（40ms），造成master与slave数据不一致；假如设置成no，则redis master会立即发送同步数据，没有延迟
repl-disable-tcp-nodelay no

# 从机竞争主机的优先级，值越小优先级越高。如有三个 replicas 节点其 priority 值分别为 10，100，25， Sentinel 会选择 priority 为 10 的节点进行提升。这个值为 0 表示 replica 节点永远不能被提升为 master 节点。
replica-priority 100

# 日志最多保存的条数,先进先出淘汰。acllog是一个记录所有ACL（访问控制列表）相关日志的列表，acllog-max-len是这个日志列表的最大长度。当超过这个长度时，最旧的日志条目将被删除。
acllog-max-len 128


# 以非阻塞方式释放内存
# 内存释放策略配置。当Redis的内存使用达到配置的maxmemory上限时，Redis需要根据一定的策略选择一些键进行淘汰，以释放内存。如果lazyfree-lazy-eviction设置为yes，则表示启用懒惰淘汰策略，即在需要淘汰键时，Redis会延迟释放这些键所占用的内存，直到有必要的时候，例如当某个写命令需要分配内存时。这样做可以减少CPU的使用，因为释放内存不需要额外的计算。如果你正在使用Redis的命令行工具或者是在运行时，你可以使用CONFIG SET命令来临时设置这个选项：CONFIG SET lazyfree-lazy-eviction yes
lazyfree-lazy-eviction no

# 当设置了过期 key 的过期时间到了，将删除 key。针对设置有TTL的键，达到过期时间后，被redis清理删除时是否采用lazy free机制；此场景建议开启，因TTL本身是自适应调整的速度。即释放过期 key 的内存，放到后台线程执行
lazyfree-lazy-expire no

# 内部删除选项，比如rename srckey destkey时，如果destkey存在需要先删除destkey。针对有些指令在处理已存在的键时，会带有一个隐式的DEL键的操作。如rename命令，当目标键已存在,redis会先删除目标键，如果这些目标键是一个big key,那就会引入阻塞删除的性能问题。此参数设置就是解决这类问题，建议可开启。
lazyfree-lazy-server-del no

# 主要用于处理Redis副本（replica）的数据刷新。当replica-lazy-flush设置为yes时，副本会延迟刷新数据直到有实际的写入操作发生。这样做可以减少不必要的数据刷新，从而提高性能。在Redis中，副本通常会在主库进行每个命令的同步时进行数据的刷新。当replica-lazy-flush设置为yes时，副本会等到接收到主库发送的命令才进行数据的刷新。
replica-lazy-flush no

#  配置选项是Redis中的一个优化参数，用于控制内存回收策略。当你删除一个键时，Redis并不会立即释放掉这个键所占用的内存，而是将其标记为要删除，并在后台的某个不太繁忙的时间点再进行实际的内存释放。这样做可以减少删除键时对Redis性能的影响。
lazyfree-lazy-user-del no

# Redis中的一个性能调优选项，用于控制Redis的内存释放策略。当你删除一个键时，Redis并不会立即释放该键所占用的内存，而是将其标记为等待释放，并在后台按照一定的策略释放内存。这样做的目的是为了减少删除键操作时对Redis性能的影响。
lazyfree-lazy-user-flush no

# 是Linux内核的一个特性，它允许进程在内存不足（OOM，Out-Of-Memory）时有更高的可能性被内核杀死。在Redis中，oom-score-adj是用来调整Redis在系统内存不足时被内核杀死（被释放）的可能性。oom-score-adj的值可以是-1000到1000之间的整数。值越高，进程被KILL的可能性越大。
oom-score-adj no

# oom-score-adj 是 Linux 内核的一个特性，用于在系统内存不足时决定哪些进程最先被杀死以释放内存。在 Redis 中，oom-score-adj-values 是用来配置 Redis 和其他 Linux 进程的 oom-score-adj 值的。这个配置将为 Redis 主进程设置 oom-score-adj 为 0，为 Redis 的 RDB 和 AOF 子进程设置为 200，为 Redis 的共享内存子进程设置为 1000。
oom-score-adj-values 0 200 800

# disable-thp是一个不太常见的配置项，它并不是Redis官方提供的标准配置选项。transparent_hugepage是Linux内核的一个特性，用于提高内存管理性能，但可能会影响Redis的性能。
disable-thp yes

# appendonly 配置指令在Redis中用于控制是否开启AOF（Append Only File）持久化策略。AOF 是 Redis 的另一种数据持久化方式，它通过保存 Redis 服务器状态的每一个写操作来记录数据变化，以文件的形式保存在磁盘上。no：不开启AOF持久化功能。everysec：每秒钟同步一次到磁盘，这是appendonly的默认值。always：每执行一个写命令，立即同步到磁盘。（最好不要将redis.conf中的appendfsync设置为always，这极影响性能）
appendonly no

# "appendfilename"选项用于设置持久化数据的文件名。当Redis需要将数据写入磁盘时，会将数据追加到该文件中。在Redis启动时，会先读取该文件中的数据并加载到内存中，以恢复数据。
appendfilename "appendonly.aof"

# "appenddirname"选项用于设置持久化数据的文件所在路径。
appenddirname "appendonlydir"

# 每次执行完命令，每秒钟调用一次 fsync，先把日志写入到AOF内存缓冲区，每隔1s同步到磁盘。
appendfsync everysec

# 该配置项为"yes"时：当用户请求写入redis的时候，这部分数据只是保存在内存中，主线程并不会马上对此数据进行aof刷盘（而是根据aof刷盘的频率由子线程进行同步），这样子不会阻塞但是会导致数据丢失；该配置项为"no"时当用户请求写入redis的时候，这部分数据直接由主线程调用操作系统的fsync()进行aof刷盘，但是，如果此刻正在进行bgrewrite的时候则会导致主线程进行aof刷盘阻塞（前提是aof的刷盘为always），这样子可以保证数据一致性但是效率低
# 当no-appendfsync-on-rewrite选项处于打开状态时，在执行BGSAVE命令或者BGREWRITEAOF命令期间，服务器会暂时停止对AOF文件的同步，从而尽可能地减少I/O阻塞。
no-appendfsync-on-rewrite no

# 用于控制自动重写 AOF（Append Only File）文件的触发条件。当 AOF 文件的体积超过上次重写的体积的指定百分比时，Redis 会自动触发一个 AOF 重写操作。"auto-aof-rewrite-percentage 100" 意为大于上次保存容量的100%时（即当AOF文件是上次日志重写得到AOF文件大小的二倍时）执行重写（自动启动新的日志重写过程）。
auto-aof-rewrite-percentage 100

# 用于控制在Append Only File（AOF）持久化策略中，自动触发重写（rewrite）AOF文件的最小体积。当AOF文件的体积超过指定的大小时，Redis会自动开始一个新的AOF文件来代替旧的AOF文件，以此减少AOF文件的体积。即允许 AOF 重写的文件最小值。（为什么重写可以缩小 AOF 文件呢？因为可以把多条命令合并成一条命令，AOF 只需要记录 Key 最新的 Value 即可，而不用记录修改的历史记录。）
auto-aof-rewrite-min-size 64mb

# 该配置项用于指定当发生AOF文件末尾截断时，加载文件还是报错退出。Redis启动并加载AOF时，可能发现AOF文件的末尾被截断了。如果Redis所在的机器运行崩溃，就可能导致该现象。特别是在不使用 data=ordered 选项挂载ext4文件系统时。（但是Redis本身崩溃而操作系统正常运行则不会出现该情况）当发生了末尾截断，Redis可以选择直接报错退出，或者继续执行并恢复尽量多的数据（默认选项）。配置项 aof-load-truncated 用于控制此行为。yes ：末尾被截断的 AOF 文件将会被加载，并打印日志通知用户。no ：服务器将报错并拒绝启动。这时用户需要使用redis-check-aof 工具修复AOF文件，再重新启动。注意：如果AOF文件在中间（而不是末尾）发生了截断，仍然会报错退出。 aof-load-truncated只适用于当Redis将尝试从AOF文件读取更多的数据，但发现没有足够的字节时。
aof-load-truncated yes

# redis 4.0之后支持同时开启rdb和aof，配置为"yes"时，rdb+aof两种机制结合使用。当aof-use-rdb-preamble设置为yes时，Redis在执行AOF重写时，会将当前的RDB快照数据作为基础，然后追加AOF命令。这样做的好处是可以减少AOF文件的大小，并且在Redis服务器重启时，可以更快地通过RDB快照和AOF命令合并来恢复数据。如果Redis没有配置AOF，或者没有开启RDB快照，那么无论aof-use-rdb-preamble设置为何值，Redis都不会使用这种混合模式。
aof-use-rdb-preamble yes

# 用于控制 AOF (Append Only File) 文件名中是否包含时间戳。当启用时，默认情况下，Redis 会在创建 AOF 文件时，将当前时间戳附加到文件名中。这样做可以防止文件名冲突，同时也提供了文件创建时间的参考信息。
aof-timestamp-enabled no

# 用于指定执行时间超过该值的命令请求将会被记录在慢查询日志中。该值的单位是微秒，当命令的执行时长超过这个值时，该命令才会被记录在慢查询日志中。如果设置为0，则记录所有命令；如果设置为负数，则不记录任何命令。
slowlog-log-slower-than 10000

# 用于设置慢查询日志的最大长度。慢查询日志记录了执行时间超过指定阈值的命令及其详细信息，以帮助了解和优化Redis的性能问题。表示慢查询最大的条数，当slowlog超过设定的最大值后，会将最早的slowlog删除，是个FIFO队列（先进先出淘汰）
slowlog-max-len 128

# 用于指定处理命令请求的最大延迟时间（单位微秒）。当一个命令的处理时间超过这个阈值时，Redis 会在日志中记录一条警告信息。值设置为 0 的时候，延迟监控系统就会关闭。延迟检测模式是关闭的，可以通过该配置打开延迟监控
latency-monitor-threshold 0

# 用于实时监听和追踪键空间的变化。"notify-keyspace-events KEA"表示所有的键空间事件。Redis 的 notify-keyspace-events 功能可以让我们实时监听键空间中的事件，例如键的过期、删除等操作。
notify-keyspace-events ""

# 使用压缩列表保存时hash集合中的最大元素个数
hash-max-listpack-entries 512

# 使用压缩列表保存时hash集合中单个元素的最大长度;当哈希对象同时满足以下两个条件时， 使用ziplist（listpack）编码:1)哈希对象保存的键值对数量小于hash-max-listpack-entries的值;2)哈希对象保存的所有键值对的键和值的字符串长度的字节数都小于hash-max-listpack-value的值。  hash类型键的字段个数小于hash-max-listpack-entries且每个字段名和字段值的长度小于hash-max-listpack-value时，Redis才会使用OBJ_ENCODING_LISTPACK来存储该键，前述条件任意一个不满足则会转化为OBJ_ENCODING_HT编码方式。
hash-max-listpack-value 64

# 用于配置 quicklist 中的每个节点的 ziplist 的大小。 当这个值配置为正数时表示 quicklist 每个节点的 ziplist 最多可存储元素数量，超过该值就会使用 linkedlist 存储。当 list-max-ziplist-size 为负数时表示限制每个 quicklistNode 的 ziplist 的内存大小，超过这个大小就会使用 linkedlist 存储数据，配置为"- 2"则表示限制每个 listpack 大小，配置项不同的值分别有以下意义：
# -5：每个 quicklist 节点上的 ziplist 大小最大 64 kb <--- 正常环境不推荐
# -4：每个 quicklist 节点上的 ziplist 大小最大 32 kb <--- 不推荐
# -3：每个 quicklist 节点上的 ziplist 大小最大 16 kb <--- 可能不推荐
# -2：每个 quicklist 节点上的 ziplist 大小最大 8 kb <--- 不错
# -1：每个 quicklist 节点上的 ziplist 大小最大 4kb <--- 不错
list-max-listpack-size -2

# 这个参数表示一个quickList两端不被压缩的节点个数。为0，表示不进行压缩，此为quickList的默认值；为1，表示quickList的两端各有1个节点不进行压缩，中间结点开始进行压缩；为2，表示quickList的首尾2个节点不进行压缩，中间结点开始进行压缩；
list-compress-depth 0


# 在Redis7.2之前，当一个集合满足以下两个条件时。Redis 会选择使用intset编码：1）集合对象保存的所有元素都是整数值；2）集合对象保存的元素数量小于等于512个（默认）；为什么加入了listpack。在redis7.2之前，sds类型的数据会直接放入到编码结构式为hashtable的set中。其中，sds其实就是redis中的string类型。而在redis7.2之后，sds类型的数据，首先会使用listpack结构当 set 达到一定的阈值时，才会自动转换为hashtable。添加listpack结构是为了提高内存利用率和操作效率，因为 hashtable 的空间开销和碰撞概率都比较高。intset 、listpack和hashtable的转换。intset 、listpack和hashtable这三者的转换时根据要添加的数据、当前set的编码和阈值决定的。如果要添加的数据是整型，且当前set的编码为intset，如果超过阈值由intset直接转为hashtable。阈值条件为："set-max-intset-entries "intset最大元素个数，默认512;如果要添加的数据是字符串，分为三种情况:
#  1)当前set的编码为intset：如果没有超过阈值，转换为listpack；否则，直接转换为hashtable;
#  2)当前set的编码为listpack：如果超过阈值，就转换为hashtable;
#  3)当前set的编码为hashtable：直接插入，编码不会进行转换.
# 阈值条件为："set-max-listpack-entries"最大元素个数，默认128;"set_max_listpack_value"最大元素大小，默认64.以上两个条件需要同时满足才能进行编码转换,下面介绍相关配置：
# 该配置项可配置intset最大元素数量。数据量小于等于set-max-intset-entries用iniset，大于set-max-intset-entries用set。set数据类型内部数据如果全部是数值型，且包含多少节点以下会采用紧凑格式存储。
set-max-intset-entries 512

# 该配置项可配置listpack最大元素数量。
set-max-listpack-entries 128

# listpack单个元素最大大小
set-max-listpack-value 64

# 在 Redis 中，为了优化内存，列表类型的数据结构可能会使用名为 Listpack 的压缩格式。这个配置项就是用来设置在一个 Listpack 中允许的最大元素数量。
zset-max-listpack-entries 128

# 用来设置在一个 Listpack 中允许的单个元素的最大大小
zset-max-listpack-value 64

# 该配置为HyperLogLog 使用稀疏数据结构的限值。HyperLogLog 是一种高级数据结构，统计基数的利器。HyperLogLog 稀疏表示的字节限制。该限制包括16字节的标头。当使用稀疏结构的HyperLogLog超过此限制时，它将转换为密集结构的HyperLogLog。建议的值为 3000，可以在不减慢过多PFADD的情况下具有占用空间更小的优点，PFADD在稀疏编码中为O（N）。当CPU够用，但空间不够的情况，并且数据集由许多基数在0-15000范围内的HyperLogLogs组成时，该值可以提高到10000。value大小小于等于hll-sparse-max-bytes使用稀疏数据结构（sparse），大于hll-sparse-max-bytes使用稠密的数据结构（dense）。一个比16000大的value是几乎没用的，建议的value大概为3000。如果对CPU要求不高，对空间要求较高的，建议设置到10000左右。
hll-sparse-max-bytes 3000

# 流数据结构是基数编码内部多个项目的大节点树。使用此配置可以配置添加新 的流条目时，单个节点的字节数，以及切换到新节点之前可能包含的最大项目数。如果以下任何配置设置为0，就会忽略限制，因此可以设置一个例如将最大入口限制，把max-byte设置为0，max-entries设置为所需的值。
#  配置项用于指定Stream数据结构中每个节点（node）的最大容量（单位字节），即一个stream内的单个节点（entry）可以存储的最大数据量。这是为了防止节点变得过大，影响内存管理和性能。
stream-node-max-bytes 4096

# 用于设置Redis Stream切换到新节点之前可能包含的最大项目数。当一个Stream的节点（node）超过这个数值时，Redis会根据LRU（最近最少使用）算法来淘汰旧条目。
stream-node-max-entries 100

# 是否重置Hash表,设置成yes后redis将每100毫秒使用1毫秒CPU时间来对redis的hash表重新hash，可降低内存的使用,当使用场景有较为严格的实时性需求,不能接受Redis时不时的对请求有2毫秒的延迟的话，把这项配置为no,如果没有这么严格的实时性要求,可以设置为 yes,以便能够尽可能快的释放内存
activerehashing yes

# 可以用来强制断开无法足够快从 redis 服务器端读取数据的客户端。保护机制规则如下：1）[hard limit] 大小限制，当某一客户端缓冲区超过设定值后，直接关闭连接；2）[soft limit] 持续时间限制，当某一客户端缓冲区持续一段时间占用过大空间时关闭连接。该参数一般用在以下几类客户端中：1）普通 client，包括 monitor；2）主从同步时的 slave client；3）Pub/Sub 模式中的 client。
client-output-buffer-limit normal 0 0 0

# replica 是指从服务器（replica/slave）。这个配置项限制了在复制过程中，从服务器缓存命令的输出缓冲区的大小;例如 "client-output-buffer-limit replica 256mb 64mb 60"这行配置的意思是，对于从服务器的输出缓冲区，当单个缓冲区超过 32MB 时，或者总共超过 256MB 时，如果从服务器在 60 秒内没有发起同步操作，那么将会拒绝写入。
client-output-buffer-limit replica 256mb 64mb 60

# 对于pubsub client，如果client-output-buffer一旦超过32mb，又或者超过8mb持续60秒，那么服务器就会立即断开客户端连接。该配置表示，对于 Pub/Sub 客户端来说，若 output-buffer 占用内存达到 32M 或者超过 8M 的时间达到 60s，则关闭客户端连接。
client-output-buffer-limit pubsub 32mb 8mb 60

# 控制着服务器的定时器频率。hz 的全称是 "every second" ，它决定了Redis对一些操作进行定时调用的频率。默认情况下，hz 的值通常为10，这意味着Redis每秒钟会进行10次定时操作。
hz 10

# 根据当前的连接数自动调整hz值（定期任务的执行频率），可规避因每次定期任务扫描的连接数过多而造成的实例卡顿。
dynamic-hz yes

# 用于在 AOF（Append Only File）文件重写期间增量地进行 fsync 操作。当 Redis 的 AOF 持久化策略启用时，Redis 会定期将内存中的数据集写入到 AOF 文件中。如果 AOF 文件非常大，Redis 需要定期对其进行重写，以减小文件大小。aof-rewrite-incremental-fsync 配置选项决定了在这个过程中是否进行增量式的 fsync 操作。当设置为 yes 时，Redis 会在 AOF 重写的过程中进行增量式的 fsync。这意味着 Redis 会在重写的不同阶段执行 fsync，而不是等到整个重写过程完成后才执行。这样做可以降低 AOF 重写对性能的影响，但也可能增加数据丢失的风险。当设置为 no 时，Redis 将只在 AOF 重写完成后执行一次 fsync。这可以最大程度地保证数据的持久性，但是在重写过程中如果发生宕机，可能会丢失最后一次 fsync 之后写入的数据。
aof-rewrite-incremental-fsync yes

# 设置 Redis 在写入 RDB 文件时是否使用增量同步。默认情况下，Redis 在写入 RDB 文件时不使用增量同步。
rdb-save-incremental-fsync yes

# 用于启用jemalloc的后台线程。jemalloc是Redis默认的内存分配器，它提供了更好的性能和更低的内存碎片化。jemalloc-bg-thread选项允许jemalloc运行一个后台线程来优化内存分配。
jemalloc-bg-thread yes


```