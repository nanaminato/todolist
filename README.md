# todolist

用于记录暂未决定实施、但值得后续继续调查或实现的项目想法、架构方案与功能计划。

## 使用方式

主要通过 GitHub Issues 记录待考虑事项。每个 Issue 尽量包含背景、目标、初步方案、需要进一步调查的问题以及当前状态。

这里的内容默认表示 **候选方案 / TODO / Idea**，不代表已经进入正式开发计划。

## 当前方向

- RelaxKonOS 及其生态扩展
- Remote Workspace / Remote Development
- IDE 与编辑器集成
- Agent / AI 开发工具集成
- 其他值得后续尝试的项目与技术方案

## Issues

具体事项请查看仓库的 Issues。



prompt:
relaxkonos
先把已有账户用于 clone/push 的第一次请求 → 给 fetch/pull 补 credential contract → 服务端统一走现有 askpass → 最后做账户管理/持久化读取 UI。这样不会为了 Gitee 再造一套 Git 引擎，自建 GitLab/Gitea/Forgejo 也自然能吃到同一套 host-based credential 机制。
