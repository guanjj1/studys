# Jenkins + GitLab + Golang + Docker 部署方案

思路：**Jenkins 侧只负责编译 Go 二进制 → scp 到目标服务器 → 目标服务器用精简的 Dockerfile 打镜像并 `docker compose up`**。这样目标服务器不需要 Go 环境和源码，构建快、镜像小。

---

## 一、整体流程

```
GitLab (源码)
    │  webhook / 轮询
    ▼
Jenkins (golang 容器里编译 → fgbox 可执行文件)
    │  scp
    ▼
目标服务器 /opt/fgbox/appBackend/fgbox
    │  docker compose up -d --build appbackend
    ▼
容器运行 (alpine + fgbox)
```

---

## 二、目标服务器目录结构

把原仓库中 `docker-compose.yml` + `appBackend/` `appDB/` `appRedis/` 目录（不含源码 `src`）拷到目标服务器 `/opt/fgbox/`，最终：

```
/opt/fgbox/
├── docker-compose.yml
├── appBackend/
│   ├── fgbox                 ← Jenkins 推送过来（二进制）
│   ├── Dockerfile            ← 改成下面精简版
│   ├── files/
│   ├── log/
│   ├── .go_env
│   └── config_production.yaml
├── appDB/
└── appRedis/
```

---

## 三、目标服务器上的 `appBackend/Dockerfile`（精简运行时）

不再需要 golang builder，因为二进制由 Jenkins 编译：

```dockerfile
FROM 192.168.3.88/library/alpine:3.20

WORKDIR /opt/

# 时区 & 证书
RUN apk add --no-cache tzdata ca-certificates && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

# 复制 Jenkins 编译好的二进制
COPY ./fgbox ./fgbox
RUN chmod +x ./fgbox

EXPOSE 9090

CMD ["./fgbox"]
```

`docker-compose.yml` **不用改**，`build.context: ./appBackend` 已经能自动找到这个 Dockerfile。

> 建议把 `docker-compose.yml` 里的 `appdb`/`appredis` 的 `healthcheck`、`depends_on` 都保留，因为后端现在也依赖它们健康。

---

## 四、Jenkins 准备工作

1. **插件**：`Git`, `Pipeline`, `SSH Agent`, `Credentials Binding`, `Docker Pipeline`（可选）。
2. **凭据**（Manage Credentials）：
   - `gitlab-ssh-cred`：SSH 私钥，能拉 GitLab 仓库。
   - `target-server-ssh`：SSH 私钥，能登录目标服务器（免密）。
3. **Jenkins 所在节点**能拉取 `192.168.3.88/library/golang:1.22.5-alpine3.20` 镜像（或 Jenkins 本机装有 Go）。
4. **目标服务器**：Jenkins 公钥写入 `~/.ssh/authorized_keys`；已安装 docker + compose 插件。

---

## 五、Jenkinsfile

放仓库根目录：

# Jenkinsfile（带详细注释）

