---
name: godot-feature-workflow
description: "Godot 功能开发工作流。当用户要求制作新功能、实现游戏机制或修改现有系统时使用；先给出简短实施方案，随后直接完成修改与验证，除非用户明确只要方案。"
---

# Godot 功能开发工作流

当用户要求实现新功能、修改现有系统或添加游戏机制时，使用此技能。

---

## 1. 需求理解

### 检查设计文档
- 优先读取仓库中已有的 GDD、设计文档或相关需求说明
- 如果没有设计文档，直接依据用户需求和现有代码继续，不要求调用不存在的命令

### 检查现有代码
- 根据仓库实际结构查找已有脚本和相关场景，不假设固定目录名
- 识别需要修改或扩展的文件

---

## 2. 方案设计

### 创建简短实施计划
在实施前先说明方案，并在信息足够时直接进入实施，不等待额外确认。方案包含：

1. **需要创建/修改的文件清单**
2. **系统交互关系**（谁调用谁）
3. **Autoload 单例设计**（如果需要）
4. **信号/回调设计**
5. **测试验证点**

### Godot 4 注意事项

**Autoload 单例命名冲突**
- Autoload 名称不能与任何脚本的 `class_name` 相同
- 如果脚本名为 `ToolSystem`，Autoload 也叫 `ToolSystem`，会导致 `"Class ToolSystem hides an autoload singleton"` 错误
- **解决方案**：Autoload 单例脚本不要使用 `class_name`，只用 `extends Node`

**正确示例：**
```gdscript
# scripts/tools/tool_system.gd
extends Node  # 不要写 class_name ToolSystem

enum ToolType { HOE, WATERING_CAN, AXE, PICKAXE, SCYTHE }
# ...
```

**Godot 4 API 注意事项**
- `Control.ANCHOR_MODE_CENTER` 在 Godot 4 中不存在
- 改用 `Control.PRESET_CENTER` 或 `anchors_preset`
- `set_pos()` 改为 `position = Vector2(x, y)`
- `strftime()` 已废弃，改用 `Time.get_datetime_string_from_system()`

**局部变量勿遮蔽基类信号/内置名（`SHADOWED_VARIABLE_BASE_CLASS`）**
- 脚本继承 `Node`、`Control` 等时，基类已声明大量**信号**与成员；若局部变量与其同名，会触发 **`SHADOWED_VARIABLE_BASE_CLASS`**（编辑器常见文案：`The local variable "X" is shadowing an already-declared signal in the base class "Node"`）。
- **示例**：在 UI 逻辑里使用 `var ready := ...` 表示「条件已就绪」，会与 **`Node.ready` 信号**同名并触发告警。应改为 `conditions_met`、`is_ready_to_continue` 等语义明确的名称。
- **原则**：新增局部变量前先想一下是否与当前 `extends` 链上的信号/方法/属性重名；优先用完整语义前缀（`is_` / `has_` / `_done`）避免缩写。

**GDScript 其它易踩坑（项目实测）**
- **`const` 初始化必须是常量表达式**：`const _ARR: PackedStringArray = PackedStringArray([...])` 在部分版本/写法下会报 `Assigned value for constant "..." isn't a constant expression` → 改用 **`static var`** 或在函数内创建数组。
- **注释与除法**：`//` 在 GDScript 中是**注释起始**，不是整除运算符；需要浮点再取整时用 `int(a / b.0)` 等写法。
- **`class_name` 与解析顺序**：若静态工具用 `class_name`，其它脚本在首帧解析时可能出现 **`Identifier "X" not declared`** → 可改为 **`const _X := preload("res://.../file.gd")`** 再调用脚本上的 `static func`（不在 Autoload 上使用与 `class_name` 冲突的同名）。
- **参数名遮蔽同文件 `static func`**：如 `static func yuan(yuan: int)` 会警告参数遮蔽函数名 → 参数改为 `amount_yuan` 等。

---

## 3. 实施流程

### 步骤 1: 创建/更新脚本

1. 在项目现有脚本目录中创建或修改对应的 `.gd` 文件
2. 如果是 Autoload 单例，确保：
   - 没有 `class_name` 声明
   - 由 Godot 的 Autoload 机制提供单例访问，不额外实现 `static var instance`

3. 如果是 UI 脚本：
   - 遵循项目现有 UI 脚本与场景目录
   - 使用 `Control.PRESET_*` 常量而非 `ANCHOR_MODE_*`

### 步骤 2: 创建/更新场景

1. 在项目现有场景目录中创建或修改 `.tscn` 文件
2. 如果需要添加新脚本：
   - 在 `.tscn` 中使用 `[ext_resource type="Script" path="..." id="X"]`
   - 在节点上设置 `script = ExtResource("X")`

### 步骤 3: 注册 Autoload

如果创建了新的 Autoload 单例，在 `project.godot` 中添加：
```ini
[autoload]

ExistingAutoload="*res://scripts/path/to/existing.gd"
NewAutoload="*res://scripts/path/to/new.gd"
```

### 步骤 4: 添加输入配置

如果需要新的输入动作，在 `project.godot` 的 `[input]` 部分添加：
```ini
[input]

new_action={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,...)]
}
```

---

## 4. 测试验证（使用 Godot MCP）

实现完成后，使用 Godot MCP 进行测试：

### 启动项目测试
```bash
使用 mcp__godot__run_project 工具
projectPath: "<当前 Godot 项目根目录>"
```

### 检查运行时错误
```bash
使用 mcp__godot__get_debug_output 工具
```
查看输出中的：
- `Parser Error:` — 语法错误
- `Runtime Error:` — 运行时错误
- `WARNING:` — 潜在问题（可能影响功能）

### 常见错误修复

**"Class X hides an autoload singleton"**
- 解决方案：移除脚本中的 `class_name X` 声明

**"Cannot find member Y"**
- 解决方案：检查 Godot 4 API，可能使用了已废弃的 API

**"Function not found in base X"**
- 解决方案：检查方法名是否正确，或基类是否支持该方法

**`SHADOWED_VARIABLE_BASE_CLASS`（局部变量遮蔽基类信号/成员）**
- **现象**：`GDScript::reload` 阶段警告，提示某局部变量与基类已声明的 **signal** 等冲突。
- **典型原因**：`extends Node` / `Control` 下命名 `ready`、`tree_exited` 等与内置信号同名。
- **解决方案**：重命名局部变量（见上文「局部变量勿遮蔽基类信号」）。

### 停止项目
```bash
使用 mcp__godot__stop_project 工具
```

---

## 5. 完成检查

在标记功能完成前，确认：

- [ ] 项目可以成功启动（无 Parser Error）
- [ ] 功能可以正常操作
- [ ] 无运行时错误
- [ ] 设计文档已更新（如果需要）
- [ ] 相关任务或设计文档已按项目惯例更新（如果需要）

---

## 6. 输出格式

功能实现完成后，输出：

```
[功能名称] 实现完成
====================

已创建/修改的文件：
- scripts/xxx.gd
- scenes/xxx.tscn

Autoload 注册：
- NewSystem (如果创建了新单例)

测试验证：
- 项目启动：<通过 / 失败 / 未运行>
- 功能测试：<通过 / 失败 / 未运行>

下一步：
- [下一个待处理任务]
```
