### 首次安装常用

```shell
# 首先升级一下所有的包
conda upgrade --all


# 添加Anaconda的TUNA镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes

conda create --name myenv python=3.8

# myenv进入环境
activate 

# 查看所有的环境
conda env list

# 删除环境
conda remove -n myenv --all
```