```groovy
// ============================================================
// Jenkinsfile - 声明式流水线
// 功能：拉取 GitLab 源码 → 用 golang 容器编译 fgbox 二进制
//       → scp 到目标服务器 → 目标服务器 docker compose 重建 appbackend
// 存放位置：仓库根目录，文件名必须是 Jenkinsfile（无扩展名）
// ============================================================

pipeline {
    // ---------- 执行节点 ----------
    // agent any：不限制节点，Jenkins 任意可用节点都能跑这个流水线
    // 如果想指定某个节点（比如装了 docker 的那台），改成：
    // agent { label 'docker-node' }
    agent any

    // ---------- 全局环境变量 ----------
    // environment 里定义的变量可以在所有 stage 的 sh/bat 中通过
    // ${VAR} 或 ${env.VAR} 引用（Groovy 字符串需用双引号）
    environment {
        // 代码仓库的 SSH 地址（改成你自己的 GitLab 仓库地址）
        GIT_URL         = 'git@your-gitlab.com:group/fgbox.git'

        // 要构建的分支
        BRANCH          = 'main'

        // 目标服务器（部署机器）的 IP 或域名
        TARGET_HOST     = '192.168.3.100'

        // 目标服务器 SSH 登录用户名
        TARGET_USER     = 'root'

        // 目标服务器上 docker-compose.yml 所在目录
        REMOTE_BASE     = '/opt/fgbox'

        // 目标服务器上存放二进制的位置（对应 compose 里 appBackend 目录）
        REMOTE_BACKEND  = '/opt/fgbox/appBackend'

        // 编译用的 golang 镜像
        // 私有仓库写法：'192.168.3.88/library/golang:1.22.5-alpine3.20'
        // 官方写法：    'golang:1.22.5-alpine3.20'
        GO_IMAGE        = '192.168.3.88/library/golang:1.22.5-alpine3.20'
    }

    // ---------- 流水线全局选项 ----------
    options {
        // 在每行日志前加时间戳，方便排查
        timestamps()

        // 禁止同一流水线并发执行，避免两次部署互相覆盖文件
        disableConcurrentBuilds()

        // 关闭 Jenkins 默认的自动 checkout
        // 因为我们在 Checkout stage 里手动 checkout，需要控制分支和凭据
        skipDefaultCheckout(true)

        // 保留最近 10 次构建记录和 5 次构建产物（可选）
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '5'))

        // 全局超时：超过 30 分钟没结束就自动失败（可选）
        timeout(time: 30, unit: 'MINUTES')
    }

    // ---------- 流水线阶段 ----------
    stages {

        // ====================================================
        // Stage 1：从 GitLab 拉取源码
        // ====================================================
        stage('Checkout') {
            steps {
                // 使用 GitSCM 插件手动 checkout
                checkout([$class: 'GitSCM',
                    // 要拉的分支，格式 */main
                    branches: [[name: "*/${env.BRANCH}"]],

                    // 仓库地址和凭据
                    userRemoteConfigs: [[
                        url: "${env.GIT_URL}",
                        // 这里填 Jenkins 里配置好的 SSH 私钥凭据 ID
                        credentialsId: 'gitlab-ssh-cred'
                    ]],

                    // 可选：扩展行为，比如清理工作区、浅克隆
                    extensions: [
                        // 每次构建前清空工作区，避免残留文件污染
                        cleanBeforeCheckout()
                    ]
                ])
            }
        }

        // ====================================================
        // Stage 2：在 golang 容器里编译出 fgbox 二进制
        // ====================================================
        stage('Build Go Binary') {
            // 该 stage 单独指定 agent：临时起一个 golang 容器执行
            agent {
                docker {
                    // 使用 environment 里定义的 GO_IMAGE
                    image "${env.GO_IMAGE}"

                    // reuseNode true：
                    //   让容器挂载当前 Jenkins 工作区（$WORKSPACE）
                    //   这样容器里 go build 生成的 fgbox 会落到宿主机工作区
                    //   后面 scp 才能拿到
                    reuseNode true

                    // 以 root 用户运行容器，避免挂载目录权限问题
                    args '-u root'

                    // 如果 Jenkins 节点上已经 docker load 过镜像，
                    // 不想每次都去 registry 拉，可加下面这行：
                    // alwaysPull false
                }
            }

            steps {
                // 进入源码目录，假设 go.mod 在 appBackend/src 下
                dir('appBackend/src') {
                    // sh 使用三引号，内部变量由 shell 处理
                    sh '''
                        set -e    # 任一命令失败立即退出，避免带病继续

                        # Go 模块代理，国内必须设，否则 go mod tidy 会超时
                        export GOPROXY=https://goproxy.cn,direct

                        # 关闭 CGO，生成的二进制是静态的，
                        # 才能在 alpine（musl libc）里跑
                        export CGO_ENABLED=0

                        # 交叉编译目标：Linux amd64
                        export GOOS=linux
                        export GOARCH=amd64

                        # 整理依赖
                        go mod tidy

                        # 编译：-s -w 去掉符号表和调试信息，减小体积
                        go build -ldflags "-s -w" -o fgbox

                        # 打印文件大小，方便确认产物
                        ls -lh fgbox
                    '''
                }

                // 把二进制归档到 Jenkins，构建记录里可下载排查
                archiveArtifacts artifacts: 'appBackend/src/fgbox', fingerprint: true
            }
        }

        // ====================================================
        // Stage 3：把 fgbox 二进制推送到目标服务器
        // ====================================================
        stage('Push Binary to Target') {
            steps {
                // sshagent：把 Jenkins 里配置的 SSH 私钥临时加载到 ssh-agent
                //           这样 scp/ssh 不需要手动指定私钥文件
                sshagent(credentials: ['target-server-ssh']) {
                    sh '''
                        set -e

                        # 在目标服务器上创建目录（如果不存在）
                        # -o StrictHostKeyChecking=no：跳过首次连接的主机指纹确认
                        ssh -o StrictHostKeyChecking=no \
                            ${TARGET_USER}@${TARGET_HOST} \
                            "mkdir -p ${REMOTE_BACKEND}"

                        # 把编译产物 scp 过去，覆盖旧文件
                        scp -o StrictHostKeyChecking=no \
                            appBackend/src/fgbox \
                            ${TARGET_USER}@${TARGET_HOST}:${REMOTE_BACKEND}/fgbox
                    '''
                }
            }
        }

        // ====================================================
        // Stage 4：目标服务器上重建并重启 appbackend 容器
        // ====================================================
        stage('Deploy via Docker Compose') {
            steps {
                sshagent(credentials: ['target-server-ssh']) {
                    sh '''
                        set -e
                        set -e 是 shell（bash/sh）里的一个开关，全称是 "exit on error"（遇到错误就退出）。

                        # 登录目标服务器执行部署
                        ssh -o StrictHostKeyChecking=no \
                            ${TARGET_USER}@${TARGET_HOST} "
                                cd ${REMOTE_BASE} &&

                                # --build           重新构建镜像（Dockerfile 会 COPY 新的 fgbox）
                                # --force-recreate  强制重建容器，确保用上新镜像
                                # -d                后台运行
                                # appbackend        只处理这一个服务，
                                #                   避免误动 mysql/redis
                                docker compose up -d --build --force-recreate appbackend &&

                                # 打印容器状态，确认启动成功
                                docker compose ps
                            "
                    '''
                }
            }
        }
    }

    // ---------- 构建结束后的通知 ----------
    post {
        // 成功时打印提示
        success {
            echo "✅ 部署成功：${env.TARGET_HOST}"
        }

        // 失败时打印提示（可扩展成发钉钉/企业微信/webhook 通知）
        failure {
            echo "❌ 部署失败，请检查 Jenkins 日志或目标服务器状态"
        }

        // 无论成功失败都执行（可扩展清理临时文件等）
        always {
            // echo "构建结束时间：${new Date()}"
        }
    }
}
```

