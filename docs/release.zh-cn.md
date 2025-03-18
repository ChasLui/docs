# 发布新版本

当我们准备发布新版本（例如 `4.18.1`）时，应遵循以下标准流程：

1. 创建一个名为 `release/4.18.1` 的新分支。
2. 更新后端项目、前端项目和 Helm 文件的版本号：

   - 对于后端，手动更新 `pyproject.toml` 中的版本号，
   - 对于每个项目（`src/frontend`、`src/frontend/apps/*`、`src/frontend/packages/*`、`src/mail`），在其目录中运行 `yarn version --new-version --no-git-tag-version 4.18.1`。这将自动更新它们的 `package.json`，
   - 对于 Helm，更新 `src/helm/env.d` 中 `preprod` 和 `production` 环境的 Docker 镜像标签：

     ```yaml
     image:
       repository: lasuite/impress-backend
       pullPolicy: Always
       tag: "v4.18.1" # 替换为新的版本号，别忘了加上 "v" 前缀
     
     ...
     
     frontend:
       image:
         repository: lasuite/impress-frontend
         pullPolicy: Always
         tag: "v4.18.1" 

      y-provider:
        image:
          repository: lasuite/impress-y-provider
          pullPolicy: Always
          tag: "v4.18.1" 
     ```

     新的镜像还不存在：它们将在后续流程中自动创建。

3. 按照 [keepachangelog](https://keepachangelog.com/en/0.3.0/) 的建议更新项目的 `Changelog`

4. 使用以下格式提交更改：使用 🔖 发布表情符号、发布类型（patch/minor/patch）和发布版本：

    ```text
    🔖(minor) bump release to 4.18.0
    ```

5. 创建拉取请求，等待同事批准并合并。
6. 检出并拉取 `main` 分支的更改，确保您有最新的更新。
7. 标记并推送您的提交：

    ```bash
    git tag v4.18.1 && git push origin tag v4.18.1
    ```

    这样做会触发 CI，并告诉它构建您在 Helm 文件中指定的新 Docker 镜像版本。

8. 确保新的[后端](https://hub.docker.com/r/lasuite/impress-frontend/tags)和[前端](https://hub.docker.com/r/lasuite/impress-frontend/tags)镜像标签已出现在 Docker Hub 上。
9. 发布完成！

# 部署

> [!TIP]
> `staging` 平台会随着 `main` 分支的每次更新自动部署。

创建新版本不会自动发布到生产环境。

部署由 ArgoCD 完成。ArgoCD 检查 `production` 标签并自动部署生产平台到目标提交。

要发布，我们用 `production` 标签标记我们想要的提交。ArgoCD 会收到标签更改的通知。然后它会部署目标提交的 Helm 文件中指定的 Docker 镜像标签。

要发布您刚刚创建的版本：

```bash
git tag --force production v4.18.1
git push --force origin production
```