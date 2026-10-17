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

```groovy
pipeline {
    agent any

    environment {
        GIT_URL         = 'git@your-gitlab.com:group/fgbox.git'   // 改成你的仓库
        BRANCH          = 'main'
        TARGET_HOST     = '192.168.3.100'                         // 目标服务器
        TARGET_USER     = 'root'
        REMOTE_BASE     = '/opt/fgbox'
        REMOTE_BACKEND  = '/opt/fgbox/appBackend'
        GO_IMAGE        = '192.168.3.88/library/golang:1.22.5-alpine3.20'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: "*/${env.BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "${env.GIT_URL}",
                        credentialsId: 'gitlab-ssh-cred'
                    ]]
                ])
            }
        }

        stage('Build Go Binary') {
            agent {
                docker {
                    image "${env.GO_IMAGE}"
                    reuseNode true                 // 复用工作区，产物会落回 Jenkins workspace
                    args  '-u root'
                }
            }
            steps {
                dir('appBackend/src') {
                    sh '''
                        set -e
                        export GOPROXY=https://goproxy.cn,direct
                        export CGO_ENABLED=0
                        export GOOS=linux
                        export GOARCH=amd64
                        go mod tidy
                        go build -ldflags "-s -w" -o fgbox
                        ls -lh fgbox
                    '''
                }
                // 归档一份方便排查
                archiveArtifacts artifacts: 'appBackend/src/fgbox', fingerprint: true
            }
        }

        stage('Push Binary to Target') {
            steps {
                sshagent(credentials: ['target-server-ssh']) {
                    sh '''
                        set -e
                        ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} \
                            "mkdir -p ${REMOTE_BACKEND}"
                        scp -o StrictHostKeyChecking=no \
                            appBackend/src/fgbox \
                            ${TARGET_USER}@${TARGET_HOST}:${REMOTE_BACKEND}/fgbox
                    '''
                }
            }
        }

        stage('Deploy via Docker Compose') {
            steps {
                sshagent(credentials: ['target-server-ssh']) {
                    sh '''
                        set -e
                        ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} "
                            cd ${REMOTE_BASE} &&
                            docker compose up -d --build --force-recreate appbackend &&
                            docker compose ps
                        "
                    '''
                }
            }
        }
    }

    post {
        success { echo "✅ 部署成功：${env.TARGET_HOST}" }
        failure { echo "❌ 部署失败，请检查日志" }
    }
}
```

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