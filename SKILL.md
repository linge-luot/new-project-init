---
name: new-project-init
description: 从 0 自动创建新项目(本地 + GitHub)端到端流程封装。用户说"新建项目"、"建项目"、"帮我建项目"、"创建项目"、"建个项目"、"new project"、"create new project"、"init project"、"scaffold project" 等明显在表达"建新项目"意图时立刻触发。Skill 接管对话,问 3-4 个短问题(项目类型大类 + 可能的子类 + 用途 + 是否接 GitHub),然后自动 mkdir + 写 CLAUDE.md/.gitignore + git init/add/commit + 用 gh CLI 建私有/公开 GitHub 仓库并 push。覆盖 4 大类项目:内容/文档、软件开发、电商业务、自定义/练手。生成的项目 CLAUDE.md 内嵌"温和式"协作约定,让 Claude 在该项目工作时主动提醒 commit、对话变长时提醒 /compact。
---

# new-project-init

把"从 0 建项目到 GitHub"的 12 步流程封装成一次自动化对话。

## 触发条件

用户消息中包含下列短语之一,且**明显**在表达"建新项目"意图时立刻激活:
- 新建项目 / 建项目 / 帮我建项目 / 创建项目 / 建个项目 / 建一个项目
- create new project / new project / init project / scaffold project

**不要触发**:用户已经在某个现有项目里改东西、问问题、调试 bug 时。

---

## 工作流(严格按 6 步执行,含 Step 0 平台检测)

每一步开始前先用一句话告诉用户你在做什么。

### Step 0:检测桌面路径(平台兼容)

在所有后续操作之前,**先跑下面这条 bash 检测平台并算出真实桌面路径**:

```bash
if [ -f /proc/version ] && grep -qi microsoft /proc/version 2>/dev/null; then
  # WSL: 用 Windows 真桌面,不是 WSL 假桌面
  WIN_USER=$(cmd.exe /c "echo %USERNAME%" 2>/dev/null | tr -d '\r\n ')
  echo "DESKTOP=/mnt/c/Users/$WIN_USER/Desktop"
  echo "PLATFORM=wsl"
elif [ "$(uname -s)" = "Darwin" ]; then
  echo "DESKTOP=$HOME/Desktop"
  echo "PLATFORM=macos"
elif [ "$(uname -s)" = "Linux" ]; then
  echo "DESKTOP=$HOME/Desktop"
  echo "PLATFORM=linux"
else
  # 兜底
  echo "DESKTOP=$HOME/Desktop"
  echo "PLATFORM=unknown"
fi
```

输出会告诉你两个值。**后续所有 `<DESKTOP>` 占位符都替换成检测到的真实路径**,例如:

| 平台 | `<DESKTOP>` 实际路径 |
|---|---|
| macOS | `/Users/<user>/Desktop` |
| Linux 原生 | `/home/<user>/Desktop` |
| WSL(Windows) | `/mnt/c/Users/<windows-user>/Desktop`(Windows 真桌面) |

记住这个路径,接下来 Step 1-5 都要用。

### Step 1:确认项目名

- 若触发消息中已含项目名(如"新建项目 my-blog") → 直接用 `my-blog`
- 否则问用户:"项目叫啥?英文小写、用 `-` 分隔(比如 `my-notes`)"

**校验**:
- 不能含空格、中文、`/`、`\`、`*`、`?` 等特殊字符 → 拒绝并要求重起名
- `<DESKTOP>/<NAME>` **不能已存在** → 若已存在,告知用户,问"改名 / 取消"

### Step 2:用 AskUserQuestion 问大类

**Q1 项目类型大类(4 选 1)**:
- 📝 内容/文档(纯 Markdown,如笔记/分享稿)
- 💻 软件开发(脚本/网页/通用程序)
- 🛒 电商业务(Amazon/独立站/Listing/Reviews)
- ✏️ 自定义/练手(自填类型名或纯练手)

### Step 3:按大类问子类(再用一次 AskUserQuestion)

**若 Q1 = 内容/文档**:
- 笔记/日记(每天写学习/想法)
- 演讲/分享稿(Markdown + 导出 PDF/PPT)

**若 Q1 = 软件开发**:
- 🐍 脚本工具(Python/Node 命令行)
- 🌐 网页应用(React/Vue/Next.js)
- 📦 通用程序(可能混语言)

**若 Q1 = 电商业务**:
- Amazon 选品/排名追踪
- 独立站运营
- Listing/Reviews 分析
- 综合电商工具

**若 Q1 = 自定义/练手**:
- 直接文字问:"这类项目叫啥?(可填'练手')" + "需要忽略什么特殊文件吗?(可填'无')"

### Step 4:问用途和 GitHub

**Q2(文字回答)**:"一句话描述这个项目干嘛用?"

**Q3 AskUserQuestion:接 GitHub?**
- 接,私有(推荐)
- 接,公开
- 不接(只本地 Git)

### Step 5:自动执行(用 Bash + Write 工具)

按以下顺序跑,绝对路径:

**5.1 建文件夹**
```bash
mkdir <DESKTOP>/<NAME>
```

**5.2 准备 CLAUDE.md 内容**

读模板:`~/.claude/skills/new-project-init/templates/CLAUDE.md.tmpl`

替换占位符:
- `{{NAME}}` → 项目名
- `{{DESC}}` → 用户 Q2 回答
- `{{TYPE}}` → 大类 / 子类(如"软件开发 / 网页应用")
- `{{DATE}}` → 今天日期 YYYY-MM-DD
- `{{GITHUB_URL}}` → GitHub URL,或"未接"
- `{{TYPE_SPECIFIC_USAGE}}` → 按子类填(见下方"占位符填充规则")

用 Write 工具写到 `<DESKTOP>/<NAME>/CLAUDE.md`

**5.3 选 .gitignore 模板**

按大类/子类映射到模板:

| 类型 | 模板文件 |
|---|---|
| 内容/文档 (任何子类) | `gitignore.notes` |
| 软件开发/脚本工具 | `gitignore.script` |
| 软件开发/网页应用 | `gitignore.webapp` |
| 软件开发/通用程序 | `gitignore.practice` |
| 电商业务 (任何子类) | `gitignore.practice` |
| 自定义/练手 | `gitignore.base` |

用 Read 读 `~/.claude/skills/new-project-init/templates/gitignore.<对应类型>`,Write 到 `<DESKTOP>/<NAME>/.gitignore`(注意:目标文件名是 `.gitignore`,不是 `gitignore.xxx`)。

**5.4 本地 Git**
```bash
cd <DESKTOP>/<NAME> && \
git init && \
git add CLAUDE.md .gitignore && \
git commit -m "初始版本:<NAME> 项目骨架"
```

**5.5 接 GitHub(若 Q3 选了接)**

私有:
```bash
gh repo create <NAME> \
  --private \
  --description "<DESC>" \
  --source=<DESKTOP>/<NAME> \
  --remote=origin \
  --push
