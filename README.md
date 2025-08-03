新建一个 Workers，选 Hello World 项目，创建完成后点右上角的按钮进入编辑代码，填入以下代


例如您的Workers项目域名为：`docker.fxxk.dedyn.io`；

### 1.官方镜像路径前面加域名

```shell
docker pull docker.fxxk.dedyn.io/stilleshan/frpc:latest
```

```shell
docker pull docker.fxxk.dedyn.io/library/nginx:stable-alpine3.19-perl
```

### 2.一键设置镜像加速

修改文件 `/etc/docker/daemon.json`（如果不存在则创建）

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://docker.fxxk.dedyn.io"]  # 请替换为您自己的Worker自定义域名
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 3. 配置常见仓库的镜像加速

#### 3.1 配置

`Containerd` 较简单，它支持任意 `registry` 的 `mirror`，只需要修改配置文件 `/etc/containerd/config.toml`，添加如下的配置：

```yaml
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://xxxx.xx.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
          endpoint = ["https://xxxx.xx.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
          endpoint = ["https://xxxx.xx.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]
          endpoint = ["https://xxxx.xx.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."ghcr.io"]
          endpoint = ["https://xxxx.xx.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]
          endpoint = ["https://xxxx.xx.com"]
```

`Podman` 同样支持任意 `registry` 的 `mirror`，修改配置文件 `/etc/containers/registries.conf`，添加配置：

```yaml
unqualified-search-registries = ['docker.io', 'k8s.gcr.io', 'gcr.io', 'ghcr.io', 'quay.io']

[[registry]]
prefix = "docker.io"
insecure = true
location = "registry-1.docker.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "registry.k8s.io"
insecure = true
location = "registry.k8s.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "k8s.gcr.io"
insecure = true
location = "k8s.gcr.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "gcr.io"
insecure = true
location = "gcr.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "ghcr.io"
insecure = true
location = "ghcr.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "quay.io"
insecure = true
location = "quay.io"

[[registry.mirror]]
location = "xxxx.xx.com"

```

#### 3.3 使用

对于以上配置，k8s 在使用的时候，就可以直接 `pull` 外部无法 pull 的镜像了。

```shell
# 手动可以直接pull配置了mirror的仓库
crictl pull registry.k8s.io/kube-proxy:v1.28.4
docker  pull nginx:1.21
```
