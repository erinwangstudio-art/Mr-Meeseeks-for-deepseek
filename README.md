# Mr. Meeseeks for DeepSeek Harness

<p align="center">
  <img src="icon.png" width="96" alt="dsh-logo-swap icon">
</p>

DeepSeek Harness（DSH）桌面版的品牌自定义插件。替换 Web 界面中的侧栏 Logo、品牌字标、对话页 Logo、欢迎语标语与预览版徽章。纯前端注入，不改 `app.asar`。

> 基于 [charce526/dsh-logo-swap](https://github.com/charce526/dsh-logo-swap)（MIT）派生。上游的全部能力都保留，本仓库的改动见「来源」。

## 功能

所有项目集中在 设置 → **Logo 自定义** 分区，各项独立配置：

| 分组 | 项目 | 说明 |
|---|---|---|
| 侧栏自定义 | Logo（左上角） | 替换品牌行图标；同时作用于标签页 favicon 与品牌行小鱼标记 |
| 侧栏自定义 | 字标（DeepSeek / Harness） | 官方字标为一张整体 SVG，插件将其拆分为两半，各自支持 官方 / 隐藏 / 文字 / 图片 四种模式 |
| 对话页自定义 | Logo（新对话首页） | 替换对话首页的品牌图标，独立于侧栏设置 |
| 对话页自定义 | 欢迎语标语 | 将「探索未至之境」替换为任意文字 |
| 对话页自定义 | 预览版徽章 | 官方 / 隐藏 / 自定义文字 三种模式。文字替换走 CSS 覆盖，不依赖 i18n 文案 |

通用能力：

- **独立缩放**：每项提供 50%–300% 缩放滑杆。侧栏 Logo / 字标放大时自动解除品牌行容器的固定高度限制；侧栏折叠态自动跳过，不影响原有布局。
- **图片导入**：支持文件选择器或本机绝对路径，格式 png / svg / webp / jpg / gif / ico，单张不超过 4 MiB，导入后以 dataURL 存于 localStorage。
- **恢复默认**：每项提供独立的恢复操作；停用插件即整体恢复官方界面。

## 出厂预设

| 项目 | 值 |
|---|---|
| 欢迎语标语 | `Gubba nub nub doo rahkah` |
| 预览版徽章 | `Go`（自定义文字模式） |

首次启动写入 localStorage（key `dsh-logo-swap.custom` 与 `dsh-logo-swap.preset-seeded`）。之后**设置面板的修改优先**，预设不会被反复覆盖。

## 配置入口

- **品牌配置**：设置 → **Logo 自定义**（分区，不是独立页面）。改欢迎语、切换徽章模式、换 icon 都在这里。
- **插件管理**：macOS 顶部菜单栏 →「**桌面插件…**」，弹出独立窗口「桌面插件」（「npm 包」输入框 +「安装」）。**不在设置页** —— 设置页的「插件」只列已装插件与启用状态。

三个后续修改入口：

1. **运行时改文案**：设置 → Logo 自定义 → 对话页自定义。改欢迎语或徽章文字，立即生效。面板底部「恢复本机预设」一键回到 `Gubba nub nub doo rahkah` + `Go`，不动 Logo 设置。
2. **改出厂预设**：编辑 `lib/client.js` 顶部 `PRESET_HEADLINE` / `PRESET_BADGE`。源码有两份（主副本 + DSH 实际加载的 profile 副本），改完按「本地开发」里的命令同步，再清空 localStorage 的两个 key 并重启 DSH。
3. **以后换 icon**：侧栏 Logo、字标两半、对话页 Logo 的导入入口全部保留，默认官方。在设置面板导入即可，不用改代码。

彻底回到官方界面：设置 → 插件 里停用 `dsh-logo-swap`，或按「安装」里的回退步骤移除。配置存在 localStorage，停用不会丢。

## 安装

### macOS 桌面版（已实机验证）

**上游的 `link:` + junction 方案在 macOS 桌面版无效。** DSH 桌面版用「包事务」独占 `~/.dsh/profiles/desktop`：启动时校验 `package.json` 的 dependencies 必须是精确 npm 版本（`link:` / `file:` 会被判非法），校验失败就把整个 profile 恢复出厂并删掉 `node_modules`。

按 DSH 自己的规则，把插件做成「已安装的精确版本包」：

```bash
SRC="$HOME/.dsh/plugins/dsh-logo-swap"                         # 主源码，保留
DST="$HOME/.dsh/profiles/desktop/node_modules/dsh-logo-swap"   # DSH 实际加载的位置
PROFILE="$HOME/.dsh/profiles/desktop"

# 1. 实体复制，不能是 symlink（validateDesktopPluginGraph 拒绝链接型私有包）
mkdir -p "$DST"
cp -R "$SRC/." "$DST/"

# 2. 先备份，再改 $PROFILE/package.json：
#    dependencies:       { "dsh-logo-swap": "1.0.0" }   ← 精确版本，不是 link:
#    dsh.profile.bundles: 内建两项之后追加 "dsh-logo-swap"
#    内建两项必须是前两位：@deepseek-ai/dsh-base、@deepseek-ai/dsh-web-app

# 3. 不要创建 pnpm-lock.yaml：lockHash 保持 sha256(空)，启动不会触发 rebuild

# 4. 完全退出（Cmd+Q）并重启 DeepSeek Harness
```

**回退**：用 `package.json.bak-*` 还原（或删掉 dependencies 与 bundles 里那项），删掉 profile 里的插件目录，重启。

**这不是官方支持的安装方式。** 官方路径是 macOS 菜单栏 →「桌面插件…」→ 输入 npm 包名安装，只接受 registry 上的精确版本包（DSH 源码里明确拒绝 `file:`，且 registry 硬编码为 `https://registry.npmjs.org/`）。本做法是让 DSH 把本地目录当成已装 registry 包；DSH 升级或执行「重置 Desktop」后可能失效，届时按上面重做一次。

### Windows（上游方案）

前提：已安装 DSH 桌面版；本机已安装 Git 与 Node.js ≥ 18。插件以链接方式挂载到 desktop profile，安装目录需保留。

```powershell
# 1. 克隆仓库到固定目录
git clone https://github.com/<you>/dsh-logo-swap.git D:\path\to\dsh-logo-swap

# 2. 在 %USERPROFILE%\.dsh\profiles\desktop\package.json 中登记
#    ① dependencies 新增 "dsh-logo-swap": "link:D:/path/to/dsh-logo-swap"
#    ② dsh.profile.bundles 新增 "dsh-logo-swap"

# 3. 建立 junction
New-Item -ItemType Junction `
  -Path "$env:USERPROFILE\.dsh\profiles\desktop\node_modules\dsh-logo-swap" `
  -Target "D:\path\to\dsh-logo-swap"

# 4. 重启 DeepSeek Harness
```

无需构建步骤；卸载时删除两条登记与 junction 并重启。

## 本地开发

```bash
# 语法检查
node --check lib/client.js

# 改完源码同步到 DSH 实际加载的位置（只覆盖、不删除）
cp -R "$HOME/.dsh/plugins/dsh-logo-swap/." \
      "$HOME/.dsh/profiles/desktop/node_modules/dsh-logo-swap/"

# 重置出厂预设：在 DSH 里清掉 localStorage 的两个 key
#   dsh-logo-swap.custom / dsh-logo-swap.preset-seeded
```

改完 Cmd+Q 完全退出并重启 DSH 才生效。配置结构为 v6，兼容 v2–v5 的旧 localStorage 数据。

本仓库保留了上游的 commit 历史，`upstream` remote 指向原仓库，需要合并上游更新时：

```bash
git fetch upstream
git merge upstream/master
```

## 目录结构

```
dsh-logo-swap/
├── package.json         # bundle 声明（patch 入口 + client 平台）
├── cordis.patch.yml     # bundle patch：注册 loader entry
├── icon.svg / icon.png  # 插件图标
├── lib/
│   ├── index.js         # 宿主端：logoswap 服务（本机图片读取 → dataURL）
│   ├── client.js        # 前端端：slot 注册、favicon/CSS、标语替换、设置面板
│   └── wm-paths.json    # 官方字标 SVG 拆分数据（DeepSeek / Harness 两半路径）
├── LICENSE              # MIT
└── README.md
```

## 实现说明

- 通过 `ctx.slots.inject` 进入 DSH 前端已声明的槽位（`sidebar.brand.mark`、`sidebar.brand.name`、`conversation.hero.brand.mark`、`settings.section`），以 priority -1 影子注册；停用插件时 dispose，官方内容自动恢复。
- 字标的文字 / 图片模式替换 `sidebar.brand.name` 槽位；官方字标矢量数据自 BrandWordmark SVG 提取，存于 `wm-paths.json`。
- 欢迎语替换基于 TreeWalker 文本节点替换与 MutationObserver；官方 i18n 注册表为只读，无法覆盖注册。
- favicon、品牌行小鱼标记通过运行时注入 `<style>` / `<link>` 实现；预览版徽章的隐藏与自定义文字同样由注入的 `<style>` 完成（`font-size:0` + `::after` 覆盖），不受 React 重渲染影响，随配置实时切换。
- 配置存于浏览器 localStorage，跨会话保留，旧版本格式自动迁移。

## 兼容性

- 依赖 DSH 前端的 slot 声明合约（`sidebar.brand.mark` 等）。这些槽位属于官方前端公开结构，版本间保持稳定；若大版本重构品牌行，插件需相应适配。
- 仅修改 Web 前端界面元素，不涉及桌面壳层（窗口图标、托盘图标、exe 内嵌图标）。
- 验证环境：macOS（darwin/arm64）、DSH 桌面版 `0.1.0-rc.22`、Node `v22.22.2`。已确认 app.asar 中存在 `sidebar.brand.mark`、`previewBadge`、`探索未至之境` 等注入点；按上述 macOS 方案挂载后，插件前端已实机加载并写入预设。上游方案已在 Windows 桌面版验证。

## 来源

基于 [charce526/dsh-logo-swap](https://github.com/charce526/dsh-logo-swap)（MIT）派生。上游的侧栏 Logo、字标拆分、对话页 Logo、图片导入等能力全部保留。本仓库的改动：

- 首次启动写入出厂预设（欢迎语 `Gubba nub nub doo rahkah`、徽章 `Go`），且只 seed 一次
- 预览版徽章从「显示/隐藏」扩展为 官方 / 隐藏 / 自定义文字 三模式，文字替换改用 CSS 覆盖
- 设置面板增加「恢复本机预设」；配置结构升级 v6 并兼容 v2–v5
- README 补充 macOS 桌面版安装、回退、配置入口与本地开发说明

## License

[MIT](LICENSE) — 上游同为 MIT，版权声明保留在 LICENSE 中。