```

公开:把 `--private` 改成 `--public`。

### Step 6:显示成功报告

用表格汇报:
```
| 项 | 值 |
|---|---|
| 📁 本地路径 | <DESKTOP>/<NAME> |
| 🔗 GitHub | <URL> 或 "未接,只本地" |
| ✏️ 首个 commit | <hash> |
| 🎨 类型 | <大类 / 子类> |
```

教学:
> 以后改东西就 3 句循环:
> `git add <文件名>` → `git commit -m "做了啥"` → `git push`

提示:
> 在这个新项目里跟 Claude 聊天,他会按 CLAUDE.md 协作约定主动提醒你 commit / /compact。

---

## 占位符填充规则

### `{{TYPE_SPECIFIC_USAGE}}` 按子类填写

| 子类 | 填写内容 |
|---|---|
| 笔记/日记 | 「一篇笔记一个 `.md` 文件,文件名建议 `YYYY-MM-DD-主题.md`」 |
| 演讲/分享稿 | 「每篇分享一个文件夹,放 `.md` 源稿;导出的 `.pptx/.pdf` 放桌面(已被 .gitignore 拦截)」 |
| 脚本工具 | 「主入口 `main.py` / `index.js` / `run.sh`,结果输出到 `output/`(已忽略)」 |
| 网页应用 | 「`npm install` → `npm run dev` 启动本地开发,生产构建 `npm run build`」 |
| 通用程序 | 「看实际语言选主入口;数据 / 构建产物放 `output/` 或 `dist/`」 |
| Amazon 选品/排名 | 「调用 `sorftime-product-research` 选品 / `amazon-rank-tracker` 跟踪排名 / `amazon-reviews` 抓评论」 |
| 独立站运营 | 「调用 `sean-frank-perspective` 做策略 / `bb-browser` 做调研抓数据」 |
| Listing/Reviews | 「用 `amazon-reviews` skill 抓评论分析,数据写 `output/`」 |
| 综合电商工具 | 「按需调用 Amazon / 独立站相关 skill,可能混 Python + Node」 |
| 自定义/练手 | 「随便玩,先建一个 `hello.md` 写一句话试 push 流程」 |

---

## 错误处理

- **项目名校验失败** → 解释为啥不行(空格/中文/特殊字符),要求重起名
- **目录已存在** → 告知用户,3 选 1:改名 / 直接进去看现状(不动) / 取消
- **`git init` 失败** → 显示错误,提示用户手动检查
- **`gh repo create` 失败**(常见原因:同名仓库已存在) → 告知用户,2 选 1:换仓库名 / 改用现有仓库手动接 remote
- **`git push` 失败** → 保留本地 commit(已建好),提示用户后面手动 `cd <项目> && git push` 重试

---

## 关键工具映射

| 任务 | 用什么 |
|---|---|
| 问用户问题 | AskUserQuestion 工具(2-4 个选项一次问) |
| 跑命令 | Bash 工具 |
| 写文件 | Write 工具(绝对路径) |
| 读模板 | Read 工具(读本 skill 的 templates/) |

---

## 设计要点

- **温和提醒**:协作约定写在生成的 CLAUDE.md 里,以后 Claude 进入该项目自动按它行事
- **轮数代理 /compact**:Claude 看不到自己 context %,用"对话 60+ 轮"作粗略代理
- **3+ 改动代理 commit**:看 Edit/Write 工具被调用次数 + 改动文件数综合判断
- **温和不烦人**:同样提醒催过一次没回应,**不要催第二次**

---

## 不要做的事

- 不要把项目放在 `~/Desktop` 之外的位置(简化逻辑)
- GitHub 仓库默认建在 `gh` CLI 当前登录的账号下(用 `gh repo create <NAME>` 不加 owner 前缀,gh 自动用当前用户)
- 不要在已有项目目录里触发(检测到 `<NAME>` 已存在就停)
- 不要修改任何现有项目
