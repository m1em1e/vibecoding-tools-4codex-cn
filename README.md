# vibecoding-tools-4codex-cn

一套面向中文开发场景的 Codex 自用配置集合，包含可复用的项目规则、专业角色（自定义 Agent）与任务型 Skill。

本仓库目前重点覆盖 Godot 4.x 游戏开发、GDScript、游戏立项、Konado Script 视觉小说脚本，以及 Java + Spring Web 后端开发。所有内容均以“先理解项目、控制改动范围、完成真实验证”为基本原则。

## 项目特点

- 中文优先：说明、工作流与交付要求均针对中文协作习惯编写。
- 面向落地：强调直接修改项目、补齐配置与测试，而非只提供概念建议。
- 边界清晰：区分项目规则、专业角色与任务 Skill 的职责。
- 谨慎修改：优先兼容现有结构，避免无关重构、破坏性操作和虚假验证。
- 可按需安装：可以只选择与你技术栈相关的角色或 Skill。

## 仓库内容

### 项目规则

| 文件 | 说明 |
| --- | --- |
| [`AGENTS.md`](./AGENTS.md) | 通用项目执行规则，覆盖上下文识别、最小改动、测试验证、Git 安全与 Agent/Skill 调用标注。 |

### 专业角色

| 角色 | 配置文件 | 适用范围 |
| --- | --- | --- |
| `game_dev_cn` | [`agents/game_dev_cn.toml`](./agents/game_dev_cn.toml) | Godot 4.x、GDScript、场景、资源、UI、动画、物理、编辑器工具、调试与发布。 |
| `java_spring_web_cn` | [`agents/java_spring_web_cn.toml`](./agents/java_spring_web_cn.toml) | Java、Spring Boot、Spring MVC、REST API、校验、异常处理、测试、配置与后端工程实践。 |

角色配置中包含默认模型、推理强度和沙箱设置。模型是否可用取决于你的 Codex 版本、账户权限与运行环境；如不可用，请修改对应 TOML 文件中的 `model` 字段。

### Skills

| Skill 名称 | 目录 | 用途 |
| --- | --- | --- |
| `game-project-initiation` | [`skills/game-project-initiation`](./skills/game-project-initiation/) | 通过结构化问答提炼核心玩法，生成 GDD 初稿、原型清单与批判性分析。 |
| `gdscript-codegen` | [`skills/godot-code-generate`](./skills/godot-code-generate/) | 生成、修改或审查 Godot 4.x GDScript，覆盖类型、信号、导出变量、模板与常见警告。 |
| `godot-feature-workflow` | [`skills/godot-feature-workflow`](./skills/godot-feature-workflow/) | 实现 Godot 新功能、玩法机制或现有系统修改，并完成必要验证。 |
| `godot-skills` | [`skills/godot-skills`](./skills/godot-skills/) | Godot 通用开发与排错入口，适合节点、资源、UI、动画、物理和兼容性问题。 |
| `godot-tscn-format` | [`skills/godot-tscn-format`](./skills/godot-tscn-format/) | 直接读取、创建和修改 Godot 4.x `.tscn` 文本场景文件。 |
| `konado-script` | [`skills/konado-skills`](./skills/konado-skills/) | 编写与检查视觉小说使用的 Konado Script（`.ks`）。 |

## 目录结构

```text
.
├── AGENTS.md
├── agents/
│   ├── game_dev_cn.toml
│   └── java_spring_web_cn.toml
└── skills/
    ├── game-project-initiation/
    ├── godot-code-generate/
    ├── godot-feature-workflow/
    ├── godot-skills/
    ├── godot-tscn-format/
    └── konado-skills/
```

## 安装方式

Codex 支持用户级和项目级配置。建议先按需选择文件，不要直接覆盖已有的 `AGENTS.md`、角色或 Skill。

### 用户级安装

用户级配置会在多个项目中生效：

- 角色：复制到 `~/.codex/agents/`
- Skills：复制到 `~/.agents/skills/`
- 通用规则：将本仓库 `AGENTS.md` 中需要的规则合并到 `~/.codex/AGENTS.md`