---

## 一、关键点速查

| 位置 | 作用 |
|---|---|
| `environment` | 集中管理地址、分支、镜像等变量，改一处生效全局 |
| `options.skipDefaultCheckout(true)` | 关闭默认 checkout，改为手动控制分支/凭据 |
| `agent { docker { image ... } }` | 该 stage 在临时容器里跑，无需在节点装 Go |
| `reuseNode true` | 容器挂载工作区，编译产物写回宿主机，供后续 scp |
| `sshagent(credentials: [...])` | 免密登录/推送文件到目标服务器 |
| `set -e` | shell 脚本严格模式，命令失败即中止 |
| `CGO_ENABLED=0` | 编译静态二进制，才能在 alpine 里跑 |
| `docker compose up -d --build appbackend` | 只重建后端容器，不动 db/redis |

---

## 二、需要你替换的地方

1. `GIT_URL` → 改成你的 GitLab 仓库 SSH 地址
2. `TARGET_HOST` / `TARGET_USER` → 改成实际服务器信息
3. `credentialsId: 'gitlab-ssh-cred'` → 改成 Jenkins 里 GitLab 凭据的实际 ID
4. `credentialsId: 'target-server-ssh'` → 改成 Jenkins 里目标服务器凭据的实际 ID
5. `GO_IMAGE` → 私有仓库或官方镜像（`golang:1.22.5-alpine3.20`）
6. `appBackend/src` → 如果 go.mod 不在这个路径，按实际调整

### 关于 `reuseNode true`
用 docker agent 时，容器里执行 `go build` 会挂载 Jenkins 工作区，产物直接写到宿主机 workspace 下的 `appBackend/src/fgbox`，所以后面 `scp` 能拿到。若你的 Jenkins 节点没有 docker，可去掉 agent docker，改成直接 `sh 'cd appBackend/src && go build ...'`（需节点装 Go）。

---

## 六、GitLab 侧触发

在 GitLab 项目 → Settings → Webhooks：
```
http://<jenkins>/project/fgbox-deploy
```
勾选 Push events，或者 Jenkins 侧用 `pollSCM('H/5 * * * *')` 也可以。

---

## 七、可选方案：镜像仓库方式（更规范）

如果以后不想在目标服务器重建镜像，可以改成：

1. Jenkins 里 `docker build -t 192.168.3.88/library/fgbox-backend:${BUILD_NUMBER} appBackend`
2. `docker push 192.168.3.88/library/fgbox-backend:${BUILD_NUMBER}`
3. 目标服务器 `docker compose` 里把 `appbackend` 的 `build:` 改成 `image: 192.168.3.88/library/fgbox-backend:${TAG}`，再 `docker compose pull && up -d`。

这样目标服务器无需 docker build，回滚也只要切换 tag，唯一代价是 Jenkins 要有 docker build/push 权限。

---

## 八、几个坑提醒

| 项 | 说明 |
|---|---|
| `healthcheck.sh` 权限 | 确保 `appDB/healthcheck.sh` 有可执行权限，否则容器会一直 unhealthy 导致 backend 起不来 |
| `docker compose` vs `docker-compose` | 服务器较新装的是 v2 插件，用 `docker compose`；旧版用 `docker-compose`，Jenkinsfile 里改一下 |
| `.go_env` / `.mysql_env` | 建议用 Jenkins 凭据注入，不要提交到 git |
| `CGO_ENABLED=0` | 一定要关，否则 alpine 里跑不起来（除非装了 glibc） |
| 首次部署 | 先在服务器上手动 `docker compose up -d` 一次，把 mysql 数据目录、redis 配置初始化好，再交给 Jenkins 自动化 |
| 回滚 | 每次 deploy 前可把 `fgbox` 备份为 `fgbox.bak`，出问题直接 `cp` 回去重建 |

需要我把 `docker-compose.yml` 里 `appbackend` 段按上面结构改好的完整版本贴出来吗？