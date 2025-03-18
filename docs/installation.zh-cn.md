# 在 k8s 集群上安装 Docs

本文档是一个分步指南，描述了如何在 k8s 集群上安装不带 AI 功能的 Docs。这是一个教学文档，用于学习其工作原理。在实际生产环境中需要进行相应调整。

## 前置条件

- 带有 nginx-ingress 控制器的 k8s 集群
- OIDC 提供商（如果没有，我们将提供一个示例）
- PostgreSQL 服务器（如果没有，我们将提供一个示例）
- Memcached 服务器（如果没有，我们将提供一个示例）
- S3 存储桶（如果没有，我们将提供一个示例）

### 测试集群

如果您没有测试集群，可以在本地 kind 集群上安装所有内容。在这种情况下，最简单的方法是使用我们的脚本 **bin/start-kind.sh**。

要使用该脚本，您需要安装：

- Docker (https://docs.docker.com/desktop/)
- Kind (https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- Mkcert (https://github.com/FiloSottile/mkcert#installation)
- Helm (https://helm.sh/docs/intro/quickstart/#install-helm)

```
./bin/start-kind.sh 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4700  100  4700    0     0  92867      0 --:--:-- --:--:-- --:--:-- 94000
0. 创建 ca
本地 CA 已安装在系统信任存储中！👍
本地 CA 已安装在 Firefox 和/或 Chrome/Chromium 信任存储中！👍


已创建新的证书，对以下名称有效 📜
 - "127.0.0.1.nip.io"
 - "*.127.0.0.1.nip.io"

提醒：X.509 通配符只能匹配一层深度，因此不会匹配 a.b.127.0.0.1.nip.io ℹ️

证书位于 "./127.0.0.1.nip.io+1.pem"，密钥位于 "./127.0.0.1.nip.io+1-key.pem" ✅

证书将于 2027 年 3 月 24 日过期 🗓

1. 创建 registry 容器（除非已存在）
2. 创建启用 containerd registry 配置目录的 kind 集群
正在创建集群 "suite" ...
 ✓ 确保节点镜像 (kindest/node:v1.27.3) 🖼
 ✓ 准备节点 📦  
 ✓ 写入配置 📜 
 ✓ 启动控制平面 🕹️ 
 ✓ 安装 CNI 🔌 
 ✓ 安装 StorageClass 💾 
将 kubectl 上下文设置为 "kind-suite"
您现在可以使用以下命令使用集群：

kubectl cluster-info --context kind-suite

感谢使用 kind！😊
3. 将 registry 配置添加到节点
4. 如果尚未连接，将 registry 连接到集群网络
5. 记录本地 registry
configmap/local-registry-hosting created
Warning: resource configmaps/coredns is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
configmap/coredns configured
deployment.apps/coredns restarted
6. 安装 ingress-nginx
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
secret/mkcert created
deployment.apps/ingress-nginx-controller patched
7. 设置命名空间
namespace/impress created
Context "kind-suite" modified.
secret/mkcert created
$ kubectl -n ingress-nginx get po
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-t55ph        0/1     Completed   0          2m56s
ingress-nginx-admission-patch-94dvt         0/1     Completed   1          2m56s
ingress-nginx-controller-57c548c4cd-2rx47   1/1     Running     0          2m56s
```
当您的 k8s 集群准备就绪（ingress nginx 控制器已启动）时，您可以开始部署。这个集群很特殊，因为它使用 *.127.0.0.1.nip.io 域名和 mkcert 证书来提供完整的 HTTPS 支持和简单的域名管理。

请记住，*.127.0.0.1.nip.io 将始终解析到 127.0.0.1，但在 k8s 集群中，我们配置 CoreDNS 使用 ingress-nginx 服务 IP 来响应。

## 准备工作

### 您将使用什么来认证用户？

Docs 使用 OIDC，所以如果您已经有 OIDC 提供商，请获取使用它所需的信息。在下一步中，我们将了解如何配置 Django（以及 Docs）来使用它。如果您没有提供商，我们将向您展示如何部署本地 Keycloak 实例（这不是生产环境部署，仅用于演示）。

```
$ kubectl create namespace impress
$ kubectl config set-context --current --namespace=impress
$ helm install keycloak oci://registry-1.docker.io/bitnamicharts/keycloak -f examples/keycloak.values.yaml
$ #等待直到
$ kubectl get po
NAME                    READY   STATUS    RESTARTS   AGE
keycloak-0              1/1     Running   0          6m48s
keycloak-postgresql-0   1/1     Running   0          6m48s
```

从这里开始，您需要的重要信息是：

```
OIDC_OP_JWKS_ENDPOINT: https://keycloak.127.0.0.1.nip.io/realms/impress/protocol/openid-connect/certs
OIDC_OP_AUTHORIZATION_ENDPOINT: https://keycloak.127.0.0.1.nip.io/realms/impress/protocol/openid-connect/auth
OIDC_OP_TOKEN_ENDPOINT: https://keycloak.127.0.0.1.nip.io/realms/impress/protocol/openid-connect/token
OIDC_OP_USER_ENDPOINT: https://keycloak.127.0.0.1.nip.io/realms/impress/protocol/openid-connect/userinfo
OIDC_OP_LOGOUT_ENDPOINT: https://keycloak.127.0.0.1.nip.io/realms/impress/protocol/openid-connect/session/end
OIDC_RP_CLIENT_ID: impress
OIDC_RP_CLIENT_SECRET: ThisIsAnExampleKeyForDevPurposeOnly
OIDC_RP_SIGN_ALGO: RS256
OIDC_RP_SCOPES: "openid email"
```

您可以在 **examples/keycloak.values.yaml** 中找到这些值

### 获取 redis 服务器连接值

Impress 需要一个 redis，所以我们将首先部署一个 redis：

```
$ helm install redis oci://registry-1.docker.io/bitnamicharts/redis -f examples/redis.values.yaml
$ kubectl get po
NAME                    READY   STATUS    RESTARTS   AGE
keycloak-0              1/1     Running   0          26m
keycloak-postgresql-0   1/1     Running   0          26m
redis-master-0          1/1     Running   0          35s
```

### 获取 postgresql 连接值

Impress 使用 postgresql 数据库作为后端，所以如果您有提供商，请获取使用它所需的信息。如果没有，您可以按以下方式安装 postgresql 测试环境：

```
$ helm install postgresql oci://registry-1.docker.io/bitnamicharts/postgresql -f examples/postgresql.values.yaml
$ kubectl get po
NAME                    READY   STATUS    RESTARTS   AGE
keycloak-0              1/1     Running   0          28m
keycloak-postgresql-0   1/1     Running   0          28m
postgresql-0            1/1     Running   0          14m
redis-master-0          1/1     Running   0          42s
```

从这里开始，您需要的重要信息是：

```
DB_HOST: postgres-postgresql
DB_NAME: impress
DB_USER: dinum
DB_PASSWORD: pass
DB_PORT: 5432
POSTGRES_DB: impress
POSTGRES_USER: dinum
POSTGRES_PASSWORD: pass
```

### 获取 s3 存储桶连接值

Impress 使用 s3 存储桶来存储文档，所以如果您有提供商，请获取使用它所需的信息。如果没有，您可以按以下方式安装本地 minio 测试环境：

```
$ helm install minio oci://registry-1.docker.io/bitnamicharts/minio -f examples/minio.values.yaml
$ kubectl get po
NAME                       READY   STATUS      RESTARTS   AGE
keycloak-0                 1/1     Running     0          38m
keycloak-postgresql-0      1/1     Running     0          38m
minio-84f5c66895-bbhsk     1/1     Running     0          42s
minio-provisioning-2b5sq   0/1     Completed   0          42s
postgresql-0               1/1     Running     0          24m
redis-master-0             1/1     Running     0          10m
```

## 部署

现在您已经准备好部署不带 AI 功能的 Impress。AI 功能需要更多依赖（openai API）。要部署 impress，您需要向 helm chart 提供所有上述信息。

```
$ helm repo add impress https://suitenumerique.github.io/docs/
$ helm repo update
$ helm install impress impress/docs -f examples/impress.values.yaml
$ kubectl get po
NAME                                         READY   STATUS      RESTARTS   AGE
impress-docs-backend-96558758d-xtkbp         0/1     Running     0          79s
impress-docs-backend-createsuperuser-r7ltc   0/1     Completed   0          79s
impress-docs-backend-migrate-c949s           0/1     Completed   0          79s
impress-docs-frontend-6749f644f7-p5s42       1/1     Running     0          79s
impress-docs-y-provider-6947fd8f54-78f2l     1/1     Running     0          79s
keycloak-0                                   1/1     Running     0          48m
keycloak-postgresql-0                        1/1     Running     0          48m
minio-84f5c66895-bbhsk                       1/1     Running     0          10m
minio-provisioning-2b5sq                     0/1     Completed   0          10m
postgresql-0                                 1/1     Running     0          34m
redis-master-0                               1/1     Running     0          20m
```

## 测试您的部署

为了测试您的部署，您需要登录到您的实例。如果您完全使用我们的示例，可以执行以下操作：

```
$ kubectl get ingress
NAME                             CLASS    HOSTS                       ADDRESS     PORTS     AGE
impress-docs                     <none>   impress.127.0.0.1.nip.io    localhost   80, 443   114s
impress-docs-admin               <none>   impress.127.0.0.1.nip.io    localhost   80, 443   114s
impress-docs-collaboration-api   <none>   impress.127.0.0.1.nip.io    localhost   80, 443   114s
impress-docs-media               <none>   impress.127.0.0.1.nip.io    localhost   80, 443   114s
impress-docs-ws                  <none>   impress.127.0.0.1.nip.io    localhost   80, 443   114s
keycloak                         <none>   keycloak.127.0.0.1.nip.io   localhost   80        49m
```

您可以在 https://impress.127.0.0.1.nip.io 上使用 impress。Keycloak 中的预置用户是 impress/impress。

