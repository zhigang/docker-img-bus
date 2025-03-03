# Docker 镜像同步工具

这是一个用于将Docker镜像同步到阿里云容器镜像服务的自动化工具。通过GitHub Actions，可以方便地将指定的Docker镜像自动同步到阿里云容器镜像仓库中。

## 功能特点

- 支持从多个来源同步Docker镜像
- 自动将镜像推送到阿里云容器镜像服务
- 支持通过配置文件管理镜像列表
- 自动处理镜像命名规范化
- 支持多种镜像仓库源（如Docker Hub、GCR、Quay.io等）
- 支持批量同步和自动化部署

## 使用方法

### 1. 配置GitHub Secrets

在使用此工具之前，需要在GitHub仓库中配置以下Secrets：

- `ALIYUN_USERNAME`: 阿里云容器镜像服务的用户名
- `ALIYUN_PASSWORD`: 阿里云容器镜像服务的密码

配置步骤：

1. 进入仓库的Settings页面
2. 选择Secrets and variables > Actions
3. 点击"New repository secret"添加上述配置项

### 2. 配置镜像列表

在`images.txt`文件中添加需要同步的Docker镜像，每行一个镜像。支持以下格式：

1. Docker Hub官方镜像：

```
nginx:1.27.4
mysql:8.0
```

2. Docker Hub组织/用户镜像：

```
langgenius/dify-api:0.15.3
minio/minio:RELEASE.2023-03-20T20-16-18Z
```

3. 其他镜像仓库：

```
k8s.gcr.io/ingress-nginx/controller:v1.12.0
docker.elastic.co/elasticsearch/elasticsearch:8.14.3
quay.io/coreos/etcd:v3.5.5
```

### 3. 工作流触发

镜像同步工作流可通过以下方式触发：

1. 自动触发：当`images.txt`文件发生变更并提交到main或master分支时
2. 手动触发：在GitHub Actions页面中手动触发workflow_dispatch事件

## 镜像命名规则

同步到阿里云后的镜像命名规则如下：

- 目标仓库：registry.cn-shanghai.aliyuncs.com/siriuszg/
- 镜像名称转换规则：
  1. 移除源地址部分（如k8s.gcr.io/、docker.elastic.co/等）
  2. 当命名空间和镜像名相同时（如elasticsearch/elasticsearch），只保留一个作为最终镜像名
  3. 其余情况下，将斜杠(/)替换为连字符(-)
- 标签：保持原始标签不变

示例：

- `nginx:1.27.4` → `registry.cn-shanghai.aliyuncs.com/siriuszg/nginx:1.27.4`
- `k8s.gcr.io/ingress-nginx/controller:v1.12.0` → `registry.cn-shanghai.aliyuncs.com/siriuszg/ingress-nginx-controller:v1.12.0`
- `docker.elastic.co/elasticsearch/elasticsearch:8.14.3` → `registry.cn-shanghai.aliyuncs.com/siriuszg/elasticsearch:8.14.3`
- `quay.io/coreos/etcd:v3.5.5` → `registry.cn-shanghai.aliyuncs.com/siriuszg/coreos-etcd:v3.5.5`

## 最佳实践

1. 镜像版本管理
   - 建议使用具体的版本标签而不是`latest`
   - 定期更新镜像版本以获取安全补丁
   - 在更新前先在测试环境验证新版本

2. 镜像同步优化
   - 按需添加镜像，避免同步不必要的镜像
   - 对于大型镜像，建议在网络良好时进行同步
   - 可以按功能或项目对镜像进行分组注释

3. 安全建议
   - 定期轮换阿里云访问凭证
   - 使用最小权限原则配置镜像仓库权限
   - 启用镜像扫描功能检测安全漏洞

## 注意事项

1. 确保已正确配置阿里云容器镜像服务的访问凭证
2. 在添加新镜像时，建议先验证镜像地址的可访问性
3. 对于大型镜像，同步过程可能需要较长时间
4. 某些镜像仓库可能需要额外的身份验证
5. 建议定期清理不再使用的镜像版本
6. 注意遵守镜像源的使用条款和限制