Windows PowerShell：

```powershell
git clone https://github.com/m1em1e/vibecoding-tools-4codex-cn.git
Set-Location vibecoding-tools-4codex-cn

New-Item -ItemType Directory -Force "$HOME\.codex\agents" | Out-Null
New-Item -ItemType Directory -Force "$HOME\.agents\skills" | Out-Null

Copy-Item ".\agents\*.toml" "$HOME\.codex\agents\" -Force
Copy-Item ".\skills\*" "$HOME\.agents\skills\" -Recurse -Force
```

macOS / Linux：

```bash
git clone https://github.com/m1em1e/vibecoding-tools-4codex-cn.git
cd vibecoding-tools-4codex-cn

mkdir -p ~/.codex/agents ~/.agents/skills
cp agents/*.toml ~/.codex/agents/
cp -R skills/* ~/.agents/skills/
```

### 项目级安装

若只希望配置在某个仓库中生效，可复制到目标项目：

```text
目标项目/
├── AGENTS.md
├── .codex/
│   └── agents/
│       └── *.toml
└── .agents/
    └── skills/
        └── <skill-name>/SKILL.md
```

如果目标项目已经有 `AGENTS.md`，请合并所需规则，不要直接覆盖。安装后建议重新启动 Codex 或开启新会话，使新配置被重新发现。

## 使用示例

Codex 可以根据任务描述自动选择匹配的 Skill，也可以在提示词中明确指定角色或 Skill。

```text
请使用 game_dev_cn 角色检查这个 Godot 4.x 项目的角色移动实现，并直接修复发现的问题。
```

```text
请调用 game-project-initiation，把这个游戏创意整理为可执行的 GDD 初稿和原型清单。
```

```text
请按照 godot-tscn-format 的规则，为现有场景添加一个带碰撞体的 CharacterBody2D。
```

实际行为仍会受到当前项目代码、项目内更深层的 `AGENTS.md`、Codex 配置、可用工具和用户当前指令影响。

## 选择建议

- 开发完整 Godot 功能：优先使用 `game_dev_cn` + `godot-feature-workflow`。
- 只生成或审查 GDScript：使用 `gdscript-codegen`。
- 直接编辑 `.tscn`：使用 `godot-tscn-format`。
- Godot 通用排错：使用 `game_dev_cn` 或 `godot-skills`。
- 从游戏创意开始立项：使用 `game-project-initiation`。
- 编写视觉小说脚本：使用 `konado-script`。
- 开发 Spring Web 后端：使用 `java_spring_web_cn`。

## 自定义

这些配置是作者当前工作流的公开版本，不一定适合所有项目。使用前建议检查并调整：

- Agent TOML 中的 `model`、`model_reasoning_effort` 与 `sandbox_mode`
- Skill 的触发描述、技术版本和输出格式
- `AGENTS.md` 中的 Git、验证、语言与调用标注规则
- 与团队现有规范、CI、测试命令和目录结构冲突的部分

修改 Skill 时，请保持每个 Skill 目录内存在有效的 `SKILL.md`，并确保其 frontmatter 中至少包含清晰的 `name` 和 `description`。

## 贡献

欢迎通过 Issue 或 Pull Request 提交：

- 新的中文专业角色或 Skill
- 对现有工作流的纠错与补充
- Codex 新版本兼容调整
- 更可靠的验证步骤与真实项目案例

提交内容应尽量说明适用版本、触发场景、行为边界和验证方式，避免加入未经验证的工具能力或宽泛模板。

## 参考文档

- [Codex customization](https://developers.openai.com/codex/concepts/customization/)
- [Codex subagents](https://developers.openai.com/codex/subagents/)
- [Codex skills](https://developers.openai.com/codex/skills/)

## 许可证

本项目采用 [MIT License](./LICENSE) 开源。你可以使用、复制、修改和分发本仓库内容，但需要保留原始版权声明和许可证文本。
