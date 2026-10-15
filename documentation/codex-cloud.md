# Codex Cloud 仓库级 Skill 配置与验收

## 安装位置

本仓库将可运行的原版 Skill 文件复制到 `.agents/skills/goutoujunshi/`：

```text
.agents/skills/goutoujunshi/
├── SKILL.md
├── references/
├── scripts/
├── assets/
├── agents/openai.yaml
└── LICENSE
```

根目录的原版文件继续保留；副本保留相同内容、相对目录和可执行权限。
`SKILL.md` 的名称为 `goutoujunshi`，原版描述保持不变。
`agents/openai.yaml` 的默认提示词包含 `$goutoujunshi`，没有禁用隐式调用。
这项仓库配置不会启用长期记忆，也不会保存聊天记录。

按照 [Codex 官方 Skill 文档](https://developers.openai.com/codex/skills/)，
Codex 从当前工作目录到仓库根目录查找 `.agents/skills`。
新任务应选中本仓库，并在仓库根目录或其子目录运行。
这项配置只适用于加载该仓库的任务；不会为其他仓库安装全局 Skill。

## 文件验证

在仓库根目录执行：

```bash
python3 -B scripts/validate_skill.py
python3 -B .agents/skills/goutoujunshi/scripts/validate_skill.py --runtime
```

两个命令都应输出 `goutoujunshi validation passed`。
副本使用原版验证器的 `--runtime` 模式，因为运行目录不包含仓库级 README 等开发文档。
新增的 `Validate Cloud Skill` 工作流还检查副本文件清单、内容和可执行权限是否与原版一致。
后续更新原版运行文件时，应同步更新对应副本，再运行上述检查。

## 新任务显式调用验收

1. 在 Codex Cloud 新建任务，选择 `zhoufanlei3-create/goutoujunshi`。
2. 合并前选择包含此配置的 PR 分支；合并后可以选择包含该提交的默认分支。
3. 使用下面的测试提示，不附带真实聊天记录，也不手动提供 Skill 文件路径：

```text
请使用 $goutoujunshi。这是虚构的首次使用测试，没有私人聊天记录。
请说明它是否出现在本任务启动时的可用 Skill 列表中，以及实际加载的来源路径。
若已可用，请按原版首次使用流程给出紧凑问卷。
不要启用长期记忆或写入个人档案。若未发现，请明确说明，不要临时安装或扫描文件来冒充自动注册。
```

验收时核对：启动时的可用 Skill 列表包含 `goutoujunshi`，实际加载路径为
`.agents/skills/goutoujunshi/SKILL.md`，回答符合原版首次使用流程。
手动找到文件、单纯复述名称或验证器通过，都不足以单独证明自动发现成功。
如果界面提供 Skill 选择器，也可检查其中是否列出该 Skill；各客户端界面可能不同。

## 新任务自动匹配验收

另开一个独立新任务，选择同一仓库和配置分支，在第一次提问中不写 Skill 名称或路径：

```text
这是虚构测试：我想梳理一段暧昧关系，分析双方投入，并决定如何邀约。
请采用当前可用的合适 Skill，说明选用了哪个 Skill，并给出首次使用需要的简短问题。
不要启用长期记忆或写入个人档案。
```

检查 Codex 是否自行选择并加载 `goutoujunshi`。
自动匹配依赖任务内容和 Skill 描述；一次未选中不等于文件缺失，应分别记录
“启动时是否发现”“显式调用是否成功”和“本次自动匹配是否成功”。

## 未发现时排查

- 确认任务选择的分支和提交确实包含 `.agents/skills/goutoujunshi/SKILL.md`。
- 确认任务工作目录位于该 Git 仓库内部，而非仓库外的上级目录。
- 创建全新任务重新加载；必要时刷新环境缓存，避免复用旧任务的 Skill 列表。
- 检查环境是否禁用本地 Skill，或用户配置是否通过 `skills.config` 禁用了此路径。
- 若文件验证通过但启动时仍未列出，保留为运行时发现未通过；不要改写原版分析规则。

以上文件检查和 CI 检查不会启动新的 Codex Cloud 任务，因此不能替代显式调用与自动匹配的实测。
