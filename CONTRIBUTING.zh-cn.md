# 项目贡献指南

感谢您抽出时间做出贡献！请遵循以下指南，以确保工作流程顺畅高效。🚀🚀🚀

要开始使用项目，请参阅[README.md](https://github.com/suitenumerique/docs/blob/main/README.md)获取详细说明。

请同时查看我们的[开发手册](https://suitenumerique.gitbook.io/handbook)，了解我们的最佳实践。

## 帮助我们翻译

您可以在[Crowdin](https://crowdin.com/project/lasuite-docs)上帮助我们进行翻译。
您的语言不在其中？请在我们的Crowdin页面上申请😊。

## 创建问题

创建问题时，请提供以下详细信息：

1.  **标题**：一个简明且描述性的问题标题。
2.  **描述**：详细解释问题，包括相关上下文或截图（如适用）。
3.  **重现步骤**：如果问题是bug，请包括重现问题所需的步骤。
4.  **预期与实际行为**：描述您期望发生的情况和实际发生的情况。
5.  **标签**：添加适当的标签以对问题进行分类（例如，bug、功能请求、文档）。

## 选择问题

我们使用[GitHub Project](https://github.com/orgs/numerique-gouv/projects/13)来优先处理我们的工作量。

请优先检查处于**todo**列中且优先级较高的问题（P0 -> P2）。

## 提交消息格式

所有提交消息必须遵循以下格式：

`<gitmoji>(type) title description`

*   <**gitmoji**>：使用gitmoji表示提交的目的。例如，✨表示添加新功能，🔥表示删除内容，查看这里的列表：<https://gitmoji.dev/>。
*   **(type)**：描述更改类型。常见类型包括`backend`、`frontend`、`CI`、`docker`等...
*   **title**：简短、描述性的更改标题，以小写字符开头。
*   **description**：包含有关更改内容和原因的其他详细信息。

### 提交消息示例

```
✨(frontend) add user authentication logic 

Implemented login and signup features, and integrated OAuth2 for social login.
```

## 更新变更日志

请在变更日志中添加一行描述您的开发。变更日志条目应包含对更改的简要摘要，这有助于有效地跟踪更改并让所有人了解情况。我们通常包括拉取请求的标题，然后是拉取请求ID来完成日志条目。变更日志行总长度应少于80个字符。

### 变更日志消息示例
```
## [Unreleased]

## Added

- ✨(frontend) add AI to the project #321
```

## 拉取请求

最好添加关于拉取请求目的的信息，以帮助审阅者理解更改的上下文和意图。如果可能，添加一些图片或小视频来展示更改。

### 别忘了：
- 检查您的提交
- 检查代码风格：`make lint && make frontend-lint`
- 检查测试：`make test`
- 添加变更日志条目

一旦所有必需的测试通过，您可以请求项目维护者进行审核。

## 代码风格

请保持代码风格的一致性。运行任何可用的代码检查工具，确保代码干净并遵循项目的约定。

## 测试

确保所有新功能或修复都有相应的测试。在推送更改之前运行测试套件，以确保没有任何内容被破坏。

## 寻求帮助

如果您在贡献过程中需要任何帮助，请随时在问题跟踪器中开启讨论或寻求指导。我们非常乐意提供帮助！

感谢您的贡献！👍
