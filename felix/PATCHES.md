# felix/opencode-v2：适配 OpenCode 2.x

本分支 = `uuie/agent-cache-optimizer` 上游 + Felix 平台的 v2 适配器。

## 为什么

OpenCode 2.0.18 起只接受 v2 插件模块（`default { id, setup }`），并且不再触发
v1 hooks（`experimental.chat.system.transform`），原插件（0.6.1，2026-06 发布）
在 OpenCode 2.x 上报 “Plugin must export a default definition with an id and an
effect or setup function.”，核心的稳定块前置完全失效。

## 改了什么

`src/index.ts` 末尾新增 v2 `default` 导出（复用原有全部重排/分类代码）：

- `ctx.session.hook("context", (input, output))`：把 `output.system`（`{type:"text",text}` 数组）
  取出为字符串，交给原 `experimental.chat.system.transform` 实现重排，再写回；
- `ctx.event.subscribe()` 事件尽力转发给原 `event` 处理器（指标/诊断）；
- 已知差异：`chat.headers`（Anthropic prompt-caching beta 头）在 v2 没有对应
  注册点，暂缺；核心缓存命中提升不受影响。

## 部署

Felix-Homelab 把本分支固定提交打进 OpenCode 模板镜像：
`/opt/agent-cache-optimizer`，入口脚本在 `opencode.json` 里以**绝对路径**引用
（OpenCode 支持 path 插件），无需 npm 安装、离线可用。

## 与上游同步

```bash
git remote add upstream https://github.com/uuie/agent-cache-optimizer.git
git fetch upstream main && git rebase upstream/main
# 冲突点只在 src/index.ts 末尾的 v2 适配器
```
