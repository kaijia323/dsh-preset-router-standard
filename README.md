# dsh-preset-router-standard

DeepSeek Harness（DSH）Agent Preset：任务感知思维模式路由（router-standard）。

本 preset 的 dsh preset 模式参考了 [dsh-routing-suite](https://github.com/yjh051108/dsh-routing-suite)（MIT）的设计，并移植了其中的 `router-standard` 路由思路：根据首条真实用户消息在 `spec`（计划优先）/ `weak`（模型内路由）/ `react`（执行者）行为带之间路由，并在首轮注入匹配的 persona 与核心工具面。

## 两种 preset 模式

本仓库与 [dsh-preset-router-ptc](https://github.com/kaijia323/dsh-preset-router-ptc) 是同一套任务感知路由思路的两种 DSH preset 模式：

| 模式 | 仓库 | 首个工具调用后的工具面 |
| --- | --- | --- |
| **Router Standard（本 preset）** | [dsh-preset-router-standard](https://github.com/kaijia323/dsh-preset-router-standard) | 开放完整 Standard 原生工具集 |
| **PTC RunCode（PTC 分支）** | [dsh-preset-router-ptc](https://github.com/kaijia323/dsh-preset-router-ptc) | 切换到 DSH 内置 PTC/run_code（Code Mode）单一入口 |

- **Router Standard**：任务感知路由 + RL 接口还原；首个持久工具调用后开放完整 Standard 工具集。
- **PTC RunCode**：保留 Router Standard 的首轮路由与 RL 接口，首个持久工具调用后改为 `agent.ctx.tools.presentAs('code')` 切换为 PTC/run_code（Code Mode）单一入口，system 保持短 prompt（persona + 协作口吻）。

## 特性

- **任务感知路由**：首条真实用户消息按关键词证据分类为 `spec` / `react`；模糊或未匹配文本进入 `weak`，由模型按任务自行决定。
- **RL 接口还原（standard 模式）**：首轮 system 仅保留 RL 训练句 + `shell` / `str_replace_editor`，模型以 think-act 反馈循环工作；首个持久工具调用后开放完整 Standard 工具集。
- **三行为带 + weak 内路由**：`spec [0, 0.15]` / `mixed [0.2, 0.45]`（过渡带，避免）/ `react [0.5, 1.0]`；`weak` 为模型自分类模式。
- **深度自适应近场引导**：weak 模式下每条真实用户消息后追加一条固定引导；简单任务使用快速收敛引导，复杂任务使用深度探索引导。
- **Agent 自优化工具**：内置 `dev_router_status` / `dev_router_mode` / `dev_mode_subagent`，会话可读取和调整自身路由。
- **会话恢复安全**：模式从 durable session events 推导，resume / reload 后保持。

## 目录结构

```text
dsh-preset-router-standard/
├── preset.yml              # 预设元信息（名称 / 描述）
├── agent.cordis.yml        # Agent 平面组合：persona、工具、plan mode、compaction、delegation 等
├── router-bootstrap.mjs    # Cordis 路由插件：首轮注入、弱带引导、dev_* 工具
└── router-core.mjs         # 纯路由逻辑：分类、行为带、persona、工具面（零依赖）
```

## 安装

从 GitHub 克隆到 DSH 的 agent-presets 目录：

```bash
mkdir -p ~/.dsh/.agent-presets
git clone --depth 1 https://github.com/kaijia323/dsh-preset-router-standard.git ~/.dsh/.agent-presets/dsh-preset-router-standard
```

然后重启 DSH，并在新会话中选择 **Router Standard** 预设。

> 如果之前安装过旧版本，可以先删除旧目录再克隆，避免残留文件：
>
> ```bash
> rm -rf ~/.dsh/.agent-presets/dsh-preset-router-standard
> ```

## 使用

新会话选择 **Router Standard** 后，可通过以下工具查看或调整路由：

```text
dev_router_status                        # 查看当前 mode / band / persona / core / override
dev_router_mode <spec|weak|mixed|react|0-100|0.0-1.0|auto>  # 设置显式路由模式
dev_mode_subagent <spec|weak|react|balanced> <task>         # 在隔离上下文中以其他模式运行任务
```

## 致谢

- 本 preset 的 dsh preset 模式由 [dsh-routing-suite](https://github.com/yjh051108/dsh-routing-suite)（MIT）作为参考设计，特别感谢 [yjh051108](https://github.com/yjh051108) 的 `router-standard` 预设与相关实测工作。
- 路由分类、行为带划分、弱带内路由、首轮工具面还原等核心思路均来自上述开源项目；本仓库在此基础上做了 DSH preset 化适配。

## License

[MIT](LICENSE)
