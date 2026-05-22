# new-project-init

把"从 0 到 GitHub"的新建项目流程封装成一个 Claude Code skill。

跟 Claude 说一句"新建项目 my-blog",Claude 自动:
- ✅ 建文件夹 `~/Desktop/<项目名>`
- ✅ 写 CLAUDE.md(填好基本信息 + 协作约定)
- ✅ 写 .gitignore(按类型选模板)
- ✅ `git init` / `git add` / `git commit`
- ✅ 用 `gh repo create` 建 GitHub 仓库并 push(若你选了接 GitHub)

---

## 📦 安装(给同事看的)

### Prerequisites

确保你电脑装了:

| 工具 | 检查命令 | 没装怎么办 |
|---|---|---|
| Claude Code | `claude --version` | https://claude.com/claude-code 下载 |
| `gh` CLI | `gh --version` | `brew install gh` 然后 `gh auth login` |
| `git` | `git --version` | Mac 自带,没的话 `brew install git` |

### 一行命令装上

```bash
cd ~/.claude/skills && git clone https://github.com/linge-luot/new-project-init.git
```

装完**重启 Claude Code**(关掉再开新对话),触发词就生效了。

### 验证装好了

新开一个 Claude Code 对话,问:

```
你能用 new-project-init skill 吗?
```

Claude 应该能识别。或者直接试触发:

```
新建项目 test-from-friend
```

Claude 会问你 3-4 个短问题,然后帮你建好项目。

---

## 🎯 触发词

- 中文:**新建项目** / **建项目** / **帮我建项目** / **创建项目** / **建个项目**
- 英文:create new project / new project / init project / scaffold project

---

## 📂 项目类型支持

Skill 会按你选的类型用对应的 .gitignore 模板:

| 大类 | 子类 | 模板 |
|---|---|---|
| 📝 内容/文档 | 笔记/日记 · 演讲分享稿 | `gitignore.notes` |
| 💻 软件开发 | 脚本工具(Python/Node) | `gitignore.script` |
| 💻 软件开发 | 网页应用(React/Vue/Next.js) | `gitignore.webapp` |
| 💻 软件开发 | 通用程序 | `gitignore.practice` |
| 🛒 电商业务 | Amazon/独立站/Listing/综合 | `gitignore.practice` |
| ✏️ 自定义/练手 | 自填类型 | `gitignore.base` |

---

## 🤝 协作约定(温和提醒)

生成的项目 CLAUDE.md 里会带一段"Claude 协作约定",让 Claude 进入该项目工作时:

- **3+ 处改动后** → 提示"建议 git add + commit"
- **对话 60+ 轮后** → 提示"可以 /compact 清下上下文"
- **用户问'下一步'** → 优先检查是否该 commit
- **温和原则**:同样提醒催过一次没动作就**不再催第二次**

---

## 🔄 更新到最新版

```bash
cd ~/.claude/skills/new-project-init && git pull
```

然后重启 Claude Code 即可。

---

## 🛠️ 自定义(给你自己)

| 想改啥 | 改这个 |
|---|---|
| 触发词 | `SKILL.md` 顶部 YAML 的 `description` |
| 协作约定 | `templates/CLAUDE.md.tmpl` 的"协作约定"段 |
| 项目类型 | `SKILL.md` Step 2-3 + 加新 `gitignore.xxx` 模板 |
| 模板内容 | `templates/gitignore.*` 文件 |

改完想 push 回上游(我的仓库)?Fork 一份 → 改 → 提 PR。

---

## 📁 项目结构

```
new-project-init/
├── SKILL.md              ← 触发词 + 工作流(给 Claude)
├── README.md             ← 给人看的(本文件)
├── LICENSE               ← MIT
└── templates/
    ├── CLAUDE.md.tmpl    ← 项目说明书模板
    ├── gitignore.base    ← 通用兜底
    ├── gitignore.notes   ← 内容/文档
    ├── gitignore.script  ← 脚本工具
    ├── gitignore.webapp  ← 网页应用
    └── gitignore.practice ← 通用/电商/练手
```

---

## ⚠️ 已知限制

- 项目固定建在 `~/Desktop/<NAME>`,不支持其他路径(简化逻辑)
- GitHub 仓库建在你 `gh auth login` 的账号下,不支持组织
- `gh` 必须先 `gh auth login` + 有 `repo` scope(若想后续删仓库还需要 `delete_repo` scope:`gh auth refresh -h github.com -s delete_repo`)

---

## License

MIT
