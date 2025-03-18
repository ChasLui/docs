<p align="center">
  <a href="https://github.com/suitenumerique/docs">
    <img alt="Docs" src="/docs/assets/docs-logo.png" width="300" />
  </a>
</p>

<p align="center">
欢迎使用Docs！这是一款开源文档编辑器，通过实时协作让您的笔记转化为知识
</p>

<p align="center">
  <a href="https://matrix.to/#/#docs-official:matrix.org">
    在Matrix上聊天
  </a> - <a href="/docs/">
    文档
  </a> - <a href="#getting-started-">
    开始使用
  </a> - <a href="mailto:docs@numerique.gouv.fr">
    联系我们
  </a>
</p>

<img src="/docs/assets/docs_live_collaboration_light.gif" width="100%" align="center"/>

## 为什么使用Docs ❓

Docs是一款协作文本编辑器，旨在解决知识构建和共享中的常见挑战。

### 写作
*   😌 简单的协作编辑，没有markdown的格式复杂性
*   🔌 离线？没问题，继续写作，您的编辑将在重新联网时同步
*   💅 使用有限但精美的格式选项创建简洁的文档，专注于内容
*   🧱 为生产力而设计（支持markdown、多种块类型、斜杠命令、键盘快捷键）
*   ✨ 借助我们的AI功能（生成、总结、校正、翻译）节省时间

### 协作
*   🤝 与您的团队实时协作
*   🔒 精细的访问控制，确保您的信息安全并只与合适的人共享
*   📑 多种格式（.odt、.doc、.pdf）的专业文档导出，支持自定义模板
*   📚 内置wiki功能，将团队协作成果转变为有组织的知识 `计划于2025年2月发布`

### 自托管
*   🚀 易于安装，可扩展且安全，是Notion、Outline或Confluence的替代选择

## 开始使用 🔧

### 测试

通过登录此[环境](https://impress-preprod.beta.numerique.gouv.fr/)在浏览器中测试Docs

```
邮箱: test.docs@yopmail.com
密码: I'd<3ToTestDocs
```

### 本地运行

> ⚠️ 使用以下方法在本地运行Docs仅用于测试目的。它基于使用Minio作为S3存储解决方案构建Docs，但您可以选择任何兼容S3的对象存储。

**前提条件**

确保您的笔记本电脑上安装了较新版本的Docker和[Docker Compose](https://docs.docker.com/compose/install)：

```shellscript
$ docker -v

Docker version 20.10.2, build 2291f61

$ docker compose version

Docker Compose version v2.32.4
```

> ⚠️ 您可能需要使用sudo运行以下命令，但可以通过将用户添加到`docker`组来避免此问题。

**项目引导**

开始处理项目的最简单方法是使用GNU Make：

```shellscript
$ make bootstrap FLUSH_ARGS='--no-input'
```

此命令构建`app`容器，安装依赖项，执行数据库迁移并编译翻译。每次从项目仓库拉取代码时使用此命令是个好主意，可以避免依赖性相关或迁移相关的问题。

您的Docker服务现在应该启动并运行 🎉

您可以通过访问<http://localhost:3000>来访问项目。

系统将提示您登录，默认凭据为：

```
用户名: impress
密码: impress
```

📝 请注意，如果您之后需要运行它们，可以使用同名的Make规则：

```shellscript
$ make run
```

⚠️ 对于前端开发人员，通常最好在本地以开发模式运行前端。

要做到这一点，请使用以下命令安装前端依赖项：

```shellscript
$ make frontend-development-install
```

并使用以下命令在开发模式下本地运行前端：

```shellscript
$ make run-frontend-development
```

要启动除前端容器外的所有服务，可以使用以下命令：

```shellscript
$ make run-backend
```

**添加内容**
您可以通过运行以下命令创建基本演示站点：

```shellscript
$ make demo
```

最后，您可以使用以下命令查看所有可用的Make规则：

```shellscript
$ make help
```

**Django管理**

您可以在以下地址访问Django管理站点：

<http://localhost:8071/admin>。

您首先需要创建超级用户账户：

```shellscript
$ make superuser
```

## 反馈 🙋‍♂️🙋‍♀️

我们很乐意听取您的想法并了解您的实验，所以欢迎在[Matrix](https://matrix.to/#/#docs-official:matrix.org)上打个招呼。

## 路线图

想知道项目的发展方向？[🗺️ 查看我们的路线图](https://github.com/orgs/numerique-gouv/projects/13/views/11)

## 许可证 📝

本作品根据MIT许可证发布（参见[LICENSE](https://github.com/suitenumerique/docs/blob/main/LICENSE)）。

虽然Docs是一个公共驱动的计划，但我们的许可证选择是邀请私营部门参与者使用、销售和贡献该项目。

## 贡献 🙌

本项目旨在由社区驱动，因此如果您对我们的实现或设计决策有任何疑问，请不要犹豫，[联系我们](https://matrix.to/#/#docs-official:matrix.org)。

您可以在[Crowdin](https://crowdin.com/project/lasuite-docs)上帮助我们进行翻译。

如果您打算提交拉取请求，请参阅[CONTRIBUTING](https://github.com/suitenumerique/docs/blob/main/CONTRIBUTING.md)获取指南。

目录结构：

```markdown
docs
├── bin - 用于各种任务的可执行脚本或二进制文件，如设置脚本、实用程序脚本或自定义命令。
├── crowdin - 用于Crowdin翻译，一种帮助管理项目翻译的工具或服务。
├── docker - 用于构建项目Docker镜像的Dockerfile和相关配置文件。这些镜像可用于开发、测试或生产环境。
├── docs - 项目文档，包括用户指南、API文档和其他有用资源。
├── env.d/development - 开发环境的特定配置文件。这些文件可能包括环境变量、配置设置或开发所需的其他设置文件。
├── gitlint - `gitlint`的配置文件，这是一个强制执行提交消息指南的工具，确保提交消息的一致性和质量。
├── playground - 实验性或临时代码，开发人员可以在不影响主代码库的情况下测试新功能或想法。
└── src - 主要源代码目录，包含项目的核心应用程序代码、库和模块。
```

## 致谢 ❤️

### 技术栈

Docs基于[Django Rest Framework](https://www.django-rest-framework.org/)、[Next.js](https://nextjs.org/)、[BlockNote.js](https://www.blocknotejs.org/)、[HocusPocus](https://tiptap.dev/docs/hocuspocus/introduction)和[Yjs](https://yjs.dev/)构建。

### 政府 ❤️ 开源

Docs是由法国🇫🇷🥖（[DINUM](https://www.numerique.gouv.fr/dinum/)）和德国🇩🇪🥨政府（[ZenDiS](https://zendis.de/)）共同努力的结果。

我们很自豪地赞助[BlockNotejs](https://www.blocknotejs.org/)和[Yjs](https://yjs.dev/)。

我们一直在寻找新的公共合作伙伴（我们目前正在引入荷兰🇳🇱🧀），如果您有兴趣使用或贡献Docs，请随时[联系我们](mailto:docs@numerique.gouv.fr)。

<p align="center">
  <img src="/docs/assets/europe_opensource.png" width="50%"/>
</p>
